# CYBERFRONT

# Architecture Review Report

**Document ID:** CF-ARCH-009  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.9 - Architecture Review  
**Date:** November 25, 2026

---

## 1. Executive Summary

Review ini mengevaluasi konsistensi delapan dokumen arsitektur CYBERFRONT:

1. CF-ARCH-001 - System Architecture Overview
2. CF-ARCH-002 - Service Boundaries & Module Responsibilities
3. CF-ARCH-003 - Frontend Architecture
4. CF-ARCH-004 - Backend Architecture
5. CF-ARCH-005 - Database Architecture
6. CF-ARCH-006 - Simulation Engine Architecture
7. CF-ARCH-007 - Simulation Engine Integration Contract
8. CF-ARCH-008 - Security Architecture

Secara umum, arsitektur telah memiliki fondasi yang baik. Pilihan Modular Monolith untuk Laravel dan Independent Deterministic Simulation Engine memberikan pemisahan tanggung jawab yang jelas tanpa memperkenalkan kompleksitas microservices terlalu dini.

Namun, beberapa keputusan masih belum konsisten atau belum cukup spesifik untuk menjadi pedoman implementasi.

**Review recommendation: Conditionally Ready for Architecture Approval.**

Arsitektur dapat dilanjutkan setelah temuan prioritas tinggi diselesaikan dan keputusan yang masih terbuka dicatat secara eksplisit.

## 2. Review Scope

Review mencakup:

- Konsistensi antar dokumen.
- Module boundaries dan dependency direction.
- Data ownership dan database integrity.
- Laravel dan Simulation Engine integration.
- Deterministic simulation requirements.
- Security boundaries dan abuse prevention.
- Battle lifecycle.
- Versioning dan replay integrity.
- Technology dan deployment assumptions.
- Architecture Decision Records.

Review ini didasarkan pada delapan dokumen yang diberikan untuk Task 02.1 sampai 02.8. Review tidak mencakup inspeksi langsung terhadap repository, source code, migration, deployment configuration, atau implementasi aktual.

## 3. Findings Summary

| ID     | Finding                                                                               | Priority | Affected Documents |
| ------ | ------------------------------------------------------------------------------------- | -------- | ------------------ |
| AR-001 | Circular dependency pada PvP dan Battles                                              | High     | 02.2, 02.4         |
| AR-002 | Battle lifecycle belum memiliki satu state machine kanonis                            | High     | 02.4, 02.7         |
| AR-003 | Diagram Redis dan Engine tidak konsisten dengan HTTP integration                      | High     | 02.1, 02.7         |
| AR-004 | Cross-module transaction ownership belum cukup spesifik                               | High     | 02.2, 02.4, 02.5   |
| AR-005 | Infrastructure connection integrity belum dijamin oleh relasi konseptual              | High     | 02.5               |
| AR-006 | Credit transaction sign convention belum final                                        | Medium   | 02.5               |
| AR-007 | PvP match dan battle cardinality belum tegas                                          | Medium   | 02.5               |
| AR-008 | Event delivery dan replay storage belum memiliki payload policy final                 | High     | 02.5, 02.6, 02.7   |
| AR-009 | HMAC dan callback reliability masih membutuhkan protokol operasional rinci            | High     | 02.7, 02.8         |
| AR-010 | Determinism requirements belum seluruhnya menjadi spesifikasi implementasi            | High     | 02.6               |
| AR-011 | Engine isolation membutuhkan deployment enforcement yang eksplisit                    | High     | 02.6, 02.7, 02.8   |
| AR-012 | Snapshot dan historical version retention belum memiliki compatibility policy lengkap | Medium   | 02.5, 02.6, 02.7   |
| AR-013 | WebSocket scope pada MVP belum konsisten                                              | Medium   | 02.1, 02.3, 02.4   |
| AR-014 | Related-document metadata dan penomoran belum konsisten                               | Low      | 02.1, 02.3, 02.4   |
| AR-015 | Simulation job persistence dan reconciliation model belum cukup eksplisit             | Medium   | 02.4, 02.5, 02.7   |

---

## 4. Detailed Findings

### AR-001 - Circular Dependency pada PvP dan Battles

**Priority:** High

**Affected documents:** CF-ARCH-002, CF-ARCH-004

CF-ARCH-002 mencantumkan:

- PvP bergantung pada Players, Infrastructure, dan Battles.
- Battles bergantung pada Players, Infrastructure, dan PvP.

Dependency matrix tersebut membentuk circular dependency.

CF-ARCH-004 sudah memberikan arah yang lebih tepat:

- PvP menginisiasi workflow melalui Battles Public Interface.
- Battles tidak perlu bergantung kembali pada PvP.
- Jika hasil battle memengaruhi status match, gunakan workflow atau event yang ditangani pemilik data terkait.

**Recommended decision:**

Gunakan dependency direction berikut:

```text
PvP Module
    |
    v
Battles Public Interface
    |
    v
Battles Application
    |
    v
Battle Lifecycle
```

Setelah battle selesai:

```text
Battles
    |
    v
BattleCompleted Event
    |
    v
PvP Event Handler
    |
    v
Update Match State
```

PvP tetap menjadi pemilik match state. Battles menjadi pemilik battle lifecycle.

Jangan menambahkan dependency langsung dari Battles ke PvP hanya untuk memperbarui status match.

**Required action:**

- Revisi dependency matrix CF-ARCH-002.
- Hapus PvP dari allowed dependencies Battles.
- Dokumentasikan event atau application workflow untuk menyelesaikan match.
- Tambahkan architecture test untuk mencegah circular dependency.

### AR-002 - Battle Lifecycle Belum Memiliki Satu State Machine Kanonis

**Priority:** High

**Affected documents:** CF-ARCH-004, CF-ARCH-007

CF-ARCH-004 mengusulkan:

```text
CREATED
  |
VALIDATING
  |
QUEUED
  |
RUNNING
  |
PROCESSING_RESULT
  |
COMPLETED
```

Failure states:

- REJECTED
- FAILED

CF-ARCH-007 mendefinisikan lifecycle yang lebih rinci:

```text
BATTLE_CREATED
  |
VALIDATING
  |
QUEUED
  |
SUBMITTING
  |
ENGINE_ACCEPTED
  |
RUNNING
  |
RESULT_RECEIVED
  |
PROCESSING_RESULT
  |
COMPLETED
```

Kedua dokumen tidak benar-benar bertentangan dalam konsep, tetapi memiliki tingkat detail dan nama state yang berbeda.

CF-ARCH-007 juga membedakan status internal Laravel dari status Engine. Perbedaan ini harus dipertahankan.

**Recommended decision:**

Tetapkan satu state machine Battles di CF-ARCH-004 atau dokumen khusus Battles.

State yang direkomendasikan:

```text
CREATED
  |
VALIDATING
  |
QUEUED
  |
SUBMITTING
  |
ENGINE_ACCEPTED
  |
RUNNING
  |
RESULT_RECEIVED
  |
PROCESSING_RESULT
  |
COMPLETED
```

Terminal states:

- REJECTED
- SUBMISSION_FAILED
- FAILED
- RESULT_REJECTED
- CANCELLED, apabila cancellation didukung

Status Engine harus dipetakan ke status internal Laravel. Jangan menggunakan status Engine sebagai pengganti battle lifecycle.

**Required action:**

- Tetapkan daftar state final.
- Tetapkan allowed transitions.
- Tetapkan retryable dan terminal states.
- Tetapkan aturan kapan battle dianggap selesai.
- Pastikan reward hanya diproses setelah hasil valid dan workflow bisnis yang diwajibkan berhasil.

### AR-003 - Diagram Redis dan Engine Tidak Konsisten dengan Integration Contract

**Priority:** High

**Affected documents:** CF-ARCH-001, CF-ARCH-007

Diagram CF-ARCH-001 menggambarkan:

```text
PvP
 |
Redis
 |
Simulation Worker
```

Diagram tersebut dapat ditafsirkan sebagai shared queue atau direct Redis integration antara Laravel dan Simulation Engine.

Namun, CF-ARCH-007 secara eksplisit menetapkan asynchronous HTTP-based integration:

- Laravel mengirim job melalui internal HTTP API.
- Engine menerima job dan memasukkannya ke execution workflow.
- Engine menjalankan simulation worker.
- Engine mengirim result melalui authenticated callback.
- Laravel menyediakan status lookup sebagai fallback.

**Recommended decision:**

MVP menggunakan asynchronous HTTP integration sesuai ADR-007.

Redis dapat tetap digunakan oleh Laravel untuk Laravel Queue dan oleh Engine untuk internal execution coordination jika diperlukan. Keduanya tidak boleh dianggap sebagai shared authoritative queue tanpa keputusan arsitektur baru.

Diagram yang disarankan:

```text
Laravel Application
       |
       v
Laravel Queue
       |
       v
Simulation Adapter
       |
       | Internal HTTP
       v
Simulation Engine API
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
Result Callback
       |
       v
Laravel Result Handler
```

**Required action:**

- Revisi diagram CF-ARCH-001.
- Tegaskan bahwa Redis bukan transport integrasi langsung Laravel-Engine pada MVP.
- Tegaskan bahwa HTTP submission, callback, dan status lookup merupakan mekanisme integrasi yang dipilih.

### AR-004 - Cross-Module Transaction Ownership Belum Cukup Spesifik

**Priority:** High

**Affected documents:** CF-ARCH-002, CF-ARCH-004, CF-ARCH-005

Dokumen menyatakan:

- Setiap modul hanya boleh menulis tabel miliknya.
- Cross-module database writes dilarang.
- Setiap modul mengelola local transaction.
- Infrastructure dapat meminta Economy memproses pembelian.
- Backend juga menyebut pembelian komponen dan pengurangan Credits sebagai operasi yang membutuhkan konsistensi atomik.

Karena seluruh modul menggunakan satu PostgreSQL database pada MVP, perlu diperjelas siapa yang mengorkestrasi transaksi lintas modul dan bagaimana partial failure ditangani.

**Recommended decision:**

Pertahankan single ownership.

- Economy menjadi satu-satunya pemilik ledger dan balance.
- Infrastructure menjadi pemilik component instance dan infrastructure state.
- Modul lain tidak boleh menulis tabel Economy.
- Orchestrator tidak boleh melewati public interface untuk melakukan direct database writes.

Untuk workflow pembelian, tentukan secara eksplisit apakah menggunakan coordinated application workflow, transactional outbox, atau mekanisme kompensasi.

Satu transaksi database lintas tabel tidak otomatis berarti ownership lintas modul boleh diabaikan.

**Required action:**

- Definisikan transaction boundary untuk purchase workflow.
- Definisikan urutan debit, persistence, dan compensation.
- Tentukan perilaku jika salah satu langkah gagal.
- Gunakan idempotency key pada seluruh langkah yang dapat diulang.

### AR-005 - Infrastructure Connection Integrity

**Priority:** High

**Affected document:** CF-ARCH-005

Database Architecture menetapkan bahwa dua komponen yang dihubungkan harus berasal dari infrastructure yang sama.

Namun, jika `infrastructure_connections` hanya memiliki:

- `infrastructure_id`
- `source_component_id`
- `target_component_id`

dan foreign key ke component ID hanya memvalidasi keberadaan komponen, database belum tentu menjamin bahwa kedua component tersebut benar-benar milik `infrastructure_id` yang sama.

**Recommended decision:**

Gunakan composite foreign key atau mekanisme database/domain constraint yang menjamin:

```text
source_component.infrastructure_id
    =
connection.infrastructure_id

target_component.infrastructure_id
    =
connection.infrastructure_id
```

Validasi application layer tetap diperlukan, tetapi database juga harus menjaga referential integrity.

**Required action:**

- Revisi logical schema.
- Tentukan composite unique key yang diperlukan.
- Definisikan composite foreign keys pada migration design.
- Tambahkan tests untuk menolak koneksi lintas-infrastruktur.

### AR-006 - Credit Transaction Sign Convention Belum Final

**Priority:** Medium

**Affected document:** CF-ARCH-005

Database Architecture menetapkan:

```text
credit_transactions.amount > 0
```

Namun, dokumen juga menyebut alternatif representasi signed amount.

**Recommended decision:**

Gunakan amount positif dan representasikan arah melalui `transaction_type`.

Contoh:

```text
transaction_type = DEBIT
amount = 100

transaction_type = CREDIT
amount = 100
```

Dengan demikian, arti angka tidak bergantung pada tanda negatif atau positif.

Tetapkan pula currency unit dan integer precision yang digunakan.

**Required action:**

- Finalisasi convention.
- Tambahkan enum atau constrained transaction type.
- Pastikan balance calculation konsisten.
- Tambahkan tests untuk debit, credit, refund, dan compensating transaction.

### AR-007 - PvP Match dan Battle Cardinality

**Priority:** Medium

**Affected document:** CF-ARCH-005

`pvp_matches` memiliki `battle_id`, sedangkan ERD menggambarkan hubungan satu match terhadap nol atau satu battle.

Relasi ini belum menjelaskan apakah semua battle harus berasal dari PvP.

Arsitektur lainnya menggambarkan Battles sebagai domain yang lebih luas daripada matchmaking.

**Recommended decision:**

Untuk fleksibilitas domain, battle tidak harus selalu berasal dari PvP.

Pertahankan:

- PvP Match sebagai pemilik matchmaking dan match state.
- Battle sebagai pemilik battle lifecycle.
- Relasi match terhadap battle bersifat opsional dari perspektif battle.

Pilih satu arah referensi sebagai sumber kebenaran. Rekomendasi: `battles.pvp_match_id` nullable apabila sistem membutuhkan battle non-PvP di masa depan.

Jika tetap menggunakan `pvp_matches.battle_id`, dokumentasikan dengan tegas aturan uniqueness dan lifecycle relasinya.

**Required action:**

- Finalisasi cardinality.
- Tentukan apakah satu match dapat memiliki lebih dari satu battle.
- Tentukan apakah battle non-PvP termasuk scope MVP atau future capability.
- Revisi ERD dan foreign key strategy.

### AR-008 - Event Delivery dan Replay Storage

**Priority:** High

**Affected documents:** CF-ARCH-005, CF-ARCH-006, CF-ARCH-007

CF-ARCH-007 memberikan contoh callback yang menyertakan seluruh event dalam satu payload.

CF-ARCH-005 dan CF-ARCH-007 juga mengakui bahwa event archive dapat dipindahkan ke object storage jika ukurannya besar.

Namun, belum ada keputusan final mengenai:

- Batas ukuran callback.
- Batas ukuran event.
- Chunking atau pagination.
- Penyimpanan artifact.
- Verifikasi kelengkapan event.
- Retention dan recovery artifact.

**Recommended decision:**

Pisahkan battle result summary dari event archive.

Result callback membawa:

- Battle identity.
- Simulation identity.
- Result summary.
- Integrity metadata.
- Event count.
- Event artifact reference atau inline events jika masih di bawah batas yang ditentukan.

Untuk replay besar, gunakan object storage atau mekanisme transfer artifact yang terkontrol.

Setiap artifact harus memiliki hash, ukuran, schema version, dan battle identity.

**Required action:**

- Tetapkan batas payload numerik.
- Tetapkan kapan event dikirim inline dan kapan menggunakan artifact.
- Tentukan format artifact.
- Tentukan upload/download authorization.
- Verifikasi event count, sequence range, dan hash.
- Definisikan retention policy.

### AR-009 - HMAC dan Callback Reliability

**Priority:** High

**Affected documents:** CF-ARCH-007, CF-ARCH-008

Dokumen telah menetapkan:

- TLS.
- HMAC-SHA256.
- Timestamp validation.
- Replay protection.
- Secret rotation.
- Callback authentication.
- Idempotency.

Ini merupakan fondasi yang tepat, tetapi protokol operasionalnya masih perlu dipatok.

**Recommended decision:**

Spesifikasi integrasi harus mendefinisikan:

1. Format canonical signature.
2. Exact bytes yang ditandatangani.
3. Timestamp format dan tolerance window.
4. Key identifier untuk secret rotation.
5. Nonce atau replay identifier.
6. Replay cache dan retention.
7. Signature verification order.
8. Callback acknowledgement semantics.
9. Duplicate callback behavior.
10. Result retention ketika Laravel tidak tersedia.

Jangan mengandalkan hash payload saja untuk autentikasi. Hash dan HMAC memiliki tujuan berbeda.

**Required action:**

Buat protocol specification yang dapat diuji lintas Laravel dan TypeScript.

### AR-010 - Determinism Belum Menjadi Spesifikasi Implementasi Lengkap

**Priority:** High

**Affected document:** CF-ARCH-006

Simulation Engine Architecture sudah menetapkan:

- Tidak menggunakan `Math.random()`.
- Tidak menggunakan `Date.now()` sebagai simulation input.
- Tidak menggunakan uncontrolled asynchronous execution.
- Menggunakan PRNG dengan seed eksplisit.
- Menggunakan deterministic discrete-event simulation.
- Menetapkan event ordering.
- Mempertimbangkan fixed-point atau integer arithmetic.
- Memiliki determinism dan golden tests.

Namun, beberapa aturan masih disebut sebagai keputusan yang akan ditentukan kemudian.

**Recommended decision:**

Sebelum implementasi Simulation Core, tetapkan:

- PRNG algorithm dan version.
- Seed encoding.
- Derivation strategy untuk RNG streams.
- Scheduler ordering key final.
- Stable entity ordering.
- Integer/fixed-point precision.
- Overflow behavior.
- Canonical serialization.
- Deterministic error handling.
- Engine/ruleset compatibility behavior.

Rekomendasi scheduler key:

```text
simulation_tick
event_priority
sequence_number
stable_entity_id
```

Urutan tersebut perlu diuji dengan fixture yang menghasilkan timestamp dan priority identik.

**Required action:**

Buat Simulation Determinism Specification sebagai turunan CF-ARCH-006 sebelum implementasi core.

### AR-011 - Engine Isolation Membutuhkan Deployment Enforcement

**Priority:** High

**Affected documents:** CF-ARCH-006, CF-ARCH-007, CF-ARCH-008

Dokumen keamanan sudah melarang:

- Arbitrary shell commands.
- Player-provided scripts.
- Arbitrary binaries.
- External network access.
- Production database access.
- Host filesystem access.
- Cross-job runtime state access.

Namun, larangan pada application logic saja tidak cukup untuk menjamin containment.

**Recommended decision:**

Enforce isolation pada execution infrastructure:

- Container atau sandbox dengan permission minimum.
- Read-only filesystem jika memungkinkan.
- Default-deny network egress.
- No cloud metadata access.
- CPU, memory, dan execution limits.
- Job-level isolation.
- Restricted service credentials.
- No privileged worker.
- Controlled temporary storage.
- Dependency dan image provenance checks.

**Required action:**

Tambahkan deployment security requirements dan containment tests sebelum Engine menerima player-controlled simulation input.

### AR-012 - Snapshot dan Historical Version Retention

**Priority:** Medium

**Affected documents:** CF-ARCH-005, CF-ARCH-006, CF-ARCH-007

Dokumen sudah membedakan:

- Snapshot schema version.
- Component catalog version.
- Component behavior version.
- Ruleset version.
- Engine version.
- Contract version.
- Event schema version.

Namun, belum ada kebijakan lengkap untuk mempertahankan compatibility artifact dan versi yang diperlukan oleh battle historis.

**Recommended decision:**

Setiap battle harus mereferensikan seluruh versi yang diperlukan untuk interpretasi hasil dan replay.

Historical replay harus menggunakan recorded events dan metadata yang sesuai, bukan otomatis menjalankan ulang battle menggunakan versi Engine terbaru.

**Required action:**

- Tentukan compatibility registry.
- Tentukan retention policy untuk component behavior dan ruleset.
- Tentukan kapan versi lama boleh dihentikan.
- Tentukan archival policy untuk snapshot dan replay artifact.
- Definisikan migration policy untuk event schema.

### AR-013 - WebSocket Scope Belum Konsisten

**Priority:** Medium

**Affected documents:** CF-ARCH-001, CF-ARCH-003, CF-ARCH-004

CF-ARCH-001 menyatakan WebSocket akan diperkenalkan ketika fitur real-time dibutuhkan.

CF-ARCH-004 mencantumkan WebSocket sebagai bagian teknologi real-time communication.

CF-ARCH-003 menyatakan runtime delivery mechanism akan ditentukan kemudian.

**Recommended decision:**

WebSocket bukan requirement untuk MVP asynchronous PvP.

MVP dapat menggunakan REST API untuk request dan status retrieval. Push notification atau WebSocket dapat ditambahkan jika kebutuhan monitoring atau user experience membenarkannya.

WebSocket tidak menggantikan asynchronous HTTP integration antara Laravel dan Engine.

**Required action:**

- Tandai WebSocket sebagai optional/future capability.
- Definisikan apakah runtime metrics perlu polling atau push pada MVP.
- Hindari menambah gateway sebelum ada kebutuhan nyata.

### AR-014 - Related Document Metadata Tidak Konsisten

**Priority:** Low

**Affected documents:** CF-ARCH-001 sampai CF-ARCH-008

Beberapa Related Documents mencantumkan nomor dokumen yang tidak sesuai dengan mapping aktual.

Contoh pada CF-ARCH-003:

- CF-ARCH-005 ditulis sebagai Simulation Engine.
- CF-ARCH-006 ditulis sebagai Database.
- CF-ARCH-007 ditulis sebagai Communication.
- CF-ARCH-008 ditulis sebagai Battle Processing.

Mapping aktual adalah:

- CF-ARCH-004: Backend Architecture.
- CF-ARCH-005: Database Architecture.
- CF-ARCH-006: Simulation Engine Architecture.
- CF-ARCH-007: Simulation Engine Integration Contract.
- CF-ARCH-008: Security Architecture.

**Recommended decision:**

Gunakan satu document registry sebagai sumber kebenaran untuk seluruh Document ID, title, path, version, dan status.

**Required action:**

Perbaiki Related Documents dan metadata yang tidak sesuai.

### AR-015 - Simulation Job Persistence dan Reconciliation

**Priority:** Medium

**Affected documents:** CF-ARCH-004, CF-ARCH-005, CF-ARCH-007

Backend menyebut simulation job references dan integration contract menetapkan request ID, simulation ID, correlation ID, idempotency key, status lookup, retry, dan callback.

Namun, logical database model belum menjelaskan secara eksplisit apakah identitas dan delivery attempts tersebut disimpan sebagai bagian dari `battles`, tabel job terpisah, atau integration records.

**Recommended decision:**

Tentukan persistence model untuk:

- Simulation ID.
- Request ID.
- Idempotency key.
- Payload fingerprint.
- Engine status mirror.
- Submission attempt.
- Callback attempt.
- Last reconciliation time.
- Terminal error metadata.

Model ini harus mendukung audit dan reconciliation tanpa menjadikan Redis sebagai sumber data authoritative.

**Required action:**

Tambahkan logical entity atau definisi field yang diperlukan ke Database Architecture.

---

## 5. Cross-Document Consistency Assessment

| Area                     | Assessment                                       | Recommendation                                |
| ------------------------ | ------------------------------------------------ | --------------------------------------------- |
| Overall architecture     | Consistent                                       | Retain Modular Monolith + Independent Engine  |
| Frontend rendering       | Consistent                                       | Retain Canvas 2D and Cytoscape separation     |
| Module boundaries        | Needs revision                                   | Fix PvP/Battles dependency                    |
| Database ownership       | Mostly consistent                                | Clarify cross-module transaction coordination |
| Persistent/runtime state | Consistent                                       | Preserve Laravel vs Engine ownership          |
| Integration transport    | Needs diagram alignment                          | Use asynchronous HTTP for MVP                 |
| Battle lifecycle         | Needs consolidation                              | One canonical state machine                   |
| Determinism              | Strong foundation, incomplete specification      | Add implementation-level determinism spec     |
| Security                 | Strong principles, deployment details incomplete | Enforce sandbox and egress restrictions       |
| Replay                   | Conceptually consistent                          | Define event artifact and retention policy    |
| Versioning               | Consistent concepts                              | Define compatibility and retention policy     |
| WebSocket                | Needs scope clarification                        | Future/optional for MVP                       |

## 6. Recommended Architecture Corrections

Sebelum approval:

1. Revisi CF-ARCH-002 dependency matrix.
2. Konsolidasikan battle lifecycle pada CF-ARCH-004 dan CF-ARCH-007.
3. Revisi diagram CF-ARCH-001 agar sesuai dengan ADR-007.
4. Definisikan transaction coordination lintas modul.
5. Perkuat referential integrity infrastructure connections.
6. Finalisasi credit transaction sign convention.
7. Finalisasi PvP Match dan Battle cardinality.
8. Definisikan event artifact dan payload limits.
9. Lengkapi HMAC dan callback protocol.
10. Buat deterministic execution specification.
11. Tambahkan deployment-level Engine isolation.
12. Definisikan historical version compatibility.
13. Klarifikasi WebSocket sebagai future/optional.
14. Perbaiki document metadata dan related references.
15. Definisikan persistence model untuk simulation job tracking.

## 7. Approval Recommendation

**Current recommendation: Conditionally Ready.**

Arsitektur dasar dapat dipertahankan. Tidak ada alasan kuat untuk mengganti Modular Monolith, PostgreSQL, TypeScript Simulation Engine, atau asynchronous HTTP integration.

Namun, temuan High Priority harus diselesaikan sebelum implementasi terkait dimulai.

Status Proposed pada dokumen sumber tidak otomatis berubah menjadi Approved karena review ini selesai. Approval akhir tetap membutuhkan keputusan eksplisit dari pemilik proyek.

## 8. Review Conclusion

CYBERFRONT sudah memiliki pemisahan tanggung jawab yang sehat:

- Laravel mengelola business state.
- Simulation Engine mengelola runtime simulation state.
- PostgreSQL menyimpan persistent authoritative data.
- Redis mendukung queue dan coordination.
- Frontend memvisualisasikan dan mengirim intent, bukan menentukan hasil teknis.
- Battles mengorkestrasi lifecycle, bukan mengimplementasikan simulation mechanics.

Prioritas berikutnya bukan menambah teknologi atau memecah sistem menjadi microservices. Prioritasnya adalah memperjelas kontrak, ownership, deterministic behavior, dan containment agar implementasi pertama tidak membawa ambiguitas desain ke dalam kode.

---

**End of Document**
