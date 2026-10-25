# Asterion Range 3 — Incident Response & Digital Forensics Write-ups

A set of five Markdown case reports documenting the controlled **Asterion Range 3** incident-response training sequence, from the M1 Trojan Research Utility through the M5 Fragmented Research Archive. Each report includes the procedure and commands recorded in its corresponding Word report, observed results, investigation notes, handoff status, lessons learned, and linked terminal screenshots.

## Suggested repository name

**`asterion-range3-ir-writeups`**

Alternative: `asterion-range3-incident-response-labs`.

## Contents

| Challenge | Markdown report | Evidence directory | Final recorded outcome |
|---|---|---|---|
| M1 — Trojan Research Utility | [`01-M1-Trojan-Research-Utility.md`](writeups/01-M1-Trojan-Research-Utility.md) | [`evidence/M1/`](evidence/M1/) | M1-to-M2 handoff validator passed |
| M2 — Sleeping Endpoint | [`02-M2-Sleeping-Endpoint.md`](writeups/02-M2-Sleeping-Endpoint.md) | [`evidence/M2/`](evidence/M2/) | M2-to-M3 handoff validator passed |
| M3 — Rewritten Experiment History | [`03-M3-Rewritten-Experiment-History.md`](writeups/03-M3-Rewritten-Experiment-History.md) | [`evidence/M3/`](evidence/M3/) | M3-to-M4 handoff validator passed |
| M4 — Timestamp-Key Exfiltration | [`04-M4-Timestamp-Key-Exfiltration.md`](writeups/04-M4-Timestamp-Key-Exfiltration.md) | [`evidence/M4/`](evidence/M4/) | M4-to-M5 handoff validator passed |
| M5 — Fragmented Research Archive | [`05-M5-Fragmented-Research-Archive.md`](writeups/05-M5-Fragmented-Research-Archive.md) | [`evidence/M5/`](evidence/M5/) | Range 3-to-Range 4 handoff validator passed |

## Investigation sequence

1. **M1:** Verify the research utility package digest and detached signature, trigger the supplied scenario against the local gateway, review simulated incident telemetry, and create the M1-to-M2 handoff.
2. **M2:** Validate the inherited handoff, inspect hibernation evidence, recover the simulated memory-slack indicators, preserve hashes, and create the M2-to-M3 handoff.
3. **M3:** Compare primary and clean PostgreSQL values, verify the backup, correlate WAL/logical-decoding evidence and transaction references, build a timeline, and create the M3-to-M4 handoff.
4. **M4:** Inspect native MongoDB replica-set topology, preserve oplog timing evidence, recover the simulated timestamp channel, and create the M4-to-M5 handoff.
5. **M5:** Analyze FAT16 metadata divergence, recover five deleted/fragmented entries using the supplied workflow, reconstruct the research archive, verify the detached vendor-package signature, and create the Range 4 handoff.

## Evidence and reproducibility notes

- `evidence/M1` through `evidence/M5` contain screenshots embedded in the corresponding finished Word reports. Their filenames use the evidence IDs used in those reports.
- The bundle contains 48 terminal screenshots across the five challenge folders. The Markdown reports preserve the procedures, selected terminal outputs, comparison tables, caveats, and learning points from the completed reports. Where a report documented a checksum or environment discrepancy, it is retained as a caveat rather than silently corrected.
- The lab scenarios were run against loopback addresses such as `127.0.0.1`. Results describe the supplied controlled environment and must not be interpreted as findings about an external production system.
- Some commands require the original challenge package and Asterion lab services to be present. The challenge ZIPs, VM images, raw memory/hibernation artifacts, full database/WAL files, raw recovered keys, and credential-bearing recovery JSON are **not included** in this repository.
- The recovered M2 ephemeral credential and M4/M5 archive key are intentionally omitted. The reports describe verified recovery outcomes without publishing raw secret values.
- A valid detached signature establishes a match between the package, signature, and certificate in the supplied lab; it does not establish that a package is safe or authorized to execute. The vendor maintenance package was not executed.
- These are training-lab write-ups, not a general endorsement to run the commands on production systems. Reproduce only in an environment you are authorized to use.

## Repository layout

```text
.
├── README.md
├── .gitignore
├── writeups/
│   ├── 01-M1-Trojan-Research-Utility.md
│   ├── 02-M2-Sleeping-Endpoint.md
│   ├── 03-M3-Rewritten-Experiment-History.md
│   ├── 04-M4-Timestamp-Key-Exfiltration.md
│   └── 05-M5-Fragmented-Research-Archive.md
└── evidence/
    ├── M1/  # terminal evidence screenshots
    ├── M2/
    ├── M3/
    ├── M4/
    └── M5/
```

## Publish to GitHub

1. Create a repository named `asterion-range3-ir-writeups` (or choose the alternative above).
2. Extract this ZIP file.
3. From the extracted repository directory, run:

   ```bash
   git init
   git add README.md .gitignore writeups evidence
   git commit -m "Add Asterion Range 3 forensic write-ups and evidence"
   git branch -M main
   git remote add origin https://github.com/<your-user>/asterion-range3-ir-writeups.git
   git push -u origin main
   ```

4. Replace `<your-user>` with your GitHub username. If the repository already exists and has a remote configured, use that remote rather than adding a second one.
5. After pushing, open each Markdown file on GitHub and check that the relative screenshot links render. Do not add the VM images, raw recovered credentials/keys, or large case artifacts to Git.

## Publishing and attribution

Before making the repository public, confirm that the challenge materials and screenshots may be redistributed under the applicable rules or license. This bundle includes the write-ups and screenshot evidence from the supplied reports, not the underlying challenge software or VM images. No license is declared here because the applicable reuse terms have not been established.
