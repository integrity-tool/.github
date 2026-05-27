# IntegrityTool — Healthcare EDI X12 837 Platform

> End-to-end healthcare claims processing system — X12 837 parsing · CMS-1500 mapping · Admin dashboard · Audit logging

[![Java](https://img.shields.io/badge/Java-17-ED8B00?style=flat-square&logo=java)](https://github.com/integrity-tool/IntegrityToolBackend)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6?style=flat-square&logo=typescript)](https://github.com/integrity-tool/IntegrityToolUI)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.2-6DB33F?style=flat-square&logo=spring-boot)](https://github.com/integrity-tool/IntegrityToolBackend)
[![Angular](https://img.shields.io/badge/Angular-17-DD0031?style=flat-square&logo=angular)](https://github.com/integrity-tool/IntegrityToolUI)

---

## What is IntegrityTool?

IntegrityTool is a full-stack healthcare EDI processing platform that parses raw X12 837 claim files, maps them to CMS-1500 form fields, and exposes them through both a patient-facing portal and a secure admin dashboard — all with end-to-end audit logging.

**The problem it solves:** X12 837 is the standard EDI format for submitting healthcare insurance claims in the US and Canada. It is notoriously difficult to parse — cryptic 2-character segment identifiers, nested loop structures, and qualifier-driven polymorphism. IntegrityTool handles all of that and turns raw EDI into clean, queryable, auditable claim records.

---

## System architecture

```
                         ┌─────────────────────────────────────────────┐
                         │              IntegrityTool Platform          │
                         └─────────────────────────────────────────────┘

  ┌──────────────────┐        ┌──────────────────┐
  │  IntegrityToolUI │        │ IntegrityToolAdmin│
  │  (User Portal)   │        │ UI (Admin Panel) │
  │                  │        │                  │
  │  • Submit claims │        │  • Claim review  │
  │  • Track status  │        │  • Audit logs    │
  │  • View EOB      │        │  • User mgmt     │
  │  TypeScript /    │        │  • Analytics     │
  │  Angular         │        │  TypeScript /    │
  └────────┬─────────┘        │  Angular         │
           │ REST API         └────────┬─────────┘
           │                          │ REST API
           ▼                          ▼
  ┌──────────────────┐        ┌──────────────────┐
  │IntegrityTool     │        │IntegrityTool     │
  │Backend           │        │Admin             │
  │(User API)        │        │(Admin API)       │
  │                  │        │                  │
  │  • X12 837       │        │  • Claim mgmt    │
  │    parsing       │        │  • Audit trail   │
  │  • CMS-1500      │        │  • Reporting     │
  │    mapping       │        │  • User roles    │
  │  • Validation    │        │  • Analytics     │
  │  Java /          │        │  Java /          │
  │  Spring Boot     │        │  Spring Boot     │
  └────────┬─────────┘        └────────┬─────────┘
           │                          │
           └──────────┬───────────────┘
                      │
                      ▼
           ┌─────────────────────┐
           │     PostgreSQL       │
           │                     │
           │  • claims           │
           │  • service_lines    │
           │  • audit_logs       │
           │  • users            │
           └─────────────────────┘
```

---

## Repositories

| Repo | Description | Stack |
|---|---|---|
| [IntegrityToolBackend](https://github.com/integrity-tool/IntegrityToolBackend) | User-facing REST API — X12 837 parsing, CMS-1500 mapping, claim submission | Java · Spring Boot · PostgreSQL |
| [IntegrityToolUI](https://github.com/integrity-tool/IntegrityToolUI) | Patient portal — claim submission, status tracking, EOB viewer | Angular · TypeScript |
| [IntegrityToolAdmin](https://github.com/integrity-tool/IntegrityToolAdmin) | Admin REST API — claim management, audit trail, reporting, user roles | Java · Spring Boot · PostgreSQL |
| [IntegrityToolAdminUI](https://github.com/integrity-tool/IntegrityToolAdminUI) | Admin dashboard — claim review, analytics, audit logs, user management | Angular · TypeScript |

---

## Tech stack

| Layer | Technology |
|---|---|
| Backend language | Java 17 |
| Backend framework | Spring Boot 3.2 |
| API documentation | SpringDoc OpenAPI (Swagger UI) |
| Frontend language | TypeScript 5 |
| Frontend framework | Angular 17 |
| UI components | Angular Material |
| Database | PostgreSQL |
| Testing | JUnit 5 · Mockito · Jasmine |
| Build tools | Maven · Angular CLI |
| Version control | Git · GitHub |

---

## X12 837 parsing — how it works

```
Raw EDI file
    │
    │  ISA*00*...*ZZ*SENDER*ZZ*RECEIVER*...*~
    │  ST*837*0001*005010X222A1~
    │  NM1*85*2*CLINIC NAME*****XX*1234567890~
    │  CLM*CLAIM001*150***11:B:1*Y*A*Y*I~
    │  HI*ABK:Z23.0~
    │  SV1*HC:99213*85*UN*1***1~
    │
    ▼
┌─────────────────────────┐
│  X12 Tokenizer           │  Split on ~ (segment) and * (element)
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Loop State Machine      │  Detect 2000A → 2000B → 2300 → 2400
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Segment Handlers        │  NM1 · CLM · HI · SV1 · DTP
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  CMS-1500 Mapper         │  Maps to all 33 form boxes
└────────────┬────────────┘
             ▼
      ClaimRecord (POJO)
      + AuditLog entry
      → stored in PostgreSQL
```

---

## Key features

**Claim processing**
- Full X12 837P (Professional) parsing across all standard loops and segments
- CMS-1500 field mapping for all 33 form boxes
- Segment-level validation with detailed error reporting
- Batch ingestion for multi-claim 837 files

**Admin dashboard**
- Real-time claim status overview and queue management
- Filterable audit log — every parse, validation, and status change tracked
- Role-based access control (admin / reviewer / read-only)
- Analytics — claim volume, error rates, processing time trends

**Engineering**
- Clean separation of concerns across 4 independent services
- Shared PostgreSQL instance with schema-level isolation
- JWT authentication across both backend services
- Swagger UI on both APIs for easy integration testing

---

## Getting started

Clone all four repos:

```bash
git clone https://github.com/integrity-tool/IntegrityToolBackend.git
git clone https://github.com/integrity-tool/IntegrityToolUI.git
git clone https://github.com/integrity-tool/IntegrityToolAdmin.git
git clone https://github.com/integrity-tool/IntegrityToolAdminUI.git
```

See the README in each repository for individual setup instructions.

---

## Author

**Krishna Solanki** — Full-stack developer, Montreal QC

[github.com/krishnasolanki](https://github.com/krishnasolanki) · [linkedin.com/in/krishnasolanki](https://linkedin.com/in/krishnasolanki) · krishnasolanki120@gmail.com
