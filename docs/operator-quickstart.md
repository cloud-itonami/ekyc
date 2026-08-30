# Operator quickstart

**Every command in this file was run, in this order, on 2026-08-30 — first
on the tree these docs were written in, then again in a fresh
`git clone` of the pushed branch, which reproduced each result.** The
release build was byte-identical across the two (`app.js`, 359,412 bytes).
The output quoted under each step is what it actually printed. If a step is
not listed here, this document does not claim it works — see [What this document does not
verify](#what-this-document-does-not-verify) at the bottom.

Read [What is actually in this tree](../README.md#what-is-actually-in-this-tree)
first. This repository is a scaffold: it builds, its tests pass, and it
implements no eKYC. The quickstart below is therefore short by nature, not
by omission.

## Prerequisites

| Tool | Used by | Version this was walked with |
|---|---|---|
| Node.js + npm | both packages | node 26.7.0 |
| `clojure` CLI + a JDK | the ClojureScript build (shadow-cljs shells out to it) | whatever `shadow-cljs` resolves |

Network access is required the first time: npm fetches packages, and the
ClojureScript build resolves a Maven and a git dependency
(`jp-go-digital-design-system`, pinned in `appview/ekyc-mcp-component/cljs/deps.edn`).

## 1. The MCP facade package (TypeScript)

```bash
cd appview/ekyc-mcp-component
npm install          # added 44 packages
npm test             # vitest
```

`npm install` here rewrites `package-lock.json` (+929 / −226 lines,
reproducibly). Your tree will be dirty afterwards; that change is npm
re-resolving the committed lockfile, not something you did.

Observed:

```
 Test Files  1 passed (1)
      Tests  1 passed (1)
```

**Read that number before you trust it.** The one test is
`test/ekyc.test.ts`, and its body is `expect(true).toBe(true)`. It passes on
every tree, including a tree with nothing in it — which is the tree you
have. `package.json` declares `"main": "src/app.ts"`, and **`src/` does not
exist**, so there is no MCP facade here to test. A green suite in this
package currently carries no information about the product.

## 2. The appview (ClojureScript)

```bash
cd appview/ekyc-mcp-component/cljs
npm install          # added 129 packages
npm test             # shadow-cljs compile test && node out/tests.js
```

Observed:

```
Ran 4 tests containing 6 assertions.
0 failures, 0 errors.
```

Compilation is 112 files / 111 compiled. How long that takes is dominated by
what is already in your `~/.m2` and git dependency caches, not by this
repository: **101.89 s** on a machine resolving them for the first time,
**9.35 s** in a fresh clone on the same machine once they were warm. Both
runs were on a workstation at load average ~49, so treat these as the shape
of the cost, not as a benchmark.

These four tests are real: they cover `:initialize-db` and the `:heading` /
`:message` subscriptions in `src/ekyc/app.cljs`, and they read the db rather
than asserting fixed strings, so they fail if the event or the subs are
changed. They cover a two-line scaffold view — that is the whole of what
this appview does today.

The `re-frame: Subscribe was called outside of a reactive context` lines
printed during the run come from subscribing directly in a test rather than
inside a component. They are warnings, not failures.

## 3. Build the appview bundle

```bash
cd appview/ekyc-mcp-component/cljs
npm run release      # rm -rf public/js && shadow-cljs release app
```

Observed: `[:app] Build completed. (110 files, 39 compiled, 0 warnings)` —
171.24 s cold, 31.54 s warm, same caveat as above — producing

```
public/js/app.js         359412 bytes
public/js/manifest.edn     3012 bytes
```

`public/index.html` is a single document that loads exactly that one bundle
and mounts it into `<div id="app">`, which is the element
`ekyc.app/main` calls `getElementById` on. One document, one bundle, one
mount — the workspace single-page-app rule.

## 4. Serve it

```bash
cd appview/ekyc-mcp-component/cljs/public
python3 -m http.server 8731
# then open http://127.0.0.1:8731/index.html
```

The server was started and stopped as written. **The rendered result was not
verified** — see below.

## What this document does not verify

Written down rather than left out, because a step that was skipped and a
step that passed look identical in a document that only lists successes.

- **The page rendering in a browser.** Headless Chrome was attempted twice
  on the authoring machine (`--headless` and `--headless=new`, via the
  Chrome for Testing build) and both runs produced an empty DOM dump and had
  to be killed at the timeout; the log shows repeated
  `CVDisplayLinkCreateWithCGDisplay failed`, and the machine was at load
  average 49. That is a fault of the authoring machine, not evidence about
  this page. What *is* verified is one step short of a render: the bundle is
  produced, and its mount id matches the one the code looks up. Treat step 4
  as unconfirmed until someone opens it.
- **Any deployment.** `README.md` describes `buf generate` and `mage Deploy`
  against `legacy-runtime/` and `cdn/`. None of those directories, and
  neither of those tools, are present. Those commands were not run and
  cannot be.
- **Anything eKYC.** No document upload, OCR, liveness check, Clerk
  authentication, MCP call, or APQC workflow exists in this tree to exercise.
  `PROJECT.jsonld` lists all seven of those as `"status": "pending"`, and
  that is accurate.
