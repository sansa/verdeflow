# VerdeFlow lab alignment review

**Audit date:** 4 November 2026
**Baseline:** `/Users/mbanyickjoof/Downloads/VERDEFLOW_LAB_SPEC.md`, version 1.0
**Scope:** Static inspection and local, non-hardware checks of this repository. No application code or running service was changed.

## Verdict

**Not ready to operate the specified shared remote physical measurement lab.** VerdeFlow has a useful prototype path for API runs, Git checkout, agent polling, software/on-board power sampling, PostgreSQL storage, and result display. The path does not establish a physical, off-target energy measurement. The first blockers are:

1. No commissioned instrument/channel/path registry or physical collector exists. `external_meter` returns an error, while PMIC and PowerJoular can silently become simulated data under their original labels ([profiler.rs](verde-flow-agent/src/profiler.rs#L100-L137), [profiler.rs](verde-flow-agent/src/profiler.rs#L205-L225)). PowerGoblin is absent.
2. The agent builds the workload and sends the requests from the same host ([runner.rs](verde-flow-agent/src/runner.rs#L77-L103), [runner.rs](verde-flow-agent/src/runner.rs#L137-L180)). There is no separate generator or logging-host coordination, so the intended energy boundary is not met.
3. A database job claim is atomic for one queue row, but there is no atomic reservation of the target **and** instrument/channel, analyzer, acquisition host, generator, and outlet. `enqueueJob` can choose a busy device; claims have no lease, fencing, attempt, or quarantine ([enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L41-L52), [scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L36-L55)).
4. Result integrity and access need immediate repair. Missing samples, collector failures, and failed ingest can still yield `completed`; energy is computed from mean power times summed request latency, not the measured window ([runner.rs](verde-flow-agent/src/runner.rs#L153-L165), [runner.rs](verde-flow-agent/src/runner.rs#L188-L215)). Ingest accepts unauthenticated writes, and some authenticated routes lack organisation checks ([ingest index.ts](verde-flow-api/apps/ingest/src/index.ts#L15-L31), [measurements.ts](verde-flow-api/apps/gateway/src/routes/measurements.ts#L8-L21)).

No percentage score is assigned. Code review cannot establish electrical compatibility, installed equipment, calibration, or successful physical acquisition.

### Status convention

**Implemented** = working code path supported by appropriate evidence; **Partial** = some behaviour exists but requirement incomplete; **Missing** = no implementation found in relevant paths; **Unverified** = plausible implementation whose necessary runtime/hardware evidence is unavailable; **Deferred** = explicit later-phase choice. A local compile does not prove a service or hardware workflow works.

## Repository map and deployment boundary

| Component | Actual implementation and boundary |
|---|---|
| Gateway | Node/TypeScript Express REST API; authentication, projects, runs, device registration, GitHub integration, agent poll/progress/complete routes. [gateway index.ts](verde-flow-api/apps/gateway/src/index.ts#L8-L70) |
| Scheduler | Separate Express process that reports queue depth and marks timed-out running rows failed. Dispatch/claim actually lives in gateway `enqueueJob` and `/api/scheduler/poll`. [scheduler index.ts](verde-flow-api/apps/scheduler/src/index.ts#L9-L27) |
| Ingest | Separate Express service inserts raw samples and summaries into PostgreSQL/TimescaleDB; no auth middleware. [ingest index.ts](verde-flow-api/apps/ingest/src/index.ts#L9-L87) |
| Estimator | Converts stored joules to carbon with one configured intensity. [estimator index.ts](verde-flow-api/apps/estimator/src/index.ts#L19-L32) |
| Agent | Rust process on a target; polls gateway, clones/builds/runs API, starts local samplers, drives endpoint requests, uploads measurements. [poller.rs](verde-flow-agent/src/poller.rs#L35-L74), [runner.rs](verde-flow-agent/src/runner.rs#L63-L215) |
| UI | React/Vite dashboard with project, device, run and detail pages. The trigger modal offers profiler labels and optional device choice; it is not a physical resource scheduler. [TriggerRunModal.tsx](verde-flow-ui/src/components/modals/TriggerRunModal.tsx#L4-L15), [RunDetailPage.tsx](verde-flow-ui/src/pages/runs/RunDetailPage.tsx#L102-L155) |
| Storage | PostgreSQL/TimescaleDB `runs`, `scheduled_jobs`, `measurements`, `raw_samples`, projects, devices, auth and alerts. MinIO is in Compose but no artifact use was found. [001_init.sql](verde-flow-api/db/migrations/001_init.sql#L98-L183), [docker-compose.yml](docker-compose.yml#L1-L42) |
| Deployment | Docker Compose exposes gateway, ingest, scheduler, DB, MinIO and UI; one containerized agent mounts Docker socket. Separate Pi systemd setup exists. [docker-compose.yml](docker-compose.yml#L44-L178), [PI_AGENT_SETUP.md](docs/PI_AGENT_SETUP.md) |

Two schema trees exist: `verde-flow-api/db/migrations/` is mounted by Compose, while `verde-flow-api/packages/storage/migrations/` contains another initial schema and `github_connected_repos` migration. The package migration is not mounted in Compose; the `db:migrate` script is a TODO ([docker-compose.yml](docker-compose.yml#L11-L17), [002_github_app.sql](verde-flow-api/packages/storage/migrations/002_github_app.sql#L27-L41), [migrate.js](verde-flow-api/db/scripts/migrate.js#L1-L3)). This makes a fresh installation's GitHub path uncertain until a real migration is exercised.

## Working path trace and stopping point

1. UI submits project, optional commit, selected profiler strings and optional device ([TriggerRunModal.tsx](verde-flow-ui/src/components/modals/TriggerRunModal.tsx#L74-L88)). Gateway verifies the project belongs to the user's org, creates a queued run, and calls `enqueueJob` ([runs.ts](verde-flow-api/apps/gateway/src/routes/runs.ts#L90-L137)). **Gap:** no immutable experiment manifest; an explicitly supplied device is not checked against the org or capabilities, and a run with no device can remain queued without a later dispatcher.
2. `enqueueJob` looks up a connected repository, chooses an online **or busy** device, writes a single-device `scheduled_jobs` row and separately updates the run ([enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L41-L103)). Gateway poll atomically claims a row with `FOR UPDATE SKIP LOCKED` and marks run/device running ([scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L36-L55)). **Gap:** no shared-channel contention or durable ownership of the whole resource set.
3. Agent clones the connected repository and checks out the requested commit, or uses a pre-running URL. It builds a Docker image or runs configured host commands ([checkout.rs](verde-flow-agent/src/checkout.rs#L57-L87), [apprunner.rs](verde-flow-agent/src/apprunner.rs#L60-L111)). The actual SHA backfill is best effort ([runner.rs](verde-flow-agent/src/runner.rs#L85-L93)). **Gap:** no build digest, protected sandbox or independent load generator.
4. The target agent starts local profiler(s) and sends sequential endpoint requests itself ([runner.rs](verde-flow-agent/src/runner.rs#L125-L182), [runner.rs](verde-flow-agent/src/runner.rs#L260-L298)). **Physical lab trace stops here:** there is no off-target logging host, commissioned DC path, AC analyzer, or collector start acknowledgement. `external_meter` cannot start; PMIC/PowerJoular failures may silently yield synthetic readings ([profiler.rs](verde-flow-agent/src/profiler.rs#L110-L135)).
5. Agent estimates per-endpoint joules as `mean_power × mean_latency × request_count`, backfills sample timestamps from the stop time, and uploads raw rows plus summaries ([runner.rs](verde-flow-agent/src/runner.rs#L153-L165), [profiler.rs](verde-flow-agent/src/profiler.rs#L67-L77), [ingest.rs](verde-flow-agent/src/ingest.rs#L7-L53)). Upload failures are non-fatal ([runner.rs](verde-flow-agent/src/runner.rs#L188-L215)). Ingest recalculates joules/request from power and latency, stores summaries and marks the run completed ([ingest index.ts](verde-flow-api/apps/ingest/src/index.ts#L40-L87)).
6. Gateway reads run measurements; UI shows one preferred profiler's aggregate and a completed badge ([runs.ts](verde-flow-api/apps/gateway/src/routes/runs.ts#L63-L87), [RunDetailPage.tsx](verde-flow-ui/src/pages/runs/RunDetailPage.tsx#L8-L13), [RunDetailPage.tsx](verde-flow-ui/src/pages/runs/RunDetailPage.tsx#L105-L155)). **Gap:** no provenance/quality gate prevents a synthetic or empty acquisition from appearing as a successful measurement.

## Requirements matrix

Evidence links refer to the current source, not verified installed hardware. Each row gives the gap, recommended action, and a practical verification. `P0/P1/P2` are the specification's priorities, not implementation estimates.

### Resource registry

| ID | Priority | Status | Evidence and gap | Recommended action and verification |
|---|---|---|---|---|
| LAB-REG-01 | P0 | Partial | `devices` combines host/agent/target; no instrument, channel, acquisition host or outlet records ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L60-L76)). | Add stable, distinct resource identities; verify a target and its meter/channel can be queried independently. |
| LAB-REG-02 | P0 | Partial | Device status only `online/offline/busy/error`; poll can set online ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L71-L75), [scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L22-L26)). | Add planned, commissioning, reserved, maintenance and quarantined states with server-side dispatch guards; test planned/quarantined refusal. |
| LAB-REG-03 | P0 | Partial | Heartbeat advertises arch/OS/version and profiler tokens, not envelopes, calibration, boundary or full capabilities ([scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L67-L92)). | Store typed capabilities with unknowns and commissioning evidence; test incompatible selection rejection. |
| LAB-REG-04 | P0 | Missing | Jobs carry profiler names but no target-to-channel/supply/outlet binding ([enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L73-L91)). | Commission explicit measurement paths; reject missing/incompatible path in API and scheduler. |
| LAB-REG-05 | P0 | Missing | `enqueueJob` chooses by device status; no channel or analyzer capacity ([enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L41-L52)). | Calculate capacity from compatible free paths; test two targets sharing one channel. |
| LAB-REG-06 | P1 | Missing | Device has mutable `metadata`; run has no hardware snapshot ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L62-L76), [001_init.sql](verde-flow-api/db/migrations/001_init.sql#L100-L117)). | Snapshot configuration/serial/firmware/calibration at run allocation; verify old run is unchanged after maintenance edit. |

### Collectors and agents

| ID | Priority | Status | Evidence and gap | Recommended action and verification |
|---|---|---|---|---|
| LAB-COL-01 | P0 | Partial | `MeasurementType` enum and sampler branches exist, but no vendor-neutral lifecycle/data contract ([profiler.rs](verde-flow-agent/src/profiler.rs#L15-L44), [profiler.rs](verde-flow-agent/src/profiler.rs#L100-L137)). | Extract collector interface around existing samplers; contract-test file, counter and streamed implementations. |
| LAB-COL-02 | P0 | Partial | Capability probe/start/stop exist locally; no validation, prepare/finalize, health or artifact retrieval ([profiler.rs](verde-flow-agent/src/profiler.rs#L90-L178)). | Add explicit lifecycle and error states; test prepare failure and bounded cleanup. |
| LAB-COL-03 | P0 | Partial | Raw numeric samples and `profiler` token reach DB, but no original artifact, source units, boundaries or quality ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L126-L183)). | Preserve immutable raw files and normalized records with units/source/boundary/quality; round-trip sample export test. |
| LAB-COL-04 | P0 | Missing | Local profiler runs in the same Rust agent as target workload ([runner.rs](verde-flow-agent/src/runner.rs#L125-L142)). | Support acquisition-host agents and separate identities; exercise target and logger as distinct simulated processes. |
| LAB-COL-05 | P0 | Partial | Empty samples can become zero summary; failed sample/results upload is non-fatal ([profiler.rs](verde-flow-agent/src/profiler.rs#L80-L88), [runner.rs](verde-flow-agent/src/runner.rs#L188-L215)). | Make collector outcome explicit and fail/partial on missing coverage; fault-inject disconnect/timeout/upload loss. |
| LAB-COL-06 | P1 | Missing | `external_meter` returns an error; no JS220, Otii or SmartPower adapter ([profiler.rs](verde-flow-agent/src/profiler.rs#L110-L114)). | Implement adapters only after model/API verification and commissioning; validate against actual units. |
| LAB-COL-07 | P1 | Missing | No PowerGoblin interface found in agent, API, scripts or manifests. | Inspect agreed Turku release; add versioned bridge for its actual interface and golden-file/real-host checks. |
| LAB-COL-08 | P1 | Partial | PowerJoular parser exists; RAPL is advertised but has no `MeasurementType` branch, so selection defaults to PMIC ([profiler.rs](verde-flow-agent/src/profiler.rs#L24-L34), [profiler.rs](verde-flow-agent/src/profiler.rs#L170-L176)). | Correct capability-to-adapter mapping; implement/verify RAPL counter and GPU scope separately. |
| LAB-COL-09 | P2 | Deferred | Current code is API/host focused; no cloud/Kubernetes, LLM or agent-event adapter found. | Extend the collector contract after physical path; test each source's distinct provenance. |

### Run lifecycle and scheduling

| ID | Priority | Status | Evidence and gap | Recommended action and verification |
|---|---|---|---|---|
| LAB-RUN-01 | P0 | Partial | Run stores optional commit/scenario and mutable JSON; job carries repo/build settings, no immutable artifact digest/window/boundary ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L100-L117), [enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L78-L91)). | Freeze validated manifest before queueing; verify exact source/build/workload/path identities persist. |
| LAB-RUN-02 | P0 | Missing | No full-set capability validation/reservation; explicit `deviceId` is accepted without org/capability check ([runs.ts](verde-flow-api/apps/gateway/src/routes/runs.ts#L105-L124), [enqueue.ts](verde-flow-api/apps/gateway/src/services/enqueue.ts#L41-L52)). | Validate complete allocation in one transaction; test foreign, incompatible and shared resources. |
| LAB-RUN-03 | P0 | Partial | `FOR UPDATE SKIP LOCKED` atomically claims one job, not exclusive target/channel leases ([scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L36-L49)). | Add DB-enforced resource lease constraints; concurrent-worker contention test. |
| LAB-RUN-04 | P0 | Missing | Assigned timestamp/timeout have no fencing token, renewal, attempt or quarantine ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L247-L259), [scheduler index.ts](verde-flow-api/apps/scheduler/src/index.ts#L17-L25)). | Durable ownership and reconciliation; simulate agent disconnect/restart before reuse. |
| LAB-RUN-05 | P0 | Partial | App readiness exists; short baseline and local acquisition start exist, but no coordinated warmup/window acknowledgement or separate outcomes ([apprunner.rs](verde-flow-agent/src/apprunner.rs#L95-L111), [runner.rs](verde-flow-agent/src/runner.rs#L105-L142)). | Add explicit window timestamps, logger ack and workload/acquisition outcomes; verify rejected start prevents load. |
| LAB-RUN-06 | P0 | Partial | Normal teardown on returned errors, but no persisted cancellation/attempt recovery; profiler shutdown is not guarded across every early error ([runner.rs](verde-flow-agent/src/runner.rs#L32-L61), [runner.rs](verde-flow-agent/src/runner.rs#L125-L215)). | Idempotent finalize/stop with quarantine on uncertainty; test cancellation, crash and redelivery. |
| LAB-RUN-07 | P0 | Missing | `run_workload` executes inside target agent ([runner.rs](verde-flow-agent/src/runner.rs#L137-L142)). | Assign generator role/host and record achieved load; test requests originate there. |
| LAB-RUN-08 | P1 | Partial | Priority/time order queue rows exist, but no booking, fair quota or contention view ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L247-L259)). | Separate bookings from execution leases; test queue fairness and quota. |
| LAB-RUN-09 | P1 | Missing | Single endpoint pass; no repetition/order/cooldown model ([runner.rs](verde-flow-agent/src/runner.rs#L137-L182)). | Add comparison plan with distinct research repetitions and attempt IDs; verify randomized seeded order. |
| LAB-RUN-10 | P1 | Partial | GitHub checkout and webhook paths exist, but no idempotent submission/result URL contract; one GitHub handler inserts `github_push`, outside schema's allowed trigger values ([github.ts](verde-flow-api/apps/gateway/src/routes/github.ts#L305-L362), [001_init.sql](verde-flow-api/db/migrations/001_init.sql#L106-L112)). | Consolidate webhook path, signature enforcement and dedupe; integration-test replay and PR/CI link. |

### Results and research validity

| ID | Priority | Status | Evidence and gap | Recommended action and verification |
|---|---|---|---|---|
| LAB-DATA-01 | P0 | Partial | Run, summary and raw-sample rows share run ID, but no raw artifact registry or quality/outcome linkage ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L100-L183)). | Add artifact identity/checksum and separate outcome/quality; verify all files resolve by run ID. |
| LAB-DATA-02 | P0 | Partial | J/W columns exist; computation uses latency-derived duration, while actual timestamps are backfilled at stop ([runner.rs](verde-flow-agent/src/runner.rs#L153-L165), [profiler.rs](verde-flow-agent/src/profiler.rs#L67-L77)). | Integrate timestamped power or delta counters with reset/wrap rules; deterministic uneven-spacing tests. |
| LAB-DATA-03 | P0 | Missing | `profiler` token is the only source descriptor; no declared physical boundary ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L158-L164)). | Store source and boundary per series/summary; test AC/DC/RAPL/GPU stay separate. |
| LAB-DATA-04 | P0 | Partial | Run start/end timestamps exist; sample times are reconstructed, no clock alignment or coverage ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L112-L117), [profiler.rs](verde-flow-agent/src/profiler.rs#L67-L77)). | Capture actual monotonic/window timestamps, clock method, sample coverage; missing coverage must remain unknown. |
| LAB-DATA-05 | P0 | Missing | PMIC/PowerJoular fallback runs `spawn_simulated` while retaining original profiler label ([profiler.rs](verde-flow-agent/src/profiler.rs#L118-L135), [profiler.rs](verde-flow-agent/src/profiler.rs#L220-L225)). | Remove silent fallback; explicitly classify simulated/estimated/imported; test every UI/export surface. |
| LAB-DATA-06 | P1 | Partial | Request count/latency/error fields exist, but denominator is all requests and `error_rate_pct` is hardcoded 0 ([ingest index.ts](verde-flow-api/apps/ingest/src/index.ts#L40-L56)). | Define window-matched successful-work denominator and repetition summary; test zero-success/unmatched windows. |
| LAB-DATA-07 | P1 | Missing | No CSV/JSON export or reproducibility manifest route found in gateway/UI. | Export run manifest, calculation versions and raw references; verify external round trip. |
| LAB-DATA-08 | P1 | Missing | No processing-method/version fields or immutable reprocessing lineage ([001_init.sql](verde-flow-api/db/migrations/001_init.sql#L126-L165)). | Version computations and append derived revisions; verify original remains accessible. |
| LAB-DATA-09 | P2 | Partial | Carbon estimator converts joules with one env intensity; DB defaults intensity to 208, with no region/time/source/method ([estimator index.ts](verde-flow-api/apps/estimator/src/index.ts#L19-L25), [001_init.sql](verde-flow-api/db/migrations/001_init.sql#L158-L160)). | Keep carbon labelled as derived and store intensity provenance; test source/unit/time handling. |

### Access, power and operations

| ID | Priority | Status | Evidence and gap | Recommended action and verification |
|---|---|---|---|---|
| LAB-OPS-01 | P0 | Partial | JWT/API-key auth and org checks exist on some routes; ingest has no auth, measurement read has no org filter, progress/complete/clone-token do not bind run to assigned device, and endpoint subroutes omit org checks ([auth.ts](verde-flow-api/apps/gateway/src/middleware/auth.ts#L11-L84), [measurements.ts](verde-flow-api/apps/gateway/src/routes/measurements.ts#L8-L21), [scheduler.ts](verde-flow-api/apps/gateway/src/routes/scheduler.ts#L102-L185), [projects.ts](verde-flow-api/apps/gateway/src/routes/projects.ts#L143-L156)). | Centralize server-side org/assignment checks, authenticate ingest and scope agent writes; cross-org/agent negative tests. |
| LAB-OPS-02 | P0 | Partial | Docker workload mode exists, but agent mounts Docker socket and command mode uses `sh -c`; no resource limits/network isolation ([docker-compose.yml](docker-compose.yml#L139-L164), [apprunner.rs](verde-flow-agent/src/apprunner.rs#L80-L88), [apprunner.rs](verde-flow-agent/src/apprunner.rs#L114-L155)). | Restrict workload runtime, privileges, network and resources; exercise malicious-workload isolation and cleanup. |
| LAB-OPS-03 | P0 | Partial | GitHub key mounted read-only, but config has `change_me` secret defaults, Compose has default DB/MinIO credentials, and clone URL embeds token in `git clone` arguments ([config index.ts](verde-flow-api/packages/config/src/index.ts#L7-L11), [docker-compose.yml](docker-compose.yml#L8-L32), [checkout.rs](verde-flow-agent/src/checkout.rs#L40-L67)). | Require production secrets; restrict admin settings/credential exposure; inspect process/log leakage in deployment test. |
| LAB-OPS-04 | P0 | Missing | No PDU/outlet model or remote power workflow found. | Add verified outlet map and authorization/interlocks only after commissioning; safe outlet dry-run and local sign-off. |
| LAB-OPS-05 | P0 | Partial | Queue is persisted, timeout marks running rows failed; no retry/reconciliation/quarantine, and a `queued` run without `scheduled_jobs` stays stranded ([runs.ts](verde-flow-api/apps/gateway/src/routes/runs.ts#L118-L135), [scheduler index.ts](verde-flow-api/apps/scheduler/src/index.ts#L17-L25)). | Reconcile orphan/expired jobs and uncertain devices; restart/failure tests with no duplicate launch. |
| LAB-OPS-06 | P0 | Partial | Heartbeat and health endpoints exist; no collector/disk/upload alarms, and `busy` is treated as live despite stale heartbeat ([poller.rs](verde-flow-agent/src/poller.rs#L8-L32), [devices.ts](verde-flow-api/apps/gateway/src/routes/devices.ts#L9-L20)). | Monitor collector, disk, lease, upload and agent liveness; inject failure and verify alert/quarantine. |
| LAB-OPS-07 | P1 | Missing | Compose has volumes and MinIO, but no NAS backup/restore procedure or tested restore found ([docker-compose.yml](docker-compose.yml#L1-L42)). | Define versioned DB/config/artifact backups and run restore drill. |
| LAB-OPS-08 | P1 | Missing | Raw samples are sent as one JSON batch, no chunk/checksum/retry/retention ([ingest.rs](verde-flow-agent/src/ingest.rs#L20-L53), [ingest index.ts](verde-flow-api/apps/ingest/src/index.ts#L15-L25)). | Spool local acquisition and upload bounded checked chunks; verify interrupted transfer recovery. |
| LAB-OPS-09 | P1 | Partial | Admin org/user views exist, but no commissioning, mapping, quarantine or manual recovery UI ([admin.ts](verde-flow-api/apps/gateway/src/routes/admin.ts#L17-L90)). | Add lab-admin maintenance/recovery screens after registry; verify role restrictions. |
| LAB-OPS-10 | P1 | Partial | Pi agent install guide/systemd files exist; no complete lab deployment, clock, meter permissions, UPS/shutdown or recovery-media runbook ([PI_AGENT_SETUP.md](docs/PI_AGENT_SETUP.md), [vf-agent.service](verde-flow-agent/deploy/vf-agent.service)). | Document and rehearse deployment/clock/permissions/UPS recovery on commissioned hosts. |

## Hardware integration matrix

These are software findings, not inventory or compatibility certifications. The attached spec describes owned and planned equipment; no instrument was operated in this audit.

| Tool/model | Collector location and transport in current code | Boundary and automated evidence | Outstanding verification |
|---|---|---|---|
| Pi 5 PMIC | Target-side Rust agent; `vcgencmd pmic_read_adc` ([profiler.rs](verde-flow-agent/src/profiler.rs#L252-L312)). | Rail-sum estimate, **not an external DC input meter**; parser has no test and failures can silently simulate. | Actual rail semantics, firmware, coverage, calibration/uncertainty and comparison to commissioned DC path. |
| PowerJoular | Target-side spawned CLI/CSV parser ([profiler.rs](verde-flow-agent/src/profiler.rs#L203-L250)). | Software power model; parser assumes column positions and can reread/duplicate prior lines; no fixture test. | Exact version/header, supported host, privileges, parser correctness and source scope. |
| RAPL | Heartbeat advertises `/sys/class/powercap/intel-rapl`, but no acquisition adapter ([profiler.rs](verde-flow-agent/src/profiler.rs#L173-L176)). | CPU/package domain; no automated measurement evidence. | Host permissions, domains, units, wrap range and chosen collector implementation. |
| Simulated | Target-side synthetic sampler ([profiler.rs](verde-flow-agent/src/profiler.rs#L354-L376)). | Development-only synthetic values; explicit `simulated` enum exists, but silent fallback masks provenance. | Test classification end to end after removing fallback. |
| SmartPower3, JS220, Otii Arc/Arc Pro | No adapter/transport/version declarations found; `external_meter` errors ([profiler.rs](verde-flow-agent/src/profiler.rs#L110-L114)). | Intended external DC boundary; no automated evidence. | Inventory/model/serial/firmware, limits, software entitlement, USB/API, rated supply/interface, calibration and logger-host access. |
| Planned Otii Ace Pro | No adapter; procurement only in spec. | Intended DC boundary; no evidence. | Supplier-confirmed supported model, drivers, current envelope, supply and automation licence. |
| Planned WT310E-class AC analyzer | No adapter/channel registry. | Intended whole-system AC boundary; no evidence. | Final model/fixture/driver, calibration, safe enclosure and shared-channel contention. |
| PowerGoblin | No code or declared transport found. | Integration target only. | Agreed Turku version, API/CLI/file format, supported meters and bridge ownership. |
| GPU telemetry, ambient logger, PDU, UPS | No collector/control implementation found. | No measured boundary or operational telemetry evidence. | Final models, safe control policy, host access and chosen metadata scope. |

## Acceptance scenarios and validation needed

No hardware-dependent scenario can be accepted from this audit. Current checks were compilation/type checks, not integration tests.

| Scenario from spec §13 | Static evidence now | Automated check to add | Hardware validation outstanding |
|---|---|---|---|
| 1. Single Pi API run | Git checkout, local sampler and API request loop exist; generator/logger/DC path absent. | Manifest, separate-role coordination and same-window energy/work counts. | Commission Pi DC wiring, logger, generator and compare raw trace. |
| 2. Instrument contention | One job-row claim is atomic; no channel lease. | Two-worker same-channel exclusion transaction test. | One real channel, two target paths, no overlap. |
| 3. Shared AC analyzer | No analyzer registry/adapter. | AC channel lease and boundary isolation test. | Workstation/frontend queue against the real analyzer. |
| 4. Collector failure | Failed start may be skipped; missing samples/upload can still complete. | Disconnect/timeout/empty/partial upload fault tests. | Safely unplug or simulate approved instrument failure. |
| 5. Agent restart/redelivery | Poll loop executes serially; no durable attempt/fence. | Kill/restart before/after claim and assert no duplicate work. | Reconcile uncertain target, quarantine before reuse. |
| 6. Repeated comparison | Commit and scenario exist; no repetition plan. | Seeded order, retry-versus-repetition and environmental metadata test. | Repeat stable baseline/variant under recorded conditions. |
| 7. AI inference | Generic endpoint workload only. | Model/config/count/boundary schema and calculation test. | AC workstation trace plus separate GPU telemetry. |
| 8. Access isolation | Several org filters; known missing checks. | Cross-org reads/writes, foreign device/outlet and agent-run ownership tests. | Identity/network review on deployed lab. |
| 9. Simulated development run | Explicit synthetic sampler exists; fallback may mislabel it. | Assert simulation provenance in storage, export and UI. | None required for simulated path. |
| 10. Export and recovery | No export or backup/restore path found. | Manifest/raw checksum round trip and restore drill. | Restore retained run from NAS on lab infrastructure. |

## Prioritised implementation plan

Effort bands are **estimates**, relative to one developer familiar with the code and excluding supplier lead time or hardware commissioning. Preserve the existing Express/React/Rust/PostgreSQL design; no new framework is required.

1. **P0, immediate containment (several days):** Disable silent simulated fallback for physical labels; distinguish measurement outcome from workload outcome; reject empty acquisition and failed ingest as completed measurements. Authenticate ingest and bind every agent mutation/clone-token request to its assigned run; add org filters to measurements/endpoints/status routes. Require production secrets and signed webhook submissions. Add focused regression tests for these failures. This protects current prototype results before extending the lab.
2. **P0, resource and run core (roughly 2–4 weeks):** Add distinct resource/path registry, commissioned states, typed capability checks and immutable run manifest. Use PostgreSQL transactions/constraints to reserve the complete compatible set; add attempt IDs, leases/fencing, heartbeats, timeout reconciliation and quarantine. Make queueing and claim idempotent. Test multi-worker channel contention, restart/redelivery and cancelled/failed cleanup.
3. **P0, first physical vertical slice (roughly 3–6 weeks after path design):** Add a logging-host collector contract, target/generator/logger coordination and exact measurement-window acknowledgement. Commission one Pi plus one verified DC instrument path; keep load generation on its own host. Store source timestamps, raw artifact/checksum, boundary, units, quality and coverage. Calculate joules by timestamped integration or correct counter delta and use successful work in the same window. Validate this exact path on hardware before scaling to ten Pis.
4. **P1, research service (roughly 4–8 weeks after P0 slice):** Add bookings/fair queue/quotas, repeated comparison plans, CSV/JSON reproducibility export, versioned processing, backup/restore and lab-admin maintenance. Add selected JS220/Otii/SmartPower and PowerGoblin adapters only against confirmed versions. Extend to the shared AC analyzer and AI/frontend targets with declared boundaries.
5. **P2, later extensions:** Add cloud/Kubernetes, LLM and event collectors through the same contract; enrich separately labelled carbon estimation with source/time/region and embodied-impact distinction.

Dependencies matter: electrical approval and supplier APIs precede a physical adapter; registry/leases precede shared analyzer concurrency; honest provenance and acquisition outcomes precede meaningful comparisons or carbon claims.

## Validation record and limits

| Check executed | Actual result |
|---|---|
| `find .. -name AGENTS.md -print` | No applicable `AGENTS.md` found under the project parent. |
| `rg --files`, `rg` searches and numbered source reads | Inspected manifests, Compose, API/agent/UI paths, schemas, scripts and docs. No automated test source was found in repository file inventory. |
| `pnpm --recursive run typecheck` in `verde-flow-api` | Exit 0. Ran package and scheduler/ingest/estimator type checks; gateway has no `typecheck` script. |
| `pnpm exec tsc --noEmit -p apps/gateway/tsconfig.json` in `verde-flow-api` | Exit 0. |
| `pnpm exec tsc --noEmit` in `verde-flow-ui` | Exit 0. |
| `cargo test --offline` in `verde-flow-agent` | Exit 0, **0 tests** run; six compiler warnings. |
| `git status --short` before audit | Existing modifications: `SETUP.md`, `docker-compose.yml`; untracked `docs/VERDEFLOW_LAB_SPEC.md`, `docs/VerdeFlow_Measurement_Audit_Design_Spec.pdf`, `verde-flow-website/`. These were not changed by this audit. |

No database migration, Compose startup, live API request, GitHub webhook, physical meter, Pi, PDU or NAS test was run. A type check does not validate migrations, authorization, system wiring or measurements. `docs/VERDEFLOW_LAB_SPEC.md` is untracked; this review uses the attached Downloads specification as the target. Static “missing” findings mean no implementation was found in the inspected repository paths, not that no external lab system exists.

## Questions for the lab team or suppliers

1. Which owned instrument units/models/serials, firmware, channels, calibration records and automation entitlements are actually available, and which Pi supply/interface arrangements have electrical approval?
2. Which exact PowerGoblin release and supported API/CLI/file output will Turku provide, and who operates its bridge?
3. Which AC analyzer, fixture, PDU/outlet map, ambient logger and UPS configuration will be commissioned, with what control/safety rules?
4. Which hosts are installed versus planned, and what maximum simultaneous measured-run capacity follows from the real channels and acquisition hosts?
5. What identity provider/network access, project sharing, artifact retention and backup/restore targets must the deployed lab enforce?
