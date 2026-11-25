# Simulation Engine Architecture

**Project:** CYBERFRONT  
**Document ID:** CF-ARCH-006  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.6 - Simulation Engine Architecture  
**Date:** November 25, 2026

**Related Documents:**

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/frontend-architecture.md`
- `docs/architecture/backend-architecture.md`
- `docs/architecture/database-architecture.md`

---

## 1. Purpose

Dokumen ini mendefinisikan arsitektur Independent Simulation Engine untuk CYBERFRONT.

Simulation Engine merupakan sistem independen yang bertanggung jawab menjalankan simulasi teknis infrastruktur virtual, termasuk:

- Infrastructure runtime.
- Component behavior.
- Resource consumption.
- Network communication.
- Service dependencies.
- Attack and defense mechanics.
- Event processing.
- Battle resolution.
- Simulation replay.

Simulation Engine menjadi sumber kebenaran untuk hasil teknis simulasi.

Laravel bertanggung jawab mengatur business workflow, sedangkan Simulation Engine bertanggung jawab menghitung hasil simulasi.

Arsitektur harus memastikan:

1. Deterministic execution.
2. Reproducible simulation.
3. Independent deployment.
4. Modular simulation domains.
5. Versioned simulation rules.
6. Isolated execution.
7. Reliable event processing.
8. Extensibility untuk fitur simulasi berikutnya.

## 2. Architectural Decisions

### 2.1 Architecture Style

Simulation Engine menggunakan **Independent Modular Simulation Engine**.

Engine dikembangkan sebagai aplikasi TypeScript terpisah dari Laravel.

Engine memiliki:

- Simulation Core.
- Domain Modules.
- Runtime State Manager.
- Event Processing System.
- Deterministic Random Generator.
- Ruleset Registry.
- Snapshot Validator.
- Simulation Result Builder.
- Transport Adapter.
- Execution Worker.

Engine tidak menjadi bagian dari Laravel application runtime.

### 2.2 Technology Stack

| Area              | Technology                 |
| ----------------- | -------------------------- |
| Language          | TypeScript                 |
| Runtime           | Node.js                    |
| Package Manager   | pnpm                       |
| Testing           | Vitest                     |
| Schema Validation | Zod atau alternatif setara |
| Build Tool        | TypeScript Compiler / tsup |
| API Integration   | Adapter-based              |
| Initial Execution | Background Worker          |
| Logging           | Structured Logging         |

Pemilihan library final dapat berubah berdasarkan kebutuhan implementasi dan hasil evaluasi.

### 2.3 Core Architectural Principle

Simulation Engine harus memisahkan:

**Simulation Core**

Berisi aturan dan perhitungan simulasi.

**Execution Infrastructure**

Menangani transport, job execution, logging, persistence adapter, dan komunikasi eksternal.

Simulation Core tidak boleh bergantung pada execution infrastructure.

## 3. High-Level Architecture

```text
             Laravel Backend
                   |
                   v
          Simulation Request
                   |
                   v
       +------------------------+
       |   Transport Adapter   |
       +------------------------+
                   |
                   v
       +------------------------+
       |    Request Validator   |
       +------------------------+
                   |
                   v
       +------------------------+
       |   Simulation Runtime   |
       +------------------------+
                   |
          +--------+--------+
          |                 |
          v                 v
    Snapshot Loader   Ruleset Registry
          |                 |
          +--------+--------+
                   |
                   v
       +------------------------+
       |    Simulation Core     |
       |                        |
       | - Infrastructure      |
       | - Components          |
       | - Resources           |
       | - Network             |
       | - Attack & Defense    |
       | - Event Processing    |
       +------------------------+
                   |
                   v
       +------------------------+
       |  Result & Event Builder|
       +------------------------+
                   |
                   v
       +------------------------+
       |    Response Adapter    |
       +------------------------+
                   |
                   v
             Laravel Backend
```

Simulation Core tidak mengetahui apakah request berasal dari Laravel, test runner, replay system, atau sistem lain.

## 4. Simulation Engine Responsibilities

Simulation Engine memiliki tanggung jawab:

### 4.1 Infrastructure Simulation

- Memuat infrastructure snapshot.
- Membuat runtime component instances.
- Memproses component dependencies.
- Memeriksa infrastructure validity.
- Mengelola runtime topology.
- Memelihara runtime state.

### 4.2 Component Simulation

- Menjalankan behavior komponen.
- Memproses configuration.
- Menghitung resource consumption.
- Menghasilkan component events.
- Memproses component failures.
- Memperbarui component runtime status.

### 4.3 Resource Simulation

- CPU consumption.
- RAM allocation.
- Storage usage.
- Bandwidth utilization.
- Resource contention.
- Resource exhaustion.

### 4.4 Network Simulation

- Network connectivity.
- Packet flow abstraction.
- Routing behavior.
- Firewall filtering.
- Network latency.
- Bandwidth constraints.
- Connection failures.

### 4.5 Attack and Defense Simulation

- Attack scenario execution.
- Attack propagation.
- Firewall enforcement.
- Reverse proxy behavior.
- Application vulnerability simulation.
- Service availability impact.
- Defense effectiveness.

### 4.6 Battle Resolution

- Memproses attack sequence.
- Menghitung defense response.
- Menghasilkan battle outcome.
- Menghasilkan event timeline.
- Menghasilkan simulation metrics.
- Menghasilkan replay-compatible event records.

## 5. Simulation Core Architecture

Simulation Core merupakan bagian terpenting dari Engine.

Core harus dapat dijalankan tanpa:

- Laravel.
- PostgreSQL.
- Redis.
- HTTP.
- External network.
- Filesystem dependency.
- Host operating system commands.

Core menerima input eksplisit dan menghasilkan output eksplisit.

### 5.1 Core Execution Model

Secara konseptual:

```typescript
function runSimulation(input: SimulationInput): SimulationResult {
  // Validate input
  // Initialize deterministic context
  // Build runtime state
  // Process simulation events
  // Resolve final outcome
  // Return immutable result
}
```

Fungsi tersebut menggambarkan kontrak konseptual, bukan implementasi final.

### 5.2 Pure Core Principle

Simulation Core harus menghindari side effects.

Dilarang menggunakan:

- `Math.random()`.
- `Date.now()`.
- Uncontrolled asynchronous execution.
- External API calls.
- Direct database queries.
- Direct filesystem access.

Semua nilai yang dapat memengaruhi hasil simulasi harus berasal dari input yang eksplisit atau state internal yang deterministik.

### 5.3 Deterministic Execution

Input yang sama harus menghasilkan output yang sama.

Secara konseptual:

```text
Simulation Input
    +
Snapshot
    +
Seed
    +
Ruleset Version
    +
Engine Version
    +
Event Sequence
    |
    v
Simulation Result
```

Jika seluruh input dan versi identik, hasil simulasi harus identik.

Determinisme mencakup:

- Random number generation.
- Event ordering.
- Component processing order.
- Resource calculations.
- Network calculations.
- State transitions.
- Battle outcome.
- Result serialization.

## 6. Deterministic Random Generator

Randomness dibutuhkan untuk beberapa mekanisme simulasi.

Contoh:

- Attack success probability.
- Randomized events.
- Failure probability.
- Detection probability.
- Random attack variation.

Engine tidak boleh menggunakan global `Math.random()`.

Gunakan deterministic pseudorandom number generator (PRNG) dengan seed yang eksplisit.

### 6.1 RNG Requirements

- Seed harus menjadi bagian dari Simulation Input.
- Algoritma RNG harus memiliki versi.
- RNG state harus dapat direproduksi.
- Random calls harus mengikuti urutan deterministik.
- Tidak boleh mengambil randomness dari sistem operasi selama simulasi.

### 6.2 RNG Isolation

Hindari satu global RNG yang digunakan sembarangan oleh seluruh komponen.

Random stream dapat dipisahkan berdasarkan domain atau subsystem agar perubahan pada satu subsystem tidak secara tidak sengaja menggeser seluruh random sequence subsystem lainnya.

Contoh:

```text
Simulation Seed
      |
      +---- Attack RNG
      |
      +---- Defense RNG
      |
      +---- Environment RNG
      |
      +---- Failure RNG
```

Strategi derivasi seed dan stream isolation harus didefinisikan dalam ruleset.

## 7. Simulation Time and Execution Model

### 7.1 Simulation Clock

Simulation Engine menggunakan simulation time internal.

Simulation time tidak boleh bergantung pada waktu aktual server.

Engine harus memiliki:

- Simulation start time.
- Current simulation tick atau time step.
- Maximum simulation duration.
- Event scheduling.
- Simulation termination condition.

### 7.2 Execution Model

Untuk MVP, gunakan **deterministic discrete-event simulation dengan bounded execution steps**.

Artinya:

- Simulation Core memproses event berdasarkan simulation time.
- Event diproses dalam urutan yang eksplisit.
- Event yang memiliki timestamp sama harus memiliki tie-breaking order.
- Engine memiliki batas maksimum jumlah event dan durasi simulasi.
- Simulation time bergerak secara deterministik.

Model ini lebih fleksibel untuk infrastruktur jaringan dan event-based battle daripada mengandalkan wall-clock execution.

### 7.3 Event Ordering

Setiap event memiliki:

- Simulation timestamp.
- Event priority.
- Sequence number.
- Event type.
- Source identity.
- Payload.

Event harus memiliki deterministic ordering key.

Contoh:

```text
1. Simulation Timestamp
2. Event Priority
3. Sequence Number
4. Stable Entity Identifier
```

Urutan tersebut merupakan rancangan awal. Aturan tie-breaking final harus ditetapkan dan diuji.

### 7.4 Execution Limits

Setiap simulation job harus memiliki:

- Maximum event count.
- Maximum simulation duration.
- Maximum component count.
- Maximum event payload size.
- Maximum execution resource budget.
- Maximum recursion atau dependency depth jika relevan.

Batas ini mencegah malformed input atau simulation scenario menyebabkan worker menghabiskan resource tanpa batas.

## 8. Resource Simulation Architecture

Resource Simulation mengatur konsumsi resource virtual.

Resource MVP:

- CPU.
- RAM.
- Storage.
- Bandwidth.

### 8.1 Resource Model

Setiap component memiliki:

- Base resource requirements.
- Runtime resource consumption.
- Maximum supported capacity.
- Resource constraints.

Contoh:

```text
Linux Server
  CPU Capacity: 4 Units
  RAM Capacity: 8 GB
  Storage Capacity: 100 GB
  Bandwidth Capacity: 100 Mbps
```

Nilai di atas merupakan contoh konfigurasi, bukan nilai final component catalog.

### 8.2 Resource Allocation

Resource allocation harus mempertimbangkan:

- Component requirements.
- Infrastructure capacity.
- Runtime demand.
- Resource contention.
- Resource exhaustion.

### 8.3 Resource State

Resource state minimal mencakup:

```text
ResourceState
  capacity
  allocated
  consumed
  available
  utilization
```

Engine harus membedakan antara resource allocation dan runtime consumption.

### 8.4 Calculation Consistency

Perhitungan resource harus konsisten dan deterministik.

Untuk nilai yang berpotensi menimbulkan perbedaan floating-point, pertimbangkan fixed-point atau integer arithmetic dengan precision yang telah ditentukan.

Jangan mengandalkan floating-point equality untuk keputusan penting seperti batas kapasitas atau hasil battle.

## 9. Component Simulation Architecture

Setiap component memiliki definisi behavior dan runtime state.

### 9.1 Component Definition

Component definition berasal dari Component Catalog.

Informasi yang diperlukan:

- Component ID.
- Component version.
- Configuration schema.
- Base resource requirements.
- Supported interfaces.
- Dependencies.
- Behavior definition reference.

### 9.2 Component Runtime

Runtime component memiliki:

```text
ComponentRuntime
  id
  definitionId
  version
  configuration
  status
  resourceState
  interfaces
  metrics
```

### 9.3 Component Behavior

Setiap jenis komponen memiliki behavior module.

Contoh:

- Linux Server Behavior.
- Router Behavior.
- Firewall Behavior.
- Reverse Proxy Behavior.
- Web Application Behavior.
- PostgreSQL Behavior.
- Monitoring Behavior.

Behavior module tidak boleh memiliki ketergantungan terhadap Laravel atau database.

### 9.4 Component Lifecycle

Contoh status:

```text
STOPPED
   |
   v
STARTING
   |
   v
RUNNING
   |
   v
DEGRADED
   |
   v
FAILED
```

Status final dan transition rules akan ditentukan berdasarkan behavior masing-masing komponen.

### 9.5 Component Configuration

Configuration harus:

- Memiliki schema.
- Memiliki versi.
- Divalidasi sebelum simulasi.
- Tidak dieksekusi sebagai arbitrary host command.
- Diproses sebagai data virtual.

Konfigurasi firewall, routing, Linux service, database, dan aplikasi harus direpresentasikan sebagai structured simulation configuration.

## 10. Network Simulation Architecture

Network Simulation memodelkan hubungan komunikasi antarkomponen.

### 10.1 Network Model

Network topology direpresentasikan sebagai graph.

```text
Node:
- Router
- Firewall
- Server
- Reverse Proxy
- Web Application
- Database

Edge:
- Network Connection
- Logical Link
- Service Dependency
```

### 10.2 Network Responsibilities

- Connectivity validation.
- Routing decisions.
- Firewall rule evaluation.
- Connection establishment.
- Bandwidth limitation.
- Latency modeling.
- Connection failure handling.

### 10.3 Network and Workspace Separation

Posisi visual komponen pada isometric workspace tidak menentukan network connectivity.

Network topology ditentukan oleh logical connections dan aturan jaringan.

### 10.4 Network Abstraction

Engine tidak perlu mengimplementasikan seluruh detail protokol jaringan nyata pada MVP.

Simulasi awal menggunakan abstraction yang cukup untuk:

- Memodelkan packet atau request flow.
- Menghitung latency.
- Memeriksa routing.
- Mengevaluasi firewall rules.
- Memodelkan bandwidth.
- Menentukan service reachability.

Detail protokol dapat ditambahkan secara bertahap.

## 11. Attack and Defense Simulation

Attack and Defense Simulation merupakan domain utama gameplay CYBERFRONT.

Seluruh aktivitas bersifat virtual dan berjalan di dalam Simulation Engine yang terisolasi.

### 11.1 Attack Model

Attack direpresentasikan sebagai simulation scenario.

Contoh kategori:

- Reconnaissance.
- Network probing.
- Service enumeration.
- Authentication attack.
- Application-layer attack.
- Resource exhaustion scenario.

Daftar tersebut merupakan kategori gameplay dan tidak mengharuskan implementasi exploit nyata.

### 11.2 Defense Model

Defense mencakup:

- Firewall filtering.
- Network segmentation.
- Reverse proxy protection.
- Authentication controls.
- Application security configuration.
- Resource limits.
- Monitoring and detection.

### 11.3 Attack Resolution

Attack outcome dipengaruhi oleh:

- Attack scenario.
- Target component.
- Network path.
- Component configuration.
- Defense rules.
- Resource availability.
- Simulation seed.
- Ruleset version.

### 11.4 Defense Resolution

Defense outcome mempertimbangkan:

- Firewall rules.
- Component availability.
- Network connectivity.
- Security configuration.
- Resource constraints.
- Detection capabilities.

### 11.5 Result Model

Battle result minimal mencakup:

- Attack objective status.
- Target availability.
- Compromised component status.
- Resource impact.
- Detection outcome.
- Event timeline.
- Final battle outcome.

Engine menghasilkan technical result. Laravel menentukan konsekuensi bisnis seperti reward, progression, dan ekonomi.

## 12. Event Processing Architecture

Event Processing merupakan mekanisme komunikasi internal antarsubsystem.

Contoh event:

```text
ComponentStarted
ComponentFailed
ResourceExhausted
NetworkConnectionEstablished
NetworkConnectionFailed
FirewallBlockedTraffic
AttackStarted
AttackDetected
AttackSucceeded
AttackFailed
ServiceUnavailable
BattleResolved
```

### 12.1 Event Structure

Setiap event memiliki:

- Event ID atau deterministic sequence identity.
- Event type.
- Simulation timestamp.
- Sequence number.
- Source entity.
- Target entity jika ada.
- Payload.
- Event schema version.

### 12.2 Event Ordering

Event tidak boleh diproses berdasarkan urutan asynchronous completion yang tidak deterministik.

Engine harus menggunakan deterministic event scheduling.

### 12.3 Event Immutability

Event yang sudah dicatat sebagai bagian dari hasil simulasi tidak boleh diubah.

Perubahan event format harus menggunakan schema version baru.

### 12.4 Event Processing

Subsystem dapat menghasilkan event yang akan diproses oleh scheduler.

Contoh:

```text
Attack Subsystem
      |
      v
Event Scheduler
      |
      v
Network Subsystem
      |
      v
Firewall Behavior
      |
      v
Resource Subsystem
      |
      v
Event Recorder
```

## 13. Simulation State Architecture

Engine membedakan beberapa jenis state:

### 13.1 Input State

Data awal yang berasal dari Defense Snapshot dan simulation request.

### 13.2 Runtime State

State yang berubah selama simulasi.

Contoh:

- Component health.
- Resource consumption.
- Network connection status.
- Service availability.
- Attack progression.

### 13.3 Event State

Event sequence yang dihasilkan selama simulasi.

### 13.4 Result State

Hasil akhir yang dikembalikan ke Laravel.

Runtime state tidak boleh secara langsung mengubah persistent business data Laravel.

### 13.5 State Ownership

Simulation Engine memiliki runtime state selama execution berlangsung.

Laravel tetap memiliki persistent business state seperti:

- Player account.
- Credits.
- Inventory.
- Infrastructure ownership.
- Progression.
- Battle business status.

## 14. Replay Architecture

Replay merupakan kemampuan untuk merekonstruksi atau memutar ulang jalannya simulasi.

### 14.1 Replay Data

Replay harus memiliki:

- Battle ID.
- Snapshot reference.
- Snapshot hash.
- Simulation seed.
- Ruleset version.
- Engine version.
- Event sequence.
- Final result metadata.

### 14.2 Replay Strategy

MVP menggunakan recorded simulation events sebagai sumber utama visual replay.

Replay tidak boleh mengandalkan simulasi ulang dengan versi Engine terbaru.

Jika re-simulation diperlukan, Engine harus menggunakan versi ruleset dan execution behavior yang kompatibel dengan simulasi asli.

### 14.3 Replay Consistency

Replay harus:

- Mempertahankan event order.
- Menggunakan event sequence yang sama.
- Memiliki schema version.
- Memvalidasi event integrity.
- Dapat diakses tanpa menjalankan ulang battle.

### 14.4 Replay Storage

Event payload dapat disimpan oleh Battles Module melalui Laravel.

Untuk volume besar, event archive dapat dipindahkan ke object storage.

Simulation Engine hanya menghasilkan event data dan tidak menentukan lokasi penyimpanan final.

## 15. Versioning Architecture

Simulation Engine harus memiliki versioning yang eksplisit.

### 15.1 Engine Version

Menandakan versi implementasi Simulation Engine.

### 15.2 Ruleset Version

Menandakan versi aturan simulasi.

### 15.3 Component Behavior Version

Menandakan versi behavior komponen.

### 15.4 Snapshot Schema Version

Menandakan struktur input snapshot.

### 15.5 Event Schema Version

Menandakan struktur event output.

### 15.6 Compatibility

Engine harus dapat menentukan apakah kombinasi berikut kompatibel:

- Snapshot schema.
- Component catalog.
- Component behavior.
- Ruleset.
- Simulation contract.

Battle historis harus mempertahankan informasi versi yang dibutuhkan untuk audit dan replay.

Jangan mengasumsikan semua battle lama bisa dijalankan ulang oleh versi Engine terbaru.

## 16. Error Handling

Simulation Engine harus membedakan:

### 16.1 Validation Errors

Contoh:

- Invalid snapshot.
- Unsupported schema version.
- Invalid component reference.
- Invalid configuration.
- Invalid network connection.

### 16.2 Execution Errors

Contoh:

- Unexpected runtime exception.
- Event processing failure.
- Internal invariant violation.
- Execution limit exceeded.

### 16.3 Compatibility Errors

Contoh:

- Unsupported ruleset version.
- Missing component behavior.
- Unsupported component version.

### 16.4 Result Errors

Contoh:

- Invalid result structure.
- Incomplete event sequence.
- Result identity mismatch.

Error response harus memiliki:

- Error code.
- Error category.
- Safe error message.
- Correlation ID.
- Retryability classification.

Jangan mengembalikan stack trace atau informasi internal sensitif ke client.

## 17. Execution Isolation and Security

Simulation Engine harus berjalan dalam lingkungan terisolasi.

Persyaratan:

- Tidak menjalankan konfigurasi pemain sebagai host command.
- Tidak mengeksekusi arbitrary shell commands.
- Tidak mengakses production network.
- Tidak menerima akses langsung ke production database.
- Tidak mempercayai snapshot tanpa validation.
- Membatasi CPU dan memory worker.
- Membatasi execution time.
- Membatasi ukuran payload.
- Menggunakan service authentication pada integrasi eksternal.
- Mencatat request dan execution identity.

Semua attack scenario hanya memengaruhi virtual simulation state.

Engine tidak boleh memiliki kemampuan menyerang sistem nyata melalui simulation input.

## 18. Performance and Scalability

Strategi awal:

- Gunakan worker-based execution.
- Hindari blocking operation pada Simulation Core.
- Batasi jumlah event per simulation.
- Gunakan efficient data structures.
- Hindari serialization yang tidak dibutuhkan.
- Gunakan deterministic processing order.
- Profiling sebelum optimasi kompleks.
- Pisahkan job execution dari API handling.
- Dukung parallel execution untuk battle yang independen.

### 18.1 Parallelization

Battle yang berbeda dapat dieksekusi secara paralel jika masing-masing memiliki input dan state yang terisolasi.

Namun, subsystem dalam satu battle tidak boleh menjalankan state mutation secara paralel jika hasil akhirnya bergantung pada urutan event.

Deterministic ordering harus tetap dipertahankan.

### 18.2 Worker Scaling

Jumlah worker dapat ditingkatkan sesuai kebutuhan throughput.

Worker tidak boleh berbagi mutable runtime state antar battle.

## 19. Testing Strategy

### 19.1 Unit Tests

Menguji:

- PRNG.
- Resource calculations.
- Component behavior.
- Network behavior.
- Firewall evaluation.
- Event scheduler.
- State transitions.
- Attack and defense rules.

### 19.2 Determinism Tests

Menguji:

- Input identik menghasilkan output identik.
- Seed identik menghasilkan random sequence identik.
- Event ordering konsisten.
- Ruleset version konsisten.
- Result serialization konsisten.

### 19.3 Golden Tests

Simpan beberapa simulation fixtures sebagai baseline.

Setiap perubahan Engine harus membandingkan output baru dengan expected output.

Perubahan hasil yang disengaja harus memiliki penjelasan dan perubahan baseline yang direview.

### 19.4 Integration Tests

Menguji:

- Snapshot loading.
- Request validation.
- Ruleset registry.
- Simulation execution.
- Result generation.
- Event serialization.

### 19.5 Compatibility Tests

Menguji kombinasi:

- Snapshot schema version.
- Component versions.
- Ruleset versions.
- Event schema versions.

### 19.6 Performance Tests

Menguji:

- Component count.
- Event count.
- Simulation duration.
- Memory consumption.
- Worker throughput.

## 20. Recommended Directory Structure

Target struktur Simulation Engine:

```text
simulation/
├── src/
│   ├── core/
│   │   ├── engine/
│   │   ├── runtime/
│   │   ├── scheduler/
│   │   ├── events/
│   │   ├── state/
│   │   ├── random/
│   │   └── time/
│   │
│   ├── domains/
│   │   ├── infrastructure/
│   │   ├── components/
│   │   ├── resources/
│   │   ├── network/
│   │   ├── attacks/
│   │   ├── defense/
│   │   └── battles/
│   │
│   ├── contracts/
│   │   ├── input/
│   │   ├── output/
│   │   └── events/
│   │
│   ├── rulesets/
│   │   ├── registry/
│   │   └── versions/
│   │
│   ├── adapters/
│   │   ├── transport/
│   │   ├── logging/
│   │   └── serialization/
│   │
│   ├── workers/
│   │
│   └── index.ts
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── determinism/
│   ├── golden/
│   └── performance/
│
├── fixtures/
│   ├── snapshots/
│   ├── scenarios/
│   └── expected-results/
│
├── package.json
├── tsconfig.json
└── README.md
```

Struktur tersebut merupakan target awal dan dapat disesuaikan saat implementasi.

## 21. Architectural Constraints

Aturan wajib:

1. Simulation Engine harus independen dari Laravel.
2. Simulation Core tidak boleh mengakses database secara langsung.
3. Simulation Core tidak boleh melakukan HTTP request.
4. Simulation Core tidak boleh menggunakan uncontrolled randomness.
5. Simulation Core tidak boleh bergantung pada wall-clock time.
6. Event ordering harus deterministic.
7. Runtime state harus terisolasi per simulation job.
8. Snapshot harus divalidasi sebelum execution.
9. Ruleset dan component behavior harus versioned.
10. Hasil simulasi harus dapat direproduksi.
11. Replay harus menggunakan recorded event sequence.
12. Resource calculations harus konsisten.
13. Attack simulation hanya berjalan pada virtual environment.
14. Execution harus memiliki resource dan time limits.
15. Laravel tetap menjadi pemilik business state dan konsekuensi ekonomi.

## 22. Architecture Decision Record

### ADR-006: Independent Deterministic Simulation Engine

**Status:** Proposed

**Context**

CYBERFRONT membutuhkan sistem simulasi teknis yang dapat menjalankan infrastructure behavior, network interaction, resource consumption, attack, defense, dan battle resolution.

Simulation harus dapat direproduksi dan tidak bergantung pada Laravel runtime.

**Decision**

Menggunakan Independent TypeScript Simulation Engine dengan deterministic Simulation Core, modular domain architecture, deterministic event scheduler, dan versioned rulesets.

Execution infrastructure dipisahkan dari Simulation Core.

**Consequences**

Positive:

- Engine independen dari backend.
- Simulation dapat diuji secara terpisah.
- Battle dapat direproduksi.
- Mendukung asynchronous PvP.
- Menjadi fondasi real-time PvP.
- Domain simulasi dapat dikembangkan secara modular.

Trade-offs:

- Memerlukan pengelolaan versi dan compatibility.
- Determinism membutuhkan disiplin implementasi.
- Event ordering dan state management lebih kompleks.
- Engine membutuhkan testing yang lebih ketat.

**Alternatives Considered**

- Simulation logic di dalam Laravel.
- Simulation logic di frontend.
- Independent TypeScript Simulation Engine.

**Selected:** Independent TypeScript Simulation Engine.

## 23. Acceptance Criteria

Simulation Engine Architecture dianggap selesai apabila:

- [ ] Independent Simulation Engine ditetapkan.
- [ ] TypeScript dan Node.js ditetapkan sebagai technology baseline.
- [ ] Simulation Core dan Execution Infrastructure dipisahkan.
- [ ] Deterministic execution requirements didefinisikan.
- [ ] Random generator strategy ditetapkan.
- [ ] Simulation time dan event ordering didefinisikan.
- [ ] Resource simulation architecture didefinisikan.
- [ ] Component simulation architecture didefinisikan.
- [ ] Network simulation architecture didefinisikan.
- [ ] Attack and defense simulation architecture didefinisikan.
- [ ] Event processing architecture didefinisikan.
- [ ] Runtime state ownership ditetapkan.
- [ ] Replay architecture didefinisikan.
- [ ] Versioning strategy ditetapkan.
- [ ] Execution isolation dan security requirements ditetapkan.
- [ ] Testing strategy ditetapkan.
- [ ] Directory structure ditetapkan.
- [ ] ADR-006 terdokumentasi.

## 24. Related Documents

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/frontend-architecture.md`
- `docs/architecture/backend-architecture.md`
- `docs/architecture/database-architecture.md`
- `docs/architecture/simulation-engine-architecture.md`

---

**End of Document**
