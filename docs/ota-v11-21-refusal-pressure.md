#
#                 █████
#                ░░███
#        ██████  ███████    ██████
#       ███░░███░░░███░    ░░░░░███
#      ░███ ░███  ░███      ███████
#      ░███ ░███  ░███ ███ ███░░███
#      ░░██████   ░░█████ ░░████████
#       ░░░░░░     ░░░░░   ░░░░░░░░
#
#    Copyright (C) 2026 — 2026, Ota. All Rights Reserved.
#
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
#
#        http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.
#
# Ota V11.21 Unsupported-Capability Pressure: Buzz

This branch pressure-tests Ota's stock `oci_local` sandbox against Buzz's real integration-test
closure. It is a refusal proof, not an attempt to make Buzz run inside stock Ota sandboxing.

## Selected Lane

`just test-integration` delegates to `scripts/run-tests.sh integration`. Before it runs tests, the
helper invokes `_ensure-migrations`, which starts the Compose topology, may pull images, runs
Postgres migrations, and seeds local state. The full Compose file also owns Redis, MinIO, Keycloak,
Adminer, Prometheus, `buzz-net`, and persistent Postgres, MinIO, and Prometheus volumes.

The contract preserves those requirements through declared services, Cargo hydration, tool
requirements, adapter state, external state, and the `target` compiler output. Its `linux/amd64`
ephemeral container lane is deliberately selected by agent mode.

## Expected Result

Stock `oci_local` cannot place typed Cargo hydration, required Compose services, or task
requirements inside its first enforced command segment. Both commands below must refuse with
`execution_started: false` before `just`, Docker Compose, migration, seed, dependency hydration,
or hooks begin:

```bash
ota run test:integration --agent --sandbox-target oci_local
ota up --workflow integration --agent --sandbox-target oci_local
```

The hosted matrix snapshots Buzz-labelled containers, `buzz-net`, Buzz volumes, repository status,
and dependency/output paths before and after the requests. It also records Docker events. Any Buzz
resource, `.env`, dependency tree, compiler output, or repository write is a pressure failure.

## Evidence Boundary

`ota run` provides the human refusal; `ota run --dry-run --json` provides its machine-readable
admission record. `ota up --receipt --json` provides an inline blocked receipt with
`execution_attempted: false`. Current Ota does not archive a pre-boundary sandbox refusal, so the
matrix separately validates the ordinary native readiness archive. On a clean hosted runner that
archive is expected to retain the same missing-`just` readiness failure; it does not call that
archive evidence of the refusal. Durable refusal archival remains an Ota audit-evidence gap.

`ota doctor --workflow integration --mode native` is intentionally a non-provider diagnosis for
this pass. A clean hosted runner is expected to report the declared `just` tool as missing; the
matrix does not install it because doing so would be host mutation outside the refusal proof.
Default container diagnosis may probe its declared image, which is a separate provider operation
and cannot demonstrate refusal-before-provider-mutation.

## Uncovered Material Behavior

| Behavior | Classification | Boundary |
| --- | --- | --- |
| Cargo dependency hydration | Explicitly bounded | Declared, but stock `oci_local` refuses typed preparation before it runs. |
| Docker Compose services and image acquisition | Explicitly bounded | Declared by service and adapter-state truth; stock `oci_local` refuses required services before Docker mutation. |
| Postgres migrations and local seed data | Repo-owned outside selected Ota scope | Performed by Buzz's `_ensure-migrations` helper after service startup; never started in this refusal pass. |
| Redis, MinIO/S3, Keycloak, Adminer, Prometheus, network, and named volumes | Explicitly bounded | Declared topology and external/adapter state; not started or observed ready. |
| Integration test task and any conditional/outcome hook | Contract-owned and proved absent | Refusal JSON reports no execution start; mutation guards confirm no task-owned output. |
| Desktop, relay, mobile, production deployment, credentials, and relay federation | Repo-owned outside selected scope | Not selected by this narrow contract. |
| Durable archive for an inline sandbox refusal | Named Ota platform gap | Current Core archives ordinary receipts, but not a pre-boundary sandbox-refusal receipt. |

This pass proves only selected-lane refusal on Linux/amd64. It does not prove Buzz application
correctness, service readiness, deployment safety, or repository-global governance.
