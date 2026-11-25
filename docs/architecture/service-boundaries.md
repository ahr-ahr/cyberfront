# CYBERFRONT

## Service Boundaries & Module Responsibilities

**Document ID:** CF-ARCH-002  
**Architecture Decision:** CF-ADR-002  
**Version:** 0.1.0  
**Phase:** 0 - Product Foundation  
**Task:** 02.2 - Service Boundaries  
**Status:** Proposed  
**Date:** November 25, 2026

---

# 1. Introduction

## 1.1 Purpose

Dokumen ini mendefinisikan batas tanggung jawab, kepemilikan data, komunikasi, dan dependensi antar modul dalam arsitektur CYBERFRONT.

CYBERFRONT menggunakan Modular Monolith untuk application backend dan Independent Simulation Engine untuk menjalankan simulasi teknologi.

Setiap modul harus memiliki tanggung jawab yang jelas dan tidak boleh mengambil alih business logic milik modul lain.

## 1.2 Architectural Goals

- Clear domain boundaries.
- Single ownership of business data.
- Low coupling between modules.
- Explicit module communication.
- Independent module testing.
- Consistent business transactions.
- Independent simulation execution.
- Future service extraction capability.

---

# 2. Module Overview

CYBERFRONT memiliki delapan modul utama pada Application Layer.

| Module            | Primary Responsibility                     |
| ----------------- | ------------------------------------------ |
| Identity & Access | Authentication and authorization           |
| Players           | Player profiles and account state          |
| Infrastructure    | Infrastructure ownership and configuration |
| Component Catalog | Technology component definitions           |
| Economy           | Credits and financial transactions         |
| Progression       | Technology unlocks and specialization      |
| PvP               | Target discovery and matchmaking           |
| Battles           | Battle lifecycle and report management     |

Simulation Engine merupakan sistem independen di luar Laravel.

---

# 3. Module Responsibilities

## 3.1 Identity & Access Module

### Responsibility

Mengelola identitas, autentikasi, dan otorisasi pengguna.

### Owned Data

- User identity.
- Authentication credentials.
- Session information.
- Account security settings.
- Access permissions.

### Main Operations

- User registration.
- User authentication.
- Logout.
- Password management.
- Account verification.
- Permission validation.

### Restrictions

Identity & Access tidak bertanggung jawab atas profil permainan, inventory, Credits, atau progression pemain.

---

## 3.2 Players Module

### Responsibility

Mengelola identitas pemain di dalam dunia permainan.

### Owned Data

- Player profile.
- Player preferences.
- Player status.
- Player statistics.
- Player metadata.

### Main Operations

- Create player profile.
- Retrieve player profile.
- Update player information.
- Retrieve player statistics.
- Manage player status.

### Dependencies

- Identity & Access.

### Restrictions

Players tidak boleh mengelola autentikasi secara langsung atau memodifikasi transaksi ekonomi.

---

## 3.3 Infrastructure Module

### Responsibility

Mengelola infrastruktur teknologi yang dimiliki pemain.

Modul ini bertanggung jawab atas struktur dan konfigurasi infrastruktur sebelum dijalankan oleh Simulation Engine.

### Owned Data

- Infrastructure identity.
- Infrastructure ownership.
- Component instances.
- Component configurations.
- Network topology.
- Interface connections.
- Infrastructure versions.
- Defense Snapshots.

### Main Operations

- Create infrastructure.
- Add component instance.
- Remove component instance.
- Update component configuration.
- Connect network interfaces.
- Validate infrastructure structure.
- Publish Defense Snapshot.
- Retrieve infrastructure versions.

### Dependencies

- Players.
- Component Catalog.
- Economy.
- Progression.

### Restrictions

Infrastructure tidak boleh mengimplementasikan perilaku runtime komponen atau menghitung hasil serangan.

Modul ini hanya bertanggung jawab atas definisi dan konfigurasi infrastruktur.

---

## 3.4 Component Catalog Module

### Responsibility

Mengelola katalog definisi komponen teknologi yang tersedia dalam permainan.

### Owned Data

- Component definitions.
- Component categories.
- Component specifications.
- Component configuration schemas.
- Component resource requirements.
- Component compatibility metadata.
- Component version information.

### Main Operations

- Register component definition.
- Retrieve component definition.
- Validate component configuration schema.
- Retrieve component requirements.
- Retrieve component compatibility information.
- Manage component availability.

### Example Components

- Linux Server.
- Router.
- Switch.
- Firewall.
- Reverse Proxy.
- Web Application.
- PostgreSQL.
- Monitoring Agent.

### Restrictions

Component Catalog mendefinisikan karakteristik dan spesifikasi komponen.

Perilaku runtime komponen menjadi tanggung jawab Simulation Engine.

Katalog dan engine harus menggunakan identitas komponen serta versi ruleset yang kompatibel.

---

## 3.5 Economy Module

### Responsibility

Mengelola seluruh aktivitas ekonomi pemain.

### Owned Data

- Player Credits balance.
- Transaction ledger.
- Purchase records.
- Operational expenses.
- Infrastructure costs.
- Economic transaction history.

### Main Operations

- Retrieve balance.
- Purchase component.
- Deduct Credits.
- Apply rewards.
- Record transaction.
- Validate transaction.
- Retrieve transaction history.

### Business Rules

Setiap perubahan saldo harus tercatat sebagai transaksi yang dapat diaudit.

Transaksi harus menggunakan mekanisme atomic dan idempotent untuk menghindari duplikasi perubahan saldo.

### Restrictions

Modul lain tidak boleh mengubah saldo Credits secara langsung.

Seluruh perubahan ekonomi harus melewati Economy Module.

Simulation Engine tidak boleh mengubah saldo pemain secara langsung.

---

## 3.6 Progression Module

### Responsibility

Mengelola perkembangan kemampuan engineering pemain.

### Owned Data

- Player experience.
- Technology unlocks.
- Progression levels.
- Specialization paths.
- Achievement records.
- Progression history.

### Main Operations

- Retrieve progression.
- Award experience.
- Unlock technology.
- Validate unlock requirements.
- Manage specialization.
- Retrieve available technologies.

### Restrictions

Progression tidak boleh mengubah definisi teknis komponen.

Komponen yang dapat digunakan pemain harus tetap mengikuti validasi Component Catalog dan Infrastructure.

---

## 3.7 PvP Module

### Responsibility

Mengelola proses pencarian target dan matchmaking.

### Owned Data

- Matchmaking preferences.
- Target eligibility.
- PvP matchmaking records.
- Matchmaking history.
- Target selection metadata.

### Main Operations

- Discover targets.
- Validate target eligibility.
- Apply matchmaking rules.
- Select target.
- Initiate battle request.

### Dependencies

- Players.
- Infrastructure.
- Battles.

### Restrictions

PvP tidak boleh menjalankan simulation logic.

Modul ini hanya mengatur proses menemukan dan memilih pertandingan.

---

## 3.8 Battles Module

### Responsibility

Mengelola seluruh lifecycle pertandingan.

Modul ini menjadi penghubung antara application backend dan Simulation Engine.

### Owned Data

- Battle identity.
- Battle participants.
- Battle status.
- Snapshot references.
- Simulation job references.
- Battle results.
- Battle reports.
- Replay metadata.

### Main Operations

- Create battle.
- Validate battle request.
- Submit simulation job.
- Track battle execution.
- Process simulation result.
- Persist battle report.
- Retrieve battle history.
- Retrieve replay data.

### Dependencies

- Players.
- Infrastructure.
- PvP.
- Simulation Engine.

### Restrictions

Battles tidak boleh mengimplementasikan algoritma serangan atau perilaku teknis komponen.

Battle resolution harus dilakukan oleh Simulation Engine.

Laravel bertanggung jawab memvalidasi hasil dan menerapkan konsekuensi business logic.

---

# 4. Independent Simulation Engine

## 4.1 Responsibility

Simulation Engine merupakan sistem independen yang menjalankan seluruh mekanisme simulasi teknologi.

Engine tidak termasuk dalam Laravel Modular Monolith.

## 4.2 Internal Domains

| Domain                 | Responsibility                           |
| ---------------------- | ---------------------------------------- |
| Simulation Core        | Execution lifecycle and state management |
| Infrastructure Runtime | Runtime infrastructure state             |
| Component Runtime      | Component behavior                       |
| Networking             | Network and connectivity behavior        |
| Attack System          | Attack execution                         |
| Defense System         | Defense evaluation                       |
| Resource System        | Technical resource modeling              |
| Event System           | Simulation event processing              |
| Battle Resolution      | Final simulation outcome                 |
| Replay System          | Event history and replay data            |

## 4.3 Input

Engine menerima:

- Battle identifier.
- Infrastructure snapshot.
- Component configurations.
- Network topology.
- Attack scenario.
- Random seed.
- Simulation ruleset version.

## 4.4 Output

Engine menghasilkan:

- Simulation status.
- Battle outcome.
- Generated events.
- Component impact.
- Resource impact.
- Security events.
- Replay information.
- Simulation metadata.

## 4.5 Restrictions

Simulation Engine:

- Tidak mengelola user authentication.
- Tidak mengelola player accounts.
- Tidak mengelola Credits.
- Tidak mengelola inventory.
- Tidak mengakses business models Laravel.
- Tidak memodifikasi data pemain secara langsung.

Engine hanya bertanggung jawab atas hasil simulasi.

---

# 5. Module Dependency Rules

## 5.1 General Rules

1. Setiap modul memiliki satu domain ownership.
2. Modul tidak boleh mengubah data milik modul lain secara langsung.
3. Cross-module operations harus menggunakan public interfaces.
4. Domain events digunakan untuk komunikasi berbasis kejadian.
5. Business logic harus tetap berada di modul pemilik domain.
6. Circular dependencies tidak diperbolehkan.
7. Shared functionality harus dibatasi pada kebutuhan yang benar-benar umum.

## 5.2 Dependency Matrix

| Consumer          | Allowed Dependencies                              |
| ----------------- | ------------------------------------------------- |
| Identity          | None                                              |
| Players           | Identity                                          |
| Infrastructure    | Players, Catalog, Economy, Progression            |
| Component Catalog | None                                              |
| Economy           | Players                                           |
| Progression       | Players                                           |
| PvP               | Players, Infrastructure, Battles                  |
| Battles           | Players, Infrastructure, PvP, Simulation Contract |

Dependencies menunjukkan akses melalui public interfaces, bukan akses bebas terhadap internal class atau database model.

## 5.3 Restricted Dependencies

Contoh dependensi yang tidak diperbolehkan:

- Economy mengubah Infrastructure secara langsung.
- Infrastructure mengubah Credits tanpa Economy.
- PvP menjalankan attack simulation.
- Players mengubah progression data secara langsung.
- Simulation Engine mengakses Eloquent models.
- Frontend menentukan hasil battle.

---

# 6. Module Communication

CYBERFRONT menggunakan tiga mekanisme komunikasi.

## 6.1 Direct Module Interfaces

Digunakan ketika modul membutuhkan respons langsung dari modul lain.

Contoh:

Infrastructure meminta Component Catalog memvalidasi definisi komponen.

## 6.2 Domain Events

Digunakan untuk memberitahukan perubahan atau kejadian dalam domain.

Initial event examples:

| Event                    | Producer       |
| ------------------------ | -------------- |
| PlayerRegistered         | Identity       |
| PlayerProfileCreated     | Players        |
| InfrastructureCreated    | Infrastructure |
| ComponentInstalled       | Infrastructure |
| DefenseSnapshotPublished | Infrastructure |
| CreditsDeducted          | Economy        |
| TechnologyUnlocked       | Progression    |
| BattleCreated            | Battles        |
| BattleCompleted          | Battles        |

Event harus memiliki schema yang jelas dan tidak mengekspos internal implementation details.

## 6.3 Asynchronous Jobs

Digunakan untuk pekerjaan yang tidak perlu diselesaikan dalam HTTP request.

Contoh:

- Battle simulation.
- Large replay generation.
- Background processing.
- Infrastructure analysis.

---

# 7. Data Ownership

CYBERFRONT menggunakan satu PostgreSQL database pada tahap awal.

Walaupun database masih sama, setiap modul memiliki ownership terhadap tabelnya sendiri.

| Data Group                 | Owner             |
| -------------------------- | ----------------- |
| Users and credentials      | Identity          |
| Player profiles            | Players           |
| Infrastructure definitions | Infrastructure    |
| Component instances        | Infrastructure    |
| Component definitions      | Component Catalog |
| Credits and transactions   | Economy           |
| Progression and unlocks    | Progression       |
| Matchmaking records        | PvP               |
| Battle records             | Battles           |
| Simulation runtime state   | Simulation Engine |

## 7.1 Database Access Rules

- Modul hanya boleh menulis tabel yang dimilikinya.
- Modul lain harus membaca data melalui public query interfaces.
- Cross-module database writes tidak diperbolehkan.
- Database transactions harus dikendalikan oleh modul pemilik operasi.
- Cross-module consistency harus menggunakan explicit coordination.

Foreign keys lintas modul dapat digunakan jika diperlukan, tetapi tidak boleh menjadi alasan untuk mengakses atau mengubah data modul lain secara langsung.

---

# 8. Cross-Module Workflow

## 8.1 Infrastructure Creation

1. Player requests infrastructure creation.
2. Infrastructure validates player ownership.
3. Infrastructure requests component definitions from Component Catalog.
4. Economy validates and processes required Credits.
5. Infrastructure persists the infrastructure definition.
6. Infrastructure publishes InfrastructureCreated event.

## 8.2 Component Installation

1. Player selects a component.
2. Infrastructure requests component specifications.
3. Progression validates unlock eligibility.
4. Economy processes purchase if required.
5. Infrastructure validates resource and compatibility constraints.
6. Infrastructure creates component instance.
7. ComponentInstalled event is published.

## 8.3 Battle Execution

1. PvP discovers an eligible target.
2. Attacker requests a battle.
3. Battles validates participants.
4. Infrastructure supplies an immutable Defense Snapshot.
5. Battles creates a simulation job.
6. Simulation Engine executes the battle.
7. Engine returns simulation results.
8. Battles validates and persists the result.
9. Economy and Progression process eligible consequences.
10. BattleCompleted event is published.

---

# 9. Shared Kernel

CYBERFRONT may use a limited Shared Kernel for truly universal technical primitives.

Examples:

- UUID identifiers.
- Common timestamp conventions.
- Standard error structures.
- Contract version metadata.
- Common serialization primitives.

Shared Kernel must not become a location for arbitrary business logic.

Domain-specific rules remain owned by their respective modules.

---

# 10. Transaction and Consistency Rules

## 10.1 Local Transactions

Each module manages its own database transactions.

## 10.2 Cross-Module Operations

Cross-module business operations must use explicit application services and clearly defined failure handling.

## 10.3 Asynchronous Consistency

Asynchronous operations must account for:

- Duplicate events.
- Retry handling.
- Idempotency.
- Failed message delivery.
- Partial execution.
- Result reconciliation.

For critical operations such as battle submission, a transactional outbox mechanism should be evaluated to prevent database state and queue delivery from becoming inconsistent.

## 10.4 Battle Result Integrity

Simulation results must be associated with:

- Unique battle identifier.
- Exact Defense Snapshot.
- Simulation ruleset version.
- Random seed.
- Input fingerprint.

Laravel must validate the battle state and result identity before committing business consequences.

---

# 11. Testing Boundaries

Each module must support independent testing.

Testing categories:

**Unit Tests**

- Domain rules.
- Value objects.
- Business logic.

**Module Tests**

- Public module interfaces.
- Module operations.
- Ownership constraints.

**Integration Tests**

- Cross-module workflows.
- Event delivery.
- Transaction behavior.

**Contract Tests**

- Backend and Simulation Engine communication.
- Input and output schema validation.
- Contract version compatibility.

**Simulation Tests**

- Deterministic execution.
- Component behavior.
- Network behavior.
- Battle resolution.

---

# 12. Future Scalability

The architecture should allow individual domains to become independent services if actual operational needs justify extraction.

Potential candidates:

- Simulation Engine.
- Real-Time Gateway.
- Battle Processing.
- Matchmaking.

Service extraction must preserve existing domain ownership and communication contracts.

No module should be extracted solely to increase the number of deployable services.

---

# 13. Architecture Decision Record

**Decision ID:** CF-ADR-002

**Title:** Domain Module Boundaries and Ownership

**Status:** Proposed

## Context

CYBERFRONT requires multiple technology and gameplay domains that interact frequently.

Without explicit boundaries, the modular monolith may become tightly coupled and difficult to maintain.

## Decision

Use eight primary application modules with explicit domain ownership.

Simulation Engine remains independent of the Laravel application.

Modules communicate through public interfaces, domain events, and asynchronous jobs.

## Consequences

Positive:

- Clear ownership.
- Reduced coupling.
- Easier testing.
- Better maintainability.
- Future service extraction capability.

Trade-offs:

- Additional interface definitions.
- More explicit communication.
- Cross-module workflows require coordination.
- Event consistency requires careful implementation.

---

# 14. Acceptance Criteria

Task 02.2 is complete when:

- [ ] All application modules are defined.
- [ ] Module responsibilities are documented.
- [ ] Data ownership is established.
- [ ] Dependency matrix is documented.
- [ ] Restricted dependencies are documented.
- [ ] Module communication rules are defined.
- [ ] Simulation Engine boundaries are established.
- [ ] Cross-module workflows are documented.
- [ ] Transaction and consistency rules are established.
- [ ] Testing boundaries are documented.
- [ ] ADR-002 is reviewed and accepted.

---

# 15. Related Documents

| Document ID | Document                       |
| ----------- | ------------------------------ |
| CF-PROD-001 | Product Foundation             |
| CF-ARCH-001 | System Architecture Overview   |
| CF-ARCH-002 | Service Boundaries             |
| CF-ARCH-003 | Frontend Architecture          |
| CF-ARCH-004 | Backend Architecture           |
| CF-ARCH-005 | Simulation Engine Architecture |
| CF-ARCH-006 | Database Architecture          |
| CF-ARCH-007 | Communication Architecture     |
| CF-ARCH-008 | Battle Processing              |

---

**Document Status:** Proposed  
**Document Version:** 0.1.0  
**Next Task:** Phase 0 - Task 02.3: Frontend Architecture

**CYBERFRONT**

Build systems like an engineer.  
Attack like an adversary.  
Defend like an operator.

---

End of Document.
