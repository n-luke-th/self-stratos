# Stratos (STR): LukeCreated Assets manager (S3 Object Manager (Serverless + CloudFront))

A minimal, production-correct, serverless system for managing private S3 objects and exposing them securely via CloudFront.

Designed as a cost-efficient, extensible foundation suitable for real-world deployment and long-term expansion.

## 🎯Stage 1 Objective

Build a clean, near-zero-cost architecture that:

- Manages private objects in Amazon S3
- Exposes read-only access through CloudFront
- Generates deterministic CDN URLs
- Supports authenticated upload & delete
- Sends transactional emails via Amazon SES
- Is fully serverless
- Is designed for future multi-bucket and microservice expansion

## 🏗 Architecture Overview

```text
VueJS (Client)
   |
   | Clerk JWT
   v
API Gateway (HTTP API)
   |
   v
Lambda (TypeScript)
   |
   ├── DynamoDB (metadata)
   ├── S3 (private storage)
   └── SES (email)

CloudFront (OAC)
   |
   v
Public Read-Only Access
```

## 🧱 Tech Stack

### Core Infrastructure

- Amazon S3 – Private object storage
- Amazon CloudFront – CDN with Origin Access Control (OAC)
- Amazon API Gateway (HTTP API) – REST layer
- AWS Lambda (TypeScript) – Control plane logic
- Amazon DynamoDB (On-Demand) – Metadata storage
- Amazon SES – Transactional email

### Application Layer

- VueJS – Frontend SPA
- Clerk – Authentication provider

## 🔐 Security Model

### Storage

- S3 bucket is private
- Public access block enabled
- No public ACLs
- CloudFront uses Origin Access Control (OAC)

### Authentication

- Clerk issues JWT
- Lambda verifies token
- All resources partitioned by userId

### Access Model

- Upload: Pre-signed S3 PUT URL
- Read: Deterministic CloudFront URL
- Delete: Authenticated API request

## 🌐 Deterministic CDN URL Strategy

CloudFront URLs are generated without runtime AWS calls.

Formula:

```url
https://<distribution-domain>/<path-prefix>/<object-key>
```

Example:

```url
https://cdn.example.com/users/abc123/images/uuid.png
```

This guarantees:

- Zero lookup overhead
- Zero additional cost
- Predictable behavior
- Easy multi-distribution expansion later

## 📦 Object Key Structure

All objects follow a strict naming convention:

```text
users/<userId>/<resourceType>/<uuid>.<ext>
```

Example:

```text
users/abc123/images/550e8400-e29b-41d4-a716-446655440000.png
```

Benefits:

- Multi-tenant isolation
- Lifecycle rule compatibility
- Easy future bucket split
- Clean migration path

## 🗄 Data Model (DynamoDB)

Single-table design.

Table: app-main

Partition Key:

```text
USER#<userId>
```

Sort Key:

```text
OBJECT#<uuid>
```

Metadata stored:

- bucketName
- objectKey
- contentType
- size
- createdAt
- status

## 🔌 API Endpoints

### Generate Upload URL

```text
POST /objects/upload-url
```

Returns:

- Pre-signed S3 PUT URL
- Object key
- Deterministic CDN URL

### Confirm Upload

```
POST /objects
```

Stores metadata.

### List Objects

```
GET /objects
```

Returns all objects for authenticated user.

### Delete Object

```
DELETE /objects/{id}
```

Deletes from S3 and soft-deletes metadata.

## ✉️ Email (Stage 1)

Transactional emails are sent directly using Amazon SES.

An internal abstraction layer ensures future compatibility with a dedicated email microservice.

## 📂 Repository Structure

```
apps/
  web/            # VueJS frontend
  api/            # Lambda handlers

packages/
  shared-types/   # Shared TS types
  email/          # SES email abstraction
  utils/

infrastructure/
  cdk/ or terraform/ # TODO: decide infras folder

docs/
  architecture.md
```

## 💰 Cost Model (Low Traffic)

| Service     | Estimated Monthly Cost |
| ----------- | ---------------------- |
| S3          | ~$0                    |
| CloudFront  | ~$0                    |
| Lambda      | ~$0                    |
| API Gateway | ~$0                    |
| DynamoDB    | ~$0                    |
| SES         | ~$0                    |
| **Total**   | **~$0–3**              |

Designed to scale from hobby-level to production without architectural rewrite.

## 🚀 Stage 2 Roadmap

Planned expansions:

- Multiple S3 buckets
- Multiple CloudFront distributions
- Private CDN signed URLs
- Dedicated email microservice (useSend + SES)
- Background job processing
- Advanced access control

_*Stage 1 decisions intentionally avoid blocking these upgrades.*_

## 🧠 Design Philosophy

- Serverless-first
- Cost discipline
- Deterministic infrastructure
- Clear separation of control plane and data plane
- No premature microservices
- Upgrade-ready without rewrite

## 📌 Why This Project Exists

This project is both:

- A practical object management system
- A portfolio demonstration of production-oriented serverless architecture

It focuses on correctness, security, and extensibility rather than feature bloat.

---

**last updated on Feb 14, 2026**
