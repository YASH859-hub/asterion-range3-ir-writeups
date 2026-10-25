# IR-Range3-M1 — Trojan Research Utility

> **Case ID:** `AST-IR-2026-0017`  
> **Handoff:** M1 → M2 handoff  
> **Scope:** Local lab gateway on 127.0.0.1:32761. APK was hashed and signature-checked, not installed or manually executed.

**Source note:** This Markdown write-up is derived from the completed Word report for the controlled Asterion Range 3 lab. The screenshots are placed inline immediately after the command/output steps they document. Their image assets are stored in `../evidence/M1/` so GitHub can render the repository-relative links. Interpret findings within the simulated lab context.

---

## 1. Executive summary

The IR-Range3-M1 Trojan Research Utility scenario was prepared and validated in the existing Ubuntu VirtualBox environment. The setup validator passed, the local gateway responded on TCP 32761, the APK digest matched the supplied expected SHA-256, and the detached signature verified with OpenSSL. The bundled Nuclei template was validated and used only against 127.0.0.1. The live attack validator passed, incident telemetry was collected and hashed, and the M1-to-M2 handoff was generated and accepted by the completion validator.

| CONTROL / CHECK | OBSERVED RESULT | STATUS |
| --- | --- | --- |
| Setup validation | [PASS] Clean setup validated | PASS |
| Local gateway | HTTP 200 for /health and /api/v1/status | PASS |
| Attack assets | [PASS] Attack assets verified | PASS |
| APK SHA-256 | Matches the expected digest | PASS |
| Detached signature | OpenSSL returned Verified OK | PASS |
| Nuclei template | All templates validated successfully | PASS |
| Live incident telemetry | One match; live-telemetry validator passed | PASS |
| Evidence manifest | Files listed in the case manifest verified OK | PASS |
| M2 handoff | [PASS] Scenario 2 handoff validated | PASS |
| Package manifest integrity | One checksum mismatch warning observed initially | REVIEW |
| Device-local secondary DEX copy | First copy attempt used a non-matching device path | FOLLOW-UP |

## 2. Scope and safety boundaries

This report records the simulated IR-Range3-M1 workflow on the local lab VM. The release endpoint referenced by the scenario is /api/v1/mdm/release on the M1 gateway, TCP 32761. The executed Nuclei command targeted 127.0.0.1. The Android APK was copied and checked with SHA-256 and detached-signature verification; it was not installed or manually executed.

- Case workspace: $HOME/AST-IR-2026-0017.
- Release/package metadata: AST-LABASSIST-2026.06; package com.asterion.labassist.
- Researcher identity recorded by the scenario: dr.mira.rao@asterion.example.
- Workstation identifier: AST-LNX-RSCH-017; campaign marker: MDRN-7F3A.
- Expected and observed APK digest: d685578d5a1f1b024a9217e07cc3fe6b4f4d75658e83c858fe893eaa7bfdae64.
## 3. Execution timeline

| VM time shown | Activity | Recorded result |
| --- | --- | --- |
| 13:05–13:07 | Extracted package, checked shell syntax and manifest, ran setup and setup validator. | Setup passed; one manifest mismatch warning noted. |
| 13:09 | Checked gateway /health and /api/v1/status; verified attack assets. | HTTP 200; state ready; incident_active false before trigger. |
| 13:14 | Copied APK, signature and certificate; verified digest and detached signature. | Digest matched; Verified OK. |
| 13:15–13:17 | Reviewed scenario templates, validator conditions and local release endpoint. | Confirmed expected source value was nuclei. |
| 13:21 | Installed Nuclei v3.11.1, validated supplied template and ran it against localhost. | One match; live telemetry validation passed. |
| 13:23–13:30 | Collected logs and hashes, reviewed events, generated the M2 handoff and re-ran completion validation. | Evidence hashes OK; initial check noted missing handoff, final handoff validator passed. |

## 4. Complete procedure and commands

### 4.1 VM preflight, package verification and setup

The initial preflight recorded the VM hostname, checked that the previous M5 service was inactive, inspected the former Python capability, and confirmed network interfaces before setup. The host-only interface was `192.168.56.103/24`.

```bash
hostname
systemctl is-active aethon-core-monitor 2>/dev/null || true
getcap /usr/bin/python3.14 2>/dev/null || true
ip -br addr
```

![E01 — Package extraction and initial VM/network preflight](../evidence/M1/E01.jpg)

*Figure E01. Initial VM identity and network preflight.*

The package was then checked and installed. One manifest mismatch warning was observed and is retained as an integrity exception rather than silently overwritten.

```bash
cd ~/Downloads/IR-Range3-M1-Trojan-Research-Utility
chmod +x setup.sh validate-setup.sh validate-attack.sh validate-completion.sh reset.sh
bash -n setup.sh && echo "Task 1 setup syntax passed"
sha256sum -c FILE-MANIFEST.sha256
sudo ./setup.sh
sudo ./validate-setup.sh
```

![E02 — Manifest check and clean setup validation](../evidence/M1/E02.jpg)

*Figure E02. Setup and clean-state validation passed; one package-manifest checksum warning was reported.*

Observed result: the previous `aethon-core-monitor` service was inactive, the host-only interface was `192.168.56.103/24`, setup reported “Setup Successfully”, the clean setup validator returned `[PASS]`, and the gateway was listed on TCP `32761`. The initial package manifest check emitted a warning that one computed checksum did not match; the screenshot record does not conclusively identify the affected entry.

### 4.2 Verify local service and attack assets

```bash
curl -i http://127.0.0.1:32761/health
curl -i http://127.0.0.1:32761/api/v1/status
cd ~/Downloads/IR-Range3-M1-Trojan-Research-Utility/Attack-VM/IR-Range3-M1
bash ./verify-attack-assets.sh
command -v nuclei || echo "Nuclei is not installed"
```

![E03 — Gateway health, status, and attack-asset verification](../evidence/M1/E03.jpg)

*Figure E03. Local health/status checks returned HTTP 200, attack assets passed, and Nuclei was not yet installed.*

The validator, scenario configuration, evidence inventory, and technical story were then reviewed read-only before the controlled trigger.

```bash
sed -n '1,220p' validate-attack.sh
cat challenge/config/scenario.json
find challenge/evidence -maxdepth 3 -type f -print | sort
sed -n '1,180p' Range-Technical-Story.md
```

![E04 — Read-only review of validator, configuration, evidence inventory, and scenario notes](../evidence/M1/E04.jpg)

*Figure E04. Read-only review of scenario prerequisites and evidence paths.*

### 4.3 Preserve and verify the release APK

The investigation case folder was created at `$HOME/AST-IR-2026-0017`. The incoming handoff, DNS manifest, clean status, APK, detached signature, and AST-MOBILE-17 certificate were copied before triggering the scenario. The APK digest was compared with the expected package value.

```bash
CASE="$HOME/AST-IR-2026-0017"
mkdir -p "$CASE"/{baseline,package,logs,analysis,volatile,notes}
EXPECTED="d685578d5a1f1b024a9217e07cc3fe6b4f4d75658e83c858fe893eaa7bfdae64"
ACTUAL="$(sha256sum "$CASE/package/AsterionLabAssist-3.6.17.apk" | awk '{print $1}')"
echo "Expected SHA-256: $EXPECTED"
echo "Actual SHA-256:   $ACTUAL"
[ "$ACTUAL" = "$EXPECTED" ] || exit 1
openssl x509 -in "$CASE/package/AST-MOBILE-17.crt" -pubkey -noout > "$CASE/package/mobile-public.pem"
openssl dgst -sha256 -verify "$CASE/package/mobile-public.pem" \
  -signature "$CASE/package/AsterionLabAssist-3.6.17.apk.sig" \
  "$CASE/package/AsterionLabAssist-3.6.17.apk"
```

![E05 — APK SHA-256 match and detached-signature verification](../evidence/M1/E05.jpg)

*Figure E05. Expected and observed APK SHA-256 values matched and OpenSSL returned `Verified OK`. The APK was not installed or executed.*

### 4.4 Validate and run the supplied Nuclei template

The supplied validator expected the incident record to show `incident_active: true` and `source: nuclei`. The templates, validator, and documented local endpoint were inspected before running Nuclei.

```bash
sed -n '1,240p' Attack-VM/IR-Range3-M1/IR-Range3-M1-Attack-Template.yaml
sed -n '1,240p' Attack-VM/IR-Range3-M1/IR-Range3-M1-ServiceAvailabilityTTP.yaml
```

![E06 — TTP templates and validator preconditions reviewed](../evidence/M1/E06.jpg)

*Figure E06. The two supplied templates and the required incident-source condition were reviewed.*

```bash
grep -nE '127\.0\.0\.1|32761|/api/v1/mdm/release|handoff|release' \
  Attack-VM/IR-Range3-M1/IR-Range3-M1-Attack-Template.yaml \
  Attack-VM/IR-Range3-M1/IR-Range3-M1-ServiceAvailabilityTTP.yaml \
  README.md Range-Technical-Story.md
```

![E07 — Local release endpoint and next-stage handoff references](../evidence/M1/E07.jpg)

*Figure E07. The local release endpoint and expected M2 handoff were identified from the supplied artifacts.*

Because Nuclei was not installed, version `3.11.1` was downloaded from the official ProjectDiscovery GitHub release, the archive checksum was checked, and the binary was installed into `$HOME/.local/bin`.

```bash
mkdir -p "$HOME/Downloads/nuclei-install" "$HOME/.local/bin"
cd "$HOME/Downloads/nuclei-install"
curl -fL -o nuclei_3.11.1_linux_amd64.zip \
  https://github.com/projectdiscovery/nuclei/releases/download/v3.11.1/nuclei_3.11.1_linux_amd64.zip
echo "ea63d4ae232808cd7c6bc00d0142428e231fab59dae01042246097d195835ab6  nuclei_3.11.1_linux_amd64.zip" | sha256sum -c -
unzip -o nuclei_3.11.1_linux_amd64.zip nuclei
install -m 0755 nuclei "$HOME/.local/bin/nuclei"
export PATH="$HOME/.local/bin:$PATH"
cd ~/Downloads/IR-Range3-M1-Trojan-Research-Utility
nuclei -validate -t ./Attack-VM/IR-Range3-M1/IR-Range3-M1-Attack-Template.yaml
```

The controlled trigger targeted only the local lab gateway.

```bash
CASE="$HOME/AST-IR-2026-0017"
TEMPLATE="./Attack-VM/IR-Range3-M1/IR-Range3-M1-Attack-Template.yaml"
nuclei -duc -u 127.0.0.1 -t "$TEMPLATE" -retries 0 -timeout 5 \
  -o "$CASE/logs/nuclei-trigger.txt"
sudo ./validate-attack.sh
```

![E08 — Nuclei template validation, one local match, and live telemetry validation](../evidence/M1/E08.jpg)

*Figure E08. Nuclei found one match on `127.0.0.1:32761`, and `validate-attack.sh` reported `[PASS] Live attack and telemetry validated`.*

### 4.5 Collect telemetry and create an evidence manifest

The first collection attempt queried some filenames that were not present. A source inventory then identified the available artifacts, including `file-events.jsonl`, `gateway-access.log`, `token-access.jsonl`, `process-events.jsonl`, and `network-events.jsonl` under `/var/log/asterion`.

```bash
CASE="$HOME/AST-IR-2026-0017"
find "$CASE" -type f ! -path "$CASE/notes/SHA256SUMS.txt" \
  -print0 | sort -z | xargs -0 sha256sum > "$CASE/notes/SHA256SUMS.txt"
sha256sum -c "$CASE/notes/SHA256SUMS.txt"
```

![E09 — Evidence inventory and case-folder hash verification](../evidence/M1/E09.jpg)

*Figure E09. A SHA-256 manifest was generated and the listed case files were checked.*

### 4.6 Review the incident record and timeline

The status record showed machine `IR-Range3-M1` and state `incident_active`, with source `nuclei`. Process telemetry described `labassist-agent` running `/opt/asterion/app/device_agent.py`. Network telemetry included a logical destination of `update-checker.net:443` and training transport `127.0.0.1:32761`. The logical destination is a simulated lab record, not proof that a real public endpoint was contacted.

```bash
cat "$CASE/baseline/incident.json"
for f in file-events.jsonl gateway-access.log token-access.jsonl process-events.jsonl network-events.jsonl; do
  if [ -f "/var/log/asterion/$f" ]; then
    echo "===== $f ====="
    tail -n 30 "/var/log/asterion/$f"
  fi
done
```

![E10 — Incident status and process/network telemetry review](../evidence/M1/E10.jpg)

*Figure E10. Incident state and process/network telemetry were reviewed; one attempted device-artifact copy used a non-matching device path.*

### 4.7 Generate and validate the M2 handoff

The first completion check reported that the M2 handoff was missing. The evidence manifest and the validator's requirements were reviewed before building the handoff.

```bash
sha256sum -c "$CASE/notes/SHA256SUMS.txt"
sudo ./validate-completion.sh
```

![E11 — Evidence manifest passed; initial completion check reported missing handoff](../evidence/M1/E11.jpg)

*Figure E11. Saved artifacts verified, while the first completion check correctly reported that the M2 handoff had not yet been generated.*

```bash
sed -n '1,220p' validate-completion.sh
sed -n '1,180p' challenge/evidence/README.md
grep -n -A 5 -B 3 -E 'handoff|build-m2-handoff|Scenario 2' README.md INTERNAL-IMPLEMENTATION.md Range-Technical-Story.md
```

![E12 — Handoff requirements and supplied builder contract reviewed](../evidence/M1/E12.jpg)

*Figure E12. The completion requirements and M1-to-M2 handoff contract were inspected read-only.*

```bash
sudo /opt/asterion/tools/build-m2-handoff.py \
  --case-id AST-IR-2026-0017 --analyst "$USER"
sudo cat /var/lib/asterion/handoff/IR-Range3-M1-to-M2.json
```

![E13 — M1-to-M2 handoff generated and inspected](../evidence/M1/E13.jpg)

*Figure E13. The generated JSON records handoff ID `R3-M1-AST-017`, source and destination machine IDs, case ID, researcher, workstation, campaign marker, and the next-stage scope indicator.*

```bash
sudo ./validate-completion.sh
```

![E14 — Final completion validator passed](../evidence/M1/E14.jpg)

*Figure E14. Final output: `[PASS] Scenario 2 handoff validated`.*

## 5. Important handoff fields

| FIELD | VALUE RECORDED IN HANDOFF |
| --- | --- |
| handoff_id | R3-M1-AST-017 |
| from_machine / to_machine | IR-Range3-M1 / IR-Range3-M2 |
| case_id / analyst | AST-IR-2026-0017 / vboxuser |
| researcher_account | dr.mira.rao@asterion.example |
| workstation_device_id | AST-LNX-RSCH-017 |
| database token collection command | /usr/local/bin/ast-token-helper --service postgresql --profile research-main --output /tmp/.ast_pg_token |
| campaign_marker | MDRN-7F3A |
| secondary DEX path referenced by builder | /var/lib/asterion/device/AST-LNX-RSCH-017/cache/.secondary.dex |

## 6. Evidence-quality notes and follow-up

> Manifest caveat
> The initial `sha256sum -c FILE-MANIFEST.sha256` output reported “WARNING: 1 computed checksum did NOT match”. The screenshot does not establish conclusively which line caused the warning. Individual checks for the APK, signature, certificate and listed case files were separately successful, but the whole package manifest should not be described as fully clean.

> Device artifact copy caveat
> One collection attempt checked paths containing AST-LAB-RSCH-017. The handoff itself names AST-LNX-RSCH-017, and the subsequent `file` command reported that the expected case-folder APK/secondary DEX paths could not be opened. The final validator passing means the handoff contract passed; it does not prove those device-local artifacts were duplicated into the case folder.

Before retiring or cloning the M1 VM, verify the two source paths below still exist and copy them to the case folder if present. Do not execute either artifact. Then regenerate the case SHA-256 manifest and verify it again.

```bash
CASE="$HOME/AST-IR-2026-0017"; DEVICE="/var/lib/asterion/device/AST-LNX-RSCH-017"
for REL in cache/.secondary.dex data/app.com.asterion.labassist.base.apk; do
  SRC="$DEVICE/$REL"
  if sudo test -f "$SRC"; then sudo cp --preserve=all "$SRC" "$CASE/package/"; else echo "NOT FOUND: $SRC"; fi
done
sudo chown -R "$USER":"$USER" "$CASE/package"
find "$CASE" -type f ! -path "$CASE/notes/SHA256SUMS.txt" -print0 | sort -z | xargs -0 sha256sum > "$CASE/notes/SHA256SUMS.txt"
sha256sum -c "$CASE/notes/SHA256SUMS.txt"
```

## 7. Learning notes

This exercise reinforced that a signed package is not automatically a safe package: signature verification checks trust-key continuity, not behaviour. A clean setup must be validated before the controlled event, and an attack template should be constrained to the local lab target. Incident telemetry is more useful when correlated across gateway, process, file and network records rather than read in isolation. Finally, a handoff validator confirms required metadata is present, but it should not replace a separate check that every intended forensic artifact was actually preserved in the case folder.

## 8. Final status

| DELIVERABLE | FINAL STATUS |
| --- | --- |
| M1 setup and clean-state validation | Passed |
| APK integrity and signature check | Passed |
| Local Nuclei trigger and attack validator | Passed |
| Case manifest for listed files | Passed; manifest caveat noted |
| M1-to-M2 handoff file generated | Passed |
| M1 completion validator | [PASS] Scenario 2 handoff validated |
| Device-local secondary DEX / installed APK copied into case folder | Follow-up recommended due path mismatch in first copy attempt |

Conclusion: Task 1 reached the challenge-defined completion state: the final M1-to-M2 handoff was generated and accepted. This is completion of the setup/trigger/telemetry/handoff workflow, not a claim that the entire Blue Team Write-up has been executed. The captured session did not document full static analysis of the APK, recovery and inspection of the secondary DEX, complete scope determination, or containment/eradication/recovery actions. Preserve the original M1 VM and address the evidence-quality notes before treating the evidence directory as a complete forensic acquisition.
