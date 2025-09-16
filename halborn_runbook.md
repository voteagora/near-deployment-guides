## NEAR Foundation High-Value Contracts
### Guide to Secure Deployment, Upgradability, and Operations

### Executive Summary
 A single unsafe deployment or upgrade can cause material loss, reputational damage, and ecosystem instability. This guide defines a secure, repeatable, and transparent way to deploy and upgrade NEAR contracts that executives can trust and engineers can execute. It combines governance, operational security, deterministic engineering, on-chain safety rails, and rigorous observability.

What you get:
- A risk-based governance model with timelocks and clear separation of powers
- Deterministic builds and on-chain verification for every change
- Step-by-step deployment and upgrade processes with embedded checklists
- NEAR-specific patterns for safe upgradability and migrations
- Monitoring, alerting, and incident response playbooks
- Templates for proposals, release manifests, attestations, and postmortems

Outcomes:
- Reduced likelihood and impact of security incidents, audit-ready evidence for all changes, faster remediation when things go wrong

---

### How to Use This Guide
- Executives: read the Executive Summary, Governance and OpSec, and Roadmap.
- Engineers and Operators: follow the Release Engineering, Deployment, and Upgrade sections and checklists.
- Security Council: use the Governance, Signing Ceremony, and Incident Response sections.
- Auditors/Observers: see the Templates, Events, and Attestations for evidence trails.

---

### 1. Purpose, Scope, and Roles
Purpose: Provide a single, authoritative blueprint for securely deploying and upgrading NF’s high-value NEAR contracts.

Scope: Deployment and Upgradability (primary lenses), underpinned by Governance and OpSec.

Key Roles (RACI):
- Proposer (R): engineering lead or product owner
- Reviewers (A/C): internal security and external auditors
- Approvers (A): Security Council signers via governance
- Executor (R): governance account (DAO/multisig)
- Verifier (C): independent observers attest to hashes and outcomes
- Initiator (I): A member from the governance team to facilitate the ceremony

---

### 2. Security Principles and Risk Model
Principles:
- Defense-in-depth: combine governance, opsec, on-chain safeguards, and monitoring
- Determinism: build reproducibly; verify on-chain code hash every time
- Least privilege: minimize full-access keys; prefer function-call keys for automation
- Explicit trust model: define who can do what, when, and how it’s logged and verified
- Rollback-ready: retain last-known-good artifacts and a tested recovery path

Risk Model (examples):
- Key compromise (signers, operators, CI)
- Unsafe upgrades and migrations (logic or data loss)
- Supply chain risks (malicious libs, toolchain drift)
- RPC tampering or indexing blind spots
- Async execution pitfalls (failed callbacks, reentrancy, gas griefing)

---

### 3. Standard Operating Procedures (SOPs)
This section provides exact, step-by-step runbooks for common operations. Follow them as written and record evidence at each step.

#### 3.1 Pre-flight Checks (run before any change)
1) Confirm network and RPC endpoints are healthy.
2) Verify current on-chain code hash and record it:
   ```bash
   near state <contract>
   ```
3) Check access keys for unexpected entries (record output):
   ```bash
   near keys <contract>
   ```
4) Confirm governance ownership/role mapping via your contract’s owner/role view methods (record outputs).
5) Validate storage headroom (storage_usage vs account balance) and gas budgets for planned actions.
6) Compute artifact digest and match manifest:
   ```bash
   sha256sum contract.wasm
   ```
7) Prepare comms plan (internal + public) and change window; notify stakeholders.

Record in evidence: CLI outputs, artifact hash, manifest URL, comms approval.

#### 3.2 New Mainnet Deployment (no prior code)
Owners: Proposer (Engineering lead), Approvers (Security Council), Executor (Governance account), Verifier (Independent).
1) Build deterministically and co-sign release manifest.
2) Deploy to Testnet/canary with production-like config; run smoke tests.
3) Create governance proposal with: artifact hash, source commit, audit links, execution window, and rollback plan.
4) Start timelock; announce window to stakeholders.
5) Day-of: validate signer devices and environment (hardware wallets, isolated network).
6) Execute deployment via governance account:
   ```bash
   near deploy --accountId <contract> --wasmFile contract.wasm
   ```
7) Verify on-chain code hash matches artifact digest:
   ```bash
   near state <contract>
   ```
8) Run smoke tests (read-only and minimal state changes) and validate logs/events.
9) Publish public attestation with tx links and artifact hash.
10) Monitor for 24–72h; archive all evidence.
Evidence (minimum): near-cli outputs, tx hashes, artifact hash, release manifest URL, signers present, minutes and screenshots, post-deploy smoke test results.

#### 3.3 Routine Upgrade (no migration)
Owners: Proposer (Engineering lead), Approvers (Security Council), Executor (Governance account), Verifier (Independent).
1) Pre-flight checks (Section 3.1).
2) Governance proposal with artifact hash and execution plan; start timelock.
3) Optional canary on a staging mainnet account.
4) Execute upgrade:
   - If external upgrade:
     ```bash
     near deploy --accountId <contract> --wasmFile contract.wasm
     ```
   - If self-upgrade (encode Wasm to base64 first):
     ```bash
     base64 -w0 contract.wasm > code.b64  # macOS: base64 < contract.wasm | tr -d '\n' > code.b64
     ```
     ```bash
     near call <contract> upgrade "{\"code\":\"$(cat code.b64)\"}" \
       --accountId <governance> --gas 300000000000000
     ```
5) Verify code hash, run smoke tests, publish attestation, monitor 24–72h, archive evidence.
Evidence (minimum): proposal link, tx hashes, on-chain code hash, artifact hash, smoke test outputs, attestation link.

#### 3.4 Upgrade With Migration (state changes)
Owners: Proposer (Engineering lead), Approvers (Security Council), Executor (Governance account), Verifier (Independent), Operator (migration runner).
1) Pre-flight checks (include data-volume estimate and gas budget).
2) Rehearse migration on Testnet/canary with production-like data.
3) Governance proposal includes migration plan and rollback plan; start timelock.
4) Execute upgrade and migration in bounded steps (idempotent, resumable):
   - Upgrade code (Section 3.3 step 4)
   - Run migration entrypoint(s) in batches if needed; verify intermediate invariants.
5) Verify code hash and post-migration invariants; publish attestation; extended monitoring window; archive evidence.
Evidence (minimum): migration plan, rehearsal results, batch logs, invariants before/after, tx hashes, attestation link.

#### 3.5 Emergency Pause / Containment
Owners: Initiator (On-call), Approver (Guardian multisig), Communicator (Comms lead), Investigator (Security/Eng).
1) Trigger: exploit indicators, code-hash mismatch, key compromise.
2) Guardian multisig proposes and executes `pause` (no upgrades):
   ```bash
   near call <contract> pause '{}' --accountId <guardian>
   ```
3) Notify internal responders; publish initial advisory if user risk exists.
4) Investigate, decide on rollback vs hotfix path.
5) Keep detailed timeline and evidence for postmortem.
Evidence (minimum): trigger indicators, pause tx hash, advisories, investigation notes, timeline.

#### 3.6 Rollback to Last-Known-Good
Owners: Proposer (Engineering/Security), Approvers (Security Council), Executor (Governance), Verifier (Independent).
1) Confirm last-known-good artifact and manifest.
2) Governance proposal to redeploy previous artifact; start timelock (unless emergency criteria met).
3) Execute redeploy and verify code hash.
4) Validate invariants, publish attestation, monitor, and complete postmortem.
Evidence (minimum): artifact and manifest of LKG, rollback tx hash, invariants, attestation, postmortem.

---

### 4. Governance and OpSec (Who should deploy, thresholds, timelocks)
Security Council:
- Owns upgrade rights via a DAO/multisig with a high approval threshold (e.g., 7/10 or 10/12 based on value-at-risk) and enforced timelocks (e.g., 48–72 hours).
- Members use hardware wallets, are organizationally and geographically diverse, and rotate keys periodically.

Guardian Group (Emergency):
- Separate, smaller multisig for pause/resume only. Cannot upgrade. Lower threshold enables fast containment.

Checks and balances:
- Timelocks to create reaction windows. Independent verifiers attest to code hashes. Public attestations increase transparency.

OpSec Controls:
- Hardened workstations, minimal extensions, isolated networks/VPN during ceremonies
- Split-custody of recovery seeds, tested recovery drills, key rotation policy
- Immutable storage for artifacts and logs; strict access controls and audit trails

Challenge assumptions:
- 10/12 is strong but heavy; consider risk-based thresholds with timelocks and independent watchers.
- One-size-fits-all does not work; tune thresholds and delays per contract criticality and change type.

Key security, backup, and recovery (SOPs):
- Objectives: prevent single-key failure, withstand device loss/compromise, and enable fast, safe recovery.

1) Key provisioning and inventory
   - Hardware wallets only (e.g., Ledger with NEAR app). Verify device authenticity and update firmware.
   - Generate keys on-device. Never export seeds to computers or cloud.
   - Use BIP39 passphrases for added security where supported; document passphrase custody separately from seed.
   - Record public keys, fingerprints, and intended roles in a central inventory with access controls.
   - Proof-of-possession: each signer signs a standard message (purpose, date, public key) and submits the signature to the inventory.

2) Backup strategy (split custody)
   - Seeds on steel (or equivalent) in tamper-evident sealed bags. No photos/scans.
   - Use Shamir Secret Sharing (M-of-N) for seeds or passphrases, distributed across independent custodians and regions.
   - Keep at least two independent backups per key (different locations). Maintain an access log for all vaults.
   - Store device PINs separately from devices; never in the same container as the seed.

3) Routine operations and hygiene
   - No full-access keys on servers. Never sign governance actions from CI.
   - Dedicated, hardened laptops for signers; isolated network/VPN during ceremonies; no extraneous browser extensions.
   - Mandatory screenshare (or in-person) during ceremonies to reduce social-engineering risk.
   - Quarterly review of key inventory: verify current signers, public keys, device status, and backup locations.

4) Rotation and recovery (lost/damaged device)
   - Immediately inform Security Council; open an incident ticket.
   - Recover key from split backups under dual control; re-provision device; generate a replacement key pair if compromise is suspected.
   - Update multisig/DAO signer set via governance proposal (add new key, then remove old key). Avoid dropping below safe thresholds during transition.
   - Update inventory and attest to the change (who, when, why, tx links).

5) Suspected compromise playbook
   - Contain: freeze pending proposals; pause non-essential governance actions.
   - Remove the compromised signer key from multisig/DAO via emergency governance if available.
   - Increase threshold temporarily if quorum security is at risk.
   - For contract accounts with full-access keys, remove/rotate immediately:
     ```bash
     near delete-key <account> <public-key>
     ```
   - For function-call keys, revoke and re-issue with minimal scopes.
   - Conduct forensics and rotate any related secrets; publish an advisory if user impact is possible.

6) Device lifecycle
   - Onboarding: provision device, register public key, record proof-of-possession, set up backups and custodians.
   - Offboarding: remove keys from governance, retire device, destroy or securely wipe; reconcile backups; update inventory.
   - Firmware: schedule periodic updates; validate checksums and provenance before installing.

7) Drills and attestations
   - Semi-annual recovery drills: simulate loss of a device and recover within defined RTO.
   - Annual signer rotation exercise for a subset of keys to validate the process end-to-end.
   - After every key change, publish an internal attestation and attach governance tx links.

---

### 5. DAO Tooling Comparison (AstroDAO vs Custom Multisig)

#### AstroDAO (Recommended for most use cases)
- **Pros:**
  - Battle-tested on NEAR mainnet with billions in TVL
  - Built-in proposal types (transfer, function call, add/remove member, config change)
  - Native timelock support via proposal voting periods
  - Role-based permissions (council, community, custom roles)
  - UI available at app.astrodao.com
  - Upgrade path via proposal system
- **Cons:**
  - Less flexibility for custom logic
  - Gas overhead for proposal creation/voting (~5-10 TGas per action)
  - Learning curve for configuration

**Setup example:**
```bash
# Create DAO with initial council
near call factory.astrodao.near create '{
  "name": "nf-security-council",
  "args": {
    "config": {
      "name": "NF Security Council",
      "purpose": "High-value contract governance",
      "metadata": ""
    },
    "policy": {
      "roles": [{
        "name": "council",
        "kind": {"Group": ["alice.near", "bob.near", "charlie.near"]},
        "permissions": ["*:*"],
        "vote_policy": {"weight_kind": "RoleWeight", "quorum": "70", "threshold": [7, 10]}
      }],
      "default_vote_policy": {
        "weight_kind": "RoleWeight",
        "quorum": "70",
        "threshold": [7, 10]
      },
      "proposal_bond": "1000000000000000000000000",
      "proposal_period": "604800000000000",
      "bounty_bond": "0",
      "bounty_forgiveness_period": "0"
    }
  }
}' --accountId deployer.near --deposit 10
```

#### Custom Multisig Contract
- **Pros:**
  - Minimal attack surface
  - Custom business logic (e.g., time-based rotation, geo-restrictions)
  - Lower gas costs for simple operations
  - Can implement novel schemes (threshold signatures, MPC)
- **Cons:**
  - Requires audit before use
  - No existing UI (must build custom interface)
  - Upgrade complexity
  - Maintenance burden

**Key features to implement:**
```rust
pub struct Multisig {
    owners: UnorderedSet<AccountId>,
    threshold: u8,
    pending_requests: HashMap<u64, Request>,
    confirmations: HashMap<u64, HashSet<AccountId>>,
    timelock_duration: u64,
    executed: HashMap<u64, bool>,
}

impl Multisig {
    pub fn propose_action(&mut self, action: Action) -> u64;
    pub fn confirm(&mut self, request_id: u64);
    pub fn execute(&mut self, request_id: u64);
    pub fn change_threshold(&mut self, new_threshold: u8);
    pub fn add_owner(&mut self, owner: AccountId);
    pub fn remove_owner(&mut self, owner: AccountId);
}
```

#### Decision Matrix
| Criteria | AstroDAO | Custom Multisig |
|----------|----------|-----------------|
| Time to deploy | 1 day | 2-4 weeks |
| Audit required | No | Yes |
| UI availability | Yes | Build required |
| Flexibility | Medium | High |
| Gas efficiency | Medium | High |
| Community trust | High | Must establish |
| Upgrade path | Built-in | Must implement |

**Recommendation:** Start with AstroDAO for immediate needs. Consider custom multisig only if you have unique requirements that AstroDAO cannot meet.

---

### 6. Audit Integration Process

#### Pre-Deployment Audit Workflow
1) **Audit preparation (2 weeks before)**
   - Freeze code version for audit
   - Prepare documentation: architecture, threat model, invariants
   - Create audit branch with commit hash
   - Deploy to testnet for auditor access

2) **During audit (2-4 weeks)**
   - Dedicated point of contact for auditor questions
   - Daily check-ins for critical findings
   - No production deployments of audited code

3) **Findings integration**
   - Classify findings: Critical → High → Medium → Low → Informational
   - Create fix branches for each finding category
   - Required actions by severity:
     - **Critical/High:** MUST fix before mainnet deployment
     - **Medium:** SHOULD fix or document acceptance with mitigation
     - **Low/Info:** MAY fix based on effort/risk assessment

4) **Remediation verification**
   ```bash
   # Document each fix
   git commit -m "fix(audit): <AUDIT-001> <description>
   
   Severity: High
   Auditor: <firm>
   Fix: <what was changed>
   Test: <how it was validated>"
   ```

5) **Re-audit requirements**
   - Critical fixes: mandatory spot re-audit
   - High fixes: auditor review via diff
   - Medium/Low: self-certify with test evidence

6) **Deployment readiness**
   - Audit report published with remediations
   - All Critical/High findings resolved
   - Medium findings documented with decisions
   - Final auditor sign-off obtained

#### Post-Deployment Audit Actions
- Monitor for issues similar to findings in other protocols
- Schedule follow-up audit for next major version
- Add audit-inspired invariant checks to monitoring

#### Audit Checklist Template
```yaml
audit_tracking:
  firm: <auditor_name>
  commit: <git_sha>
  start_date: <date>
  end_date: <date>
  findings:
    - id: AUDIT-001
      severity: High
      title: "Reentrancy in withdraw"
      status: Fixed
      fix_commit: <sha>
      verified_by: <auditor>
  sign_offs:
    - auditor: <name>
    - engineering: <name>
    - security_council: <name>
```

---

### 7. Performance Benchmarks and Metrics

#### Deployment Performance Targets
| Operation | Expected Time | Gas Usage | Storage |
|-----------|--------------|-----------|---------|
| New contract deployment | 2-5 seconds | 10-20 TGas | 200-500 KB |
| Code upgrade (no migration) | 2-5 seconds | 15-25 TGas | Delta only |
| Small migration (<1000 items) | 5-10 seconds | 50-100 TGas | Variable |
| Large migration (>10000 items) | Multiple txs | 250-300 TGas each | Variable |
| Emergency pause | 1-2 seconds | 5-10 TGas | Minimal |

#### Gas Profiling Commands
```bash
# Measure deployment gas
near deploy --accountId <contract> --wasmFile contract.wasm \
  --initFunction new --initArgs '{}' | grep "Transaction cost"

# Profile specific function
near call <contract> <method> '<args>' --gas 300000000000000 \
  --accountId <caller> | grep -A2 "gas_burnt"

# View transaction details
near tx-status <tx_hash> --accountId <account>
```

#### Storage Metrics
```bash
# Check before deployment
near state <contract> | jq '.storage_usage'

# Monitor growth rate
watch -n 60 'near state <contract> | jq ".storage_usage"'

# Calculate storage cost
echo "scale=2; $(near state <contract> | jq '.storage_usage') * 0.00001 / 1000" | bc
```

#### Performance Monitoring Dashboard
Key metrics to track:
- **Deployment duration:** Time from initiation to verified on-chain
- **Gas efficiency:** Actual vs allocated gas per operation
- **Storage growth rate:** Bytes/day, projection to limits
- **Migration throughput:** Items processed per transaction
- **RPC latency:** Time to confirm deployment
- **Ceremony duration:** End-to-end time for governed deployment

#### Optimization Guidelines
1) **Contract size optimization**
   ```toml
   [profile.release]
   opt-level = "z"  # Optimize for size
   lto = true       # Link-time optimization
   strip = true     # Strip symbols
   ```

2) **Migration batching**
   ```rust
   const BATCH_SIZE: u64 = 100;  // Items per transaction
   
   pub fn migrate_batch(&mut self, start_index: u64) {
       let end = min(start_index + BATCH_SIZE, self.total_items);
       for i in start_index..end {
           self.migrate_item(i);
       }
   }
   ```

3) **Gas reservation patterns**
   ```rust
   const BASE_GAS: Gas = 20_000_000_000_000;
   const GAS_PER_ITEM: Gas = 5_000_000_000_000;
   
   let required_gas = BASE_GAS + (items.len() as u64 * GAS_PER_ITEM);
   assert!(env::prepaid_gas() >= required_gas, "Insufficient gas");
   ```

#### Warning Thresholds
- Contract size > 4MB: Consider splitting
- Deployment gas > 200 TGas: Optimize or split initialization
- Storage usage > 50MB: Plan for archival
- Migration time > 60s: Implement resumable pattern
- Storage growth > 1MB/day: Review retention policy

---

### 8. Release Engineering (Deterministic builds and verification)
Deterministic Builds:
- Pin Rust toolchain and `near-sdk-*` versions. Build in a pinned container image. Disable incremental builds.
- Produce artifact: Wasm binary, SHA-256 digest, size, Cargo.lock, toolchain versions. Co-sign a release manifest (engineering + security).

Example configuration (Cargo.toml):
```toml
[profile.release]
codegen-units = 1
opt-level = "z"
lto = true
debug = false
panic = "abort"
overflow-checks = true
```

Example pinned toolchain (rust-toolchain.toml):
```toml
[toolchain]
channel = "1.75.0"
components = ["rustfmt", "clippy"]
targets = ["wasm32-unknown-unknown"]
```

Sample reproducible Dockerfile:
```dockerfile
FROM rust:1.75.0
RUN rustup target add wasm32-unknown-unknown
WORKDIR /build
COPY . .
RUN cargo build --target wasm32-unknown-unknown --release
```

Verification (always compare on-chain code hash to artifact digest):
- Compute SHA-256 locally: `sha256sum contract.wasm`
- Read on-chain code hash (CLI): `near state <contract>`
- Read on-chain via RPC:
```bash
curl -s -X POST <rpc-url> -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"query","params":{"request_type":"view_code","account_id":"<contract>","finality":"final"},"id":1}'
```
- Compare digests; handle encoding (hex vs base58) as needed in your verification script. Treat mismatches as incidents.

Hash Verification SOP (copy-paste):
1) Compute local SHA-256 (hex) of the artifact:
   ```bash
   sha256sum contract.wasm | awk '{print $1}'
   ```
2) Compare against on-chain code bytes using only standard libraries:
   ```bash
   cat > verify_code_hash.py << 'PY'
   import json, sys, base64, hashlib, urllib.request
   if len(sys.argv) != 3:
       print("usage: python3 verify_code_hash.py <rpc_url> <contract>")
       sys.exit(2)
   rpc_url, account_id = sys.argv[1], sys.argv[2]
   payload = {
       "jsonrpc":"2.0",
       "method":"query",
       "params": {"request_type":"view_code", "account_id": account_id, "finality":"final"},
       "id":1
   }
   req = urllib.request.Request(rpc_url, data=json.dumps(payload).encode(), headers={"Content-Type":"application/json"})
   resp = urllib.request.urlopen(req).read()
   result = json.loads(resp)["result"]
   onchain_hex = hashlib.sha256(base64.b64decode(result["code_base64"])) .hexdigest()
   local_hex = hashlib.sha256(open("contract.wasm","rb").read()).hexdigest()
   print("local=", local_hex)
   print("onchain=", onchain_hex)
   print("match=", str(local_hex == onchain_hex).lower())
   PY
   python3 verify_code_hash.py https://rpc.mainnet.near.org <contract>
   ```
3) If `match=false`, treat as an incident: pause, investigate, and do not proceed with user flows.

Supply Chain:
- Lock dependencies (`Cargo.lock`), use `cargo audit`/`cargo deny`, sign commits/tags, and protect branches.
- CI builds artifacts; deployment requires human-in-the-loop signing with hardware wallets.

---

### 9. Secure Deployment Process (end-to-end)
1) Plan and risk assess
   - Define scope, invariants, blast radius; select timelock and rollout plan; schedule window.
2) Build and attest
   - Reproducible build; record SHA-256; co-sign release manifest.
3) Pre-production validation
   - Deploy to Testnet/canary with production-like config; validate cross-contract calls; profile gas; simulate ceremony.
4) Governance proposal
   - Include artifact hash, source commit, audit links, checklist, execution window; start timelock.
5) Day-of deployment
   - Execute via governance account with hardware wallets; verify on-chain code hash live; run smoke tests.
6) Post-deploy monitoring
   - Heightened monitoring (24–72h); confirm invariants; publish public attestation; archive evidence.
   - If quorum temporarily lost (e.g., signer travel or device failure), freeze non-essential governance actions and prioritize key recovery/rotation SOP (Section 4). Document deviation and owner approvals.

Embedded deployment checklist provided in Section 14.

---

### 10. Upgradability and Migrations (patterns and safeguards)
When and how to allow upgrades:
- Immutable by default for the most critical contracts; future changes require new deployments and migration tooling. Remove full-access keys post-deploy.
- Governance-gated external upgrade: DAO/multisig executes a deploy action that replaces the code on the account.
- Governance-gated self-upgrade: contract exposes `upgrade(code: Vec<u8>)` guarded by governance, timelock, and pre-committed code hash.
- Commit–reveal: store expected code hash in advance; later reveal bytes; contract verifies before deploying.
- Factory pattern: deploy new versions via a factory and migrate state via cross-contract calls when appropriate.

Self-upgrade snippet (Rust, near-sdk):
```rust
#[near_bindgen]
impl Contract {
    #[private]
    pub fn upgrade(&mut self, code: Vec<u8>) {
        self.assert_governance_allows_upgrade();
        self.assert_timelock_elapsed();
        self.assert_code_hash_precommitted(&code);
        Promise::new(env::current_account_id()).deploy_contract(code);
    }
}
```

Migrations:
- Maintain explicit storage version; verify in `init`/`migrate` and upgrade paths.
- Make migrations idempotent and resumable; respect the 300 TGas per-transaction limit and 100MB storage limit.
- Rehearse migrations with production-like data volumes; create and test rollback plans.

---

### 11. On-Chain Safety Rails (pausability, events, rate limits)
- Ownership and roles: explicitly model owner, governance, and guardian roles; log role changes.
- Pausability: critical entrypoints respect a pause switch controlled by the guardian; pause is narrow and reversible.
- Code-hash allowlist: self-upgrade accepts only pre-committed hashes.
- Rate limits: enforce minimum time between upgrades; cap frequency.
- Events: emit NEP-297 compatible JSON events for proposals, approvals, upgrades, pauses, and migrations; include version fields.
- Storage versioning: track and validate schema versions; prevent unsafe access when versions mismatch.
- Gas and state size: ensure functions fit within gas limits; avoid unbounded loops; monitor storage usage growth.

---

### 12. Signing Ceremony (people, process, evidence)
People and Devices:
- Hardware wallets only; updated firmware; provenance verified. Hardened workstations, isolated network/VPN, minimal extensions.

Process:
- Dry-run using the exact proposal bundle. During execution, independent verifiers compute artifact hash and compare to on-chain `code_hash` immediately after deployment.

Evidence:
- Record minutes, signer list, timestamps, tx hashes, artifact digests, source commit, audit links. Publish a public attestation for major changes.

Stakeholder communication template (pre/post):
```text
Subject: [NF] Scheduled <Deployment/Upgrade> for <contract> on <date/time UTC>

What: <brief description>
Why: <benefit/risk mitigation>
When: <start-end window>
Impact: <none/brief read-only disruption/etc>
Rollbacks: <link to plan>
Contacts: <on-call name/channel>
Artifacts: manifest <url>, commit <sha>, audit <link>
```
```text
Subject: [NF] Completed <Deployment/Upgrade> for <contract> — Verified

Summary: <1-2 lines>
Tx: <explorer link>
Hashes: wasm sha256=<hash>
Monitoring: heightened for 72h, current status green
Issues: <none/notes>
```

---

### 13. Monitoring, Alerting, and Incident Response
Monitoring targets:
- Contract metrics (throughput, error rate, gas), state invariants, NEP-297 events
- Access key changes, storage usage and balance for storage staking
- RPC health, indexer lag, cross-contract failures, receipt delays during congestion

Alerting:
- Page on pause toggles, upgrades, invariant failures, anomalous error spikes; use severity tiers and clear escalation paths.

Incident response:
- Triggers: code-hash mismatch, exploit indicators, key compromise, RPC tampering
- Actions: assess → contain (pause) → communicate internally → choose rollback vs hotfix → public comms → forensics → postmortem with corrective actions

Drills:
- Quarterly game-days for pause/rollback and upgrade rehearsal; validate monitoring during shard congestion or high gas events.

---

### 14. Embedded Checklists
Deployment (mainnet):
- Pre-commit: risk assessment; internal review; audits addressed; dependencies pinned; tests (unit/simulation/cross-contract)
- Build & attest: clean pinned toolchain; reproducible build; Wasm SHA-256 recorded; release manifest co-signed
- Pre-prod: Testnet/canary; identical config; storage migration tested; cross-contract calls validated; gas profiled; dry-run ceremony
- Governance: proposal includes artifact hash, commit, audit links; timelock set; comms plan ready
- Day-of: secure environment; execute via governance account; verify on-chain code hash; smoke tests pass
- Post: heightened monitoring; invariants hold; public attestation; archive artifacts/logs; audit access keys; ensure events indexed

Upgrade (code or state):
- Prep: classify change (emergency, bugfix, feature, migration); migration plan idempotent/resumable; rollback plan ready; events in place
- Build & verify: reproducible build; SHA-256 recorded; manifest co-signed; expected on-chain code hash computed
- Testing: unit/integration/simulation; Testnet rehearsal; canary mainnet if feasible
- Governance: threshold met; timelock ≥ policy; stakeholder/public comms prepared
- Execute: secure ceremony; perform upgrade; verify on-chain code hash; post-migration invariants pass
- Post: heightened monitoring; publish attestation; archive evidence

---

### 15. Templates
Release Manifest (YAML):
```yaml
project: <name>
version: <semver>
source_commit: <git-sha>
artifact:
  wasm_sha256: <hex>
  size_bytes: <int>
  build_cmd: "cargo build --target wasm32-unknown-unknown --release"
  toolchain:
    rustc: <version>
    near_sdk: <version>
    wasm_opt: <version>
attestations:
  engineering: <name-signature-or-url>
  security: <name-signature-or-url>
created_at: <iso8601>
```

Governance Proposal (text):
```text
Title: Upgrade <contract> to v<version>
Summary: <one-paragraph business/tech rationale>
Artifact: wasm sha256=<hash>, source=<commit>, manifest=<url>
Scope: <methods/features>, Migration: <yes/no + plan>, Risks: <top risks>
Timeline: propose=<date>, timelock=<duration>, execute=<window>
Rollbacks: <plan>
Contacts: proposer=<name>, reviewers=<names>, on-call=<name>
```

Public Attestation (post-change):
```text
On <date>, we deployed/upgraded <contract> to v<version>.
Wasm sha256: <hash> (matches manifest)
Tx: <explorer-link>
Source commit: <commit-url>
Audits: <links>
No anomalies detected during the 72h heightened monitoring window.
```

Postmortem Template:
```text
Summary: <what happened>
Impact: <users, funds, duration>
Timeline: <detailed timestamps>
Root Cause: <technical + organizational>
What Worked: <controls that helped>
What Failed: <gaps>
Action Items: <owned, dated, measurable>
```

Change Classification (matrix):
- Emergency: pause-only or narrowly scoped hotfix; higher guardian threshold; no arbitrary upgrade
- Bugfix: minor logic changes; standard timelock
- Feature: new functionality; longer timelock; broader testing
- Migration: schema changes; rehearsed and resumable; extended monitoring

---

### 16. Command and RPC References
NEAR CLI (examples):
```bash
# Compute local artifact digest
sha256sum contract.wasm

# Inspect account state (includes code_hash)
near state <contract>

# Deploy a contract
near deploy --accountId <contract> --wasmFile contract.wasm

# Remove a full-access key
near delete-key <account> <public-key>

# Add a function-call key with restricted methods and allowance
near add-key <account> <public-key> \
  --contract-id <contract> --method-names <csv-methods> --allowance <amount>

# Call a self-upgrade method (if implemented)
near call <contract> upgrade '{"code":"<base64>"}' \
  --accountId <governance> --gas 300000000000000
```

RPC (code hash):
```bash
curl -s -X POST <rpc-url> -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"query","params":{"request_type":"view_code","account_id":"<contract>","finality":"final"},"id":1}'
```

Testing (near-workspaces): prefer `near-workspaces-rs` for integration and cross-contract scenarios.

---

### 17. Roadmap (30/60/90 days)
Days 1–30: constitute Security Council; set thresholds and timelocks; define policies; pin build toolchains; create manifest template; set up monitors and dashboards.

Days 31–60: implement contract events, pausability, optional self-upgrade commit–reveal; build verification scripts; run Testnet ceremonies.

Days 61–90: first mainnet deployment under this process; conduct postmortem; refine thresholds; schedule quarterly drills.

---

### 18. Appendix (terms, tools, references)
Terms: multisig, timelock, deterministic build, artifact hash, commit–reveal, NEP-297.

Tools: near-cli, near-api-js, near-workspaces, NEAR Explorer, NEAR Lake/Indexers, cargo-audit, cargo-deny.

References: consult NEAR protocol documentation for current protocol config (gas limits, storage costs, and code size constraints) and NEP-297 event specification.
