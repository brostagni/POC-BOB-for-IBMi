# Bob sheet: redeploy IBM i + FLGHT400, then play the lab

> **Note:** the `ibmi-lab/` and `scripts/` folders cited in this sheet (`tz.py`, `tunnel.sh`, `install_flght400.sh`, `.env`, etc.) **are not provided in this repository**. They illustrate the working environment used during the lab. IP addresses, passwords and local paths have been masked (`xxx`, `_set_password_`, `<WORKSPACE>`).

## Launch prompt

**Solo session (1 user):**
```
I would like to run these labs from ibmi-lab/FICHE-REDEPLOIEMENT.md, can you set up a test environment for me on TZ?
```

**Multi-user session (replace X with the number of participants):**
```
I would like to run these labs from ibmi-lab/FICHE-REDEPLOIEMENT.md, can you set up a test environment for me on TZ for X users?
```

> Bob reads this sheet, runs phase A (TechZone + FLGHT400), then for X > 1 continues with the Multi-user section to create the libraries `FLGHT401`…`FLGHT4XX`.
> Lab reference: `flight400-demo/README.md` (exercises 1 to 7). Solo library: **`FLGHT400`**; multi-user: **`FLGHT4nn`** assigned by the instructor.

## Rules for Bob (mandatory)

1. Never display the password, the private key or the TechZone token. The scripts below write them to files in mode 600 without displaying them.
2. **TechZone tools — strict rule:**
   - **Creating a reservation**: `scripts/tz.py create` **only**. The TechZone MCP tools (`platform-mcp-platform-list`, `request-mcp-techzone-create-request`, etc.) have no access to the IBM i PowerVS catalog — using them for creation leads to a guaranteed failure.
   - **Status / credentials after creation**: `scripts/tz.py status|wait|creds` first. The `request-mcp-techzone-get-request` MCP can be used **as a fallback** if `tz.py` fails, by passing the `requestId` returned by `tz.py create` (its output contains secrets — never display it raw).
3. Before `--force` (deletes and restores `FLGHT400`, thus erasing the changes made during the exercises): ask the user for confirmation.
4. If the user says **"new environment"** or **"new IBM i"**: create a TechZone reservation without checking the existing one. Do not offer to reuse an active reservation.
5. Creating a reservation consumes TechZone quota: ask for confirmation before `create`, unless the user has already given it in the prompt.
6. Close the tunnels opened during the session (`scripts/tunnel.sh close`) before wrapping up.
7. If a step fails: stop, give the exact message, offer (a) fix it, (b) continue without it.
8. If the launch prompt mentions **"for X users"** (X > 1): after phase A, automatically continue with the **Multi-user** section — check A0c, upload the script, dry-run, then run for real with `-n X`.

## Prerequisites: deployment files in `ibmi-lab/`

`ibmi-lab/` is self-contained: everything Bob needs must be **in this folder**. Before starting phase A, Bob checks that the following two files are present.

### A0 — Check (and retrieve if needed) `FLGHT400.FILE`

```bash
ls ibmi-lab/FLGHT400.FILE
```

**If the file is missing:** it is not in the GitHub repo — it is only on IBM Box.

> ⬇️ **Manual download (human action)**: open https://ibm.box.com/v/flight400-box and download **`FLGHT400.FILE`** (≈ 275 MB) directly into `ibmi-lab/`.

No CLI shortcut works on Box without IBM authentication. The user must do it from their browser.

### A0b — Check (and retrieve if needed) `Install-Flight400.sql`

```bash
ls ibmi-lab/Install-Flight400.sql
```

**If the file is missing:** Bob retrieves it directly from GitHub:

```bash
curl -sL "https://raw.githubusercontent.com/bmarolleau/flight400-demo/main/instructor-setup/Install-Flight400.sql" \
     -o ibmi-lab/Install-Flight400.sql
```

> ℹ️ `install_flght400.sh` does not use `Install-Flight400.sql` directly — it runs the CL commands itself. This file remains useful as a reference or for the manual method (Option 2 of the instructor README).

### A0c — Check (and retrieve if needed) `scripts/setup_flight400_lab.sh` (multi-user mode only)

Only needed for multi-participant labs (see the **Multi-user** section at the end of this sheet). If missing:

```bash
curl -sL "https://raw.githubusercontent.com/bmarolleau/flight400-demo/main/instructor-setup/setup_flight400_lab.sh" \
     -o ibmi-lab/scripts/setup_flight400_lab.sh
chmod +x ibmi-lab/scripts/setup_flight400_lab.sh
```

---

## Phase A: deployment (Bob alone, no human action unless a check fails)

All paths are relative to `ibmi-lab/`. Prerequisite: be connected to the IBM VPN if the network requires it.

| # | Bob's action | Command | Expected result |
|---|---|---|---|
| A1 | Check the previous reservation (**unless the user asked for a new environment → skip straight to A2**) | `python3 scripts/tz.py status` | `Ready` with a future end date → **skip to A5** (nothing to recreate). Otherwise continue. |
| A2 | Create the reservation (confirmation required, unless already given in the prompt) | `python3 scripts/tz.py create --purpose Test --geography europe` | Displays the identifier and the link; the identifier is stored in `.tz_request_id`. `Education`/`Learning` is blocked by TechZone policy (environment not tied to an official course): use `Test`, or `Demo`/`Pilot` with `--opportunity`. With `europe`, the server picks the region (fra04 or mad02). `Test` gives a short duration (about 12 h on Oct 27): extend it in the TechZone interface if needed. |
| A3 | Wait for provisioning to finish (10 to 25 min) | `python3 scripts/tz.py wait` | `status=Ready` |
| A4 | Write the access details | `python3 scripts/tz.py creds` | Writes `ssh/ssh_private_key.pem` and `.env`, displaying nothing but the host and the user |
| A5 | Install FLGHT400 | `scripts/install_flght400.sh` (add `--node` only for exercise 7) | `RESULT: OK`, `PGM=46 MENU=3` |
| A6 | Summarize for the user | – | IP, user, reservation end date, A5 result |

**Fallback if `tz.py creds` fails** (manual backup, minimum human interaction): Bob asks the user
1. to paste the user and password into `ibmi-lab/.env` (fields `IBMI_USER`, `IBMI_PASSWORD`, `IBMI_HOST`), keeping the values in single quotes;
2. to drop the private key downloaded from the reservation page into `ibmi-lab/ssh/ssh_private_key.pem`, then `chmod 600`.
Then Bob resumes at A5. The template file is `.env.example`.

Verified technical notes:
- The TechZone gateway rejects Python's default HTTP agent (HTTP 403, code 1010). `tz.py` sends a different agent: do not remove it.
- `install_flght400.sh` uses `ssh/known_hosts`: a new machine that takes over an old IP does not block the connection.
- "First installation" path (library absent): same sequence as the one run by hand on Oct 22 (`CRTSAVF`, `CPYFRMSTMF`, `RSTLIB`, `CHGOWN`). The `--force` path and `create` have not been run under real conditions.

## Phase B: settings in Bob IDE (graphical interface, human action)

Bob can neither click in the interface nor open the IBM i connection (the "How would you like to connect?" selector waits for a click). It therefore presents this phase to the user **with the exact values to enter**, then waits for "connected" (after B2) or "phase B done" (after B5).

**Do not call `execute_sql_statement`, `execute_cl_command` or any IBM i tool before the user has confirmed the connection**: each call reopens the selector and loops.

### Source of the values: the IP folder (Bob reads it on every run)
The reservation access details are dropped by hand by the user into `ibmi-lab/<public IP>/` (e.g. `ibmi-lab/161.156.153.xxx/`). This folder is ignored by git. It contains:
- `<IP>.txt`: the reservation details, in the format *label on one line, value on the next line*;
- `ssh_private_key*.pem`: the deployment private key (the name may contain `(1)`).

Bob's procedure before presenting B2:
1. List `ibmi-lab/` and take the folder whose name is an IPv4 address (if there are several, take the most recent one and say so).
2. Read `<IP>.txt` and extract, by label, only: *Deployment user name* (user), *Virtual machine 1 public ip address* (host), *SSH port* (port), *IBM Cloud PowerVS datacenter* (e.g. `eu-de-1` = fra04).
3. **Never display the value following "Deployment password"**, which is on the line after the label: line-by-line masking is not enough. Extract the wanted fields by label (e.g. `grep -A1 'Deployment user name'`), without a `cat` of the whole file. The password is not used for the SSH connection, which uses the key.
4. Find the key with `ssh_private_key*.pem` in this folder, run `chmod 600` on it (otherwise SSH refuses it: "permissions too open"), then give the user its **absolute path**.
5. Check: `ssh-keygen -lf` on this key and on `ibmi-lab/ssh/ssh_private_key.pem` must give the same fingerprint (verified on Oct 27). Otherwise, report it.
6. Fill in the B2 table with these values.

### B1. Extensions
Bob uses `list_extensions` to check the extensions already installed and reports the missing ones.

Extensions present on Oct 27 (nothing to do):
- *IBM Bob Premium Package for i* (v2.0.0): installed and active.
- *Code for IBM i* (v3.0.13): installed and active.
- *IBM i Testing* (v1.3.4): installed and active (required for exercise 6).

**Missing** extension to install **before step 1a**:
- **IBM i Renderer** (publisher: halcyontechltd): absent on Oct 27. Provides the `Preview all` and `Preview XXXXXXXX` links in the DDS source editor. Without it, no preview link appears in `FRS001DF` or `FRS021DF`. Install: `Cmd+Shift+X` → search "IBM i Renderer" → **Install**.

Note: the "IBM i Development Pack" extension does not appear in the list and is not needed. *Code for IBM i* replaces it.

### B2. Connecting to IBM i
In the "How would you like to connect?" selector, choose **Open connection browser**.

Case 1, a `Flight 400 Demo` connection already exists: Bob updates its `host` field in `~/Library/Application Support/IBM Bob/User/settings.json` (key `code-for-ibmi.connections`) after making a backup copy `settings.json.bak-*`, then the user selects it. If the old IP is displayed, reload the window.

Case 2, it does not exist: **Add new connection**, then fill in the form with the values read from the IP folder.

| Form field | Value (Oct 27 example, to be re-read from the IP folder) |
|---|---|
| Connection Name | `Flight 400 Demo` |
| Host or IP Address | `161.156.153.xxx` (*Virtual machine 1 public ip address*) |
| Port (SSH) | `22` (*SSH port*) |
| Username | `ITZUSER` (*Deployment user name*) |
| Password | **leave empty**: a single authentication method |
| Save Password | unchecked |
| Enable MFA | unchecked |
| Private Key | absolute path of the `.pem` in the IP folder, e.g. `<WORKSPACE>/ibmi-lab/161.156.153.xxx/ssh_private_key(1).pem`; or *Select File* |

Check: the status bar or the IBM i view displays `ITZUSER@<IP>`.

### B3. Library list
In the **User Library List** view of the IBM i panel:
- **+** button: add `FLGHT400`;
- right-click on `FLGHT400`: *Set as Current Library*.

Mandatory check: `FLGHT400` appears in the list **and** carries the current-library indication. The `currentLibrary` setting in `settings.json` is not enough — it may not be attached to the new connection, and `/init` then fails ("No current library is configured for this connection"). If needed, use **Change Current Library** in the view and choose `FLGHT400`.

### B4. Object Browser filter
Create a filter (or reuse the existing one):

| Field | Value |
|---|---|
| Name | `FLGHT400` |
| Library | `FLGHT400` |
| Object | `*` |
| Types | `*ALL` |

Check: the expanded filter shows the source files (`QRPGSRC`, `QRPGLESRC`, `QDDSSRCD`…) and the 46 programs.

### B5. New Bob task
| Step | Value |
|---|---|
| Launch | **+** → *New Task in Library List* |
| Mode | **IBM i Developer** |
| Task permissions | enable **Documentation** and **Understand** |

Bob checks over SSH, with the key and without the IBM i tools, that `BOBLIB` exists (used in exercise 1c for the *Understand* metadata): `ssh -i <key> ITZUSER@<IP> 'system "DSPLIBD LIB(BOBLIB)"'`. Finding of Oct 27 on the new machine: **`BOBLIB` does not exist** (`CPF9810`). Before exercise 1c, either create it (`CRTLIB LIB(BOBLIB)`, after the user's agreement), or use the `understand` library configured in the settings (`vscode-ibmi-bob.understand.installLibrary`). Finding of Oct 27: `UNDERSTAND` does not exist either (`CPF9810`). The *Understand* tools will therefore only respond after *Understand* has been installed from its dashboard (exercise 1c), to be done by the user.

### B6. Custom rules for IBM i Developer mode (before exercise 3 only)
> ⚠️ **To be redone on every new environment** (new TechZone deployment, new machine, new Bob installation). These rules are stored in Bob IDE's local settings — they do not transfer from one workstation to another. If B6 has not been done, exercise 3 produces diffs with wrong columns and Bob compiles without waiting for approval.

Without these rules, Bob may insert DDS and RPG lines with wrong columns and compile without waiting for your approval. To be done only once, at any time before exercise 3:

1. Click the **settings icon** at the top right of the Bob panel.
2. Click **Modes** in the left sidebar, then select **IBM i Developer**.
3. Click the **pencil icon** (edit) at the top right to open the mode editor.
4. Scroll down to the **Custom Instructions** section, add the text below **under** the existing "Interaction Guidelines", then click **Save**.

```
## When a user asks to plan or implement code changes
- Make insert-only changes using apply_diff with surrounding context; do not use insert_content.
- When inserting, anchor the diff on the line ABOVE the insertion point and do not include trailing structural lines (DDS key "K ..." or record-format "R ..." lines) in the replacement - leave those lines untouched.
- When adding a fixed-form entry that mirrors an existing one (RPG I-spec, O-spec, or DDS field), copy the model line's EXACT column positions and change only the field name and value. Re-pad so numbers stay right-justified in their original columns and the data-type letter stays in its column. If the new field name is a different length than the model, adjust the padding so every downstream column is unchanged. Never eyeball the spacing.
- Preserve exact DDS column positions for field-definition entries (name, length, type). Keyword continuation lines like COLHDG only need to start after position 44 - do not fixate on their exact alignment.
- Preserve the existing OPM RPG fixed-column style.
- Do not add ALWNULL, do not use SQL ALTER TABLE, do not change the field to 5P 1.
- COLHDG belongs only in the physical file, never in the display file.
- Always show the diff and wait for my approval before saving; never compile until I say so.
```

Check: after saving, these lines appear in the IBM i Developer mode's Custom Instructions. Source: [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 3a, copied here so the sheet is self-sufficient.

### B7. Install Understand and RPGUnit (to be done before exercises 1c and 6)

These two components are installed the same way from the IBM i panel. Understand is needed from exercise 1c; RPGUnit only for exercise 6, but you might as well install everything at once.

#### Open the IBM i panel

The IBM i panel is in the **left activity bar** (IBM i icon). If it has been closed:
- hover over the **status bar at the bottom** → the connection name appears (e.g. `Flight 400 Demo`) with a settings icon → click it, or
- right-click on any icon of the left activity bar → check **IBM i** to show it again.

Once the panel is open: **right-click on the connection** (e.g. `Flight 400 Demo`) → **Connection Settings** → **Components** tab.

#### Understand

State observed on Oct 27 and 28: **`BOBLIB` and `UNDERSTAND` do not exist** on the new machine. The Understand component creates its metadata library during installation.

1. In the **Components** tab: **Add Component** → choose **Understand** → **Install**.
2. Check: the *Understand* dashboard (graph icon in the activity bar) is accessible without error.

> **Finding of Oct 28:** the installer **does not offer** a choice of metadata library — it directly creates `UNDERSTAND` (default value of `vscode-ibmi-bob.understand.installLibrary`). Do not anticipate a prompt that does not appear. **At step 1c, use `UNDERSTAND` as the metadata library** (and not `BOBLIB`).

#### RPGUnit (exercise 6 only)

State observed on Oct 27: **`RPGUNIT` does not exist** (`CPF9810`); `QDEVTOOLS` already exists.

1. **IBM i Testing** extension: open the extensions panel (`Cmd+Shift+X`), search "IBM i Testing" and check that it is *Installed* (it is on Oct 27, version 1.3.4).
2. In the **Components** tab: **Add Component** → choose **RPGUnit** → **Install**.
3. Add `RPGUNIT` then `QDEVTOOLS` to the **User Library List** (**+** button, as in B3). The list must contain `FLGHT400`, `RPGUNIT` and `QDEVTOOLS`.

Check (Bob, over SSH): `ssh -i <key> ITZUSER@<IP> 'system "DSPLIBD LIB(RPGUNIT)"'` no longer returns `CPF9810`. Source: [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § Exercise 6 prerequisites.

## Phase C: play the lab (instructions to give to Bob, in order)

Replace `FLGHT4nn` with `FLGHT400` everywhere. Total duration about 2 h. The quoted blocks are to be sent to Bob as is.

### How to launch each task (**+** button above the chat)
The **+** button offers three scopes: *New task in Bob_L4*, *New task in Library List* and *New task in IFS Working Directory*. In your screenshot's selector, they are displayed as follows:

| Option displayed | Scope | Choose for |
|---|---|---|
| **New task in Library List** (`FLGHT400, QGPL, QTEMP`) | IBM i libraries (QSYS) | **All exercises 1 to 6**, **IBM i Developer** mode (or **IBM i Database** at steps 1e and 4). The only option that allows `/init`, `@FLGHT400`, reading the sources and compiling. |
| New task in Bob_L4 | Local folder on the Mac (*Local Workspace*) | **Only exercise 7, step 3** (create the skill, in **Agent** mode). Never to work on FLGHT400. |
| New task in IFS Working Directory (`/home/ITZUSER`) | Home directory on the IBM i | Not needed for the lab. Exercise 7 does not need it: the application is created in the IFS from the *Library List* scope. |

Rules to follow:
- **A new task for each change of scope** (**+** button). For a mode change only (IBM i Developer to IBM i Database), [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 4a says to switch the mode with the chat mode selector.
- Check before sending the first instruction: the displayed mode is the right one, the displayed scope is *Library List*, and `FLGHT400` is on the library list with the current-library indication (B3).
- A task may chain several steps of the same exercise. For exercise 3, keep **a single task** from start to finish, so that Bob keeps the plan validated at step 3c.
- The "Human" steps (*Understand* dashboard, *Workflows* icon) are done in the interface, not in the chat. For *Workflows*, first choose the *Library List* scope ([`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 4d and § 6b).

### Exercise 1: documentation (40 min)
| Step | Instruction | Mode / remark |
|---|---|---|
| 1a | Human, without Bob: browse the application in the Object Browser. (1) Expand **User Library List** and **Object Browser**; filter `FLGHT400`, types `*ALL`. (2) Under **Source Files**, open a few `QRPGSRC` members to see the fixed-format RPG style. (3) In `QDDSSRCD`, double-click `FRS001DF` to open it: **Preview All** requires the **IBM i Renderer** extension (not installed on Oct 27). Without it, the link does not appear at all in the editor. To install it: `Cmd+Shift+X` → search "IBM i Renderer" (publisher: halcyontechltd) → **Install**. Once installed, a `Preview all` link appears **above line 1**, in the editor gutter, in very small grey text (verified on Oct 27 on `FRS001DF`). A `Preview XXXXXXXX` link also appears under each screen format name. Click `Preview all` to see the visual rendering of all the screens in the file. Also try `FRS021DF` (the exercise 3 screen). (4) In the Object Browser, **right-click on `FRS000.PGM`** → **Detail**: a side panel displays the compiled object's metadata, including the compilation date (1997). | No instruction to Bob |
| 1b | **Before**: in the **User Library List** view, right-click on `FLGHT400` → *Set as Current Library* (or **Change Current Library**). Then `/init` | IBM i Developer, *Library List* scope. Creates `FLGHT400/QBOBSRC(AGENTS)`. Without a current library, `/init` stops without creating anything (QGPL and QTEMP are refused) |
| 1c | **Prerequisite: Understand installed (B7).** Human: in the activity bar, click the **Understand** icon (graph) → **New Application**, name `FLGHT400`, library `FLGHT400`, metadata library **`UNDERSTAND`** (the installer does not offer a choice — it creates `UNDERSTAND` directly, confirmed on Oct 28) → **Scan**. Then: "What are the high-level functional areas of FLGHT400?" | The scan takes 30 to 60 s. Expected: about 46 programs, 0 dynamic SQL |
| 1d | "Generate a comprehensive architecture overview of the FLGHT400 application in QSYS in Markdown format. Include a high-level description, the main program flows, key programs and their roles, a Mermaid architecture diagram, and a summary of the database tables used." | Save as `FLGHT400-Architecture.md` |
| 1e | `/erd FLGHT400` | **IBM i Database** mode |
| 1f (optional) | "Analyze the FLGHT400 application from the library list and generate a draw.io architecture diagram showing the main programs, menus, and database files. Save the file as `FLGHT400-architecture.drawio` in `$HOME/docs/` on IBM i." | Draw.io Integration extension |
| 1g (optional) | Human: *Workflows* icon → *Business Rules Extraction*. Library `FLGHT400`, file `QRPGLESRC`, member `FRS401.RPGLE`, output `/home/ITZUSER/flght400/docs/business-rules/FRS401-report.md` | |

### Exercise 2: modernize `FRS409` (15 min)
1. Open `FLGHT400/QRPGSRC(FRS409)`, then: "What does this program do?"
2. "Can you modernize this program?" → *Start workflow* (RPG Modernization), source `FLGHT400/QRPGSRC`, member `FRS409`, output `FLGHT400/QRPGLESRC/FRS409.RPGLE`.

### Exercise 3: add *Total Flight Hours* (30 min)
> ⚠️ This exercise **modifies** the library. To replay it, restore with `--force` then redo B6.

**Prerequisite:** B6 done (custom rules saved in IBM i Developer mode). **IBM i Developer** mode, **Library List (QSYS)** scope.

**Step 3a — Custom rules:** = **B6** (already done in phase B). No additional action here.

**Spec of the field to add** (informational — this is the spec Bob will implement in the following steps):
| Layer | Name | Definition |
|---|---|---|
| Database (PF) | `FLHRS` | 4P 0 (packed, 4 digits, 0 decimals) |
| Logical / RPG | `FHRS` | `RENAME(FLHRS)` |
| Screen (DDS) | `SFLHRS` | 4/0, input/output, after Mileage |

**Step 3b — Explore the FRS021DF screen:**
```
Preview the display file FRS021DF from FLGHT400/QDDSSRCD and list every field on the Flight Maintenance screen, noting the input/output type of each and the screen-field naming convention (e.g. SFLGHT, SMILES, SSEATS, SPRICE). Do not modify anything.
```

**Step 3c — Plan (write nothing):**
```
Let's plan this change together first — do not write, edit, save, or compile any code; just produce the plan with me. Produce a step-by-step implementation plan to add "Total Flight Hours" to the Flight Maintenance application in FLGHT400.

Spec: database field FLHRS (4P 0); expose to RPG as FHRS RENAME(FLHRS); screen field SFLHRS (4 digits, 0 decimals, input/output) placed immediately after Mileage with the visible label "Flight Hours".

Base the plan on how the existing Mileage field flows today: FLIGHTS.MILEAGE → FLIGHTSZ.MILES → FRS021 → FRS021DF.SMILES. Mirror that exact path for the new field.

The plan must cover and bake in these constraints:
- FLGHT400/QDDSSRCF(FLIGHTS): add FLHRS 4P 0 with a COLHDG consistent with the existing PF DDS.
- FLGHT400/QDDSSRCF(FLIGHTSZ): add FHRS RENAME(FLHRS).
- FLGHT400/QDDSSRCD(FRS021DF): add input/output field SFLHRS with visible label "Flight Hours", CHECK(RZ) to match nearby numeric fields, no COLHDG, no overlap with existing fields/message/function-key areas.
- FLGHT400/QRPGSRC(FRS021): increase the FLIGHTSZ record length from 233 to 236 for the 3-byte packed field; add FHRS to the input spec at positions 234-236 — note FHRS is packed (4P 0, 3 bytes), so use the P data-type in column 43 of the I-spec, unlike the neighboring FMILES which is binary (B); after every successful CHAIN that loads an existing flight, explicitly move FHRS to SFLHRS; move SFLHRS to FHRS on add/update; include FHRS in the ADDFLT and UPDFLT output specs — mirror the exact column alignment of the neighboring FMILES O-spec entry so the end position stays right-justified; validate consistently with existing numeric screen fields.
- Build order at the end: CHGPF FLIGHTS, CRTLF FLIGHTSZ, CRTDSPF FRS021DF, CRTRPGPGM FRS021.

Only these four objects may change: FLIGHTS, FLIGHTSZ, FRS021DF, FRS021. Any other affected program is follow-up work only. Flag any other programs that use FLIGHTS or FLIGHTSZ as follow-up work only — do not include them in the plan's changes.
```
> ✅ Checkpoint: the plan only touches `FLIGHTS`, `FLIGHTSZ`, `FRS021DF`, `FRS021`, correctly uses `FLHRS`/`FHRS`/`SFLHRS`, mentions the 233→236 change and the post-CHAIN `FHRS→SFLHRS` move. Approve: *"I approve this plan — let's implement it."*

**Step 3d — Implement the DDS (one member at a time):**
```
Execute the approved plan for the DDS members only, one at a time: (1) FLIGHTS, (2) FLIGHTSZ, (3) FRS021DF. For each, show the diff, wait for my approval, then save the source member only — do not compile. Follow the standing rules exactly.
```
Check for each member before approving:
- `FLIGHTS`: `FLHRS 4P 0` with `COLHDG`; key line `K FLIGH00001` untouched.
- `FLIGHTSZ`: `FHRS RENAME(FLHRS)` after `MILES`; key line `K FLGHTN` untouched.
- `FRS021DF`: `SFLHRS` 4/0 I/O after Mileage; label "Flight Hours"; `CHECK(RZ)` present; **no** `COLHDG`.

**Step 3e — Implement the RPG program (3 separate diffs):**

*5.1 — F-spec and I-spec:*
```
Apply just the F-spec and I-spec changes to FLGHT400/QRPGSRC(FRS021): increase the FLIGHTSZ record length from 233 to 236, and add the FHRS input-spec entry at positions 234-236 using data-type P (packed) in column 43, mirroring the exact column alignment of the neighboring FMILES entry. Show the diff and wait for my approval; do not compile.
```
Check: the F-spec reads `236`; I-spec `FHRS` at 234-236 with `P` in the data-type column (not `B`).

*5.2 — C-specs:*
```
Now apply just the C-spec changes to FRS021: after every successful CHAIN that loads an existing flight, move FHRS to SFLHRS; move SFLHRS to FHRS on both the add and update paths (mirroring the existing Z-ADD SMILES/FMILES lines); and add an SFLHRS validation consistent with the SMILES check using the next free indicator. Show the diff and wait for my approval; do not compile.
```

*5.3 — O-specs (the part most sensitive to alignment):*
```
Finally apply just the O-spec changes to FRS021: add FHRS to both the ADDFLT and UPDFLT output records. Copy the exact column positions of the neighboring FMILES O-spec line and change only the field name and end position — the end position must stay right-justified in the same columns, with the packed data-type letter in the same column as FMILES's B. Do not eyeball the spacing. Show the diff and wait for my approval; do not compile.
```
> ⚠️ The end position of `FHRS` must line up with that of `FMILES` (`233`) — otherwise the compiler reads `238` and fails.

**Step 3f — Compile the 4 objects in order:**
```
Build only the four demo objects in order: (1) apply DDS for FLIGHTS, (2) rebuild FLIGHTSZ, (3) compile FRS021DF, (4) compile FRS021. Use the correct IBM i command for each source type. If one fails, stop, report the compile error briefly, and wait for my direction. Do not touch unrelated programs.
```
Expected commands:
```
CHGPF FILE(FLGHT400/FLIGHTS) SRCFILE(FLGHT400/QDDSSRCF) SRCMBR(FLIGHTS)
CRTLF FILE(FLGHT400/FLIGHTSZ) SRCFILE(FLGHT400/QDDSSRCF) SRCMBR(FLIGHTSZ)
CRTDSPF FILE(FLGHT400/FRS021DF) SRCFILE(FLGHT400/QDDSSRCD) SRCMBR(FRS021DF)
CRTRPGPGM PGM(FLGHT400/FRS021) SRCFILE(FLGHT400/QRPGSRC) SRCMBR(FRS021) REPLACE(*YES)
```

**Step 3g — Validate:**
```
Validate the completed change: confirm FLHRS exists in FLGHT400/FLIGHTS, FHRS is available through FLIGHTSZ, SFLHRS appears immediately after Mileage in the FRS021DF Previewer, FRS021 compiled, and FRS021DF has no COLHDG. Finish with a one-line end-to-end field mapping and list any additional impacted programs as follow-up. Do not modify anything.
```

### Exercise 4: SQL optimization (15 min)
**Step 4a — Change mode:** switch to **IBM i Database** via the mode selector in the Bob panel.

**Step 4b — Query review:**
1. In the Bob chat, type `/review_sql` (the slash command must be visible), then paste the following query (replace `FLGHT400` with your library):

```sql
SELECT
    f.FLIGH00001                                    AS FLIGHT_NUMBER,
    f.DEPARTURE                                     AS FROM_CITY,
    f.ARRIVAL                                       AS TO_CITY,
    f.AIRLINES                                      AS AIRLINE,
    f.DAY_O00001                                    AS DAY_OF_WEEK,
    f.DEPAR00002                                    AS DEPARTURE_TIME,
    f.ARRIV00002                                    AS ARRIVAL_TIME,
    f.MILEAGE,
    f.TICKE00001                                    AS TICKET_PRICE,
    f.SEATS00001                                    AS SEATS_AVAILABLE,
    ag.AGENT_NAME,
    COUNT(DISTINCT o.CUSTO00001)                    AS UNIQUE_CUSTOMERS,
    SUM(o.TICKE00001)                               AS TOTAL_TICKETS_SOLD,
    SUM(CASE WHEN o.CLASS = 'F' THEN o.TICKE00001 ELSE 0 END) AS FIRST_CLASS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'B' THEN o.TICKE00001 ELSE 0 END) AS BUSINESS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'E' THEN o.TICKE00001 ELSE 0 END) AS ECONOMY_TICKETS,
    MIN(o.DEPAR00001)                               AS EARLIEST_BOOKING_DATE,
    MAX(o.DEPAR00001)                               AS LATEST_BOOKING_DATE
FROM FLGHT400/FLIGHTS       f
JOIN FLGHT400/ORDERS        o  ON o.FLIGH00001  = f.FLIGH00001
JOIN FLGHT400/AGENTS        ag ON ag.AGENT_NO   = o.AGENT_NO
LEFT JOIN FLGHT400/CUSTOMERS c  ON c.CUSTO00001  = o.CUSTO00001
WHERE o.DEPAR00001 >= TIMESTAMP('2004-02-08-00.00.00')
  AND o.DEPAR00001 <  TIMESTAMP('2004-02-11-00.00.00')
GROUP BY
    f.FLIGH00001, f.DEPARTURE, f.ARRIVAL, f.AIRLINES,
    f.DAY_O00001, f.DEPAR00002, f.ARRIV00002,
    f.MILEAGE, f.TICKE00001, f.SEATS00001, ag.AGENT_NAME
ORDER BY
    o.DEPAR00001,
    f.FLIGH00001
FETCH FIRST 100 ROWS ONLY;
```
> 💡 Intentional problem: `ORDER BY o.DEPAR00001` references a column that is not in the `GROUP BY`. Bob must detect it and suggest `MIN(o.DEPAR00001)`.

**Step 4c — Performance (optional):**
Ask: *"Is any table a performance bottleneck and why?"*
Expected: Bob identifies `ORDERS` as a large table, the filter on `DEPARTURE_DATE` as highly selective, and recommends an index.

**Step 4d — Index Advisor Workflow:**
1. *Workflows* icon (top of the Bob panel) → **Library List** scope → **SQL Index Strategy Advisor**.
2. Parameters:

| Parameter | Value |
|---|---|
| Data Source | Capture New Performance Data |
| Capture Method | `DUMP_PLAN_CACHE_TOPN` |
| Output Library | `FLGHT400` |
| Output Object Name | `FLGHT400P` |
| Top N Queries | 20 |
| Top N Category | Runtime |

3. When Bob proposes the indexes: *"Apply the highest-priority index only for FLGHT400"*
   Expected result:
   ```sql
   CREATE INDEX FLGHT400.ORDERS_IDX_DEPDT_FLT
       ON FLGHT400.ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER);
   ```

### Exercise 5: system questions (10 min), IBM i Developer mode
1. "Which active jobs have accumulated the most CPU time? For the top jobs, distinguish cumulative CPU time from their current elapsed CPU percentage."
2. "Which programs in the FLGHT400 library have not been recompiled in the last 5 years?"

### Exercise 6: RPGUnit tests (20 min)
**Prerequisite:** B7 done (RPGUnit installed, `RPGUNIT` and `QDEVTOOLS` in the library list). **IBM i Developer** mode.

**Step 6a — Create the source member:**
In the Object Browser, `FLGHT400/QRPGLESRC` → right-click → **New Member** → name `CUSTCHK.SQLRPGLE`. Paste the following code and save (`Cmd/Ctrl+S`):

```rpgle
**free
ctl-opt nomain;

dcl-proc checkCustomerExists export;
  dcl-pi *n ind;
    custId packed(9:0) const;
  end-pi;

  dcl-s rowCount int(10);

  exec sql
    select count(*)
      into :rowCount
      from CUSTOMRZ
      where CUSTNO = :custId;

  return (SQLCODE = 0 and rowCount > 0);
end-proc;
```

**Step 6b — RPGUnit Test Plan Creation workflow:**
*Workflows* icon → **RPGUnit Test Plan Creation** (scope: library list).
1. **Get Started** → select the `FLGHT400` library.
2. IFS folder: `/home/ITZUSER/flght400`.
3. Choose **create a new test plan** → **default recommended path** → let Bob find the files automatically (it must find `CUSTCHK.SQLRPGLE`).
4. Select the exported procedure → **validate the environment**.
> 💡 If Bob offers to rerun the Test Plan Creation workflow, answer **No thanks**.

**Step 6c — RPGUnit Test Suite Implementation workflow:**
*Workflows* icon → **RPGUnit Test Suite Implementation** (scope: library list).
1. **Proceed** (the 6b plan is already ready).
2. Library: `FLGHT400`.
3. IFS folder: `/home/ITZUSER/flght400` (same as in 6b). Confirm the path to the suites.
4. **validate the environment**. If Bob offers to download RPGUnit, click **Install** then **Cancel** on the next popup.
> 💡 If a test fails: "Explain the failure and help fix it."

### Exercise 7 (optional): React application from the green screen (30 min)
> 📌 **Default values (solo demo): library `FLGHT400`, port `3001`.**
> If the participant has an assigned number (e.g. `403`), replace `FLGHT400` → `FLGHT403` and port `3001` → the port assigned in the Setup table.

**Prerequisite:** SSH tunnel active on the application port + Node 22 installed on the IBM i.

> ⚠️ **Why a tunnel?** The Vite server runs in PASE on an internal IBM i IP (of the `192.168.x.x` type) that is **never** directly reachable from your Mac. The only path is an SSH tunnel through the TechZone public IP. **Do not use the internal IP in the SSH command**: it fails. The tunnel target must be `localhost:<port>`.

**Step 7.0 — Install Node 22 (if not done yet):**
```bash
scripts/install_flght400.sh --node
```

**Step 7.1 — Generate the React app:**
**IBM i Developer** mode, click **+** → `FLGHT400` library. Attach the image `flight400-demo/docs/img/flight400.png`, then send:
```
Given this screenshot of the 5250 flight order screen from the Application Flight400 in @FLGHT400, Build a single-page React 18 + Vite 4 app on IBM i (PASE) using @carbon/react ^1.x with the g100 dark theme that modernises the IBM i 5250 screen shown in the attached screenshot. Create the app in the IFS at $HOME/flight400-frontend-apps/screen-name/. Use the g100 dark theme. All fields should have a list of values to select from. The dev server must run in the background using nohup ... & and write output to /tmp/vite-dev.log. Pin the Vite dev server to port 3001 if available.
```

**Step 7.2 — Create the helper skill:**
**Agent** mode, click **+** → **Local Workspace**. Copy the contents of `flight400-demo/SAMPLE-SKILL.md`, paste it into the chat, then send:
> *"Create a skill from the pasted text."*

**Step 7.3 — Start the dev server:**
```
Please start the dev server for the app you just created.
```
Check in Bob's log that Vite confirms startup:
```
VITE vX.X.X  ready in XXXX ms
➜  Local:   http://localhost:3001/
```

**Step 7.4 — Open the SSH tunnel from your Mac:**
In a terminal on your Mac, from `ibmi-lab/`:
```bash
scripts/tunnel.sh open 3001
```
Expected response:
```
port 3001: open (pid XXXXX) -> http://localhost:3001
```
If the tunnel is already open (`port 3001: already open`) → go straight to the next step.

Equivalent command without the script (foreground, stop with `Ctrl+C`):
```bash
set -a; . ./.env; set +a
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" -L 3001:localhost:3001 -N "$IBMI_USER@$IBMI_HOST"
```

**Step 7.5 — Open in the browser:**
```
http://localhost:3001/
```

**Common pitfalls**

| Symptom | Cause | Fix |
|---|---|---|
| `192.168.x.x:3001` timeout | Internal PASE IP — unreachable from the Mac | Use the tunnel, not the direct IP |
| `ssh -L 3001:192.168.x.x:3001` fails | The target must be `localhost`, not the internal IP | Use `localhost:3001` as the target |
| `bind: Address already in use` | Tunnel already open | That's OK — go straight to the browser |
| The IP changes between two TechZone reservations | `.env` stores `IBMI_HOST` — update it if needed | Re-source `.env` after each new reservation |

**Step 7.6 — Close the tunnel after the demo:**
```bash
scripts/tunnel.sh close 3001
```

**Step 7.7 — Stop the app:**
```
Stop the web service for FLGHT400 on port 3001.
```

### Exercise 8 (optional): FLGHT400 in web mode (5250 to HTML)
Prototype of the 5250 screens in HTML, viewed through an SSH tunnel. Tunnel required on port **57700** (see below). Estimated duration: 30 min. No change to the `FLGHT400` library.

**Goal:** show that a 5250 screen can be rebuilt as a web page without touching the RPG code. The prototype emulates the terminal (black background, green text, monospaced font, input fields, function key bar) from the DDS sources. Target screens: `FRS000DF` (logon) and `FRS001DF` (order creation). The prototype handles no data: it only reproduces the display.

#### Step 1 — Open the tunnel (port 57700)
The Apache server listens on port **57700** but is only reachable from the IBM i. Open the tunnel from `ibmi-lab/`:
```bash
scripts/tunnel.sh open
```
The script reads `.env`, uses the key from `ssh/`, opens port 57700 in the background and displays `port 57700: open`. To close: `scripts/tunnel.sh close`. Equivalent command without the script, in the foreground (stop with `Ctrl+C`):
```bash
set -a; . ./.env; set +a
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" -L 57700:localhost:57700 -N "$IBMI_USER@$IBMI_HOST"
```

#### Step 2 — Generate the prototype
**IBM i Developer** mode, *Library List* scope with `FLGHT400`. Send to Bob:

```
This FLGHT400 application uses 5250 screens. Read the DDS sources in
FLGHT400/QDDSSRCD, starting with FRS000DF and FRS001DF, then generate a
web prototype in HTML, CSS and JavaScript with no framework that faithfully
emulates the green-on-black terminal (input fields, function key bar).
Deploy the files to the IFS, in /www/qiccqxml/htdocs/flght400/: the
existing QICCQXML Apache server already serves them. Do not start any other server.
Do not modify any object in the FLGHT400 library.
```
Expected result: `index.html`, `style.css` and `app.js` under `/www/qiccqxml/htdocs/flght400/`.

#### Step 3 — View the prototype
Open in the browser: <http://localhost:57700/flght400/index.html>

#### Technical notes
| Item | Detail |
|---|---|
| What works | SSH tunnel with the key from `ibmi-lab/ssh/` to port 57700. Verified on Oct 27: the page answers `HTTP 200` through the tunnel. |
| Web server | `QICCQXML` Apache instance (`QHTTPSVR` subsystem). `/www/qiccqxml/conf/httpd.conf`: `Listen *:57700`, `DocumentRoot /www/qiccqxml/htdocs`. |
| Direct access `http://<IP>:57700` | Does not respond from the Mac (timeout, verified). Go through the tunnel. |
| Python server | To be avoided: same problem and pointless since Apache already serves the folder. |
| SSH key | The IBM i key is identical to the one of the TechZone reservation (same SHA256 fingerprint verified). Never display it with `cat`. |
| Other port | `scripts/tunnel.sh open 3001` for exercise 7; several ports at the same time: `open 57700 3001`. |

## Reset between two runs
- Replay the exercises on the same machine: `scripts/install_flght400.sh --force` (confirmation required). This only restores the `FLGHT400` library: the web lab files (`/www/qiccqxml/htdocs/flght400/`) and those of exercise 7 (in the home directory) stay in place.
- New machine: resume at A2. The web files and those of exercise 7 must be regenerated (they live on the old machine, deleted at the end of the reservation).
- Extend the reservation: to be done in the TechZone interface; no MCP tool allows it.

## Live 5250 test — input values for FRS000 and FRS001

> ⚠️ These values are specific to the FLGHT400 demo application. They have no value in production.

To reach the 5250 screens, use ACS (IBM Access Client Solutions) or any 5250 emulator connected to the IBM i of the TechZone reservation (IP and password in `ibmi-lab/.env` and `ibmi-lab/<IP>/<IP>.txt`). Start `FRS000` from the menu `FLGHT400/QMNUSRC(FRSMNUDF)` or with `CALL FLGHT400/FRS000`.

### Screen 1 — `FRS000DF`: LOGON

| Field | Description | Value to enter |
|---|---|---|
| **Agent Name** (`AGNTNM`) | 10 chars, ref. AGENTS | `Julia` |
| **Password** (`PASSWD`) | 10 chars, ref. AGENTS | `_set_password_` |

All agents share the password **`_set_password_`**. Valid names: `Julia`, `Mary`, `Alex`, `Debby`, `Amanda`, `Sharon`, `Suzanne`, `Helen`, `Terry`.

Confirm with **F10** (LOGON).

### Screen 2 — `FRS001DF`: Create Order (flight reservation)

Two parts: **Flight Information** (left) and **Ticket Order Information** (right).

#### Left part — Flight information

| Field | Description | Value to enter |
|---|---|---|
| **DOFMON / DOFDAY / DOFYR** | Flight date MM / DD / YYYY | `07` / `15` / `2025` |
| **FCITY** | From City | `Albany` |
| **TCITY** | To City | `Albuquerque` |

After entering FCITY and TCITY, press **F6** (Flights) to display the available flights. Examples from the database:

| Flight# | Airline | From | To | Departure | Arrival | Price |
|---|---|---|---|---|---|---|
| `1100001` | AMA | Albany (ABY) | Albuquerque (ALB) | 09:01 AM | 03:01 PM | $169 |
| `1200002` | AMA | Albany (ABY) | Albuquerque (ALB) | 09:03 AM | 11:03 AM | $179 |
| `1300003` | AMA | Albany (ABY) | Albuquerque (ALB) | 10:06 AM | 12:06 PM | $199 |
| `1500005` | AMA | Albany (ABY) | Albuquerque (ALB) | 12:55 PM | 02:55 PM | $299 |

#### Right part — Ticket Order

| Field | Description | Value to enter |
|---|---|---|
| **CUSNAM** | Customer name (25A) | `Aaronson, Lynn` |
| **COSFIR** | First class (1A) | `Y` or empty |
| **COSBUS** | Business class (1A) | `Y` or empty |
| **COSECO** | Economy class (1A) | `Y` |
| **TCKETS** | Number of tickets (2D) | `2` |

Press **F7** to display the customer list and select `Aaronson, Lynn`. Confirm with **F10** (Make Reservation).

### Function keys

| Key | `FRS000` | `FRS001` |
|---|---|---|
| F2 | Refresh | Refresh |
| F3 | Exit | — |
| F4 | Agent Prompt | FROM Cities |
| F5 | — | TO Cities |
| F6 | — | Flights |
| F7 | — | Customers |
| F10 | LOGON | Make Reservation |
| F12 | Cancel | Cancel |

## Files
| File | Role |
|---|---|
| `scripts/tz.py` | Create, track and retrieve the access details of a reservation (without displaying secrets) |
| `scripts/install_flght400.sh` | Upload the save file, restore `FLGHT400`, verify |
| `scripts/tunnel.sh` | Open, close or list SSH tunnels to the IBM i (port 57700 for the web site, 3001 for exercise 7) |
| `.env`, `ssh/ssh_private_key.pem` | Access details (secrets). Ignored by git |
| `.tz_request_id`, `.tunnels/` | Local state (reservation identifier, tunnel PIDs). No secrets, ignored by git |
| `FLGHT400.FILE` | Application backup (275 MB). **Must be in `ibmi-lab/`** — if missing, download from https://ibm.box.com/v/flight400-box (see A0) |
| `Install-Flight400.sql` | Manual restore SQL script (Option 2). Can be retrieved from GitHub if missing (see A0b) |
| `scripts/setup_flight400_lab.sh` | Multi-user cloning script. Can be retrieved from GitHub if missing (see A0c) |

## Multi-user (Bobathon / training)

This section applies when **several participants** play the lab at the same time on the **same IBM i**.

### Principle

- `FLGHT400` = **intact master**. Never assign it to a participant. Bob restores it via `install_flght400.sh` (phase A5).
- Each participant receives an isolated copy: `FLGHT401`, `FLGHT402`… up to `FLGHT450`.
- The copy is made with `CPYLIB` (IBM i CL). Each participant uses **only their own library** throughout the lab.
- For exercise 7, each number `nn` has its own dedicated port: participant 1 → port `3001`, participant 2 → port `3002`, etc.

### Before installing: announce the number of participants

Decide on the number **before** starting the installation, so the clones are prepared in a single pass.

### Option 1 — Automatic script (recommended, ≥ 2 participants)

Check that `scripts/setup_flight400_lab.sh` is present (see A0c if missing).

**Step 1 — Upload the save file and the script to the IBM i**

```bash
cd ibmi-lab
set -a; . ./.env; set +a
# Upload the save file (if not already done by install_flght400.sh)
scp -i "$IBMI_SSH_KEY" -P "$IBMI_SSH_PORT" FLGHT400.FILE "$IBMI_USER@$IBMI_HOST:$IBMI_IFS_DIR/FLGHT400.FILE"
# Upload the script
scp -i "$IBMI_SSH_KEY" -P "$IBMI_SSH_PORT" scripts/setup_flight400_lab.sh "$IBMI_USER@$IBMI_HOST:$IBMI_IFS_DIR/setup_flight400_lab.sh"
```

**Step 2 — Dry run (preview, no changes)**

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh -f $IBMI_IFS_DIR/FLGHT400.FILE -n <N> --dry-run"
```

Replace `<N>` with the number of participants. Check that the libraries displayed are `FLGHT401` … `FLGHT4NN`.

**Step 3 — Real run**

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh -f $IBMI_IFS_DIR/FLGHT400.FILE -n <N>"
```

The script:
1. Restores `FLGHT400` from the save file (if not already done).
2. Clones `FLGHT401` … `FLGHT4NN` via `CPYLIB`.
3. Displays the participants / library / port / React URL table — to be handed out to the participants.

**Useful variants**

| Need | Command |
|---|---|
| Master already restored (phase A5 done), clones only | `setup_flight400_lab.sh --skip-restore -n <N>` |
| Restore the master only, no clones | `setup_flight400_lab.sh -f <IFS_PATH> --restore-only` |
| Explicit range (e.g. participants 3 to 8) | `setup_flight400_lab.sh -f <IFS_PATH> --range 3 8` |
| Overwrite existing libraries | add `--force` |

### Option 2 — Manual CPYLIB via Bob (small group)

In an **IBM i Developer** task (*Library List* scope):

> "Run `CPYLIB FROMLIB(FLGHT400) TOLIB(FLGHT401)` and so on so each participant gets their own isolated copy. I have X participants."

Bob chains the `CPYLIB` commands up to `FLGHT4XX`.

### Instructions for participants

Each participant:
1. Receives their number `nn` (01, 02…).
2. Adds **only** `FLGHT4nn` to their User Library List (not `FLGHT400`).
3. Replaces `FLGHT400` with `FLGHT4nn` in **all** the lab instructions.
4. For exercise 7, uses port `30nn` (e.g. participant 3 → port `3003`).

### Resetting a participant

If a participant corrupts their library:

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh --skip-restore --range <nn> <nn> --force"
```

This recreates only `FLGHT4nn` from the `FLGHT400` master, without touching the others.

> Source: [`flight400-demo/instructor-setup/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/instructor-setup/README.md)

