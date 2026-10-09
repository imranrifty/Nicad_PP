# NiCad++ — An Incremental Clone Detection tools 
# Author : Imran Rahman Ifty
# Khulna University 
NiCad++ is a wrapper around NiCad 6.2. It runs NiCad on the first revision, then
re-extracts only the changed files and compares only pairs with a new fragment.
The clone pairs and classes are the same as full NiCad's.

## Contents

| File | Purpose |
|---|---|
| `install.sh` | installs everything into your NiCad folder |
| `scripts/git_downloader` (+ `.py`) | Git repository → revision snapshots `systems/<name>/r<N>/` |
| `scripts/NiCadInc` (+ `nicadinc.py`) | runs one setup over a window of revisions |
| `scripts/NiCadCompare` | runs the three setups and compares them |
| `scripts/FindClonePairs2` | incremental clone-pair step (used by NiCadInc) |
| `scripts/compare_blocks.py` | optional: compares the extracted fragments of the setups |
| `tools/clonepairs2.c` | NiCad's `clonepairs` changed to compare only new fragments (`.diff`: the change) |
| `tools/clonepairs_rev.c` | alternative clone-pair program (`ENGINE=rev`) |
| `config/type3.cfg` | configuration of the paper (functions, θ = 0.30, 2–500 lines) |

## Requirements

- Windows with **Cygwin** (packages `gcc-core`, `make`, `git`), or Linux
- **TXL 10.8b** and **NiCad 6.2**, built (`tools/clonepairs.x` exists)
- **Python 3.6+** (Cygwin's or Windows' python)

## Install

```bash
cd /cygdrive/d/NiCad-6.2           # your NiCad folder (has tools/ scripts/ config/)
unzip NiCadPP.zip                  # creates NiCadPP/ inside it
bash NiCadPP/install.sh
```

The installer copies the scripts and tools, removes Windows line endings, compiles
`tools/clonepairs2.x` and `tools/clonepairs_rev.x`, and checks txl, gcc, python and git.
Run every command below **from the NiCad folder**.

## Step 1 — download revisions

```bash
./scripts/git_downloader https://github.com/wumpz/jhotdraw.git --info       # 1082 revisions
./scripts/git_downloader https://github.com/wumpz/jhotdraw.git jhotdraw 2 200 --lang java
```

Creates `systems/jhotdraw/r2 … r200` (source files only, first-parent history), plus
`commits.xml`, `changes.csv` and `manifests/`. Revision 1 of JHotDraw is an empty
CVS import, so the window starts at r2. A local clone path also works instead of a URL.

## Step 2 — run the three setups (one command)

```bash
bash scripts/NiCadCompare functions java systems/jhotdraw type3 2 200
```

It runs, one after another:

| Setup | What it does | Output folder |
|---|---|---|
| B  | full NiCad on every revision | `systems/jhotdraw/nicadfull_type3/` |
| IB | incremental extraction + full detection | `systems/jhotdraw/nicadincext_type3/` |
| II | NiCad++: both phases incremental | `systems/jhotdraw/nicadinc2_type3/` |

and then writes the comparison (times, comparisons, equality with B):
`systems/jhotdraw/nicad_3setups_functions_type3.md` and `.csv`.

Options: add `ib,ii` to run only some setups; add a number (e.g. `8`) to run the
baseline on 8 revisions at a time (each revision is still timed on its own).

## Step 2 (alternative) — run one setup at a time

```bash
MODE=full   ./scripts/NiCadInc functions java systems/jhotdraw type3 2 200     # B
MODE=incext ./scripts/NiCadInc functions java systems/jhotdraw type3 2 200     # IB
ENGINE=clonepairs2 ./scripts/NiCadInc functions java systems/jhotdraw type3 2 200   # II
```

## Step 3 — check and compare

```bash
ENGINE=clonepairs2 MODE=verify  ./scripts/NiCadInc functions java systems/jhotdraw type3 2 200   # II = B?
ENGINE=clonepairs2 MODE=compare ./scripts/NiCadInc functions java systems/jhotdraw type3 2 200   # table
```

## Output of each setup

- `nicadinc_functions_type3.csv` — one row per revision: change set, fragments, pairs,
  classes, comparisons and the time of each part (extraction, detection, classes, bookkeeping)
- `r<N>_functions…-clones/` — NiCad's clone pairs and classes for every revision
  (class ids are kept across revisions)
- `nicadinc_functions_type3.log` — every step

## Good to know

- **Resume:** a stopped run continues where it stopped; run the same command again.
  Delete a setup's output folder to start it again.
- **Initial revisions:** the first revision, and a revision where ≥ 80 % of the files changed,
  run full NiCad in every setup.
- **Other settings:** `STEPS=1` prints every step, `PROGRESS=0` hides the progress bar,
  `FULL_PCT=off` disables the 80 % rule.
- **Errors after copying from Windows** (`$'\r': command not found`): run `bash NiCadPP/install.sh` again.
- **"in use by another run":** a previous run was killed; delete `systems/<name>/.nicad_lock`.
