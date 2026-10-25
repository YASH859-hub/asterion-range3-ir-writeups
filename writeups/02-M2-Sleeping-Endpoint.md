# IR-Range3-M2 — Sleeping Endpoint

> **Case ID:** `AST-MEM-20260628-092012`  
> **Handoff:** M2 → M3 handoff  
> **Scope:** Local lab gateway on 127.0.0.1:32762. Hibernation image and recovered slack were analysed using supplied tools; raw ephemeral credential is not included.

**Source note:** This Markdown write-up is derived from the completed Word report for the corresponding controlled Asterion Range 3 lab. The evidence images are the terminal screenshots embedded in that report. Interpret findings within the simulated lab context.

---

## 1. Executive Summary

IR-Range3-M2, “Sleeping Endpoint,” continues the verified Scenario 1 handoff from IR-Range3-M1. The M2 exercise creates a controlled hibernation-evidence artifact, requires inspection of compressed-page allocation slack, and forwards the account, timestamp, transaction identifiers, and evidence hashes to IR-Range3-M3 for PostgreSQL WAL scoping.

> FINAL RESULT
> The supplied local Nuclei template matched once; validate-attack.sh passed; the recovered summary contained the expected PostgreSQL account, compromise time, and three transaction identifiers; the evidence SHA-256 manifest verified; and validate-completion.sh returned “[PASS] Scenario 3 handoff validated.”

### Outcome at a glance

| CHECKPOINT | RESULT | EVIDENCE |
| --- | --- | --- |
| M2 setup / clean state | PASS | E04 |
| Attack template asset structure | PASS; one manifest hash mismatch recorded | E01 |
| First trigger attempt | No match; prerequisite issue diagnosed | E05–E07 |
| Controlled trigger after permission correction | PASS; 1 match | E08 |
| Memory-fragment recovery | Completed; expected indicators visible in sanitized summary | E09 |
| Evidence SHA-256 verification | All listed saved artifacts reported OK | E10 |
| M2-to-M3 handoff / completion validator | PASS | E10 |

### Key recovered indicators

| INDICATOR | RECOVERED VALUE |
| --- | --- |
| PostgreSQL account | ast_research_sync |
| True compromise start (UTC) | 2026-06-28T09:20:12.043864+00:00 |
| Suspicious transaction IDs | 884201, 884219, 884227 |
| Researcher | dr.mira.rao@asterion.example |
| Workstation | AST-LNX-RSCH-017 |
| Database session | PGS-AST-017-77A1 |
| Campaign marker | MDRN-7F3A |

> CREDENTIAL HANDLING
> The scenario reported an ephemeral credential as recovered. The credential value is deliberately omitted from this report and was not used to authenticate. The generated handoff stores a credential-evidence SHA-256, not the raw credential.

## 2. Scope, Environment & Evidence Handling

Scope was limited to the local M2 virtual-machine challenge package and its loopback services. The supplied Nuclei template was run against 127.0.0.1; no external endpoint was scanned. The endpoint events and account/transaction records are simulated challenge telemetry, not evidence that a real external system was contacted.

| COMPONENT | OBSERVED / CONFIGURED VALUE |
| --- | --- |
| Scenario | IR-Range3-M2 — Sleeping Endpoint |
| Terminal hostname during M2 workflow | ir-range3-m2 |
| M2 attack gateway | TCP 32762; local target 127.0.0.1 |
| PostgreSQL evidence simulator | 127.0.0.1:15432 |
| Nuclei | v3.11.1; templates reported as validated |
| Inbound handoff ID | R3-M1-AST-017 |
| Case directory | ~/AST-MEM-20260628-092012 |
| Handoff output | /var/lib/asterion/handoff/IR-Range3-M2-to-M3.json |

### Evidence handling approach

- Preserved the clean status, installed scenario configuration, and inbound M1-to-M2 handoff before the investigation progressed.
- Copied hibernation image and metadata, runtime findings, service/access logs, and incident state into the case folder while retaining original timestamps/metadata where cp --preserve=all was used.
- Generated a hibernation-structure JSON, recovered-slack JSON, sanitized recovery summary, timeline JSON, and M2-to-M3 handoff JSON.
- Generated notes/SHA256SUMS.txt from the saved case folder and checked each listed file with sha256sum -c.
### Important execution boundary

> VM CLONING WAS DEFERRED BY OPERATOR INSTRUCTION
> The terminal initially showed ir-range3-m1. The hostname was changed to ir-range3-m2 and the M2 setup was then run on the current VM before cloning. The M2 setup changes Asterion runtime files and services, so the live VM should not be described as an untouched M1 baseline after this point. The M1 report and its saved case folder remain separate documentation artifacts. For future replays, create a full VM clone or snapshot before running a scenario that mutates shared runtime state.

### Integrity exception

> ONE MANIFEST HASH MISMATCH WAS RECORDED
> FILE-MANIFEST.sha256 reported a mismatch for Attack-VM/IR-Range3-M2/IR-Range3-M2-Attack-Template.yaml. The package-provided verify-attack-assets.sh nevertheless returned “[PASS] Attack assets verified,” and the template later validated and produced one controlled local match. The report preserves this exception; the manifest was not edited to conceal it.

## 3. Procedure & Commands

### Step 1 — Check package integrity and syntax

The archive was extracted and the manifest checked. All listed package files except the M2 attack-template YAML matched the manifest. The setup and validation scripts passed bash syntax checks, and the package-provided asset verifier passed.

```bash
cd ~/Downloads/IR-Range3-M2-Sleeping-Endpoint
sha256sum -c FILE-MANIFEST.sha256
bash Attack-VM/IR-Range3-M2/verify-attack-assets.sh
for f in setup.sh reset.sh validate-setup.sh validate-attack.sh validate-completion.sh; do
  bash -n "$f" || exit 1
  echo "$f: syntax OK"
done
```

**E01 — **Package contents, manifest warning, template hash values, built-in asset verification, and syntax-check results.

### Step 2 — Set the M2 hostname and correct script permissions

The M2 hostname was configured as ir-range3-m2. The first direct execution of setup.sh and validate-setup.sh returned “Permission denied” because the scripts did not have execute permissions. The script permission bits were then set and setup was re-run.

```bash
sudo hostnamectl set-hostname ir-range3-m2
cd ~/Downloads/IR-Range3-M2-Sleeping-Endpoint
chmod +x setup.sh reset.sh validate-setup.sh validate-attack.sh validate-completion.sh
ls -l setup.sh validate-setup.sh
```

**E02–E03 — **Hostname change and the initial executable-permission failure.

### Step 3 — Install and validate clean M2 state

```bash
sudo ./setup.sh
sudo ./validate-setup.sh
```

Setup created the M2 gateway and PostgreSQL simulator service entries. The setup output showed the Scenario 1 handoff identifier R3-M1-AST-017 and gateway TCP 32762. The clean-state validator returned “[PASS] Clean setup validated.”

**E04 — **Successful setup, gateway port, handoff identifier, and clean-state validation.

### Step 4 — Validate and attempt the local scenario trigger

```bash
export PATH="$HOME/.local/bin:$PATH"
cd ~/Downloads/IR-Range3-M2-Sleeping-Endpoint
CASE="$HOME/AST-MEM-20260628-092012"
TEMPLATE="./Attack-VM/IR-Range3-M2/IR-Range3-M2-Attack-Template.yaml"
nuclei -duc -validate -t "$TEMPLATE"
export PATH="$HOME/.local/bin:$PATH"
CASE="$HOME/AST-MEM-20260628-092012"
TEMPLATE="./Attack-VM/IR-Range3-M2/IR-Range3-M2-Attack-Template.yaml"
nuclei -duc -u 127.0.0.1 -t "$TEMPLATE" -retries 0 -timeout 5 \
  -o "$CASE/logs/nuclei-trigger.txt"
sudo ./validate-attack.sh
```

The first scan produced no matches. The validator reported that /var/lib/asterion/memory/AST-LNX-RSCH-017.hiber was missing. The exercise was not marked successful at this stage; service status and request logs were checked to identify the failed prerequisite.

**E05 — **First Nuclei scan: template loaded but no results; the hibernation artifact was absent.

### Step 5 — Diagnose HTTP 412 and check prerequisites

```bash
curl -i --max-time 3 http://127.0.0.1:32762/health
curl -i --max-time 3 http://127.0.0.1:32762/api/v1/status
sudo tail -n 20 /var/log/asterion/gateway-access.log
sudo systemctl status asterion-range3-m2-gateway.service --no-pager -l
sudo cat /etc/asterion/IR-Range3-M1-to-M2.json
sudo ss -ltnp | grep ':15432'
```

Health returned HTTP 200 and the gateway status was ready, but the incident remained inactive and hibernate_image_ready was false. The access log showed the attempted hibernate endpoint returned HTTP 412 (Precondition Failed). All handoff/configuration comparisons passed, the PostgreSQL simulator listened on 127.0.0.1:15432, and the helper was inspected next.

**E06–E07 — **The gateway precondition failure, all handoff comparisons, PostgreSQL listener, and token-helper metadata.

### Step 6 — Correct token-helper access and retrigger

The helper was owned by root:root with mode 0750, preventing the scenario account astresearch (group astops) from executing it. The group was changed to astops, retaining restrictive 0750 permissions. Both helper execute access and database-configuration read access were tested as astresearch.

```bash
sudo chgrp astops /usr/local/bin/ast-token-helper
sudo chmod 0750 /usr/local/bin/ast-token-helper
id astresearch
ls -l /usr/local/bin/ast-token-helper
sudo -u astresearch test -x /usr/local/bin/ast-token-helper \
  && echo "PASS: helper is executable"
sudo -u astresearch test -r /etc/asterion/db-ephemeral.json \
  && echo "PASS: database configuration is readable"

export PATH="$HOME/.local/bin:$PATH"
CASE="$HOME/AST-MEM-20260628-092012"
TEMPLATE="./Attack-VM/IR-Range3-M2/IR-Range3-M2-Attack-Template.yaml"
nuclei -duc -u 127.0.0.1 -t "$TEMPLATE" -retries 0 -timeout 5 \
  -o "$CASE/logs/nuclei-trigger.txt"
sudo ./validate-attack.sh
```

After the permissions correction, the supplied template found one match and the validator returned “[PASS] Live attack and hibernation evidence validated.”

**E08 — **Successful access tests, one Nuclei match, and passed attack-evidence validation.

## 4. Forensic Recovery & Findings

### Step 7 — Preserve incident artifacts

Evidence was copied to ~/AST-MEM-20260628-092012 with baseline, evidence, logs, analysis, handoff, and notes subdirectories. Copied artifacts included the hibernation image, swap/hibernation metadata, runtime findings, hibernation and PostgreSQL event streams, gateway access/journal records, the original M1-to-M2 handoff, scenario configuration, and incident status.

```bash
CASE="$HOME/AST-MEM-20260628-092012"
mkdir -p "$CASE"/{baseline,evidence,logs,analysis,handoff,notes}
# Preserve each source artifact with metadata, for example:
sudo cp --preserve=all /var/lib/asterion/memory/AST-LNX-RSCH-017.hiber "$CASE/evidence/"
sudo cp --preserve=all /var/lib/asterion/memory/AST-LNX-RSCH-017.swapmeta.json "$CASE/evidence/"
sudo cp --preserve=all /var/lib/asterion/evidence/runtime-findings.json "$CASE/evidence/"
sudo cp --preserve=all /var/log/asterion/hibernate-events.jsonl "$CASE/logs/"
sudo cp --preserve=all /var/log/asterion/postgres-events.jsonl "$CASE/logs/"
sudo chown -R "$USER":"$USER" "$CASE"
```

### Step 8 — Inspect the hibernation image, recover slack, and build a timeline

```bash
python3 challenge/tools/inspect-hibernation.py \
  "$CASE/evidence/AST-LNX-RSCH-017.hiber" --json \
  > "$CASE/analysis/hibernation-structure.json"
python3 challenge/tools/recover-slack.py \
  "$CASE/evidence/AST-LNX-RSCH-017.hiber" \
  -o "$CASE/analysis/recovered-slack.json"
python3 challenge/tools/build-timeline.py "$CASE/logs" \
  -o "$CASE/analysis/timeline.json"
```

The recovered-slack summary confirmed the account, compromise time, transaction identifiers, researcher, workstation, database session, and campaign marker. A separate summary stored only whether the ephemeral credential was recovered and its SHA-256, not the credential value. Hibernation-structure and timeline JSON outputs were retained in the case folder.

**E09 — **Sanitized recovery summary. The screenshot deliberately displays only the credential hash, never the raw credential.

### Verified recovered indicators

| FIELD | VALUE | WHY IT MATTERS FOR M3 |
| --- | --- | --- |
| PostgreSQL account | ast_research_sync | Scope account and related database activity. |
| True compromise start | 2026-06-28T09:20:12.043864+00:00 | Use as the authoritative start of the review window. |
| Suspicious transactions | 884201, 884219, 884227 | Target records for PostgreSQL WAL correlation. |
| Database session | PGS-AST-017-77A1 | Correlate redacted database records with memory evidence. |
| Researcher | dr.mira.rao@asterion.example | Associate the activity with the correct account context. |
| Workstation | AST-LNX-RSCH-017 | Tie the memory and endpoint records to the affected device. |
| Campaign marker | MDRN-7F3A | Cross-reference the scenario’s distribution/research artifacts. |

### Step 9 — Generate M2-to-M3 handoff

```bash
sudo python3 challenge/tools/build-m3-handoff.py \
  --case-id AST-MEM-20260628-092012 --analyst "$USER"
sudo cp --preserve=all \
  /var/lib/asterion/handoff/IR-Range3-M2-to-M3.json "$CASE/handoff/"
sudo chown -R "$USER":"$USER" "$CASE"
cat "$CASE/handoff/IR-Range3-M2-to-M3.json"
```

The handoff identifies IR-Range3-M2 → IR-Range3-M3 and carries the recovered PostgreSQL account, compromise start, transaction IDs, workstation/session references, hibernation-image path and SHA-256, credential-evidence SHA-256, and the preservation note directing the next stage to scope PostgreSQL WAL evidence.

| HANDOFF FIELD | VALUE / M3 USE |
| --- | --- |
| Handoff ID / route | R3-M2-PG-884201; IR-Range3-M2 → IR-Range3-M3 |
| Database scope | ast_research_sync; session PGS-AST-017-77A1 |
| Time and transaction scope | 2026-06-28T09:20:12.043864+00:00; transactions 884201, 884219, 884227 |
| Integrity references | Hibernation-image SHA-256 and credential-evidence SHA-256 are carried in handoff; raw credential is not needed for the handoff. |

> NEXT-STAGE OBJECTIVE
> In M3, correlate the specified PostgreSQL transaction IDs and session with WAL evidence and preserve the matching database records. The M2 completion pass is a handoff gate, not a claim that WAL review or remediation is complete.

### Step 10 — Hash the case folder and validate completion

```bash
find "$CASE" -type f \
  ! -path "$CASE/notes/SHA256SUMS.txt" -print0 \
  | sort -z | xargs -0 sha256sum > "$CASE/notes/SHA256SUMS.txt"
sha256sum -c "$CASE/notes/SHA256SUMS.txt"
sudo ./validate-completion.sh
```

Every listed case artifact shown in the final terminal evidence reported OK. The completion validator then returned “[PASS] Scenario 3 handoff validated.”

**E10 — **M2-to-M3 handoff content, successful hash verification, and final validator pass.

## 5. Evidence Register & Final State

| CASE-PATH ARTIFACT | PURPOSE / RESULT |
| --- | --- |
| baseline/clean-status.json | Preserved clean-state status from before the scenario. |
| baseline/incident.json | Incident-state evidence copied after activation. |
| baseline/M1-to-M2-handoff.json | Original input handoff for continuity checks. |
| baseline/scenario.json | Scenario configuration used for field comparisons. |
| evidence/AST-LNX-RSCH-017.hiber | Primary hibernation artifact. |
| evidence/AST-LNX-RSCH-017.swapmeta.json | Hibernation metadata. |
| evidence/runtime-findings.json | Runtime evidence findings from the scenario. |
| logs/hibernate-events.jsonl | Hibernation activity log. |
| logs/postgres-events.jsonl | Database-simulator event log. |
| logs/gateway-access.log and gateway-journal.txt | Gateway requests and service journal. |
| logs/nuclei-trigger.txt | Supplied local template trigger output. |
| analysis/hibernation-structure.json | Parsed image structure result. |
| analysis/recovered-slack.json | Recovered indicators; contains the simulated credential—keep private. |
| analysis/recovered-slack-summary.json | Sanitized findings and credential hash only. |
| analysis/timeline.json | Correlated event timeline output. |
| handoff/IR-Range3-M2-to-M3.json | Validated outgoing handoff. |
| notes/SHA256SUMS.txt | Manifest used to verify the saved case files. |

### Final validation record

> COMPLETION VALIDATOR
> Terminal output: “[PASS] Scenario 3 handoff validated.” SHA-256 verification reported OK for every case file listed in the screenshot. This validates the M2 lab gate and outgoing handoff; it does not by itself confirm that M3 WAL analysis or remediation has been performed.

### Known limitations and open follow-up

- The M2 attack-template file had a package manifest checksum mismatch. The built-in asset verifier, Nuclei template validation, controlled match, and validators passed, but the mismatch remains an integrity exception worth retaining in the audit trail.
- The initial trigger returned no match and the attempted hibernation endpoint returned HTTP 412. The helper’s group access was corrected; after that correction the supplied template matched and the validator passed.
- The raw recovered credential is sensitive evidence and is intentionally excluded from this report. The recovery output and the validator-created /var/lib/asterion/evidence/recovered-slack-validation.json should also be treated as sensitive. Do not use the credential to authenticate; keep any raw output under strict case access controls.
- M3 should correlate the three transaction IDs and session PGS-AST-017-77A1 with PostgreSQL WAL evidence using the recovered true compromise start time.
- Containment, eradication, workstation rebuild/recovery, and database-WAL review are downstream operational activities. They are not asserted as completed by the M2 completion validator.
## 6. Learning Points

| 1 | Validate the handoff before scoping.<br>The M1-to-M2 handoff anchors the researcher, endpoint, secondary artifact, token-helper command, and campaign marker to the same investigation chain. |
| --- | --- |

| 2 | A healthy service is not the same as a ready scenario.<br>The gateway returned HTTP 200 for health/status while the incident trigger returned HTTP 412 and the hibernation image was not ready. Inspect preconditions and service logs instead of repeatedly triggering. |
| --- | --- |

| 3 | Least-privilege permissions can block a controlled workflow.<br>The helper’s restrictive 0750 mode was appropriate, but its group ownership prevented the intended scenario account from executing it. Correct group access and test effective access as the service user. |
| --- | --- |

| 4 | Compressed-memory slack can retain evidence after a process exits.<br>The recovered hibernation slack exposed the authoritative compromise timestamp, PostgreSQL account/session, and transaction IDs that were necessary to scope the next stage. |
| --- | --- |

| 5 | Hash manifests support integrity and chain of custody.<br>Hashing the saved case files and checking the manifest makes later changes detectable. Record package hash exceptions explicitly; do not edit a manifest simply to make verification green. |
| --- | --- |

### Conclusion

The M2 lab gate was completed. The initial trigger issue was diagnosed, the controlled scenario was activated, the hibernation evidence and indicators were recovered, the case artifacts were hashed, and the M2-to-M3 handoff passed validation. The next stage is PostgreSQL WAL evidence scoping in IR-Range3-M3.

## Appendix A — Terminal Evidence Screenshots

The following screenshots are included as direct visual evidence from the terminal session. Captions summarize what each screen supports. Source files are named by evidence ID and timestamp in the report asset package.

### E01  |  Package and initial integrity checks

The M2 package inventory was checked. The manifest reported one mismatch for the attack-template YAML; the package’s built-in attack-asset verifier later passed. Setup-shell syntax also passed.

![Terminal evidence E01](../evidence/M2/E01.png)

*Figure 1. E01 — image(20261024-180454).png*

### E02  |  M2 hostname set and VM identity checked

The terminal shows the hostname being changed to ir-range3-m2. This documents the operator-directed decision to continue on the current VM before making a separate clone.

![Terminal evidence E02](../evidence/M2/E02.png)

*Figure 2. E02 — image(20261024-180736).png*

### E03  |  Initial setup attempt and permission error

The first setup invocation failed because the scripts lacked executable permissions. The permission issue was corrected with chmod; this was a packaging/permission issue, not a setup-script syntax error.

![Terminal evidence E03](../evidence/M2/E03.png)

*Figure 3. E03 — image(20261024-181237).png*

### E04  |  M2 setup and clean-state validation passed

After fixing script execute bits, setup completed. The gateway was reported on TCP 32762 and validate-setup.sh returned [PASS] Clean setup validated.

![Terminal evidence E04](../evidence/M2/E04.png)

*Figure 4. E04 — image(20261024-181416).png*

### E05  |  Initial trigger did not produce evidence

Nuclei validated the template but the first scan had no results. The attack validator correctly reported the missing hibernation artifact; the scenario was not treated as successful.

![Terminal evidence E05](../evidence/M2/E05.png)

*Figure 5. E05 — image(20261024-181944).png*

### E06  |  Gateway diagnosis: precondition failure

Gateway health and status were available. Status showed ready, a verified inbound handoff, incident_active false, and hibernate_image_ready false. The attempted hibernate request returned HTTP 412.

![Terminal evidence E06](../evidence/M2/E06.png)

*Figure 6. E06 — image(20261024-182228).png*

### E07  |  Handoff values matched; simulator and helper inspected

The eight handoff/configuration comparisons passed. PostgreSQL simulator was listening on 127.0.0.1:15432. The token-helper permissions were then investigated as a likely prerequisite.

![Terminal evidence E07](../evidence/M2/E07.png)

*Figure 7. E07 — image(20261024-182554).png*

### E08  |  Helper access corrected; trigger and validator passed

After assigning the helper to astops while retaining mode 0750, access tests passed. The supplied local Nuclei template matched once and validate-attack.sh returned [PASS] Live attack and hibernation evidence validated.

![Terminal evidence E08](../evidence/M2/E08.png)

*Figure 8. E08 — image(20261024-184013).png*

### E09  |  Sanitized memory-recovery summary

Recovered data confirmed the PostgreSQL account, exact compromise timestamp, transaction IDs, researcher, workstation, database session, and campaign marker. The raw ephemeral credential was not included in the report.

![Terminal evidence E09](../evidence/M2/E09.png)

*Figure 9. E09 — image(20261024-184723).png*

### E10  |  Handoff, evidence hashes, and final completion validation

The handoff was written and the SHA-256 manifest verified the saved case artifacts. Final output: [PASS] Scenario 3 handoff validated.

![Terminal evidence E10](../evidence/M2/E10.png)

*Figure 10. E10 — image(20261024-185533).png*

## Appendix B — Sanitized Memory-Recovery Summary

The recovered-slack JSON contains the simulated ephemeral credential and must remain private. The following command creates a separate summary that includes the challenge indicators and records only a boolean plus SHA-256 for the credential. It never prints or copies the raw credential into the summary.

```bash
umask 077
CASE="$HOME/AST-MEM-20260628-092012"
chmod 600 "$CASE/analysis/recovered-slack.json"
python3 - "$CASE/analysis/recovered-slack.json" \
  "$CASE/analysis/recovered-slack-summary.json" <<'PY'
import hashlib, json, sys
source, destination = sys.argv[1:3]
with open(source, encoding="utf-8") as f:
    data = json.load(f)
keys = [
    "postgresql_account", "true_compromise_start", "suspicious_transaction_ids",
    "researcher_account", "workstation_device_id", "database_session_id", "campaign_marker",
]
summary = {key: data.get(key) for key in keys}
credential = data.get("ephemeral_credential")
summary["ephemeral_credential_recovered"] = bool(credential)
summary["ephemeral_credential_sha256"] = (
    hashlib.sha256(credential.encode()).hexdigest() if credential else None
)
with open(destination, "w", encoding="utf-8") as f:
    json.dump(summary, f, indent=2)
    f.write("\n")
print(json.dumps(summary, indent=2))
PY
chmod 600 "$CASE/analysis/recovered-slack-summary.json
```

> SENSITIVE FILES
> Keep analysis/recovered-slack.json and /var/lib/asterion/evidence/recovered-slack-validation.json restricted to the authorized case operator. Do not attempt to authenticate with the recovered credential. The report and handoff carry only its evidence hash.
