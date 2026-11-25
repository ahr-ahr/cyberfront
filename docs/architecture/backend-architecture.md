# Backend Architecture

**Project:** CYBERFRONT  
**Document ID:** CF-ARCH-004  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.4 - Backend Architecture  
**Date:** November 25, 2026  
**Related Documents:**

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/frontend-architecture.md`

---

## 1. Purpose

Dokumen ini mendefinisikan arsitektur backend CYBERFRONT menggunakan Laravel 12 sebagai Modular Monolith.

Backend menjadi pusat pengelolaan business logic, persistent data, API, autentikasi, ekonomi, progression, PvP, battle lifecycle, serta komunikasi dengan Independent Simulation Engine.

Arsitektur harus mendukung:

- Modularitas dan pemisahan tanggung jawab.
- Business logic yang dapat diuji secara independen.
- Konsistensi data dan transaksi.
- Pemrosesan asynchronous melalui queue.
- Komunikasi dengan Simulation Engine.
- Deterministic simulation workflow.
- Keamanan dan integritas data pemain.
- Pengembangan fitur tanpa menciptakan ketergantungan antarmodul yang tidak terkendali.

Dokumen ini menetapkan struktur dan aturan desain backend. Implementasi aktual dilakukan pada fase development berikutnya.

## 2. Architectural Decisions

### 2.1 Backend Architecture Style

Backend menggunakan **Laravel 12 Modular Monolith**.

Seluruh modul backend berjalan dalam satu aplikasi Laravel dan menggunakan satu deployment utama pada tahap awal.

Setiap modul memiliki:

- Tanggung jawab bisnis yang spesifik.
- Application layer.
- Domain logic.
- Infrastructure atau persistence layer.
- Interface publik untuk komunikasi dengan modul lain.
- Test suite sendiri.

Modular Monolith dipilih karena memberikan keseimbangan antara kesederhanaan operasional dan pemisahan domain.

### 2.2 Core Technology

| Area                    | Technology                    |
| ----------------------- | ----------------------------- |
| Backend Framework       | Laravel 12                    |
| Language                | PHP                           |
| Database                | PostgreSQL                    |
| Queue & Cache           | Redis                         |
| API                     | REST API                      |
| Authentication          | Laravel Sanctum               |
| Background Processing   | Laravel Queue                 |
| Real-time Communication | WebSocket, integration layer  |
| Simulation              | Independent TypeScript Engine |
| Testing                 | PHPUnit / Pest                |

### 2.3 Architectural Principles

1. Separation of Concerns: setiap layer memiliki tanggung jawab jelas.
2. Explicit Module Boundaries: modul tidak boleh mengakses internal modul lain secara langsung.
3. Single Ownership: setiap domain data memiliki satu pemilik.
4. Business Logic Isolation: business rules tidak ditempatkan di controller.
5. Transaction Integrity: operasi finansial dan perubahan state penting harus konsisten.
6. Asynchronous by Default: pekerjaan berat diproses melalui queue.
7. Contract-Based Integration: komunikasi dengan Simulation Engine menggunakan kontrak eksplisit dan berversi.
8. Security by Design: seluruh input client dianggap tidak tepercaya.
9. Testability: business logic harus dapat diuji tanpa menjalankan seluruh aplikasi.
10. Incremental Complexity: hindari abstraksi dan infrastruktur yang belum dibutuhkan.

## 3. High-Level Backend Structure

Backend terdiri dari beberapa lapisan utama:

```text
Client
  |
  v
API Layer
  |
  v
Application Layer
  |
  v
Domain Layer
  |
  v
Infrastructure Layer
  |
  +------ PostgreSQL
  |
  +------ Redis
  |
  +------ External Simulation Engine
```

### 3.1 API Layer

Bertanggung jawab atas komunikasi antara frontend dan backend.

Komponen:

- Routes
- Controllers
- Form Requests
- API Resources
- Middleware
- Authentication dan authorization

API Layer tidak boleh menjadi tempat business logic utama.

Controller hanya bertugas:

1. Menerima HTTP request.
2. Memastikan request melewati autentikasi dan validasi.
3. Memanggil application use case.
4. Mengembalikan HTTP response melalui API Resource.

### 3.2 Application Layer

Application Layer mengatur pelaksanaan use case.

Contoh:

- CreateInfrastructure
- InstallComponent
- UpdateComponentConfiguration
- ConnectInfrastructureComponents
- PublishDefenseSnapshot
- CreateBattle
- SubmitBattleSimulation
- CompleteBattle
- PurchaseComponent

Tanggung jawab:

- Mengorkestrasi business workflow.
- Memanggil domain logic.
- Mengatur transaksi.
- Memanggil interface modul lain.
- Menjadwalkan background job.
- Menghasilkan application-level result.

Application Layer tidak boleh menjadi tempat menumpuk seluruh aturan bisnis. Aturan yang merupakan bagian dari domain harus tetap berada di Domain Layer.

### 3.3 Domain Layer

Domain Layer berisi aturan dan konsep bisnis utama.

Contoh:

- Credit balance dan ledger rules.
- Infrastructure constraints.
- Component compatibility.
- Resource allocation rules.
- Snapshot publication rules.
- Battle lifecycle rules.
- Progression requirements.

Domain Layer tidak boleh bergantung langsung pada:

- HTTP Request.
- Controller.
- Redis.
- Laravel Queue.
- External Simulation Engine.
- Frontend implementation.

Domain logic harus dapat diuji tanpa memerlukan HTTP request atau koneksi ke layanan eksternal.

### 3.4 Infrastructure Layer

Infrastructure Layer menangani implementasi teknis.

Contoh:

- Eloquent persistence.
- Database transactions.
- Redis integration.
- Queue dispatching.
- External HTTP client.
- Simulation Engine adapter.
- Event publishing.
- Logging.

Infrastructure Layer mengimplementasikan kebutuhan teknis yang digunakan oleh Application dan Domain Layer.

## 4. Backend Module Architecture

Backend terdiri dari delapan modul utama.

### 4.1 Identity & Access Module

Tanggung jawab:

- Registration.
- Login dan logout.
- Authentication.
- Session atau token management.
- Account security.
- Authorization policy.
- Account status.

Modul ini menjadi pemilik identitas dan akses pengguna.

### 4.2 Players Module

Tanggung jawab:

- Player profile.
- Player settings.
- Player statistics.
- Player preferences.
- Player account metadata.

Modul ini tidak memiliki kewenangan mengubah Credit Ledger secara langsung.

### 4.3 Infrastructure Module

Tanggung jawab:

- Infrastructure ownership.
- Infrastructure workspace.
- Component installation.
- Component placement metadata.
- Network topology metadata.
- Component configuration.
- Resource allocation requests.
- Defense Snapshot creation dan publication.

Modul ini mengelola definisi infrastruktur yang dimiliki pemain.

Runtime behavior tetap menjadi tanggung jawab Simulation Engine.

### 4.4 Component Catalog Module

Tanggung jawab:

- Component definitions.
- Component specifications.
- Component categories.
- Configuration schemas.
- Compatibility requirements.
- Base resource requirements.
- Catalog versioning.

Catalog menjadi sumber definisi komponen yang digunakan oleh backend dan Simulation Engine.

Perubahan definisi komponen yang berdampak pada simulasi harus memiliki versioning yang jelas.

### 4.5 Economy Module

Tanggung jawab:

- Credit balance.
- Credit ledger.
- Transactions.
- Purchase validation.
- Reward processing.
- Economic rules.

Aturan penting:

- Economy adalah satu-satunya pemilik Credit Ledger.
- Modul lain tidak boleh menulis saldo pemain secara langsung.
- Simulation Engine tidak boleh mengubah saldo pemain.
- Setiap perubahan saldo harus memiliki catatan transaksi.
- Operasi ekonomi harus menggunakan database transaction dan idempotency protection.

### 4.6 Progression Module

Tanggung jawab:

- Player progression.
- Experience points.
- Technology unlocks.
- Research requirements.
- Achievement tracking.
- Progression rewards.

Modul ini dapat meminta Economy memproses reward atau biaya, tetapi tidak boleh memodifikasi ledger secara langsung.

### 4.7 PvP Module

Tanggung jawab:

- Matchmaking.
- Opponent discovery.
- PvP eligibility.
- Match request.
- Match coordination.
- PvP-related rules.

PvP menginisiasi workflow battle melalui interface publik Battles Module.

PvP tidak boleh mengimplementasikan battle resolution atau attack simulation.

### 4.8 Battles Module

Tanggung jawab:

- Battle lifecycle.
- Battle creation.
- Attack request validation.
- Defense Snapshot reference.
- Simulation job orchestration.
- Simulation result handling.
- Battle Report persistence.
- Replay metadata.
- Battle completion state.

Battles Module mengoordinasikan simulasi, tetapi tidak menghitung hasil simulasi secara mandiri.

### 4.9 Independent Simulation Engine

Simulation Engine merupakan sistem terpisah dari Laravel.

Tanggung jawab:

- Runtime infrastructure simulation.
- Component behavior.
- Network behavior.
- Attack and defense mechanics.
- Resource consumption.
- Event processing.
- Deterministic battle resolution.
- Replay event generation.

Laravel hanya berkomunikasi dengan Engine melalui kontrak yang telah ditentukan.

## 5. Recommended Directory Structure

Struktur berikut merupakan target arsitektur backend:

```text
app/
├── Modules/
│   ├── IdentityAccess/
│   │   ├── Application/
│   │   │   ├── Commands/
│   │   │   └── Queries/
│   │   ├── Domain/
│   │   │   ├── Entities/
│   │   │   ├── ValueObjects/
│   │   │   ├── Events/
│   │   │   └── Exceptions/
│   │   ├── Infrastructure/
│   │   │   ├── Persistence/
│   │   │   └── Services/
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   ├── Requests/
│   │   │   └── Resources/
│   │   └── Providers/
│   │
│   ├── Players/
│   ├── Infrastructure/
│   ├── ComponentCatalog/
│   ├── Economy/
│   ├── Progression/
│   ├── Pvp/
│   └── Battles/
│
├── Shared/
│   ├── Contracts/
│   ├── Exceptions/
│   ├── Support/
│   └── ValueObjects/
│
└── Providers/
```

Setiap modul mengikuti struktur internal yang konsisten, tetapi tidak harus memiliki semua direktori apabila belum dibutuhkan.

Contoh struktur lengkap satu modul:

```text
Infrastructure/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── DTOs/
│   └── Services/
│
├── Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Events/
│   ├── Rules/
│   └── Exceptions/
│
├── Infrastructure/
│   ├── Persistence/
│   │   ├── Models/
│   │   └── Repositories/
│   └── Services/
│
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   └── Resources/
│
├── Providers/
└── Tests/
    ├── Unit/
    ├── Feature/
    └── Integration/
```

Struktur ini adalah pedoman, bukan alasan untuk membuat puluhan class kosong. Gunakan abstraksi ketika memang ada kebutuhan.

## 6. Dependency Rules

### 6.1 Layer Dependency

Aturan dependency:

- HTTP Layer boleh memanggil Application Layer.
- Application Layer boleh menggunakan Domain Layer.
- Infrastructure Layer boleh mengimplementasikan interface yang dibutuhkan Application atau Domain.
- Domain Layer tidak boleh bergantung pada HTTP atau Infrastructure Layer.
- Controller tidak boleh memanggil Eloquent model modul lain untuk menjalankan workflow bisnis.

### 6.2 Module Dependency

Komunikasi antarmodul harus menggunakan salah satu mekanisme:

1. Public Application Interface.
2. Domain Event.
3. Explicit Contract.

Contoh:

```text
PvP Module
    |
    v
Battles Public Interface
    |
    v
Battles Application
```

Contoh lainnya:

```text
Infrastructure Module
    |
    v
Economy Public Interface
    |
    v
Credit Transaction
```

Modul tidak boleh mengakses repository, model, atau tabel internal modul lain untuk melewati interface tersebut.

### 6.3 Dependency Direction

Dependency harus mengarah ke interface publik, bukan implementasi internal.

Hindari circular dependency.

Contoh yang dilarang:

```text
PvP Module ------> Battles Module
     ^                   |
     |___________________|
```

Contoh yang disarankan:

```text
PvP Module
    |
    v
Battles Public Interface
    |
    v
Battles Application
```

Battles tidak perlu bergantung kembali pada PvP. Jika hasil battle perlu memengaruhi status match, gunakan application workflow atau event yang ditangani oleh pemilik data terkait.

## 7. API Architecture

### 7.1 API Style

Backend menggunakan REST API dengan version prefix:

```text
/api/v1/
```

Contoh endpoint:

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout

GET    /api/v1/player/profile

GET    /api/v1/infrastructures
POST   /api/v1/infrastructures
GET    /api/v1/infrastructures/{id}
PATCH  /api/v1/infrastructures/{id}

POST   /api/v1/infrastructures/{id}/components
PATCH  /api/v1/components/{id}/configuration
POST   /api/v1/infrastructures/{id}/connections

GET    /api/v1/component-catalog

GET    /api/v1/economy/balance
GET    /api/v1/economy/transactions

POST   /api/v1/pvp/matches

POST   /api/v1/battles
GET    /api/v1/battles/{id}
GET    /api/v1/battles/{id}/report
GET    /api/v1/battles/{id}/replay
```

Endpoint tersebut merupakan rancangan awal dan belum menjadi kontrak API final.

### 7.2 Request Lifecycle

Alur request:

```text
HTTP Request
    |
    v
Route
    |
    v
Middleware
    |
    v
Form Request Validation
    |
    v
Controller
    |
    v
Application Use Case
    |
    v
Domain Logic
    |
    v
Persistence / External Service
    |
    v
API Resource
    |
    v
HTTP Response
```

### 7.3 API Response Format

Response sukses:

```json
{
  "data": {
    "id": "resource-id",
    "name": "Main Infrastructure"
  },
  "meta": {
    "request_id": "request-id"
  }
}
```

Response error:

```json
{
  "error": {
    "code": "INFRASTRUCTURE_NOT_FOUND",
    "message": "Infrastructure not found.",
    "details": {}
  },
  "meta": {
    "request_id": "request-id"
  }
}
```

Response harus konsisten dan tidak mengekspos stack trace, internal path, secret, atau detail implementasi.

### 7.4 Validation

Validasi dilakukan pada beberapa tingkat:

- HTTP validation untuk format dan tipe input.
- Application validation untuk prasyarat use case.
- Domain validation untuk aturan bisnis.
- Database constraints untuk integritas data.

Validasi frontend hanya membantu UX. Backend tetap menjadi sumber kebenaran.

## 8. Database Architecture

### 8.1 Database Strategy

PostgreSQL menjadi primary persistent database.

Pada tahap awal, seluruh modul menggunakan satu database dengan kepemilikan tabel yang dipisahkan secara logis.

Contoh:

| Module            | Table Prefix / Ownership                                               |
| ----------------- | ---------------------------------------------------------------------- |
| Identity & Access | users, personal_access_tokens                                          |
| Players           | players, player_settings                                               |
| Infrastructure    | infrastructures, infrastructure_components, infrastructure_connections |
| Component Catalog | component_definitions, component_versions                              |
| Economy           | credit_accounts, credit_transactions                                   |
| Progression       | player_progressions, technology_unlocks                                |
| PvP               | pvp_matches, matchmaking_records                                       |
| Battles           | battles, battle_reports, battle_events                                 |

Nama tabel final dapat berubah mengikuti kebutuhan implementasi.

### 8.2 Data Ownership

Setiap modul hanya boleh mengubah data yang dimilikinya.

Contoh:

- Economy memiliki dan mengubah credit ledger.
- Infrastructure memiliki konfigurasi dan topology.
- Battles memiliki battle lifecycle dan battle reports.
- Component Catalog memiliki definisi komponen.

Modul lain boleh membaca informasi melalui public interface atau read model yang disepakati, tetapi tidak boleh menulis tabel milik modul lain.

### 8.3 Database Transactions

Gunakan transaksi database untuk operasi yang membutuhkan konsistensi atomik.

Contoh:

- Pembelian komponen dan pengurangan Credits.
- Instalasi komponen dan pencatatan penggunaan resource.
- Publikasi Defense Snapshot.
- Penyelesaian battle dan pencatatan hasil ekonomi.

Untuk workflow yang melibatkan beberapa sistem atau proses asynchronous, jangan menganggap satu database transaction dapat mencakup semuanya.

Gunakan state transition, idempotency, retry policy, dan compensating action jika diperlukan.

### 8.4 Snapshot Integrity

Defense Snapshot harus immutable setelah dipublikasikan.

Snapshot menyimpan atau mereferensikan:

- Infrastructure state.
- Component configuration.
- Network topology.
- Resource allocation.
- Component catalog version.
- Simulation ruleset version.
- Snapshot schema version.
- Snapshot hash.

Battle harus menggunakan snapshot yang sama sepanjang lifecycle-nya.

Perubahan pada infrastruktur pemain setelah snapshot diterbitkan tidak boleh mengubah snapshot yang sudah digunakan oleh battle.

## 9. Queue and Background Processing

Redis digunakan untuk queue dan cache.

Laravel Queue digunakan untuk pekerjaan asynchronous seperti:

- Simulation job dispatch.
- Simulation result processing.
- Battle report generation.
- Replay event persistence.
- Notification delivery.
- Non-critical analytics processing.

### 9.1 Queue Design

Job harus:

- Memiliki identitas yang dapat dilacak.
- Mendukung retry dengan batas yang jelas.
- Memiliki timeout.
- Aman dijalankan ulang jika terjadi retry.
- Tidak mengandalkan state lokal worker.
- Mencatat kegagalan secara terstruktur.

### 9.2 Idempotency

Operasi yang dapat dipanggil berulang harus memiliki perlindungan idempotency.

Contoh:

- Submit battle simulation.
- Process simulation result.
- Apply battle reward.
- Purchase component.
- Complete battle.

Satu hasil simulasi tidak boleh menyebabkan reward diberikan dua kali hanya karena job diproses ulang.

### 9.3 Transactional Outbox

Untuk event penting yang harus dikirim setelah database transaction berhasil, gunakan pola Transactional Outbox jika diperlukan.

Contoh:

```text
Database Transaction
    |
    +-- Save Battle State
    |
    +-- Save Outbox Event
    |
    v
Commit
    |
    v
Outbox Publisher
    |
    v
Queue / External Service
```

Pola ini mengurangi risiko database berhasil menyimpan perubahan tetapi event gagal dikirim.

Transactional Outbox merupakan kebutuhan desain untuk workflow kritis, bukan kewajiban menerapkannya pada setiap event sederhana.

## 10. Simulation Engine Integration

### 10.1 Integration Principle

Laravel dan Simulation Engine merupakan dua sistem independen.

Laravel:

- Mengelola business state.
- Memvalidasi permintaan.
- Menyiapkan Defense Snapshot.
- Mengatur simulation job.
- Mengelola status battle.
- Memvalidasi dan menyimpan hasil.
- Mengaplikasikan konsekuensi ekonomi dan progression.

Simulation Engine:

- Menjalankan simulasi.
- Memproses event.
- Menghitung hasil attack dan defense.
- Menghasilkan simulation result.
- Menghasilkan event log untuk replay.

### 10.2 Integration Adapter

Backend harus menyediakan adapter khusus untuk komunikasi dengan Simulation Engine.

Contoh:

```text
Battles Module
    |
    v
Simulation Application Interface
    |
    v
Simulation Adapter
    |
    v
Transport Layer
    |
    v
Simulation Engine
```

Battles Module tidak boleh mengetahui detail internal protokol atau implementasi transport Engine.

### 10.3 Simulation Job Lifecycle

```text
Battle Requested
    |
    v
Validate Battle Request
    |
    v
Create Immutable Snapshot Reference
    |
    v
Create Simulation Job
    |
    v
Dispatch Simulation Request
    |
    v
Engine Executes Simulation
    |
    v
Receive Simulation Result
    |
    v
Validate Result Contract
    |
    v
Persist Battle Report
    |
    v
Apply Authorized Business Effects
    |
    v
Battle Completed
```

### 10.4 Simulation Contract

Request dan response harus memiliki:

- Contract version.
- Battle ID.
- Snapshot ID atau immutable snapshot payload.
- Snapshot hash.
- Simulation seed.
- Ruleset version.
- Request ID.
- Result status.
- Event sequence atau event reference.
- Result metadata.

Engine harus menggunakan seed dan ruleset yang sama agar simulasi dapat direproduksi.

Laravel tidak boleh mengubah hasil simulasi secara diam-diam. Jika hasil tidak valid, proses harus ditolak atau masuk ke state failure yang dapat diaudit.

### 10.5 Failure Handling

Contoh failure:

- Engine unavailable.
- Request timeout.
- Invalid contract version.
- Invalid snapshot hash.
- Duplicate result.
- Incomplete event sequence.
- Simulation execution failure.

Setiap failure harus menghasilkan status dan error code yang dapat ditelusuri.

Retry tidak boleh menghasilkan battle result atau reward ganda.

Transport final, mekanisme authentication antarsistem, dan detail payload akan ditentukan pada Task 02.7: Simulation Engine Integration Contract.

## 11. Battle Lifecycle Management

Battles Module mengelola state transition battle.

State awal yang diusulkan:

```text
CREATED
   |
   v
VALIDATING
   |
   v
QUEUED
   |
   v
RUNNING
   |
   v
PROCESSING_RESULT
   |
   v
COMPLETED
```

Failure state:

```text
VALIDATING ------> REJECTED

QUEUED ----------> FAILED

RUNNING ---------> FAILED

PROCESSING_RESULT -> FAILED
```

Ketentuan:

- State transition harus eksplisit.
- Transition yang tidak valid harus ditolak.
- Setiap perubahan state harus dapat diaudit.
- Status `COMPLETED` hanya diberikan setelah hasil valid berhasil dipersist.
- Reward dan konsekuensi bisnis harus diproses secara idempotent.

State final dan kebijakan retry akan diperinci pada desain Battles Module.

## 12. Authentication and Authorization

### 12.1 Authentication

Laravel Sanctum menjadi pilihan awal untuk authentication API.

Authentication bertanggung jawab memastikan identitas pengguna.

### 12.2 Authorization

Authorization menggunakan policy atau mekanisme permission yang sesuai dengan domain.

Contoh:

- Player hanya dapat mengubah infrastrukturnya sendiri.
- Player tidak dapat membaca private Defense Snapshot milik pemain lain.
- Player tidak dapat mengubah Battle Report.
- Economy operation harus melewati otorisasi dan validasi.
- Admin operation harus memiliki permission khusus.

Authentication tidak otomatis memberikan akses ke seluruh resource.

### 12.3 Resource Ownership

Setiap operasi terhadap resource harus memeriksa ownership atau permission.

Jangan hanya mengandalkan UUID yang sulit ditebak. ID bukan sistem keamanan, meskipun manusia sering memperlakukannya seperti itu.

## 13. Security Architecture

Backend harus menerapkan:

- Input validation.
- Authorization checks.
- Rate limiting.
- CSRF protection untuk konteks yang relevan.
- Secure authentication.
- Secret management melalui environment atau secret manager.
- Safe logging.
- SQL injection protection melalui parameterized queries dan ORM.
- Protection terhadap mass assignment.
- Output serialization yang aman.
- Audit trail untuk operasi kritis.
- Request correlation ID.

Konfigurasi teknis yang dikirim pemain adalah data simulasi.

Backend tidak boleh mengeksekusi konfigurasi pemain sebagai arbitrary host command atau menjalankannya langsung pada server produksi.

Simulation Engine juga harus berjalan dalam lingkungan terisolasi.

## 14. Observability and Logging

Backend harus menyediakan observability untuk debugging dan operasional.

Komponen:

- Structured application logs.
- Request ID.
- Correlation ID.
- Queue job ID.
- Battle ID.
- Simulation request ID.
- Error tracking.
- Queue failure monitoring.
- Database performance monitoring.

Setiap workflow battle harus dapat ditelusuri dari request awal hingga Battle Report selesai.

Jangan memasukkan password, access token, secret, atau informasi sensitif ke log.

## 15. Testing Strategy

### 15.1 Unit Tests

Menguji:

- Domain rules.
- Value objects.
- Credit ledger rules.
- Resource allocation rules.
- Snapshot validation.
- Battle state transitions.
- Application use cases.

### 15.2 Feature Tests

Menguji:

- API endpoints.
- Authentication.
- Authorization.
- Request validation.
- API response format.
- Database persistence.

### 15.3 Integration Tests

Menguji:

- Module interfaces.
- Database transactions.
- Queue dispatching.
- Event publishing.
- Simulation adapter.
- Result processing.

### 15.4 Architecture Tests

Menggunakan automated checks untuk memastikan:

- Domain tidak bergantung pada HTTP layer.
- Modul tidak mengakses internal persistence modul lain.
- Circular dependency tidak muncul.
- Controller tidak memuat business logic kompleks.
- Modul tidak menulis tabel milik modul lain.

### 15.5 End-to-End Tests

Menguji workflow penting:

- Player registration.
- Infrastructure creation.
- Component installation.
- Configuration update.
- Defense Snapshot publication.
- Battle submission.
- Simulation result processing.
- Battle Report retrieval.

## 16. Performance and Scalability

Strategi awal:

- Gunakan pagination untuk collection API.
- Gunakan eager loading untuk menghindari N+1 queries.
- Gunakan database indexes berdasarkan pola query.
- Gunakan Redis untuk cache data yang tepat.
- Gunakan queue untuk pekerjaan berat.
- Hindari menjalankan simulasi di dalam HTTP request.
- Gunakan lazy loading pada data yang tidak diperlukan.
- Monitor query performance sebelum melakukan optimasi besar.

Modular Monolith dapat diskalakan secara horizontal pada level application dan worker jika dibutuhkan.

Pemisahan menjadi microservices bukan target otomatis. Ekstraksi modul hanya dipertimbangkan jika ada kebutuhan scaling, deployment, reliability, atau ownership yang benar-benar membenarkannya.

## 17. Configuration and Environment Management

Konfigurasi aplikasi harus dipisahkan dari source code.

Contoh environment variables:

```env
APP_ENV=
APP_DEBUG=
APP_URL=

DB_CONNECTION=pgsql
DB_HOST=
DB_PORT=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=

REDIS_HOST=
REDIS_PORT=
REDIS_PASSWORD=

SIMULATION_ENGINE_URL=
SIMULATION_ENGINE_TIMEOUT=
SIMULATION_ENGINE_CONTRACT_VERSION=
```

Nilai secret tidak boleh disimpan di repository.

Environment yang disiapkan:

- Local Development.
- Testing.
- Staging.
- Production.

Setiap environment harus memiliki konfigurasi dan kredensial yang terpisah.

## 18. Architectural Constraints

Aturan yang wajib dipertahankan:

1. Laravel menggunakan Modular Monolith.
2. Setiap modul memiliki ownership domain yang jelas.
3. Controller tidak menyimpan business logic utama.
4. Domain Layer tidak bergantung pada framework-specific HTTP implementation.
5. Modul tidak mengakses tabel internal modul lain untuk melakukan perubahan.
6. Economy menjadi satu-satunya pemilik Credit Ledger.
7. Simulation Engine tetap independen dari Laravel.
8. Laravel tidak menghitung hasil simulasi.
9. Battle Snapshot harus immutable.
10. Simulation workflow harus idempotent.
11. Queue digunakan untuk pekerjaan asynchronous.
12. API dan Engine integration harus menggunakan versioned contract.
13. Semua operasi kritis harus dapat diaudit.
14. Tidak ada konfigurasi pemain yang boleh dieksekusi sebagai arbitrary host command.

## 19. Architecture Decision Record

### ADR-004: Backend Modular Monolith Architecture

**Status:** Proposed

**Context**

CYBERFRONT membutuhkan backend yang dapat menangani berbagai domain bisnis dan workflow kompleks, tetapi belum membutuhkan kompleksitas operasional microservices.

**Decision**

Menggunakan Laravel 12 Modular Monolith dengan pemisahan modul, layer, dan ownership yang eksplisit.

**Consequences**

Positive:

- Deployment lebih sederhana.
- Business workflow lebih mudah dikelola.
- Modul dapat dikembangkan secara terpisah.
- Database transaction lebih mudah dikelola dalam satu aplikasi.
- Dapat diekstrak menjadi service terpisah jika memang diperlukan.

Trade-offs:

- Disiplin dependency harus dijaga.
- Modul berbagi runtime dan deployment.
- Isolasi resource antarmodul tidak sekuat microservices.
- Tanpa architecture tests, batas modul dapat perlahan melemah.

**Alternatives Considered**

- Traditional Layered Monolith.
- Microservices.
- Modular Monolith.

**Selected:** Modular Monolith.

## 20. Acceptance Criteria

Backend Architecture dianggap selesai apabila:

- [ ] Laravel 12 Modular Monolith ditetapkan.
- [ ] Layer responsibilities terdokumentasi.
- [ ] Delapan modul backend didefinisikan.
- [ ] Directory structure disepakati.
- [ ] Dependency rules ditetapkan.
- [ ] API architecture terdokumentasi.
- [ ] Database ownership ditetapkan.
- [ ] Queue dan background processing strategy ditetapkan.
- [ ] Simulation Engine integration boundary ditetapkan.
- [ ] Battle lifecycle awal didefinisikan.
- [ ] Authentication dan authorization strategy ditetapkan.
- [ ] Security requirements terdokumentasi.
- [ ] Testing strategy ditetapkan.
- [ ] ADR-004 terdokumentasi.

## 21. Related Documents

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/frontend-architecture.md`
- `docs/architecture/backend-architecture.md`

---

**End of Document**
