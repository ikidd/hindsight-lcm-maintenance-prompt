# Agent prompt: build Hindsight and LCM maintenance on this Hermes host

You are operating directly on a Hermes Agent host for its owner. Build, install, schedule, and verify a maintainable, host-specific subsystem-maintenance package for **Hindsight** and **LCM/LCM-X**.

You are not receiving any existing scripts or skills to copy. Reconstruct the solution from the live system, installed package/plugin source, official current documentation, and the requirements below. Do not assume paths, profile names, service names, providers, versions, database engines, plugin identities, cron delivery targets, or update mechanisms from another installation.

## Authorization and safety boundary

You are authorized to:

- perform read-only discovery across the local Hermes installation;
- create a maintenance skill, references, scripts, tests, manifests, and profile-local cron-visible copies;
- create or update recurring health/update-monitoring jobs in the selected owner profile;
- run non-destructive health checks and real bounded smoke tests;
- restart **Hindsight** when required to validate a Hindsight-specific configuration or compatibility fix, provided you first create a verified backup and a rollback path;
- install the completed maintenance package after tests pass.

You are **not** authorized merely by this prompt to:

- delete, deduplicate, rewrite, or prune Hindsight memories;
- delete or rotate Hermes sessions;
- alter which profiles use Hindsight or LCM;
- enroll a profile that currently uses neither subsystem;
- upgrade Hermes, Hindsight, LCM, host packages, models, or providers just because an update is available;
- switch model/provider identity;
- perform destructive database repair;
- replace an LCM database;
- reboot the host;
- publish credentials or secret-bearing configuration;
- force-kill or synchronously restart the gateway from the agent session that depends on it.

For those actions, build and document the safe workflow, report the finding, and obtain separate owner approval before applying it. Installing the maintenance framework itself is authorized.

## Authoritative references

Before changing anything:

1. Load the installed `hermes-agent` skill and follow it. Do not guess Hermes commands or configuration behavior.
2. Discover and load any installed Hindsight, memory-hygiene, LCM, plugin-maintenance, cron, or skill-authoring skills that are relevant.
3. Use the current official Hermes documentation at `https://hermes-agent.nousresearch.com/docs` for Hermes behavior.
4. Inspect the installed Hindsight packages/service and LCM plugin source/manifest for version-specific behavior.
5. Use upstream release metadata and documentation for the exact installed projects. Track stable releases, not arbitrary repository `main`.
6. Treat live state as authoritative. Historical notes and recalled memory are orientation only.

If local documentation conflicts with current live behavior, report the discrepancy and follow the behavior verified by the installed version unless an approved migration is required.

## Working discipline

- Inspect before designing and changing.
- Separate observed facts, inferences, proposals, applied changes, and verified results.
- Back up before any state-changing package, service, plugin, database, cron, or configuration operation.
- Never print OAuth tokens, API keys, database URLs, passwords, refresh tokens, raw process environments, or secret-bearing `.env` files.
- Use restrictive permissions (`0700` directories and `0600` secret-bearing files/backups where appropriate).
- Use exact file hashes and read-back verification.
- Prefer simple Python standard-library checkers with bounded timeouts and deterministic output.
- Silent success must truly produce empty stdout for unattended watchdog jobs.
- Actionable findings should normally exit successfully so notification delivery is not confused with checker failure; reserve nonzero status for a broken checker or failed required operation.
- Do not stop after writing templates. Run the scripts, unit tests, health checks, cron jobs, and post-install verification.

## Phase 1: discover the real installation

Create a concise inventory before implementation. At minimum determine:

### Hermes and profiles

- Hermes executable, version, installation method, home directory, active gateway service, and gateway launch profile.
- Every configured profile discovered dynamically from live configuration—not a hard-coded list.
- Which profile will own this maintenance package and its notifications. Prefer an existing operations/admin profile. If no clear owner exists, ask the owner once rather than inventing one.
- For each profile:
  - profile home and config path;
  - whether it intentionally requests Hindsight, LCM, both, or neither;
  - plugin enabled and disabled lists;
  - context engine and memory provider;
  - compression settings;
  - plugin paths and whether they are real directories or links;
  - relevant database path.
- Do not classify a profile as broken merely because it intentionally uses neither subsystem.

### Hindsight

Discover rather than assume:

- API/server package and installed version;
- Hermes-side client package and installed version;
- service manager and service unit;
- effective health endpoint;
- database backend and connectivity method;
- effective LLM provider/model and embeddings/reranker configuration, without exposing credentials;
- whether local ML dependencies are installed inside an isolated tool environment;
- authentication compatibility files or token-sync workarounds, if any;
- current service state, restart behavior, journal errors, and warm-up time;
- latest stable package/release versions from authoritative sources.

### LCM

Discover whether the installation uses legacy LCM or its successor identity. Determine:

- plugin name and context-engine name;
- manifest version and `requires_hermes` compatibility;
- installation source and provenance: bundled, catalog, package, reviewed release, or custom checkout;
- current stable upstream release and exact release identity;
- plugin load timeout requirements;
- database path, WAL/SHM state, schema version, and integrity;
- active tools and slash commands;
- current gateway journal evidence proving activation or fallback;
- profiles sharing one reviewed plugin install versus independent copies.

### Existing maintenance

Inventory existing skills, scripts, cron jobs, timers, backups, reports, and update checks. Do not create duplicate jobs. Consolidate or supersede only after confirming ownership and preserving rollback.

Write the inventory to a non-secret Markdown report under the owner profile’s workspace or maintenance directory.

## Phase 2: design the package from the discovered state

Build three clearly separated concerns.

### A. Subsystem health and update maintenance

Create a skill named appropriately, preferably `hermes-subsystem-maintenance`, owned by the selected operations profile. It must document:

- responsibilities and ownership;
- read-only health and update monitoring;
- Hindsight update workflow;
- LCM audit/update/migration workflow;
- backup, rollback, restart, and verification requirements;
- secret-handling rules;
- canonical script paths and cron-visible copies;
- schedules and delivery destinations;
- the boundary between detection and authorized update application.

### B. Hindsight memory hygiene

Keep memory quality separate from service/package health. Create or extend a separate memory-hygiene skill that documents:

- history versus durable-memory roles;
- database backup before deletion;
- inventory by source/type/tag/time;
- stable user preferences and credential-location pointers that must be preserved;
- project-anchor creation before pruning;
- exact-ID export/archive/delete discipline;
- conservative deduplication and contradiction handling;
- post-cleanup recall tests;
- cleanup self-noise removal;
- prohibition on destructive unattended cleanup without an approved policy.

Install the skill, but do not delete memory as part of this build. A scheduled hygiene job may be report-only unless the owner separately approves deletion rules.

### C. LCM operational and recall guidance

Create or extend an LCM skill/reference set that explains:

- raw stored messages versus summaries;
- current-session versus cross-session versus external Hermes history;
- status, inspect, doctor, grep/search, recall, expand, session-load, evidence compilation, and exact-computation workflows available in the installed plugin;
- exact-source recovery for historical claims;
- safe session lifecycle and rotate/backup behavior;
- preview and authorization requirements for mutation/repair commands.

Do not fabricate tools that the installed plugin does not provide.

## Phase 3: implement canonical scripts

Create canonical scripts inside the subsystem-maintenance skill. Use host-specific discovered values through a small configuration section or safe dynamic discovery—do not scatter hard-coded paths.

### 1. Broad subsystem checker

Create a script equivalent in purpose to `check_subsystem_updates.py`. It must support:

- normal silent watchdog mode;
- `--always` or equivalent manual full report;
- `--skip-network` where useful for deterministic testing;
- bounded subprocess and network timeouts;
- secret-safe errors.

It must check:

#### Hindsight

- API health and database connectivity;
- service active state;
- installed API/server and client versions;
- latest stable versions from authoritative registries;
- import smoke for the client;
- one real, minimal, bounded LLM-powered operation through the effective running provider/model—not only a health endpoint;
- clear differentiation among unsupported model, authentication rejection, quota/rate limit, timeout, transport failure, and malformed response without printing raw provider exceptions;
- local embedding/reranker imports when that installation uses them;
- recent service journal for crash/restart/fallback indicators.

#### LCM

For every dynamically discovered profile that requests LCM/Hindsight:

- correct plugin enabled state;
- enabled/disabled conflicts;
- correct context-engine identity;
- required plugin load timeout;
- compression enabled where required;
- secondary-profile plugin path resolving to the reviewed shared install when that is the local architecture;
- plugin manifest identity/version/provenance;
- Hermes compatibility;
- read-only database `PRAGMA quick_check` or the installed plugin’s supported equivalent;
- expected schema version discovered from installed code/migrations, not copied from this prompt;
- gateway runtime journal with no load timeout, inactive-engine report, or fallback to another compressor;
- positive activation evidence, not merely absence of errors.

Do not flag profiles intentionally using neither subsystem.

#### Package integrity and host maintenance

- maintenance-package sanity status;
- relevant cron jobs present and enabled only in the owning profile;
- duplicate jobs in other profiles;
- host updates and reboot-required markers as findings only;
- stale/deprecated plugin identities that require an explicit migration.

### 2. Lightweight LCM release watcher

Create a separate daily checker equivalent in purpose to `check_hermes_lcm_update.py`:

- read installed manifest identity/version;
- verify expected installation provenance;
- query latest stable upstream release;
- stay silent when current and healthy;
- report installed/latest release and review URL when action is needed;
- never apply an update itself.

The daily checker is only for release/provenance drift. The weekly broad checker owns deeper runtime, configuration, profile, and database auditing.

### 3. Hindsight authentication compatibility watcher

Only build this if discovery proves it is needed. If Hindsight reads provider credentials from a compatibility location while Hermes refreshes another source:

- copy only the required fields without printing them;
- preserve required account identifiers;
- compare expiry/refresh state before changing anything;
- back up the destination credential file;
- write atomically at mode `0600`;
- restart only the Hindsight service and only after a real change;
- wait for bounded health recovery;
- remain silent when no sync is needed.

If the running versions share credentials correctly without a compatibility sync, explicitly omit this script and document why.

### 4. Sanity checker and manifest

Create a package sanity script and a JSON manifest containing package version and SHA-256 hashes. Verify:

- Python compilation;
- expected scripts and references;
- manifest hashes;
- canonical versus cron-visible copies byte-for-byte;
- cron-visible scripts are real files if the cron sandbox rejects external symlinks;
- expected jobs exist in the owner profile;
- duplicate/legacy jobs are disabled or clearly reported;
- skill documentation contains the current package version.

### 5. Regression tests

Create isolated tests for at least:

- profile discovery beyond a hard-coded set;
- profiles intentionally using neither subsystem;
- enabled/disabled plugin conflict;
- launch-profile disablement affecting a multiplex gateway;
- plugin identity/provenance mismatch;
- missing or wrong plugin link;
- database integrity/schema findings;
- runtime log classification: successful activation, timeout, inactive engine, fallback;
- Hindsight model-probe result classification;
- silent-success versus actionable-output behavior;
- manifest/copy drift detection.

Tests must use fixtures or temporary directories and must not mutate live profiles, databases, credentials, services, or cron jobs.

## Phase 4: backup before installation

Before writing into live skill/script/cron locations:

- back up the selected profile configuration and existing relevant skill/script directories;
- export current cron-job definitions for all profiles;
- create a verified Hindsight database backup using its actual backend;
- record installed package versions and dependency freezes for the Hindsight API environment and Hermes client environment;
- preserve the LCM database consistently, including WAL/SHM state, using the plugin’s supported online backup or a verified writer-free cold copy;
- back up plugin manifest/configuration and every affected profile config;
- write SHA-256 checksums and verify backup readability;
- record rollback commands without secret values.

A checksum or dump listing is not a full restore test; label verification truthfully.

## Phase 5: install and schedule

Install the tested package into the selected owner profile.

- Keep canonical scripts under the skill directory.
- Place cron-visible real-file copies under the profile’s permitted script directory if required by the Hermes cron sandbox.
- Set safe ownership and modes.
- Create/update jobs idempotently; do not duplicate jobs on rerun.
- Use the host’s configured local timezone and show its resolved abbreviation/offset.
- Prefer:
  - a weekly broad subsystem health/update report;
  - a daily lightweight LCM release/provenance watcher;
  - an authentication compatibility sync only if needed;
  - an optional weekly report-only memory-hygiene review.
- Choose schedules that avoid backup windows, package maintenance, model warm-up windows, and other expensive jobs.
- Use no-agent/script execution for deterministic watchdogs where Hermes supports it.
- Deliver findings to the owner-approved destination; stay silent on healthy runs.
- Do not schedule destructive memory cleanup.

If schedule or delivery destination cannot be inferred safely, ask the owner one consolidated question before creating the jobs.

## Phase 6: verification

Do not claim completion until all applicable checks pass with fresh output.

Run and record:

1. unit/regression tests;
2. Python compilation for every script;
3. package sanity checker;
4. broad checker in `--always` mode;
5. broad checker in normal mode, proving empty stdout when healthy;
6. lightweight LCM checker in verbose/manual mode and normal silent mode;
7. Hindsight health endpoint and database connectivity;
8. real Hindsight recall and one LLM-powered reflection/completion operation;
9. LCM status/inspect/doctor calls exposed by the installed plugin;
10. LCM database integrity and schema checks for every participating profile;
11. positive runtime activation evidence from the current gateway start;
12. one ordinary message/session through every intended profile, verifying that it appears in the correct LCM database when appropriate;
13. exact read-back of cron schedules, enabled state, execution mode, owner profile, and delivery target;
14. one manual cron execution of each new job, confirming expected output/delivery behavior;
15. hashes proving cron-visible copies match canonical scripts;
16. rollback artifact existence and checksums.

Do not test plugin slash commands by sending them through a CLI path that intercepts slash commands before plugin dispatch. Use a real agent session or registered tools appropriate to the installed version.

## Gateway and service restart safety

- Never perform a blocking restart of the gateway from the gateway-hosted agent turn.
- A quiet process tree does not prove there is no active model call.
- If gateway restart is required, create an independent, durable, idle-gated activation and rollback runner first.
- Preserve pre-change configs.
- Queue a non-blocking restart.
- Require a replacement PID and successful health/tool/profile smokes.
- Automatically restore configs and queue a recovery restart if activation fails.
- Do not escalate a slow graceful stop to a forced kill without separate authorization.

Hindsight may be restarted independently when authorized above, but verify the actual warm-up behavior rather than imposing an unrealistically short timeout.

## Update runbooks to include, but do not execute without separate approval

### Hindsight update runbook

Document exact commands for the discovered installation to:

- back up database/configuration;
- freeze both API and Hermes-client environments;
- upgrade server/API and client separately;
- detect whether a tool upgrade removed local-ML extras;
- restore exact compatible CPU ML dependencies if needed;
- restart Hindsight;
- wait through realistic model warm-up;
- verify versions, health, imports, recall, reflection, and effective running client code;
- restart the gateway safely only if required to load a new client library.

### LCM update/migration runbook

Document exact commands to:

- inventory all profiles;
- preserve all databases consistently;
- stage the exact stable tag/SHA or reviewed catalog release;
- verify `requires_hermes` compatibility;
- run upstream tests and a temporary-home/database smoke;
- validate the disabled install before live config changes;
- stage guarded profile changes and rollback;
- perform independent idle-gated activation;
- verify tools, logs, profile ingestion, database integrity and schema;
- retain rollback material until every profile passes.

Never substitute an arbitrary repository URL or `main` branch when the local policy requires a reviewed catalog package or exact stable release.

## Final report

Provide a concise but complete report separating:

- observed pre-change state;
- decisions and assumptions;
- files created or changed;
- installed package version and manifest hash;
- schedules, timezone, execution modes and destinations;
- test commands and actual results;
- Hindsight health/model verification;
- LCM profile/runtime/database verification;
- available updates found but not applied;
- backup and rollback locations;
- remaining warnings;
- any owner decisions still required.

Attach or link the generated non-secret skill, scripts, tests and report where the platform permits. Do not expose raw memory archives, databases, tokens, process environments or credentials.

Completion means the maintenance package is installed, scheduled, exercised, and verified on this host—not merely drafted.