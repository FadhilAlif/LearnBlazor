# TaskFlow — Product Requirements Document

> **Mini Project Management Tool** · Built with Blazor Server + .NET 8

|                   |                               |
| ----------------- | ----------------------------- |
| **Versi dokumen** | 1.0.0                         |
| **Tanggal**       | Mei 2026                      |
| **Status**        | Draft — Review Internal       |
| **Author**        | Frontend Dev (Blazor Learner) |
| **Platform**      | Blazor Server / .NET 8        |

---

## Daftar Isi

1. [Ringkasan Eksekutif](#1-ringkasan-eksekutif)
2. [Latar Belakang & Konteks](#2-latar-belakang--konteks)
3. [Scope & Batasan](#3-scope--batasan)
4. [Fitur & Functional Requirements](#4-fitur--functional-requirements)
5. [Arsitektur Teknis](#5-arsitektur-teknis)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [User Stories & Acceptance Criteria](#7-user-stories--acceptance-criteria)
8. [Milestones & Timeline](#8-milestones--timeline)
9. [Risiko & Mitigasi](#9-risiko--mitigasi)
10. [Appendix — Referensi Teknis](#10-appendix--referensi-teknis)

---

## 1. Ringkasan Eksekutif

TaskFlow adalah aplikasi manajemen proyek berbasis web yang dibangun dengan Blazor Server dan .NET 8. Aplikasi ini dirancang sebagai learning project yang secara sengaja mencakup seluruh konsep penting dalam ekosistem Blazor, mulai dari komponen dasar, autentikasi, real-time communication, hingga file attachment.

TaskFlow memungkinkan tim kecil (2–20 orang) untuk berkolaborasi dalam proyek menggunakan tampilan Kanban board. Setiap anggota tim dapat membuat proyek, mengelola card tugas, memberikan komentar secara real-time, serta melacak aktivitas seluruh anggota.

### 1.1 Tujuan Pembelajaran

Proyek ini dirancang khusus agar builder memahami dan mempraktikkan konsep-konsep berikut secara end-to-end:

- Blazor Server components, lifecycle, dan cascading state
- ASP.NET Identity dengan role-based authorization (Owner, Member, Viewer)
- Entity Framework Core 8 dengan PostgreSQL — relasi kompleks dan migration
- SignalR untuk real-time collaboration (komentar, drag & drop sync)
- JavaScript Interop untuk drag & drop Kanban
- File upload dan attachment management
- Email notifikasi dengan MailKit
- Clean Architecture pattern untuk separation of concerns

---

## 2. Latar Belakang & Konteks

### 2.1 Problem Statement

Banyak developer frontend yang beralih ke .NET tidak memiliki proyek referensi yang cukup kompleks untuk memahami pola-pola enterprise Blazor secara menyeluruh. Tutorial sederhana hanya menutupi satu atau dua konsep, sehingga developer tidak siap menghadapi codebase production yang memiliki banyak moving parts.

### 2.2 Solusi

TaskFlow dirancang sebagai _reference project_ yang cukup realistis untuk menduplikasi pola-pola yang akan ditemukan di project Blazor enterprise, namun tetap cukup sederhana untuk diselesaikan dalam 3–4 minggu oleh seorang developer yang baru belajar Blazor.

### 2.3 Target Pengguna Aplikasi

| Persona           | Deskripsi                                 | Kebutuhan Utama                           |
| ----------------- | ----------------------------------------- | ----------------------------------------- |
| **Project Owner** | Membuat & mengelola proyek, invite member | Full CRUD, user management, analytics     |
| **Team Member**   | Mengerjakan task, update status, komentar | Drag & drop card, real-time collaboration |
| **Viewer**        | Memantau progress proyek tanpa edit       | Read-only dashboard, laporan progress     |

---

## 3. Scope & Batasan

### 3.1 In Scope (MVP)

- Manajemen proyek: buat, edit, hapus, archive proyek
- Kanban board dengan kolom kustom (To Do, In Progress, Done, dll.)
- Card management: judul, deskripsi, assignee, due date, priority, label
- Drag & drop card antar kolom dengan sync real-time
- Role-based access: Owner, Member, Viewer per proyek
- Komentar real-time pada card menggunakan SignalR
- File attachment pada card (gambar & dokumen, maks 10 MB/file)
- Activity log per card dan per proyek
- Email notifikasi: assigned card, komentar baru, due date reminder
- Dashboard statistik: total card per status, overdue cards

### 3.2 Out of Scope (Fase 2)

- Mobile native app — hanya web responsive
- Time tracking dan timesheet
- Gantt chart dan dependency antar card
- Integrasi GitHub/GitLab
- Billing dan subscription management
- API publik untuk integrasi pihak ketiga

### 3.3 Asumsi & Constraint

- Maksimal 5 proyek aktif per user (batasan untuk learning project)
- File attachment maksimal 10 MB per file, total 100 MB per proyek
- Blazor Server dipilih (bukan WASM) karena latency lebih rendah dan lebih umum di enterprise
- Database: PostgreSQL (atau SQLite untuk development lokal)
- Autentikasi hanya menggunakan email & password (ASP.NET Identity)

---

## 4. Fitur & Functional Requirements

### 4.1 Modul Autentikasi

#### FR-AUTH-01: Registrasi & Login

User dapat mendaftarkan akun baru menggunakan email dan password. Validasi dilakukan di sisi server menggunakan DataAnnotations. Password minimal 8 karakter dengan kombinasi huruf dan angka.

| ID      | Fitur    | Deskripsi                                         | Blazor Concept                            |
| ------- | -------- | ------------------------------------------------- | ----------------------------------------- |
| AUTH-01 | Register | Form register dengan validasi email unik          | `EditForm`, `DataAnnotations`             |
| AUTH-02 | Login    | Login dengan remember me & redirect setelah login | `CascadingAuthState`, `NavigationManager` |
| AUTH-03 | Logout   | Logout clear session & redirect ke landing        | `AuthenticationStateProvider`             |
| AUTH-04 | Profile  | Edit nama, avatar, ubah password                  | JS Interop (image preview), `HttpClient`  |

---

### 4.2 Modul Proyek

#### FR-PROJ-01: Project CRUD

Owner dapat membuat proyek baru dengan nama, deskripsi, warna, dan visibility (private/team). Setiap proyek memiliki satu atau lebih board Kanban.

| ID      | Fitur            | Deskripsi                                  | Blazor Concept                         |
| ------- | ---------------- | ------------------------------------------ | -------------------------------------- |
| PROJ-01 | Buat proyek      | Form dengan color picker dan icon selector | Component parameter, `EventCallback`   |
| PROJ-02 | Invite member    | Undang via email, set role saat invite     | Service layer, email integration       |
| PROJ-03 | Manage roles     | Owner dapat ubah role atau remove member   | `AuthorizationService`, Policy         |
| PROJ-04 | Archive proyek   | Soft delete — data tetap ada, tidak aktif  | EF Core soft delete pattern            |
| PROJ-05 | Project settings | Edit nama, deskripsi, kolom default        | `CascadingValue` untuk project context |

---

### 4.3 Modul Kanban Board

#### FR-BOARD-01: Board & Kolom

Board adalah tampilan utama untuk melihat dan mengelola card. Board memiliki kolom-kolom yang dapat dikustomisasi. Urutan kolom dapat diubah dengan drag & drop.

| ID       | Fitur          | Deskripsi                                         | Blazor Concept                                   |
| -------- | -------------- | ------------------------------------------------- | ------------------------------------------------ |
| BOARD-01 | Kanban view    | Kolom horizontal, card di dalam kolom             | Component composition, `RenderFragment`          |
| BOARD-02 | Drag & drop    | Pindah card antar kolom dengan mouse/touch        | JS Interop (SortableJS), `DotNetObjectReference` |
| BOARD-03 | Real-time sync | Drag & drop ter-sync ke semua user di board       | SignalR Hub, Group broadcast                     |
| BOARD-04 | Quick add card | Klik '+' di kolom untuk tambah card cepat         | Inline edit component, focus management          |
| BOARD-05 | Filter board   | Filter card berdasarkan assignee, label, due date | LINQ query, reactive state                       |

---

### 4.4 Modul Card (Task)

#### FR-CARD-01: Card Detail

Setiap card memiliki halaman detail yang dapat dibuka sebagai modal overlay. Card detail menampilkan semua informasi, komentar, dan attachment.

**Konten card:**

- Judul dan deskripsi dengan rich text (markdown renderer)
- Assignee: dapat assign lebih dari satu member
- Due date dengan date picker — visual alert jika overdue
- Priority: `Low` / `Medium` / `High` / `Critical` dengan warna badge
- Label: tag warna yang dapat dikustom per proyek
- Checklist: sub-task dalam card dengan progress bar
- File attachment: upload multiple files, preview gambar inline
- Komentar: real-time via SignalR, support markdown, mention `@user`
- Activity log: semua perubahan card tercatat dengan timestamp

| ID      | Fitur        | Deskripsi                                 | Blazor Concept                                |
| ------- | ------------ | ----------------------------------------- | --------------------------------------------- |
| CARD-01 | Card modal   | Open card detail sebagai modal overlay    | `CascadingParameter`, `RenderFragment`        |
| CARD-02 | Inline edit  | Edit judul/deskripsi langsung di modal    | Two-way binding, debounced autosave           |
| CARD-03 | Checklist    | Sub-task dengan checkbox & progress bar   | EF Core one-to-many, computed property        |
| CARD-04 | File upload  | Multi-file upload dengan drag & drop area | `IBrowserFile`, `IFormFile`, streaming upload |
| CARD-05 | Komentar RT  | Komentar masuk realtime tanpa refresh     | SignalR Hub, `OnInitializedAsync`             |
| CARD-06 | Activity log | Semua perubahan card tercatat otomatis    | Service Interceptor, background logging       |

---

## 5. Arsitektur Teknis

### 5.1 Tech Stack

| Layer        | Teknologi                    | Keterangan                            |
| ------------ | ---------------------------- | ------------------------------------- |
| Frontend     | Blazor Server (.NET 8)       | Render di server, state di server     |
| Styling      | Tailwind CSS + CSS Isolation | Utility-first, scoped per komponen    |
| Real-time    | ASP.NET Core SignalR         | WebSocket untuk komentar & board sync |
| ORM          | Entity Framework Core 8      | Code-first, migration, PostgreSQL     |
| Auth         | ASP.NET Core Identity        | Cookie-based, role claims             |
| Email        | MailKit + MimeKit            | SMTP atau SendGrid relay              |
| File Storage | Local disk / Azure Blob      | Configurable via `IStorageService`    |
| Database     | PostgreSQL _(dev: SQLite)_   | EF Core provider abstraction          |
| JS Library   | SortableJS                   | Drag & drop via JS Interop            |

---

### 5.2 Folder Structure (Clean Architecture)

```
TaskFlow/
├── TaskFlow.Domain/          # Entities, Interfaces, Enums — zero dependencies
├── TaskFlow.Application/     # Use cases, DTOs, Service interfaces, CQRS handlers
├── TaskFlow.Infrastructure/  # EF Core DbContext, Repositories, Email, File storage
└── TaskFlow.Web/             # Blazor components, Pages, Hubs, Startup, wwwroot
    ├── Components/
    │   ├── Board/
    │   │   ├── BoardPage.razor
    │   │   ├── ColumnComponent.razor
    │   │   └── CardComponent.razor
    │   ├── Card/
    │   │   ├── CardModal.razor
    │   │   ├── ChecklistComponent.razor
    │   │   └── CommentSection.razor
    │   └── Shared/
    ├── Hubs/
    │   └── BoardHub.cs
    └── wwwroot/
        └── js/
            └── sortable-interop.js
```

---

### 5.3 Data Model (Entity Utama)

```
User ──────────────────────────────────────────────┐
 │                                                  │
 ├──< ProjectMember >──── Project                   │
 │                          │                       │
 │                          ├──< Board              │
 │                          │      └──< Column      │
 │                          │             └──< Card │
 │                          │                   │   │
 │                          └──< ActivityLog    │   │
 │                                              │   │
 ├──────────────────────────────< CardAssignee ─┘   │
 ├──────────────────────────────< Comment ──────────┘
 └──────────────────────────────< Attachment
```

| Entity          | Relasi Utama                             | Field Penting                                                     |
| --------------- | ---------------------------------------- | ----------------------------------------------------------------- |
| `User`          | —                                        | `Id`, `Email`, `DisplayName`, `AvatarUrl` _(ASP.NET Identity)_    |
| `Project`       | Owner: User · Members: `ProjectMember[]` | `Name`, `Description`, `Color`, `IsArchived`, `CreatedAt`         |
| `ProjectMember` | Project + User                           | `ProjectId`, `UserId`, `Role` (Owner/Member/Viewer)               |
| `Board`         | Project (1 proyek = 1+ board)            | `ProjectId`, `Name`, `Position`, `IsDefault`                      |
| `Column`        | Board                                    | `BoardId`, `Name`, `Position`, `Color`, `WipLimit`                |
| `Card`          | Column, Assignees, Labels                | `Title`, `Description`, `DueDate`, `Priority`, `Position`         |
| `CardAssignee`  | Card + User _(many-to-many)_             | `CardId`, `UserId`, `AssignedAt`                                  |
| `Checklist`     | Card                                     | `CardId`, `Title`, `Items: ChecklistItem[]`                       |
| `Comment`       | Card + User                              | `CardId`, `UserId`, `Content`, `CreatedAt`, `IsEdited`            |
| `Attachment`    | Card                                     | `CardId`, `FileName`, `FileUrl`, `FileSize`, `UploadedAt`         |
| `ActivityLog`   | Card + User                              | `CardId`, `UserId`, `Action`, `OldValue`, `NewValue`, `CreatedAt` |

---

## 6. Non-Functional Requirements

| Kategori            | Target                         | Cara Implementasi                                     |
| ------------------- | ------------------------------ | ----------------------------------------------------- |
| **Performance**     | Halaman load < 2 detik         | Lazy load komponen, pagination, EF query optimization |
| **Security**        | Tidak ada unauthorized access  | `AuthorizeView`, Policy-based auth, CSRF protection   |
| **Availability**    | 99% uptime _(learning target)_ | Health check endpoint, proper error handling          |
| **Scalability**     | Hingga 50 concurrent users     | SignalR connection grouping, connection limit config  |
| **Maintainability** | Clean, testable code           | Clean Architecture, unit test dengan xUnit + bUnit    |
| **Accessibility**   | WCAG 2.1 AA dasar              | Semantic HTML, ARIA attributes, keyboard navigation   |

---

## 7. User Stories & Acceptance Criteria

### 7.1 Epic: Project Management

---

**US-01** — Sebagai **Project Owner**, saya ingin membuat proyek baru agar tim saya dapat mulai berkolaborasi.

**Acceptance Criteria:**

1. Form memiliki validasi nama (wajib, min 3 karakter)
2. Color picker tersedia dengan 12 pilihan warna
3. Setelah buat, redirect ke board view proyek baru
4. Creator otomatis menjadi Owner

---

**US-02** — Sebagai **Owner**, saya ingin mengundang anggota via email agar mereka dapat bergabung ke proyek.

**Acceptance Criteria:**

1. Input email dengan validasi format
2. Role selector tersedia: Member atau Viewer
3. Email undangan terkirim dalam 5 menit
4. Link undangan valid selama 7 hari

---

**US-03** — Sebagai **Member**, saya ingin memindahkan card antar kolom agar status tugas selalu up-to-date.

**Acceptance Criteria:**

1. Drag & drop berfungsi di desktop dan mobile
2. Perubahan tersync ke user lain dalam 1 detik
3. Activity log tercatat otomatis dengan timestamp
4. Undo tersedia selama 5 detik setelah move

---

**US-04** — Sebagai **Member**, saya ingin berkomentar pada card agar diskusi tetap terpusat.

**Acceptance Criteria:**

1. Komentar muncul realtime tanpa refresh halaman
2. Support markdown: bold, italic, code block
3. Dapat mention `@username` dengan autocomplete
4. Edit & hapus komentar sendiri dalam batas 1 jam

---

**US-05** — Sebagai **Viewer**, saya ingin melihat progress board tanpa mengubah apapun.

**Acceptance Criteria:**

1. Tidak ada tombol edit, drag & drop di-disable
2. Dapat melihat semua card dan komentar
3. Dapat download attachment
4. Tidak dapat invite member atau ubah role

---

## 8. Milestones & Timeline

> Diasumsikan alokasi waktu ~3–4 jam per hari untuk developer yang belajar sambil membangun.

| Minggu | Milestone        | Deliverable                                           | Konsep Baru                                |
| :----: | ---------------- | ----------------------------------------------------- | ------------------------------------------ |
| **1**  | Fondasi & Auth   | Project setup, Identity, layout, DB migration         | DI, EF Core, Identity, Middleware          |
| **2**  | Project & Board  | CRUD proyek, board view statis, member invite         | Routing, EditForm, Service pattern, Email  |
| **3**  | Card & Real-time | Card CRUD, drag & drop, SignalR komentar              | JS Interop, SignalR Hub, Cascading State   |
| **4**  | Polish & Deploy  | Attachment, activity log, notifikasi, deploy ke Azure | File upload, BackgroundService, Deployment |

---

## 9. Risiko & Mitigasi

| Risiko                                          |   Level   | Mitigasi                                                                                  |
| ----------------------------------------------- | :-------: | ----------------------------------------------------------------------------------------- |
| SignalR tidak berfungsi di environment tertentu | 🟡 Medium | Test di localhost dulu. Fallback ke polling jika diperlukan.                              |
| Drag & drop JS Interop kompleks                 |  🔴 High  | Gunakan SortableJS yang sudah proven. Mulai tanpa realtime, tambahkan SignalR setelahnya. |
| EF Core N+1 query problem                       | 🟡 Medium | Gunakan `.Include()` secara eksplisit, aktifkan EF Core logging di development mode.      |
| Scope creep — fitur melebar                     |  🔴 High  | Pegang ketat daftar in-scope. Fitur di luar PRD ini masuk backlog fase 2.                 |
| File storage management                         |  🟢 Low   | Mulai dengan local disk, abstraksi ke `IStorageService` agar mudah migrasi ke Azure Blob. |

---

## 10. Appendix — Referensi Teknis

### 10.1 Blazor Concepts Coverage Map

| Fitur              | Konsep Utama                                        | File / Komponen                              |
| ------------------ | --------------------------------------------------- | -------------------------------------------- |
| Login & Register   | `EditForm`, `DataAnnotations`, `CascadingAuthState` | `LoginPage.razor`, `RegisterPage.razor`      |
| Kanban Board       | Component composition, `RenderFragment`, `@key`     | `BoardPage.razor`, `ColumnComponent.razor`   |
| Drag & Drop        | JS Interop, `IJSRuntime`, `DotNetObjectReference`   | `DragDropService.cs`, `sortable-interop.js`  |
| Real-time Komentar | SignalR Hub, `OnInitializedAsync`, `IDisposable`    | `BoardHub.cs`, `CardModal.razor`             |
| Role-based UI      | `AuthorizeView`, `IAuthorizationService`, Policy    | `Shared/AuthorizeView`, `Policies.cs`        |
| File Upload        | `IBrowserFile`, `IFormFile`, streaming              | `AttachmentService.cs`, `FileUpload.razor`   |
| Global State       | `CascadingValue`, scoped service                    | `ProjectStateService.cs`, `MainLayout.razor` |
| Email Notifikasi   | `IHostedService`, MailKit, background queue         | `EmailService.cs`, `NotificationWorker.cs`   |

---

### 10.2 API Endpoints (Minimal API)

TaskFlow menggunakan Blazor Server untuk UI. Beberapa endpoint Minimal API diperlukan untuk file download dan health check.

| Method | Endpoint                   | Auth   | Deskripsi                                  |
| ------ | -------------------------- | ------ | ------------------------------------------ |
| `GET`  | `/health`                  | Public | Health check endpoint untuk monitoring     |
| `GET`  | `/api/attachments/{id}`    | User   | Download file attachment dengan auth check |
| `GET`  | `/api/exports/{projectId}` | Owner  | Export project data ke JSON                |
| `WS`   | `/hubs/board`              | User   | SignalR Hub endpoint untuk real-time board |

---

### 10.3 Environment & Configuration

```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=taskflow.db"
  },
  "SignalR": {
    "MaximumReceiveMessageSize": 32768
  },
  "Storage": {
    "Provider": "Local",
    "LocalPath": "wwwroot/uploads"
  },
  "Email": {
    "Provider": "Smtp",
    "SmtpHost": "localhost",
    "SmtpPort": 1025
  }
}
```

---

> _TaskFlow PRD v1.0 · Dokumen ini bersifat living document dan dapat diperbarui seiring perkembangan proyek._
