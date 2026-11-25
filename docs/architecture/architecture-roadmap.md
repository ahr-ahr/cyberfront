# CYBERFRONT

# Architecture Roadmap

**Document ID:** CF-ARCH-011  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.9 - Architecture Review  
**Date:** November 25, 2026

---

## 1. Purpose

Dokumen ini mendefinisikan urutan pekerjaan setelah Architecture Review untuk mempersiapkan CYBERFRONT menuju implementation phase.

Roadmap disusun berdasarkan dependency antar pekerjaan, risiko teknis, dan kebutuhan untuk mempertahankan konsistensi arsitektur.

Roadmap ini bukan jadwal kalender yang mengikat. Urutan dan completion criteria menjadi acuan utama.

## 2. Roadmap Overview

| Stage | Workstream                            | Priority | Exit Condition                              |
| ----- | ------------------------------------- | -------- | ------------------------------------------- |
| 1     | Architecture Corrections              | Critical | High findings resolved                      |
| 2     | Architecture Approval                 | Critical | ADR dan baseline disetujui                  |
| 3     | Technical Specifications              | High     | Contracts dan rules dapat diimplementasikan |
| 4     | Repository and Development Foundation | High     | Development environment siap                |
| 5     | Core Backend Foundation               | High     | Backend modules dan persistence siap        |
| 6     | Simulation Engine Foundation          | High     | Deterministic core dapat diuji              |
| 7     | Integration Foundation                | High     | Laravel dan Engine berkomunikasi            |
| 8     | MVP Feature Implementation            | High     | Core gameplay workflow berjalan             |
| 9     | Quality and Security Validation       | High     | MVP memenuhi acceptance criteria            |

---

## 3. Stage 1 - Architecture Corrections

**Priority:** Critical

Tujuan: menyelesaikan temuan yang berpotensi menghasilkan implementasi salah atau saling bertentangan.

### 3.1 CF-ARCH-002 - Service Boundaries

Tasks:

- [ ] Perbaiki dependency matrix PvP dan Battles.
- [ ] Hapus circular dependency.
- [ ] Definisikan BattleCompleted event handling.
- [ ] Tegaskan ownership match state dan battle state.
- [ ] Perjelas cross-module transaction coordination.
- [ ] Tambahkan dependency enforcement requirements.

### 3.2 CF-ARCH-001 - System Architecture Overview

Tasks:

- [ ] Perbarui diagram backend-to-engine.
- [ ] Hapus interpretasi shared Redis queue antara Laravel dan Engine.
- [ ] Tampilkan asynchronous HTTP submission.
- [ ] Tampilkan Engine execution queue.
- [ ] Tampilkan authenticated result callback.
- [ ] Perjelas peran Redis sebagai supporting infrastructure.

### 3.3 CF-ARCH-004 - Backend Architecture

Tasks:

- [ ] Konsolidasikan canonical battle lifecycle.
- [ ] Definisikan state transition rules.
- [ ] Definisikan retryable dan terminal failure states.
- [ ] Definisikan transaction coordination.
- [ ] Klarifikasi WebSocket sebagai optional/future capability.
- [ ] Definisikan simulation job tracking responsibility.

### 3.4 CF-ARCH-005 - Database Architecture

Tasks:

- [ ] Finalisasi credit transaction sign convention.
- [ ] Definisikan same-infrastructure connection integrity.
- [ ] Finalisasi PvP Match dan Battle cardinality.
- [ ] Definisikan simulation job persistence.
- [ ] Definisikan replay artifact metadata.
- [ ] Perjelas historical snapshot retention.
- [ ] Perjelas cross-module foreign key dan write ownership rules.

### 3.5 CF-ARCH-006 - Simulation Engine Architecture

Tasks:

- [ ] Finalisasi event ordering requirements.
- [ ] Definisikan PRNG implementation requirements.
- [ ] Definisikan RNG stream isolation.
- [ ] Tetapkan numeric precision policy.
- [ ] Definisikan canonical serialization.
- [ ] Definisikan Engine compatibility requirements.
- [ ] Perjelas runtime isolation boundary.

### 3.6 CF-ARCH-007 - Simulation Engine Integration Contract

Tasks:

- [ ] Finalisasi HMAC signing protocol.
- [ ] Definisikan callback acknowledgement.
- [ ] Definisikan callback retry dan reconciliation.
- [ ] Tetapkan payload size limits.
- [ ] Definisikan event artifact transfer.
- [ ] Definisikan simulation result retention.
- [ ] Definisikan status polling dan callback race behavior.

### 3.7 CF-ARCH-008 - Security Architecture

Tasks:

- [ ] Definisikan worker sandbox requirements.
- [ ] Definisikan default-deny network egress.
- [ ] Definisikan filesystem restrictions.
- [ ] Definisikan worker privileges.
- [ ] Definisikan resource enforcement.
- [ ] Definisikan containment testing requirements.
- [ ] Perjelas service credential rotation procedure.

### 3.8 Documentation Consistency

Tasks:

- [ ] Perbaiki Related Documents.
- [ ] Verifikasi Document ID dan title.
- [ ] Verifikasi version metadata.
- [ ] Verifikasi ADR references.
- [ ] Verifikasi architecture diagrams.
- [ ] Sinkronkan terminology antar dokumen.

**Exit criteria:**

Semua High Priority findings dalam CF-ARCH-009 telah diperbaiki atau memiliki keputusan eksplisit yang disetujui.

---

## 4. Stage 2 - Architecture Approval

**Priority:** Critical

Tujuan: menetapkan baseline arsitektur yang menjadi acuan implementasi.

Tasks:

- [ ] Review revisi delapan dokumen.
- [ ] Review Consolidated Architecture Decisions.
- [ ] Tentukan status ADR-002 sampai ADR-008.
- [ ] Putuskan additional ADR yang diperlukan.
- [ ] Setujui canonical battle lifecycle.
- [ ] Setujui database ownership dan transaction rules.
- [ ] Setujui Engine integration contract.
- [ ] Setujui Simulation Engine security boundary.
- [ ] Finalisasi architecture baseline.
- [ ] Catat approval date dan approver.

**Exit criteria:**

Architecture baseline disetujui dan seluruh keputusan penting memiliki status yang jelas.

Dokumen yang masih Proposed tidak boleh dianggap sebagai keputusan final hanya karena sudah masuk repository.

---

## 5. Stage 3 - Technical Specifications

**Priority:** High

Setelah architecture baseline disetujui, buat spesifikasi yang lebih dekat dengan implementasi.

### 5.1 API Specification

Dokumen yang direkomendasikan:

`docs/specifications/api-specification.md`

Isi:

- REST API conventions.
- Endpoint definitions.
- Request schemas.
- Response schemas.
- Authentication requirements.
- Authorization requirements.
- Error codes.
- Pagination.
- API versioning.
- Idempotency behavior.

OpenAPI dapat digunakan sebagai format machine-readable.

### 5.2 Database Specification

Dokumen yang direkomendasikan:

`docs/specifications/database-specification.md`

Isi:

- Final ERD.
- Table definitions.
- Column types.
- Primary keys.
- Foreign keys.
- Composite constraints.
- Unique constraints.
- Check constraints.
- Index definitions.
- Transaction boundaries.
- Migration ordering.

### 5.3 Simulation Determinism Specification

Dokumen yang direkomendasikan:

`docs/specifications/simulation-determinism.md`

Isi:

- PRNG algorithm.
- Seed representation.
- RNG stream derivation.
- Event scheduler ordering.
- Simulation clock.
- Numeric precision.
- Overflow behavior.
- Canonical serialization.
- Determinism guarantees.
- Golden test requirements.

### 5.4 Simulation Contract Specification

Direkomendasikan menggunakan shared contracts:

`contracts/simulation/v1/`

Isi:

- Submit request schema.
- Submit response schema.
- Status response schema.
- Result callback schema.
- Error schema.
- Event schema.
- Version compatibility rules.
- Signature specification.
- Payload size limits.
- Event artifact reference.

### 5.5 Component Catalog Specification

Dokumen yang direkomendasikan:

`docs/specifications/component-catalog-specification.md`

Isi:

- Component identity.
- Component version.
- Configuration schema.
- Resource requirements.
- Compatibility rules.
- Network interfaces.
- Behavior version reference.
- Component lifecycle.
- Validation rules.

### 5.6 Security Implementation Specification

Dokumen yang direkomendasikan:

`docs/specifications/security-implementation.md`

Isi:

- Authentication and authorization.
- Service authentication.
- HMAC implementation.
- Secret rotation.
- Worker isolation.
- Network policies.
- Filesystem restrictions.
- Resource limits.
- Audit requirements.
- Security tests.

**Exit criteria:**

Implementer dapat membangun modul dan Engine berdasarkan spesifikasi tanpa harus menebak aturan yang belum diputuskan.

---

## 6. Stage 4 - Repository and Development Foundation

**Priority:** High

Tujuan: menyiapkan struktur repository dan tooling sebelum fitur gameplay mulai dikembangkan.

### 6.1 Repository Structure

Target:

```text
cyberfront/
├── app/
├── resources/
├── routes/
├── database/
├── tests/
├── simulation/
│   ├── src/
│   ├── tests/
│   └── fixtures/
├── contracts/
│   └── simulation/
│       └── v1/
├── docs/
│   ├── product/
│   ├── architecture/
│   └── specifications/
├── infrastructure/
├── composer.json
├── package.json
└── README.md
```

Struktur aktual dapat disesuaikan dengan kebutuhan Laravel dan deployment.

### 6.2 Development Tooling

Tasks:

- [ ] PHP dan Composer setup.
- [ ] Laravel setup.
- [ ] Node.js dan pnpm setup.
- [ ] TypeScript configuration.
- [ ] Vite configuration.
- [ ] PostgreSQL development database.
- [ ] Redis setup.
- [ ] Simulation Engine workspace.
- [ ] PHPUnit/Pest setup.
- [ ] Vitest setup.
- [ ] Linting dan formatting.
- [ ] Environment configuration.
- [ ] Git workflow documentation.

### 6.3 CI Foundation

Tasks:

- [ ] PHP tests.
- [ ] TypeScript tests.
- [ ] Type checking.
- [ ] Linting.
- [ ] Contract schema validation.
- [ ] Architecture dependency checks.
- [ ] Build validation.

**Exit criteria:**

Repository dapat dijalankan secara lokal dan seluruh automated checks berjalan.

---

## 7. Stage 5 - Core Backend Foundation

**Priority:** High

Tujuan: membangun fondasi backend sebelum fitur battle dan simulation integration.

### Recommended implementation order

1. Identity & Access
2. Players
3. Component Catalog
4. Infrastructure
5. Economy
6. Progression
7. PvP
8. Battles

### 7.1 Identity & Access

- [ ] Registration.
- [ ] Login.
- [ ] Logout.
- [ ] Authentication.
- [ ] Authorization policies.
- [ ] Account ownership.

### 7.2 Players

- [ ] Player profile.
- [ ] Player settings.
- [ ] Player statistics baseline.

### 7.3 Component Catalog

- [ ] Component definitions.
- [ ] Component versions.
- [ ] Configuration schemas.
- [ ] Resource requirements.
- [ ] Compatibility metadata.

### 7.4 Infrastructure

- [ ] Infrastructure CRUD.
- [ ] Component instances.
- [ ] Component placement metadata.
- [ ] Logical network connections.
- [ ] Configuration validation.
- [ ] Infrastructure constraints.
- [ ] Defense Snapshot publication.

### 7.5 Economy

- [ ] Credit accounts.
- [ ] Append-only ledger.
- [ ] Debit and credit operations.
- [ ] Idempotency.
- [ ] Purchase workflow.
- [ ] Reconciliation.

### 7.6 Progression

- [ ] Experience.
- [ ] Levels.
- [ ] Technology unlocks.
- [ ] Progression requirements.
- [ ] Reward coordination.

### 7.7 PvP

- [ ] Target discovery.
- [ ] Eligibility validation.
- [ ] Matchmaking records.
- [ ] Match creation.
- [ ] Battle initiation through Battles interface.

### 7.8 Battles

- [ ] Battle creation.
- [ ] Canonical lifecycle.
- [ ] Snapshot reference.
- [ ] Simulation job tracking.
- [ ] Result processing.
- [ ] Battle Report.
- [ ] Replay metadata.

**Exit criteria:**

Core business modules memiliki domain ownership, persistence, API, dan tests yang sesuai.

---

## 8. Stage 6 - Simulation Engine Foundation

**Priority:** High

Tujuan: membangun deterministic simulation runtime secara independen dari Laravel.

### 8.1 Engine Core

- [ ] Simulation input model.
- [ ] Simulation result model.
- [ ] Runtime state manager.
- [ ] Simulation clock.
- [ ] Deterministic PRNG.
- [ ] Event scheduler.
- [ ] Event recorder.
- [ ] Ruleset registry.
- [ ] Snapshot validator.

### 8.2 MVP Simulation Domains

Urutan yang direkomendasikan:

1. Infrastructure Runtime.
2. Component Runtime.
3. Resource System.
4. Network System.
5. Defense System.
6. Attack System.
7. Battle Resolution.
8. Replay Event Generation.

### 8.3 Determinism Validation

- [ ] Identical input produces identical result.
- [ ] Identical seed produces identical random sequence.
- [ ] Event ordering remains stable.
- [ ] Numeric calculations remain consistent.
- [ ] Result serialization remains canonical.
- [ ] Golden tests pass.

### 8.4 Isolation

- [ ] Worker resource limits.
- [ ] Job-level runtime isolation.
- [ ] Network egress restriction.
- [ ] Filesystem restriction.
- [ ] No host command execution.
- [ ] No production database access.

**Exit criteria:**

Simulation Core dapat menjalankan satu battle scenario secara deterministik, terisolasi, dan dapat diuji tanpa Laravel.

---

## 9. Stage 7 - Integration Foundation

**Priority:** High

Tujuan: menghubungkan Laravel dengan Simulation Engine melalui kontrak yang sudah disetujui.

### 9.1 Contract Integration

- [ ] Shared JSON Schemas.
- [ ] Laravel contract validation.
- [ ] TypeScript contract validation.
- [ ] Contract version compatibility.
- [ ] Contract tests.

### 9.2 Submission Workflow

- [ ] Simulation job creation.
- [ ] Stable simulation identity.
- [ ] Idempotent submission.
- [ ] Internal HTTP API.
- [ ] Engine job acceptance.
- [ ] Status lookup.

### 9.3 Result Delivery

- [ ] HMAC authentication.
- [ ] Callback verification.
- [ ] Result identity validation.
- [ ] Snapshot hash validation.
- [ ] Event sequence validation.
- [ ] Result persistence.
- [ ] Callback acknowledgement.
- [ ] Retry dan reconciliation.

### 9.4 Business Processing

- [ ] Battle completion.
- [ ] Reward eligibility.
- [ ] Economy processing.
- [ ] Progression processing.
- [ ] BattleCompleted event.
- [ ] PvP match state update.

**Exit criteria:**

Laravel dapat mengirim simulation job, Engine menjalankan job, dan Laravel menerima serta memvalidasi hasil secara idempotent.

---

## 10. Stage 8 - MVP Feature Implementation

**Priority:** High

Tujuan: menyatukan backend, frontend, dan Engine menjadi gameplay loop yang dapat digunakan.

### 10.1 Infrastructure Workspace

- [ ] Isometric Canvas renderer.
- [ ] Camera pan and zoom.
- [ ] Component selection.
- [ ] Component placement.
- [ ] Component movement.
- [ ] Component Inspector.
- [ ] Draft state.
- [ ] Infrastructure persistence.

### 10.2 Network Topology

- [ ] Cytoscape graph.
- [ ] Node rendering.
- [ ] Edge rendering.
- [ ] Logical connection editing.
- [ ] Topology validation.
- [ ] Synchronization with infrastructure data.

### 10.3 Configuration

- [ ] Monaco Editor.
- [ ] Component schema integration.
- [ ] Configuration validation.
- [ ] Unsaved changes.
- [ ] Safe configuration persistence.

### 10.4 Monitoring

- [ ] CPU metrics.
- [ ] RAM metrics.
- [ ] Storage metrics.
- [ ] Bandwidth metrics.
- [ ] Latency metrics.
- [ ] Availability metrics.
- [ ] Component status visualization.

### 10.5 Battle Gameplay

- [ ] Publish Defense Snapshot.
- [ ] Discover opponent.
- [ ] Create battle.
- [ ] Select attack scenario.
- [ ] Submit simulation.
- [ ] Monitor battle status.
- [ ] Retrieve Battle Report.
- [ ] Replay event sequence.

### 10.6 MVP Completion

MVP dianggap memiliki core gameplay loop apabila pemain dapat:

1. Membuat infrastructure.
2. Memasang dan mengonfigurasi components.
3. Membuat logical network connections.
4. Mengelola resource allocation.
5. Mempublikasikan Defense Snapshot.
6. Menemukan opponent.
7. Mengirim asynchronous attack simulation.
8. Menerima Battle Report.
9. Memutar ulang battle events.

**Exit criteria:**

Seluruh gameplay loop berjalan end-to-end dan tidak bergantung pada frontend sebagai sumber kebenaran.

---

## 11. Stage 9 - Quality and Security Validation

**Priority:** High

Tujuan: memastikan MVP aman, konsisten, dan dapat dioperasikan.

### 11.1 Functional Testing

- [ ] Authentication.
- [ ] Infrastructure management.
- [ ] Component installation.
- [ ] Configuration update.
- [ ] Economy operations.
- [ ] Progression.
- [ ] PvP matchmaking.
- [ ] Battle submission.
- [ ] Result processing.
- [ ] Battle replay.

### 11.2 Integrity Testing

- [ ] Snapshot immutability.
- [ ] Snapshot hash verification.
- [ ] Credit ledger reconciliation.
- [ ] Idempotent reward processing.
- [ ] Duplicate callback handling.
- [ ] Event sequence integrity.
- [ ] Historical version compatibility.

### 11.3 Determinism Testing

- [ ] Repeatability.
- [ ] Golden fixtures.
- [ ] PRNG compatibility.
- [ ] Scheduler ordering.
- [ ] Numeric consistency.
- [ ] Serialization consistency.

### 11.4 Security Testing

- [ ] Authorization and IDOR.
- [ ] Rate limiting.
- [ ] Input validation.
- [ ] HMAC verification.
- [ ] Replay protection.
- [ ] Worker isolation.
- [ ] Network egress containment.
- [ ] Filesystem restrictions.
- [ ] Resource exhaustion limits.
- [ ] Secret handling.

### 11.5 Operational Testing

- [ ] Queue failure recovery.
- [ ] Engine unavailable.
- [ ] Callback delivery failure.
- [ ] Laravel unavailable.
- [ ] Result reconciliation.
- [ ] Database backup and restore.
- [ ] Logging and monitoring.

**Exit criteria:**

MVP memenuhi functional, integrity, security, determinism, dan operational acceptance criteria.

---

## 12. Recommended Work Order

Urutan implementasi keseluruhan:

```text
Architecture Corrections
        |
        v
Architecture Approval
        |
        v
Technical Specifications
        |
        v
Repository Foundation
        |
        v
Core Backend Foundation
        |
        v
Simulation Engine Foundation
        |
        v
Integration Foundation
        |
        v
MVP Feature Implementation
        |
        v
Quality and Security Validation
        |
        v
MVP Release Candidate
```

Frontend workspace dapat mulai dikembangkan setelah API contract dan infrastructure domain model cukup stabil. Jangan membuat frontend bergantung pada endpoint atau data model yang masih berubah.

## 13. Scope Control

Fitur berikut tetap berada di luar MVP kecuali ada keputusan produk baru:

- Real-time PvP.
- Guild atau clan.
- Marketplace.
- Advanced distributed systems.
- Multi-region cloud infrastructure.
- Kubernetes simulation.
- Complex economic trading.
- Advanced autonomous attack scenarios.

Fokus MVP adalah menghasilkan satu gameplay loop yang utuh dan dapat dipercaya.

## 14. Final Recommendation

Prioritas CYBERFRONT setelah Task 02.9 adalah:

1. Menyelesaikan architecture corrections.
2. Mendapatkan architecture approval.
3. Menulis technical specifications.
4. Menyiapkan repository dan development tooling.
5. Membangun backend dan deterministic simulation foundation.
6. Mengintegrasikan kedua sistem.
7. Mengembangkan gameplay MVP.
8. Melakukan quality dan security validation.

Jangan memulai dengan membangun semua fitur sekaligus. Infrastruktur teknis, deterministic core, dan business ownership harus menjadi fondasi sebelum fitur gameplay berkembang.

---

**End of Document**
