# ADR 0001 — `IMPLEMENTATION_STATUS.md` was never true of this tree

- **Status:** accepted
- **Date:** 2026-08-30
- **Supersedes:** nothing. This is the first ADR in this repository.

## Context

`IMPLEMENTATION_STATUS.md` opened with a section titled `## ✅ Completed
Tasks` and described three finished Go components — a Tesseract-backed OCR
engine at `internal/ocr/engine.go`, an OpenCV liveness detector with
anti-spoofing at `internal/liveness/detector.go`, and an integrated verifier
at `internal/verification/verifier.go` — down to function names, dependency
versions and the tessdata paths they load. Its security checklist marked
five controls done:

```
- [x] Clerk JWT authentication
- [x] Org-scoped access control
- [x] Admin role verification
- [x] JWKS rotation (1 hour)
- [x] service mesh mTLS
```

`README.md` matched it, documenting performers at `legacy-runtime/`
and `cdn/`, a service table for `ekyc.etzhayyim.com`, and deployment via
`buf generate` and `mage Deploy`.

None of it is in this repository. The tree was 22 files when this was
written — metadata, four READMEs, a ClojureScript scaffold that renders a
heading and a status line, and a TypeScript package whose `package.json`
sets `"main": "src/app.ts"` next to no `src/` directory. This ADR and the
quickstart it cites bring it to 25.

Three independent pieces of evidence establish that this is not deletion —
the code was never here:

1. **The repository's own machine-readable metadata already says so.**
   `PROJECT.jsonld` carries seven `tasks`, and **all seven are
   `"status": "pending"`** — including *"Create ekyc-service gRPC backend
   with Clerk JWT validation"*, *"Implement proto/v1/ekyc.proto"*, and
   *"Implement document upload, face liveness, verification logic"*. The two
   documents contradicted each other, and the machine-readable one was
   right.
2. **The extraction manifest is exact and was honoured.** `migration.edn`
   records the source as `etzhayyim/root` path
   `60-apps/etzhayyim-project-ekyc`, tree
   `57e392fedecf2e7f44d79a717804e98a79ed7112`, `:tracked-files 23`. Reading
   that tree back out of GitHub returns 23 blobs, and they are the files we
   have (plus the Svelte scaffold since migrated to ClojureScript). No Go,
   no proto, no manifests.
3. **The implementation is absent from the source repository too.** At
   source revision `089210a043264dc53659c1187a67ea4a8ffe292f`, every path in
   `etzhayyim/root` matching `ekyc` is one of those same 23 files. The
   extraction was faithful; the claims were already aspirational upstream.

The consequence is not cosmetic. `OWNERS` lists
`etzhayyim-security-team` and `etzhayyim-compliance-team` as reviewers, and
this workspace maintains a SOC 2 / ISO 27001 control crosswalk. A reader
arriving at a checklist that says mTLS and JWT authentication are done has
been told that controls exist. Nothing in the repository contradicted them
except a JSON-LD file nobody reads.

## Decision

**Correct the documents in place, and keep the design.**

- `IMPLEMENTATION_STATUS.md` states, at the top, that nothing in it is
  implemented in this tree, and gives the evidence above. The `✅` markers
  and the `[x]` security checkboxes are removed. The engineering content —
  which parsers, which anti-spoofing techniques, which state transitions —
  is retained as the design it always was, under headings that say so.
- `README.md` leads with what the tree contains. The DoDAF architecture
  stays, marked as designed-not-built, with the missing paths named.
- `docs/operator-quickstart.md` records the commands that were actually run,
  their real output, and — separately and explicitly — the things it did not
  verify.

## Alternatives rejected

- **Delete the aspirational documents.** They are the only surviving design
  record for this project. Deleting them would lose the OCR parser
  breakdown, the anti-spoofing approach and the eKYC → AML/CTF state
  machine, and would leave a reader unable to tell whether an
  implementation had once existed. The defect is the tense, not the content.
- **Leave them and add a disclaimer at the top.** `README.md` already had
  one, added during the Svelte→ClojureScript migration, and it did not work:
  the nine kilobytes below it still read as description, and the security
  checkboxes were untouched. A note that a document is wrong is weaker than
  the document being right.
- **Implement enough to make the claims true.** Out of scope, and it would
  invert the problem — the point is that the documents must track the tree,
  whichever way the gap is closed.

## Consequences

- A reader can now tell, from the first screen of any of these documents,
  that this repository implements no eKYC. That is a downgrade in apparent
  maturity and an upgrade in accuracy.
- The `[x]` security controls are gone. If those controls exist somewhere,
  they exist in a system that is not this repository, and claiming them here
  was attributing another system's posture to this one.
- `PROJECT.jsonld` needed no change. It was already correct, and it is now
  the cross-check the prose is measured against: if a future change marks a
  task complete there, the prose may say so too.
- `NOTICE` refers to `CHARTER-RIDER.md`, which is also absent from this
  tree. Recorded, not fixed — it is a licensing question, not a status one.
