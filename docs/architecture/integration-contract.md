# Simulation Engine Integration Contract

**Project:** CYBERFRONT  
**Document ID:** CF-ARCH-007  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.7 - Simulation Engine Integration Contract  
**Date:** November 25, 2026

**Related Documents:**

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/backend-architecture.md`
- `docs/architecture/database-architecture.md`
- `docs/architecture/simulation-engine-architecture.md`

---

## 1. Purpose

Dokumen ini mendefinisikan kontrak komunikasi antara Laravel Backend dan Independent TypeScript Simulation Engine.

Kedua sistem memiliki tanggung jawab yang berbeda:

**Laravel Backend**

- Mengelola business state.
- Memvalidasi battle request.
- Menyediakan Defense Snapshot.
- Membuat dan mengirim simulation job.
- Mengelola battle lifecycle.
- Menerima dan memvalidasi simulation result.
- Menyimpan Battle Report.
- Mengaplikasikan konsekuensi ekonomi dan progression.

**Simulation Engine**

- Menerima simulation request.
- Memvalidasi simulation input.
- Menjalankan deterministic simulation.
- Memproses component behavior.
- Memproses network dan resource behavior.
- Menghitung attack dan defense outcome.
- Menghasilkan event sequence.
- Mengembalikan simulation result.

Dokumen ini menjadi kontrak integrasi yang harus dipatuhi kedua sistem tanpa bergantung pada implementasi internal masing-masing.

## 2. Integration Architecture

### 2.1 Integration Style

MVP menggunakan **asynchronous HTTP-based integration**.

Laravel mengirim simulation job melalui internal HTTP API. Simulation Engine menerima job, memasukkannya ke execution workflow, lalu menjalankan simulasi melalui worker.

Setelah selesai, Engine mengirim hasil melalui authenticated callback ke Laravel.

Laravel juga menyediakan mekanisme status lookup sebagai fallback apabila callback terlambat atau gagal diterima.

### 2.2 High-Level Flow

```text
        Laravel Backend
              |
              v
      Create Simulation Job
              |
              v
       Laravel Queue Worker
              |
              v
       HTTP Submit Request
              |
              v
      Simulation Engine API
              |
              v
       Request Validation
              |
              v
       Engine Execution Queue
              |
              v
       Simulation Worker
              |
              v
       Deterministic Core
              |
              v
       Simulation Result
              |
              v
       Result Callback
              |
              v
       Laravel Callback API
              |
              v
       Result Validation
              |
              v
       Persist Battle Report
              |
              v
       Apply Business Effects
```

### 2.3 Integration Boundaries

Laravel tidak boleh:

- Menghitung attack outcome.
- Menghitung defense effectiveness.
- Mengubah simulation result secara diam-diam.
- Mengakses runtime state internal Engine.
- Mengasumsikan job langsung selesai setelah request diterima.

Simulation Engine tidak boleh:

- Mengubah player Credits.
- Mengubah inventory.
- Mengubah player progression.
- Mengubah infrastructure ownership.
- Mengakses Laravel database secara langsung.
- Menentukan business rewards.

## 3. Transport Architecture

### 3.1 MVP Transport

Transport awal:

**Internal HTTP REST API**

Alasan:

- Mudah diuji.
- Memiliki kontrak eksplisit.
- Tidak mengharuskan kedua sistem menggunakan bahasa atau runtime yang sama.
- Mendukung asynchronous processing.
- Memungkinkan deployment terpisah.
- Memudahkan monitoring dan debugging.

Komunikasi dilakukan melalui internal network yang terlindungi.

### 3.2 Transport Responsibilities

| Direction        | Purpose                               |
| ---------------- | ------------------------------------- |
| Laravel → Engine | Submit simulation job                 |
| Laravel → Engine | Query simulation status               |
| Engine → Laravel | Deliver simulation result             |
| Engine → Laravel | Deliver terminal failure notification |

### 3.3 Transport Independence

Simulation Core tidak boleh mengetahui detail HTTP transport.

Transport Adapter bertugas:

- Menerima HTTP request.
- Memvalidasi transport-level authentication.
- Mengubah payload menjadi internal command.
- Mengirim command ke execution workflow.
- Mengubah hasil menjadi response contract.

Perubahan transport di masa depan tidak boleh mengubah domain simulation logic.

## 4. API Versioning

### 4.1 API Version

Semua endpoint Engine menggunakan version prefix:

```text
/api/v1/
```

Contoh:

```text
POST /api/v1/simulations
GET  /api/v1/simulations/{simulation_id}
```

### 4.2 Contract Version

Setiap request dan response harus memiliki:

```json
{
  "contract_version": "1.0.0"
}
```

Contract version digunakan untuk mengidentifikasi format payload dan perilaku komunikasi.

### 4.3 Versioning Rules

- Breaking changes memerlukan major version baru.
- Backward-compatible additions menggunakan minor version.
- Documentation atau metadata corrections menggunakan patch version.
- Engine harus menolak contract version yang tidak didukung.
- Laravel harus memvalidasi response berdasarkan contract version.
- Versi lama harus dipertahankan selama masih ada job yang membutuhkan compatibility.

### 4.4 Independent Versioning

Contract version tidak sama dengan:

- Engine version.
- Snapshot schema version.
- Component catalog version.
- Ruleset version.
- Event schema version.

Setiap versi memiliki tujuan sendiri.

## 5. Simulation Job Identity

Setiap simulation request harus memiliki identitas unik.

### 5.1 Required Identifiers

| Field           | Purpose                           |
| --------------- | --------------------------------- |
| request_id      | Identitas satu pengiriman request |
| simulation_id   | Identitas execution job           |
| battle_id       | Identitas battle di Laravel       |
| snapshot_id     | Identitas Defense Snapshot        |
| correlation_id  | Melacak workflow lintas sistem    |
| idempotency_key | Mencegah duplicate submission     |

### 5.2 Identity Rules

- `simulation_id` harus stabil selama lifecycle satu simulation job.
- Retry submission harus menggunakan `simulation_id` yang sama.
- Retry tidak boleh membuat execution baru untuk logical job yang sama.
- `request_id` dapat berbeda pada setiap HTTP attempt.
- `correlation_id` tetap sama sepanjang workflow.
- Satu battle MVP memiliki satu logical simulation job utama.

Jika battle memerlukan rerun administratif, rerun harus dibuat sebagai job baru yang memiliki identity dan audit trail berbeda.

## 6. Authentication and Request Security

### 6.1 Service Authentication

Komunikasi antarservice wajib menggunakan service-to-service authentication.

Pilihan awal:

- TLS untuk transport encryption.
- HMAC request signing untuk autentikasi dan integritas payload.
- Rotatable service secret.
- Timestamp validation.
- Request replay protection.

Mutual TLS dapat ditambahkan sesuai kebutuhan deployment.

### 6.2 Request Signature

Request signature secara konseptual dihitung dari:

```text
HMAC-SHA256(
  secret,
  timestamp + "." + raw_request_body
)
```

HTTP headers:

```text
X-CF-Service-Id
X-CF-Timestamp
X-CF-Signature
X-CF-Request-Id
```

Ketentuan:

- Signature dibandingkan menggunakan constant-time comparison.
- Timestamp harus berada dalam toleransi waktu yang ditentukan.
- Request ID dan idempotency key harus diperiksa untuk mencegah replay.
- Secret tidak boleh disimpan di source code.
- Secret harus dapat dirotasi.

Signature algorithm, canonicalization, dan key rotation procedure harus memiliki implementasi yang konsisten pada kedua service.

### 6.3 Callback Authentication

Callback dari Engine ke Laravel menggunakan mekanisme autentikasi yang setara.

Laravel harus memverifikasi:

- Service identity.
- Request signature.
- Timestamp.
- Request ID.
- Callback payload.
- Simulation identity.

### 6.4 Network Security

- Engine API tidak diekspos secara publik tanpa kebutuhan.
- Komunikasi dilakukan melalui internal network.
- Gunakan firewall dan service access control.
- Batasi ukuran payload.
- Terapkan rate limiting.
- Callback URL ditentukan melalui konfigurasi service, bukan URL bebas dari request pemain.

## 7. Submit Simulation API

### 7.1 Endpoint

```http
POST /api/v1/simulations
```

### 7.2 Purpose

Mengirim simulation job baru dari Laravel ke Simulation Engine.

### 7.3 Request Payload

Contoh konseptual:

```json
{
  "contract_version": "1.0.0",
  "request_id": "uuid",
  "simulation_id": "uuid",
  "battle_id": "uuid",
  "correlation_id": "uuid",
  "idempotency_key": "battle-simulation-uuid",
  "simulation": {
    "engine_version": "1.0.0",
    "ruleset_version": "1.0.0",
    "rng_algorithm_version": "1.0.0",
    "seed": "123456789",
    "max_simulation_ticks": 10000,
    "max_event_count": 50000
  },
  "snapshot": {
    "snapshot_id": "uuid",
    "snapshot_hash": "sha256-hash",
    "schema_version": "1.0.0",
    "catalog_version": "1.0.0",
    "payload": {}
  },
  "scenario": {
    "scenario_id": "uuid",
    "scenario_version": "1.0.0",
    "attacker": {
      "player_id": "uuid"
    },
    "defender": {
      "player_id": "uuid"
    },
    "target_component_ids": []
  },
  "metadata": {
    "created_at": "2026-11-25T12:00:00Z"
  }
}
```

Payload snapshot di atas harus berisi seluruh data yang diperlukan Engine untuk menjalankan simulasi.

Engine tidak boleh mengambil konfigurasi tambahan dari Laravel database.

### 7.4 Request Validation

Engine harus memvalidasi:

- Contract version.
- Required fields.
- Simulation identity.
- Battle identity.
- Snapshot hash.
- Snapshot schema version.
- Component references.
- Component catalog compatibility.
- Ruleset compatibility.
- RNG algorithm compatibility.
- Scenario validity.
- Execution limits.

Jika input tidak valid, Engine harus menolak request dengan error yang sesuai.

### 7.5 Accepted Response

Jika job diterima:

HTTP Status:

```text
202 Accepted
```

Response:

```json
{
  "contract_version": "1.0.0",
  "request_id": "uuid",
  "simulation_id": "uuid",
  "battle_id": "uuid",
  "status": "ACCEPTED",
  "accepted_at": "2026-11-25T12:00:01Z"
}
```

HTTP 202 berarti job diterima untuk diproses, bukan berarti simulation berhasil.

Laravel tidak boleh mengubah battle menjadi COMPLETED hanya karena menerima response ini.

## 8. Simulation Status API

### 8.1 Endpoint

```http
GET /api/v1/simulations/{simulation_id}
```

### 8.2 Purpose

Memeriksa status execution ketika callback belum diterima atau perlu dilakukan reconciliation.

### 8.3 Response

Contoh:

```json
{
  "contract_version": "1.0.0",
  "simulation_id": "uuid",
  "battle_id": "uuid",
  "status": "RUNNING",
  "progress": {
    "completed_ticks": 250,
    "total_ticks": 1000,
    "processed_events": 500
  },
  "updated_at": "2026-11-25T12:01:00Z"
}
```

Progress bersifat observability metadata dan tidak boleh memengaruhi hasil deterministik simulasi.

### 8.4 Status Values

Status yang diusulkan:

| Status    | Meaning                               |
| --------- | ------------------------------------- |
| ACCEPTED  | Job berhasil diterima                 |
| QUEUED    | Menunggu execution                    |
| RUNNING   | Sedang disimulasikan                  |
| SUCCEEDED | Simulation berhasil                   |
| FAILED    | Simulation gagal                      |
| REJECTED  | Input tidak valid atau tidak didukung |

Status final:

- SUCCEEDED.
- FAILED.
- REJECTED.

Terminal status tidak boleh berubah kembali menjadi RUNNING atau QUEUED.

## 9. Simulation Result Callback

### 9.1 Endpoint

```http
POST /api/v1/internal/simulation-results
```

Endpoint ini dimiliki Laravel dan hanya dapat dipanggil oleh service yang terautentikasi.

### 9.2 Successful Result Payload

Contoh:

```json
{
  "contract_version": "1.0.0",
  "request_id": "uuid",
  "simulation_id": "uuid",
  "battle_id": "uuid",
  "correlation_id": "uuid",
  "status": "SUCCEEDED",
  "integrity": {
    "snapshot_id": "uuid",
    "snapshot_hash": "sha256-hash",
    "ruleset_version": "1.0.0",
    "engine_version": "1.0.0",
    "event_schema_version": "1.0.0"
  },
  "result": {
    "outcome": "DEFENSE_SUCCESS",
    "objectives": [],
    "metrics": {
      "simulation_ticks": 1000,
      "processed_events": 2500,
      "components_affected": 2
    },
    "component_results": [],
    "summary": {}
  },
  "events": {
    "event_count": 3,
    "events": [
      {
        "sequence_number": 1,
        "simulation_tick": 10,
        "event_type": "ATTACK_STARTED",
        "payload": {}
      },
      {
        "sequence_number": 2,
        "simulation_tick": 20,
        "event_type": "ATTACK_DETECTED",
        "payload": {}
      },
      {
        "sequence_number": 3,
        "simulation_tick": 30,
        "event_type": "ATTACK_BLOCKED",
        "payload": {}
      }
    ]
  },
  "result_hash": "sha256-hash",
  "completed_at": "2026-11-25T12:05:00Z"
}
```

Contoh tersebut menunjukkan bentuk kontrak awal. Daftar outcome, event type, metrics, dan component result harus ditentukan melalui domain contract berikutnya.

### 9.3 Failed Result Payload

Contoh:

```json
{
  "contract_version": "1.0.0",
  "request_id": "uuid",
  "simulation_id": "uuid",
  "battle_id": "uuid",
  "correlation_id": "uuid",
  "status": "FAILED",
  "error": {
    "code": "SIMULATION_EXECUTION_FAILED",
    "category": "EXECUTION",
    "message": "Simulation execution failed.",
    "retryable": false
  },
  "completed_at": "2026-11-25T12:05:00Z"
}
```

Error response tidak boleh mengekspos stack trace, internal file path, secret, atau informasi sensitif.

### 9.4 Callback Acceptance

Laravel harus:

1. Memverifikasi authentication.
2. Memvalidasi contract version.
3. Memverifikasi simulation identity.
4. Memeriksa snapshot identity dan hash.
5. Memeriksa ruleset dan engine compatibility.
6. Memeriksa result structure.
7. Memeriksa event sequence.
8. Memeriksa duplicate delivery.
9. Menyimpan hasil secara transaction-safe.
10. Menjadwalkan business consequence processing.

Jika callback valid dan berhasil diterima:

```text
HTTP 200 OK
```

Response:

```json
{
  "request_id": "uuid",
  "simulation_id": "uuid",
  "status": "RESULT_ACCEPTED"
}
```

Jika callback diterima tetapi hasil masih menunggu proses asynchronous internal Laravel, status response hanya berarti callback sudah diterima dengan aman, bukan semua business effects telah selesai.

## 10. Result Integrity and Validation

Laravel tidak boleh mempercayai result hanya karena berasal dari Engine.

Validasi wajib:

### 10.1 Identity Validation

- Simulation ID sesuai dengan job yang dibuat.
- Battle ID sesuai dengan battle yang aktif.
- Correlation ID sesuai dengan workflow.
- Snapshot ID sesuai dengan snapshot yang digunakan.

### 10.2 Snapshot Validation

- Snapshot hash harus sama dengan hash yang tercatat.
- Snapshot tidak boleh berubah setelah dipublikasikan.
- Snapshot schema harus kompatibel.

### 10.3 Version Validation

- Contract version didukung.
- Ruleset version sesuai dengan request.
- Engine version sesuai dengan compatibility policy.
- Event schema version didukung.

### 10.4 Event Validation

- Sequence number harus unik.
- Sequence harus terurut.
- Jumlah event harus sesuai metadata.
- Event payload harus sesuai schema.
- Event tidak boleh berasal dari battle lain.

### 10.5 Result Hash

Engine harus menghasilkan result hash menggunakan canonical serialization dan algoritma yang telah disepakati.

Hash digunakan untuk mendeteksi perubahan payload dan membantu audit integritas.

Hash bukan pengganti authentication atau signature.

Laravel harus memverifikasi hash sesuai spesifikasi canonicalization yang sama.

## 11. Idempotency and Duplicate Handling

Integrasi harus aman terhadap duplicate requests dan callbacks.

### 11.1 Submission Idempotency

- Satu logical simulation job memiliki satu simulation ID.
- Retry menggunakan simulation ID dan idempotency key yang sama.
- Engine tidak boleh membuat execution baru jika job identik telah diterima.
- Reuse idempotency key dengan payload berbeda harus ditolak.

### 11.2 Callback Idempotency

- Callback dengan simulation ID dan result yang sama dapat diterima ulang secara aman.
- Callback duplicate tidak boleh membuat Battle Report baru.
- Callback duplicate tidak boleh memberikan reward tambahan.
- Callback dengan simulation ID sama tetapi result berbeda harus ditolak dan dicatat sebagai integrity incident.

### 11.3 Business Effects Idempotency

Laravel harus menggunakan idempotency key untuk:

- Battle reward.
- Credit transaction.
- Progression update.
- Battle completion.
- Achievement processing.

Simulation Engine tidak pernah memberikan reward secara langsung.

## 12. Retry and Timeout Strategy

### 12.1 Submission Retry

Submission retry dilakukan apabila:

- Connection timeout.
- Temporary network failure.
- HTTP 5xx.
- Engine unavailable.

Retry menggunakan exponential backoff dengan jitter.

Contoh kebijakan awal:

| Attempt | Delay      |
| ------- | ---------- |
| 1       | 2 seconds  |
| 2       | 5 seconds  |
| 3       | 15 seconds |
| 4       | 30 seconds |
| 5       | 60 seconds |

Nilai ini merupakan konfigurasi awal dan harus disesuaikan dengan kapasitas Engine.

### 12.2 Non-Retryable Errors

Jangan melakukan retry otomatis untuk:

- Unsupported contract version.
- Invalid snapshot.
- Invalid configuration.
- Unsupported ruleset.
- Invalid component definition.
- Authentication failure.

Error tersebut membutuhkan perbaikan input, konfigurasi, atau deployment.

### 12.3 Timeout Categories

Bedakan:

- HTTP connection timeout.
- HTTP response timeout.
- Simulation execution timeout.
- Callback delivery timeout.
- Laravel result processing timeout.

Timeout pada HTTP submission tidak otomatis berarti simulation job gagal.

Laravel harus memeriksa status menggunakan simulation ID sebelum membuat keputusan retry yang dapat menyebabkan duplicate execution.

### 12.4 Execution Timeout

Simulation Engine harus menerapkan execution budget yang telah ditentukan dalam request dan dibatasi oleh konfigurasi server.

Engine harus menghentikan execution jika batas terlampaui.

## 13. Failure Recovery

### 13.1 Engine Unavailable

Jika Engine tidak tersedia:

- Laravel menyimpan job dalam queue.
- Laravel melakukan retry sesuai policy.
- Battle tetap berada dalam state menunggu atau submission failure sementara.
- Tidak ada reward yang diberikan.

### 13.2 Callback Delivery Failure

Jika callback gagal:

- Engine melakukan retry callback.
- Retry menggunakan simulation ID dan result identity yang sama.
- Laravel menangani duplicate callback secara idempotent.
- Status simulation tetap dapat diperiksa melalui status API.

### 13.3 Laravel Unavailable

Jika Laravel tidak tersedia:

- Engine mempertahankan completed result sesuai retention policy internal.
- Engine mencoba callback kembali.
- Laravel dapat melakukan status reconciliation setelah pulih.

### 13.4 Result Mismatch

Jika result identity atau integrity tidak cocok:

- Laravel menolak hasil.
- Battle tidak boleh menjadi COMPLETED.
- Integrity incident dicatat.
- Manual investigation dapat dilakukan.

### 13.5 Terminal Failure

Setelah retry habis:

- Job ditandai sebagai FAILED atau masuk ke dead-letter handling.
- Error dicatat.
- Battle tidak boleh dianggap berhasil.
- Retry manual harus melalui workflow yang dapat diaudit.

## 14. Data Ownership and Persistence

### 14.1 Laravel Ownership

Laravel menyimpan:

- Battle identity.
- Simulation job reference.
- Defense Snapshot reference.
- Battle lifecycle.
- Simulation status mirror.
- Validated Battle Report.
- Replay metadata.
- Business consequences.

### 14.2 Engine Ownership

Engine memiliki:

- Runtime state selama execution.
- Internal execution status.
- Simulation event generation.
- Simulation result generation.

Engine tidak menjadi pemilik:

- Player account.
- Credit ledger.
- Inventory.
- Infrastructure ownership.
- Progression.

### 14.3 Result Persistence

Laravel bertanggung jawab menyimpan result yang telah divalidasi.

Battle Report, event records, dan business status harus dipersist melalui Battles Module.

Jika event volume terlalu besar, Laravel dapat menyimpan event archive di object storage dan menyimpan metadata, reference, serta integrity hash di PostgreSQL.

## 15. Battle Lifecycle Integration

Alur lifecycle:

```text
BATTLE_CREATED
      |
      v
VALIDATING
      |
      v
QUEUED
      |
      v
SUBMITTING
      |
      v
ENGINE_ACCEPTED
      |
      v
RUNNING
      |
      v
RESULT_RECEIVED
      |
      v
PROCESSING_RESULT
      |
      v
COMPLETED
```

Failure states:

```text
VALIDATING ------> REJECTED
SUBMITTING ------> SUBMISSION_FAILED
RUNNING ---------> FAILED
RESULT_RECEIVED -> RESULT_REJECTED
PROCESSING_RESULT -> FAILED
```

State internal Laravel dapat berbeda dari status Engine.

Contohnya:

- Engine `ACCEPTED` dipetakan menjadi Laravel `ENGINE_ACCEPTED`.
- Engine `SUCCEEDED` dipetakan menjadi Laravel `RESULT_RECEIVED`.
- Laravel baru menetapkan `COMPLETED` setelah result validation, persistence, dan business workflow yang diwajibkan selesai.

Status final harus ditentukan berdasarkan state machine Battles Module.

## 16. Callback Delivery Policy

### 16.1 Callback Destination

Callback endpoint ditentukan melalui konfigurasi deployment.

Engine tidak boleh menerima callback destination arbitrary dari player-controlled payload.

### 16.2 Callback Retry

Callback retry harus:

- Menggunakan result identity yang sama.
- Menggunakan request ID baru untuk setiap delivery attempt.
- Mempertahankan simulation ID dan correlation ID.
- Mempertahankan result payload yang sama.
- Menggunakan exponential backoff.
- Memiliki maximum retry count.

### 16.3 Callback Acknowledgement

Engine menganggap callback berhasil terkirim apabila menerima acknowledgement yang valid dari Laravel.

HTTP response 5xx atau network failure harus dianggap sebagai delivery failure.

Duplicate callback aman karena Laravel menerapkan idempotency.

## 17. Observability

Kedua service harus menggunakan correlation metadata.

Field penting:

- request_id.
- simulation_id.
- battle_id.
- snapshot_id.
- correlation_id.
- engine_version.
- ruleset_version.
- contract_version.

Monitoring minimum:

- Submission success rate.
- Simulation execution duration.
- Queue waiting time.
- Simulation failure rate.
- Callback delivery success rate.
- Callback retry count.
- Result validation failure rate.
- Duplicate submission count.
- Duplicate callback count.

Log harus menggunakan structured format dan tidak boleh memuat secret atau data sensitif.

## 18. Contract Testing

### 18.1 Schema Tests

Menguji:

- Required fields.
- Field types.
- Enum values.
- Version compatibility.
- Payload size limits.
- Invalid payload rejection.

### 18.2 Consumer Contract Tests

Laravel harus menguji bahwa request yang dikirim sesuai dengan kontrak Engine.

### 18.3 Provider Contract Tests

Engine harus menguji bahwa response dan callback yang dihasilkan sesuai dengan kontrak yang disepakati.

### 18.4 Integration Tests

Menguji:

- Successful submission.
- Duplicate submission.
- Invalid snapshot.
- Unsupported version.
- Successful result callback.
- Duplicate callback.
- Failed simulation.
- Callback delivery failure.
- Engine unavailable.
- Result identity mismatch.

### 18.5 End-to-End Tests

Menguji keseluruhan workflow:

```text
Create Battle
    |
    v
Publish Defense Snapshot
    |
    v
Submit Simulation Job
    |
    v
Engine Executes
    |
    v
Receive Callback
    |
    v
Validate Result
    |
    v
Persist Battle Report
    |
    v
Apply Business Effects
    |
    v
Retrieve Battle Report
```

## 19. Recommended Contract Structure

Kontrak integrasi harus dikelola sebagai artifact yang dapat digunakan oleh kedua sistem.

Target struktur:

```text
contracts/
└── simulation/
    └── v1/
        ├── common.schema.json
        ├── submit-request.schema.json
        ├── submit-response.schema.json
        ├── status-response.schema.json
        ├── result-callback.schema.json
        ├── error-response.schema.json
        ├── event.schema.json
        └── README.md
```

Schema menggunakan format yang dapat divalidasi secara otomatis.

JSON Schema menjadi pilihan awal untuk kontrak lintas bahasa.

Laravel dan TypeScript Engine dapat menghasilkan atau menggunakan type definitions berdasarkan schema yang sama.

Kontrak tidak boleh diduplikasi secara manual tanpa automated validation.

## 20. Architectural Constraints

Aturan wajib:

1. Laravel dan Simulation Engine berkomunikasi melalui explicit versioned contract.
2. MVP menggunakan asynchronous HTTP integration.
3. Simulation Core tidak mengetahui detail transport.
4. Engine tidak boleh mengubah business state Laravel.
5. Submission menggunakan stable simulation identity.
6. Request retry harus idempotent.
7. Callback harus authenticated.
8. Duplicate callback harus aman.
9. Result harus divalidasi sebelum dipersist.
10. Snapshot hash harus diverifikasi.
11. Ruleset dan Engine version harus tercatat.
12. Result hash menggunakan canonical serialization.
13. Callback destination harus dikontrol oleh deployment configuration.
14. Status lookup tersedia sebagai fallback.
15. Battle tidak boleh COMPLETED sebelum result berhasil divalidasi dan dipersist.
16. Retry dan failure harus dapat diaudit.

## 21. Architecture Decision Record

### ADR-007: Asynchronous HTTP Integration Contract

**Status:** Proposed

**Context**

Laravel dan Simulation Engine merupakan dua sistem independen yang harus bertukar simulation request dan result secara reliable.

Simulasi dapat memerlukan waktu pemrosesan yang tidak cocok dijalankan sebagai bagian dari HTTP request normal frontend.

**Decision**

Menggunakan asynchronous HTTP-based integration.

Laravel mengirim simulation job melalui internal API. Engine menjalankan job melalui execution worker dan mengirim hasil melalui authenticated callback.

Laravel menyediakan status lookup sebagai fallback dan menerapkan idempotency pada submission serta result processing.

**Consequences**

Positive:

- Backend tidak perlu menunggu simulation execution selesai.
- Engine dapat diskalakan secara independen.
- Transport contract eksplisit.
- Retry dan recovery lebih mudah ditelusuri.
- Result processing dapat dilakukan secara asynchronous.

Trade-offs:

- Membutuhkan callback delivery mechanism.
- Membutuhkan idempotency dan status reconciliation.
- Ada tambahan operational complexity.
- Kedua sistem harus menjaga compatibility contract.

**Alternatives Considered**

- Synchronous HTTP execution.
- Shared database integration.
- Direct shared queue integration.
- Asynchronous HTTP submission with callback.

**Selected:** Asynchronous HTTP submission with authenticated callback and status lookup.

## 22. Acceptance Criteria

Integration Contract dianggap selesai apabila:

- [ ] Integration style ditetapkan.
- [ ] Transport strategy ditetapkan.
- [ ] API versioning ditetapkan.
- [ ] Simulation job identity ditetapkan.
- [ ] Service authentication didefinisikan.
- [ ] Submit Simulation API didefinisikan.
- [ ] Simulation Status API didefinisikan.
- [ ] Result Callback API didefinisikan.
- [ ] Request dan response payload dirancang.
- [ ] Result integrity validation didefinisikan.
- [ ] Idempotency strategy ditetapkan.
- [ ] Retry dan timeout strategy ditetapkan.
- [ ] Failure recovery didefinisikan.
- [ ] Data ownership ditetapkan.
- [ ] Battle lifecycle integration didefinisikan.
- [ ] Callback delivery policy ditetapkan.
- [ ] Observability requirements ditetapkan.
- [ ] Contract testing strategy ditetapkan.
- [ ] Shared contract structure ditetapkan.
- [ ] ADR-007 terdokumentasi.

## 23. Related Documents

- `docs/product/product-foundation.md`
- `docs/architecture/system-overview.md`
- `docs/architecture/service-boundaries.md`
- `docs/architecture/backend-architecture.md`
- `docs/architecture/database-architecture.md`
- `docs/architecture/simulation-engine-architecture.md`
- `docs/architecture/integration-contract.md`

---

**End of Document**
