# CYBERFRONT

# Consolidated Architecture Decisions

**Document ID:** CF-ARCH-010  
**Version:** 0.1.0  
**Status:** Proposed  
**Phase:** Phase 0  
**Task:** 02.9 - Architecture Review  
**Date:** November 25, 2026

---

## 1. Purpose

Dokumen ini mengonsolidasikan Architecture Decision Records (ADR) CYBERFRONT agar keputusan arsitektur dapat dilacak dan digunakan sebagai referensi implementasi.

Dokumen ini mencakup:

- Existing ADR.
- Current decision status.
- Consolidated architecture direction.
- Required decision revisions.
- Additional decisions recommended by Architecture Review.

Dokumen ini tidak secara otomatis mengubah Proposed ADR menjadi Approved atau Accepted.

## 2. ADR Status Summary

| ADR     | Decision                                         | Source      | Current Status | Review                          |
| ------- | ------------------------------------------------ | ----------- | -------------- | ------------------------------- |
| ADR-001 | Modular Monolith + Independent Simulation Engine | CF-ARCH-001 | Accepted       | Retain                          |
| ADR-002 | Domain Module Boundaries and Ownership           | CF-ARCH-002 | Proposed       | Revise                          |
| ADR-003 | Frontend Rendering and Interaction Architecture  | CF-ARCH-003 | Proposed       | Retain with clarification       |
| ADR-004 | Backend Modular Monolith Architecture            | CF-ARCH-004 | Proposed       | Retain with clarification       |
| ADR-005 | PostgreSQL Database Architecture                 | CF-ARCH-005 | Proposed       | Retain with integrity revisions |
| ADR-006 | Independent Deterministic Simulation Engine      | CF-ARCH-006 | Proposed       | Retain with specification work  |
| ADR-007 | Asynchronous HTTP Integration Contract           | CF-ARCH-007 | Proposed       | Retain                          |
| ADR-008 | Defense-in-Depth Security Architecture           | CF-ARCH-008 | Proposed       | Retain with enforcement details |

## 3. Consolidated Architecture Decisions

### ADR-001: Modular Monolith + Independent Simulation Engine

**Source:** CF-ARCH-001  
**Current status:** Accepted  
**Review recommendation:** Retain

**Context**

CYBERFRONT membutuhkan backend untuk mengelola business workflows dan sistem simulasi teknis yang dapat berkembang secara independen.

**Decision**

Gunakan:

- Laravel 12 Modular Monolith untuk application backend.
- Independent TypeScript Simulation Engine untuk technical simulation.
- PostgreSQL sebagai primary persistent database.
- Redis sebagai supporting infrastructure.

**Consequences**

- Business logic tetap berada di Laravel.
- Simulation logic tetap berada di Engine.
- Engine dapat diuji dan dikembangkan secara independen.
- Integrasi membutuhkan explicit versioned contract.
- Ownership antara business state dan runtime state harus dipertahankan.

**Review notes**

Keputusan ini konsisten dengan seluruh dokumen dan tidak perlu diganti.

### ADR-002: Domain Module Boundaries and Ownership

**Source:** CF-ARCH-002  
**Current status:** Proposed  
**Review recommendation:** Revise before acceptance

**Context**

Backend terdiri dari delapan modul dengan domain ownership berbeda.

**Decision**

Pertahankan delapan modul:

1. Identity & Access
2. Players
3. Infrastructure
4. Component Catalog
5. Economy
6. Progression
7. PvP
8. Battles

Simulation Engine tetap berada di luar Laravel Modular Monolith.

Modul berkomunikasi melalui:

- Public Application Interfaces.
- Domain Events.
- Explicit Contracts.
- Asynchronous Jobs.

**Required revision**

Dependency direction harus bersifat acyclic.

PvP dapat memanggil Battles Public Interface untuk menginisiasi battle. Battles tidak boleh memiliki dependency langsung kembali ke PvP.

Battle completion harus dikomunikasikan melalui event atau workflow yang ditangani oleh pemilik match state.

**Consequences**

- Domain ownership tetap jelas.
- Circular dependency dapat dicegah.
- Cross-module workflow membutuhkan explicit coordination.
- Architecture tests wajib memverifikasi dependency boundaries.

### ADR-003: Frontend Rendering and Interaction Architecture

**Source:** CF-ARCH-003  
**Current status:** Proposed  
**Review recommendation:** Retain with clarification

**Context**

CYBERFRONT membutuhkan workspace isometric, logical network graph, dan technical configuration editor.

**Decision**

Gunakan hybrid frontend architecture:

- TypeScript untuk complex frontend logic.
- Alpine.js untuk lightweight reactive UI.
- Canvas 2D untuk isometric workspace.
- Cytoscape.js untuk logical network topology.
- Monaco Editor untuk technical configuration.
- Vite untuk asset bundling.

**Consequences**

- Isometric rendering dan logical network graph memiliki model visual terpisah.
- Screen coordinate tidak menjadi sumber kebenaran konektivitas.
- Frontend tidak menghitung authoritative battle result.
- Frontend state harus membedakan UI, Draft, Persistent, dan Runtime State.

**Review notes**

Keputusan teknologi dipertahankan.

Dokumentasi perlu memperjelas bahwa komunikasi frontend ke backend pada MVP dapat menggunakan REST API. WebSocket bukan dependency wajib untuk asynchronous PvP.

### ADR-004: Backend Modular Monolith Architecture

**Source:** CF-ARCH-004  
**Current status:** Proposed  
**Review recommendation:** Retain with clarification

**Context**

Backend harus mengelola identity, infrastructure, economy, progression, PvP, battle lifecycle, dan integrasi Engine tanpa kompleksitas microservices.

**Decision**

Gunakan Laravel 12 Modular Monolith dengan:

- API Layer.
- Application Layer.
- Domain Layer.
- Infrastructure Layer.

Setiap modul memiliki ownership, public interface, dan testing boundary.

**Consequences**

- Business logic tidak ditempatkan di controller.
- Domain Layer tidak bergantung pada HTTP, Redis, atau framework-specific infrastructure.
- Cross-module access harus melalui interface atau event.
- Queue digunakan untuk pekerjaan asynchronous.
- Engine communication menggunakan adapter.

**Required clarification**

Definisikan transaction coordination untuk workflow yang melibatkan lebih dari satu modul.

Tetapkan satu canonical battle lifecycle yang mencakup submission, engine acceptance, result processing, dan terminal failure states.

WebSocket diperlakukan sebagai optional/future integration, bukan requirement simulation transport MVP.

### ADR-005: PostgreSQL Database Architecture

**Source:** CF-ARCH-005  
**Current status:** Proposed  
**Review recommendation:** Retain with integrity revisions

**Context**

CYBERFRONT membutuhkan relational integrity, transactional consistency, historical records, JSONB configuration, snapshots, dan battle replay metadata.

**Decision**

Gunakan PostgreSQL sebagai primary persistent database dengan satu database pada MVP.

Setiap modul memiliki logical ownership terhadap tabelnya.

Redis bukan authoritative persistent data store.

JSONB digunakan secara selektif untuk:

- Component configuration.
- Snapshot payload.
- Battle result metadata.
- Event payload.
- Settings dan structured metadata.

**Required clarification**

- Gunakan positive transaction amount dengan direction pada transaction type.
- Tegakkan same-infrastructure component references.
- Finalisasi PvP Match dan Battle cardinality.
- Definisikan simulation job persistence.
- Definisikan replay artifact storage dan retention.
- Tegaskan cross-module write prohibition meskipun foreign key lintas modul diperbolehkan.

### ADR-006: Independent Deterministic Simulation Engine

**Source:** CF-ARCH-006  
**Current status:** Proposed  
**Review recommendation:** Retain with implementation specification

**Context**

CYBERFRONT membutuhkan simulation runtime yang independen, reproducible, dan dapat menjalankan technical attack/defense scenarios dalam virtual environment.

**Decision**

Gunakan independent TypeScript/Node.js Simulation Engine dengan:

- Pure Simulation Core.
- Domain Modules.
- Deterministic PRNG.
- Deterministic Event Scheduler.
- Versioned Rulesets.
- Isolated Runtime State.
- Execution Worker.
- Transport Adapter.

**Consequences**

- Engine tidak bergantung pada Laravel.
- Core tidak menggunakan uncontrolled randomness atau wall-clock time.
- Input dan output harus eksplisit.
- Battle historis harus memiliki version metadata.
- Replay menggunakan recorded events.

**Required clarification**

Sebelum Simulation Core implementation, tetapkan:

- PRNG algorithm dan version.
- RNG stream derivation.
- Scheduler tie-breaking.
- Numeric precision.
- Canonical serialization.
- Deterministic state transition rules.

### ADR-007: Asynchronous HTTP Integration Contract

**Source:** CF-ARCH-007  
**Current status:** Proposed  
**Review recommendation:** Retain

**Context**

Laravel dan Simulation Engine merupakan dua sistem independen. Simulasi tidak boleh bergantung pada lifecycle HTTP request frontend.

**Decision**

MVP menggunakan asynchronous HTTP integration:

1. Laravel submit simulation job melalui internal REST API.
2. Engine menerima dan memvalidasi request.
3. Engine menjalankan job melalui execution worker.
4. Engine mengirim result melalui authenticated callback.
5. Laravel memvalidasi dan mempersist hasil.
6. Laravel menyediakan status lookup sebagai fallback.

**Consequences**

- Simulation execution tidak memblokir frontend request.
- Engine dapat diskalakan secara independen.
- Submission dan callback harus idempotent.
- Result harus diverifikasi sebelum business effects.
- Status reconciliation harus tersedia.

**Required clarification**

- Final HMAC signing protocol.
- Payload size policy.
- Event artifact strategy.
- Callback retry and acknowledgement policy.
- Simulation status retention.
- Reconciliation behavior saat callback dan polling bersamaan.

### ADR-008: Defense-in-Depth Security Architecture

**Source:** CF-ARCH-008  
**Current status:** Proposed  
**Review recommendation:** Retain with enforcement details

**Context**

CYBERFRONT memproses player-controlled configuration dan virtual attack scenarios. Sistem harus mencegah simulation input memengaruhi host atau external systems.

**Decision**

Terapkan:

- Defense in Depth.
- Zero Trust.
- Least Privilege.
- Secure by Default.
- Fail Secure.
- Separation of Duties.
- Assume Breach.

**Consequences**

- Frontend dianggap untrusted.
- Backend memvalidasi ownership dan business operations.
- Engine memvalidasi seluruh simulation input.
- Economy dan battle results harus memiliki auditability.
- Simulation worker harus terisolasi.

**Required clarification**

Enforce containment melalui execution infrastructure, bukan hanya application-level validation.

Minimum controls:

- Default-deny outbound network.
- No arbitrary host commands.
- No privileged worker.
- Restricted filesystem.
- CPU, memory, and execution limits.
- No production database access.
- Job-level runtime isolation.
- Security containment testing.

---

## 4. Additional ADRs Recommended by Review

ADR berikut merupakan keputusan tambahan yang direkomendasikan. Nomor dan status final perlu dikonfirmasi sebelum dimasukkan ke ADR registry resmi.

### Proposed ADR-A: Canonical Battle Lifecycle

**Status:** Proposed

**Context**

Backend Architecture dan Integration Contract menggunakan daftar status yang berbeda tingkat detailnya.

**Recommended decision**

Satu canonical Battles state machine digunakan oleh Laravel.

Engine status dipetakan ke internal Laravel state.

Battle hanya menjadi COMPLETED setelah result validation, persistence, dan business workflow yang diwajibkan berhasil.

**Consequences**

- Tidak ada state transition yang ambigu.
- Retry dan failure dapat diaudit.
- Reward processing memiliki satu completion boundary.

### Proposed ADR-B: Replay Event Artifact Strategy

**Status:** Proposed

**Context**

Simulation callback dapat membawa event sequence yang besar.

**Recommended decision**

Result summary dipisahkan dari event archive.

Event kecil dapat dikirim inline. Event besar menggunakan artifact storage dengan:

- Battle identity.
- Event schema version.
- Event count.
- Sequence range.
- Content hash.
- Artifact size.
- Retention metadata.

**Consequences**

- Callback payload lebih terkontrol.
- Replay dapat disimpan dan diambil secara efisien.
- Integrity dapat diverifikasi secara terpisah.

### Proposed ADR-C: Deterministic Execution Specification

**Status:** Proposed

**Context**

Simulation Engine membutuhkan determinism lintas execution dan versi.

**Recommended decision**

Buat specification implementasi yang menetapkan:

- PRNG.
- RNG stream derivation.
- Event ordering.
- Numeric precision.
- Canonical serialization.
- Version compatibility.
- Golden test requirements.

**Consequences**

- Deterministic behavior dapat diuji.
- Engine changes dapat dievaluasi secara konsisten.
- Battle replay dan audit lebih dapat dipercaya.

### Proposed ADR-D: Simulation Worker Isolation

**Status:** Proposed

**Context**

Simulation Engine memproses input yang dikendalikan pemain dan attack scenario virtual.

**Recommended decision**

Setiap worker menjalankan job dalam isolated execution boundary dengan default-deny network egress, restricted filesystem, dan bounded resource usage.

**Consequences**

- Simulation tidak dapat digunakan sebagai real-world attack host.
- Malformed input memiliki dampak yang dibatasi.
- Runtime state antar job tidak saling bocor.

---

## 5. Consolidated Architecture Baseline

Setelah keputusan di atas diselesaikan, baseline arsitektur yang direkomendasikan adalah:

| Area                   | Decision                                       |
| ---------------------- | ---------------------------------------------- |
| Backend                | Laravel 12 Modular Monolith                    |
| Simulation             | Independent TypeScript/Node.js Engine          |
| Persistent database    | PostgreSQL                                     |
| Queue/cache            | Redis                                          |
| Frontend               | TypeScript + Alpine.js                         |
| Isometric workspace    | Canvas 2D                                      |
| Network topology       | Cytoscape.js                                   |
| Configuration editor   | Monaco Editor                                  |
| Backend API            | REST API                                       |
| Engine integration     | Asynchronous HTTP                              |
| Engine result delivery | Authenticated callback                         |
| Battle mode MVP        | Asynchronous PvP                               |
| Snapshot               | Immutable, versioned, hashed                   |
| Simulation             | Deterministic discrete-event model             |
| Replay                 | Recorded event sequence                        |
| Security               | Defense in Depth + isolated virtual simulation |

## 6. Decision Governance

Aturan pengelolaan ADR:

1. Setiap keputusan arsitektur penting harus memiliki ADR.
2. ADR yang sudah Accepted tidak boleh diubah diam-diam.
3. Perubahan keputusan harus memiliki alasan dan consequences.
4. Superseded ADR harus tetap disimpan sebagai historical record.
5. Dokumen arsitektur harus mengacu pada ADR yang relevan.
6. Implementasi tidak boleh mengandalkan keputusan yang masih Proposed tanpa persetujuan yang sesuai.
7. Setiap ADR harus memiliki owner dan tanggal keputusan ketika governance implementasi mulai diberlakukan.

## 7. Approval

Status dokumen ini tetap Proposed sampai pemilik proyek menyetujui keputusan yang direkomendasikan.

**Recommended next step:** Resolve High Priority findings in CF-ARCH-009, update affected documents, then review ADR status before architecture approval.

---

**End of Document**
