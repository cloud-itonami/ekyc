# etzhayyim-project-ekyc

eKYC (electronic Know Your Customer) — Clerk-integrated identity
verification with MCP integration for APQC 12.4.5 KYC workflows.

**This repository implements none of that yet.** Read the next section
before the architecture below it.

## What is actually in this tree

25 tracked files. Two runnable packages, neither of which does any identity
verification:

| Path | What it is | State |
|---|---|---|
| `appview/ekyc-mcp-component/` | TypeScript package for the MCP JSON-RPC facade | `package.json` sets `"main": "src/app.ts"`; **`src/` does not exist**. Its one test is `expect(true).toBe(true)` |
| `appview/ekyc-mcp-component/cljs/` | ClojureScript appview — shadow-cljs + reagent + re-frame + `jp-go-dds` | Builds and runs. Renders one heading and one status line. 4 tests, 6 assertions, green |

Everything else is metadata (`PROJECT.jsonld`, `README.edn`,
`kotodama.jsonld`, `NOTICE`, `OWNERS`, `migration.edn`) and documentation.

To build and test it, see [`docs/operator-quickstart.md`](docs/operator-quickstart.md),
which lists only commands that were actually run and what they printed.

### What is designed but not built

The architecture below describes a system that does not exist in this
repository, at paths that are not in it:

- `legacy-runtime/ekyc-service-ephj2jf6/` — absent
- `cdn/ekyc-ui-hzaooy0f/` — absent
- `internal/ocr/`, `internal/liveness/`, `internal/verification/`,
  `internal/server/` — absent
- `proto/v1/ekyc.proto` — absent
- `wasm/` — absent

The deployment commands in [Deployment](#deployment) (`buf generate`,
`mage Deploy`) operate on those directories and cannot be run here.

This is not deletion. `PROJECT.jsonld` in this directory lists all seven
build tasks as `"status": "pending"`, and the extraction manifest
`migration.edn` records that 23 files were brought from `etzhayyim/root` —
which is the set present. The design was never implemented at the source
either. Evidence:
[ADR 0001](docs/adr/0001-implementation-status-was-never-true-of-this-tree.md).

`IMPLEMENTATION_STATUS.md` previously presented parts of this as complete,
including five ticked security controls. It has been corrected; see the
same ADR.

### Frontend stack

The appview was migrated from Svelte 5 + Vite to ClojureScript on
2026-08-26 (workspace standard: cljs + reagent + re-frame + `jp-go-dds`).
The port was faithful and functionality-preserving — the Svelte scaffold
rendered a heading and a status paragraph, and the ClojureScript version
renders the same. It added no eKYC behaviour, because there was none to
carry over.

---

**Everything from here down is the original design document, retained as
the only surviving record of the intended system. Read it in the future
tense.**

## Architecture (DoDAF v2)

### Capability (CV-1)
**eKYC Verification** — Electronic identity verification capability integrating Clerk authentication with KYC document validation and MCP-enabled workflow automation (APQC 12.4.5).

### Activities (OV-5b)
- Identity document upload and validation (OCR, authenticity checks)
- Face liveness detection (gesture-based)
- Clerk user authentication and org-scoped verification
- MCP-enabled APQC workflow integration
- Verification status tracking and reporting

### Performers (OV-2)
- **ekyc-service** (`legacy-runtime/ekyc-service-ephj2jf6/`) — XRPC MCP service (Tier 2, Scale-to-Zero)
- **ekyc-ui** (`cdn/ekyc-ui-hzaooy0f/`) — SvelteKit static UI (Tier 3)
- **APQC 12.4.5 KYC performer** — MCP integration target (`etzhayyim-performer-sys-etzhayyim-actors-pba7d22f-svc-apqc-12-4-5-kyc-v2`)

### Services (SvcV-1)

Neither service is deployed; the host below is NXDOMAIN as of 2026-08-30.

| Service | Type | Protocol | URL |
|---|---|---|---|
| ekyc-service | XRPC MCP | XRPC (HTTP/2) | `ekyc.etzhayyim.com` |
| ekyc-ui | Static UI | HTTP/1.1 | `ekyc.etzhayyim.com` |

### Resource Flows (OV-2)
```
User (Browser) → Clerk Auth → ekyc-ui (SvelteKit)
                                    ↓ XRPC (Connect-gRPC Bridge)
                              ekyc-service (App runtime)
                                    ↓ App state store
                              PostgreSQL (org-statestore)
                                    ↓ MCP CallTool
                              APQC 12.4.5 KYC performer
```

## Components

### legacy-runtime/ekyc-service-ephj2jf6 (not present)

Designed as a gRPC eKYC service with Clerk JWT validation and MCP integration.

**Features:**
- Clerk JWKS-based JWT authentication
- Document upload and validation (OCR placeholder)
- Face liveness check (gesture-based)
- App state store integration (PostgreSQL)
- MCP service implementation (Initialize, ListTools, CallTool, ListResources, ReadResource)
- APQC 12.4.5 KYC workflow integration via MCP CallTool

**Proto:** `proto/v1/ekyc.proto`
- `EKYCService` — SubmitVerification, GetVerificationStatus, ListVerifications, UpdateVerificationStatus, InitiateLivenessCheck, SubmitLivenessCheck
- `MCPService` — Initialize, ListTools, CallTool, ListResources, ReadResource

**Environment:**
```bash
CLERK_JWKS_URL=https://clerk.etzhayyim.com/.well-known/jwks.json
POSTGRES_STATE_STORE=org-statestore
MCP_APQC_KYC_ENDPOINT=etzhayyim-performer-sys-etzhayyim-actors-pba7d22f-svc-apqc-12-4-5-kyc-v2:8080
APP_GRPC_ENDPOINT=http://shared-ekyc-service-legacy-runtime:50001
```

**Ports:**
- 8080: XRPC server
- 9090: Prometheus `/metrics`

### cdn/ekyc-ui-hzaooy0f (not present)

Designed as an eKYC frontend with Clerk authentication.

**Features:**
- Clerk JS SDK authentication
- Document upload UI (front/back images)
- Liveness check UI (face image + gestures)
- Verification status tracking
- MCP workflow status display

**Tech stack, as designed:**
- Clerk JS SDK
- Connect-gRPC web client

The ClojureScript appview that *does* exist,
`appview/ekyc-mcp-component/cljs/`, is not this component. It shares the
stack the workspace standardises on (shadow-cljs + reagent + re-frame +
`jp-go-dds`, migrated from Svelte on 2026-08-26) and none of the features
listed above.

**URL, as designed:** `https://ekyc.etzhayyim.com` — **does not resolve**
(NXDOMAIN, checked 2026-08-30). The service table above names the same host.

## Deployment

**Not runnable in this tree** — the directories and the `buf` / `mage`
toolchain these steps use are absent. For what does run, see
[`docs/operator-quickstart.md`](docs/operator-quickstart.md).

### Deploy ekyc-service (XRPC backend)

```bash
cd 60-apps/etzhayyim-project-ekyc/legacy-runtime/ekyc-service-ephj2jf6/

# Generate proto code (Go + TypeScript)
buf generate

# Deploy to K8s via current mage flow
mage Deploy
```

**Generated Resources:**
- Deployment: `ekyc-service`
- Service: `ekyc-service` (ClusterIP)
- GRPCRoute: `ekyc.etzhayyim.com` → `ekyc-service:8080`
- KEDA ScaledObject: HTTP trigger, 0-5 replicas
- App runtime Pulumi Application: `shared-ekyc-service`

### Deploy ekyc-ui (Static frontend)

```bash
cd appview/ekyc-mcp-component/cljs/

# Install dependencies
npm install

# Build (shadow-cljs release, output to public/js/)
npm run release

# Deploy to K8s via current mage flow
mage Deploy
```

**Generated Resources:**
- Deployment: `ekyc-ui` (nginx)
- Service: `ekyc-ui` (ClusterIP)
- HTTPRoute: `ekyc.etzhayyim.com` → `ekyc-ui:80`
- KEDA HTTPScaledObject: 0-3 replicas

## MCP Integration with APQC 12.4.5 KYC

ekyc-service integrates with APQC 12.4.5 KYC performer via MCP:

1. **Submit Verification** → Calls APQC KYC performer's `CallTool` RPC
   - Tool: `submit_kyc_verification`
   - Arguments: `{ verification_id, user_id, org_id, document_info }`
   - Returns: `{ workflow_id }`

2. **Get Workflow Status** → Calls APQC KYC performer's `ReadResource` RPC
   - Resource: `apqc://kyc/workflows/{workflow_id}`
   - Returns: `{ status, steps, last_updated }`

3. **List Verifications** → Exposes as MCP Resource
   - Resource: `ekyc://verifications`
   - Consumers: APQC performers, monitoring tools

## Authentication Flow

1. User signs in via Clerk (`ekyc-ui` → `clerk.etzhayyim.com`)
2. Clerk issues JWT with claims: `{ sub, org_id, org_slug, org_role, email }`
3. ekyc-ui sends XRPC requests with `Authorization: Bearer <jwt>`
4. ekyc-service validates JWT via JWKS (`clerk.etzhayyim.com/.well-known/jwks.json`)
5. ekyc-service extracts `user_id` and `org_id` from JWT claims
6. All operations are org-scoped (user can only view their org's verifications)

## Verification Flow

1. **Document Upload** (UI)
   - User selects document type (Passport, Driver's License, etc.)
   - Uploads front + back images (base64)
   - Enters document number, issuing country, expiry date

2. **Liveness Check** (UI)
   - UI calls `InitiateLivenessCheck` → receives `session_id`
   - User performs gestures (nod, smile, turn head)
   - UI captures face image + gesture sequence
   - UI calls `SubmitLivenessCheck` → receives liveness result

3. **Submit Verification** (Backend)
   - ekyc-service receives `SubmitVerificationRequest`
   - Saves to App state store (`ekyc:verification:{id}`)
   - Starts background verification:
     - Check 1: Image quality
     - Check 2: Document authenticity (placeholder)
     - Check 3: OCR extraction (placeholder)
     - Check 4: Expiry validation
   - Triggers APQC KYC workflow via MCP
   - Returns `verification_id` + `mcp_workflow_id`

4. **Status Tracking** (UI)
   - UI polls `GetVerificationStatus` with `verification_id`
   - Backend returns:
     - Verification status (PENDING, PROCESSING, APPROVED, REJECTED, REQUIRES_REVIEW)
     - Check results
     - MCP workflow status

## State Store Schema

### Verification Record

```json
{
  "verification_id": "abc123",
  "user_id": "user_xyz",
  "org_id": "org_abc",
  "status": "VERIFICATION_STATUS_PROCESSING",
  "message": "Processing verification",
  "document_info": {
    "document_type": "DOCUMENT_TYPE_PASSPORT",
    "document_number": "AB123456",
    "issuing_country": "JP",
    "expiry_date": "2030-12-31T00:00:00Z"
  },
  "liveness_result": {
    "status": "LIVENESS_STATUS_PASS",
    "confidence_score": 0.92,
    "message": "Liveness check passed",
    "checked_at": "2026-02-17T10:00:00Z"
  },
  "checks": [
    {
      "check_name": "image_quality",
      "status": "CHECK_STATUS_PASS",
      "message": "Image quality verification",
      "checked_at": "2026-02-17T10:00:00Z"
    }
  ],
  "created_at": "2026-02-17T09:00:00Z",
  "updated_at": "2026-02-17T10:00:00Z",
  "metadata": {},
  "mcp_workflow_id": "kyc-wf-abc123",
  "admin_notes": ""
}
```

**State Key:** `ekyc:verification:{verification_id}`

### Liveness Session

```json
{
  "session_id": "sess_abc",
  "user_id": "user_xyz",
  "org_id": "org_abc",
  "session_token": "token_xyz",
  "expires_at": "2026-02-17T10:05:00Z",
  "created_at": "2026-02-17T10:00:00Z"
}
```

**State Key:** `ekyc:liveness:{session_id}`

## Measures (StdV-1)

- Verification success rate (target: >95%)
- Processing time (target: <30s for auto-approval)
- KEDA scale-to-zero efficiency (target: <5s cold start)
- MCP workflow completion rate (target: >99%)

## Security, as designed

**None of these is implemented in this repository**, which contains no
authentication, authorisation or transport security of any kind. They are
requirements on the system described above, not properties of this tree,
and must not be cited as operating controls. See
[ADR 0001](docs/adr/0001-implementation-status-was-never-true-of-this-tree.md).

- **Clerk JWT authentication** — every XRPC request to require a valid Clerk JWT
- **Org-scoped access** — users to see only their own org's verifications
- **Admin role check** — `UpdateVerificationStatus` to require `org:admin`
- **JWKS rotation** — JWKS keys to refresh hourly
- **Service mesh mTLS** — inter-component traffic to be encrypted
- **KEDA scale-to-zero** — to reduce attack surface when idle

## Next Steps

1. Integrate real OCR service (Google Vision API, AWS Textract)
2. Integrate real liveness detection (FaceTec, iProov)
3. Implement document authenticity checks (hologram detection)
4. Complete APQC KYC performer integration (real MCP CallTool)
5. Add admin dashboard for manual review
6. Add audit logging for compliance
7. Add webhook notifications for status updates
