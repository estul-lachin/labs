# attack-structure-drill

## Concepts

**ATT&CK** — *Adversarial Tactics, Techniques, and Common Knowledge* — is **MITRE**'s free
catalogue of how attackers behave, built from intrusions that were actually investigated. It does not
list vulnerabilities or signatures; it lists **actions attackers take**, each with a permanent ID that
vendors, analysts and reports all use. The idea behind it is **threat-informed defense**: decide what
to protect and detect first by looking at what real attackers do, rather than working down a generic
list. (Three domains: Enterprise, Mobile, ICS.)

- **The Pyramid of Pain** (David Bianco). It orders indicators by how much it costs an attacker to
  replace one after defenders start detecting it, cheapest at the bottom:
  ```
  TTPs                    top: costly to change      <- where ATT&CK works
  Tools
  Network/Host Artifacts
  Domain Names
  IP Addresses
  Hash Values             bottom: changed in seconds
  ```
  A new hash or IP costs the attacker almost nothing; changing **TTPs**, the way they work, means
  retooling and retraining. A detection written at that level keeps working much longer.
- **The object model** — layers of *what* and *how*:
  - **Tactic** — the goal, the *why* (e.g. Persistence); the matrix **columns**.
    - **Technique** — the method used to reach that goal (e.g. Command and Scripting Interpreter,
    T1059); the **cells**.
  - **Sub-technique** — a narrower form of a technique (e.g. PowerShell, T1059.001).
  - **Procedure** — how one specific group or tool actually did it, listed under "Procedure Examples".
- **IDs.** Every ATT&CK object has a prefix that says what kind of object it is:

  | Prefix | Kind of object | For example |
  |---|---|---|
  | `TA` | Tactic | TA0003, Persistence |
  | `T` | Technique, and `T….nnn` for a sub-technique | T1053, Scheduled Task/Job; T1053.003, Cron |
  | `S` | Software: a malware family or tool | S0002, Mimikatz |
  | `G` | Group: a tracked adversary | G0007, APT28 |
  | `C` | Campaign: activity over a set period | C0001 |
  | `M` | Mitigation | M1036, Account Use Policies |

- **Related models:** the **Cyber Kill Chain** (7 linear phases) and the **Diamond Model** (Adversary,
  Capability, Infrastructure, Victim). ATT&CK goes into far more detail than
  the Kill Chain, and it works alongside the Diamond Model rather than replacing it.
- **The v18 detection model.** Before v18, a technique had a vague "Detection" paragraph and a list of
  "Data Sources". v18 retired that for a modular chain —
  **Technique → Detection Strategy → Analytic → Log Sources / Data Components**:
  - **Detection Strategy (`DET####`)** — the **plan**: what behaviour to look for.
  - **Analytic (`AN####`)** — the **implementation**: platform-specific detection logic (one per
    platform) pointing at the exact telemetry.
  - **Data Component** (e.g. Process Creation) — the specific data a system produces; in v18 **Log
    Sources live inside Data Components**, named like `wineventlog:security` or `auditd:SYSCALL`.

## Tasks

Start the lab with `./lab up attack-structure-drill`, then open a shell:
`./lab shell attack-structure-drill`.

Read the three reference files, then fill in **`/root/answers.txt`**:

```
cat /opt/attack/ids.txt       # A. classify each ID by object type
cat /opt/attack/pyramid.txt   # B. the top and bottom of the Pyramid of Pain
cat /opt/attack/levels.txt    # C. tactic vs technique vs procedure
cat /opt/attack/detection.txt # E. the v18 detection model
```

- **A.** `ID_A`…`ID_G` — the object type of each ID (tactic, technique, sub-technique, software,
  group, campaign, mitigation).
- **B.** `PAIN_MOST` / `PAIN_LEAST` — the top (most pain) and bottom (least pain) pyramid levels.
- **C.** `LEVEL_1`…`LEVEL_3` — the object-model level of each item.
- **D.** `ACRONYM`, `MAINTAINER`, `PHILOSOPHY` — the foundations.
- **E.** `PLAN`, `IMPLEMENTATION`, `DET_PREFIX`, `AN_PREFIX`, `LOGSOURCE_HOME`, `WIN_SECURITY` — the
  v18 detection model.

Check your work from the host: `./lab check attack-structure-drill`.

<details>
<summary>Hints</summary>

- The ID prefix gives it away: `TA`→tactic, `T`→technique, `T….001`→sub-technique, `S`→software,
  `G`→group, `C`→campaign, `M`→mitigation.
- Pyramid: hashes are easiest for the attacker to change (least pain); TTPs are hardest (most pain).
- A matrix **column** is a tactic (why); a **cell** is a technique (how); a named real-actor example
  is a procedure.

</details>

## My notes

<!-- Screenshots: ./lab shot attack-structure-drill ~/Screenshots/<file>.png <name>, then link it as
     ![what it shows](screenshots/<name>.png). Prefer code blocks for terminal output. -->

## Related

- `attack-tactic-triage` — classify observed behaviours by the 15 tactics (the matrix columns).
- `ransomware-kill-chain` — map one intrusion to the Cyber Kill Chain and ATT&CK.
