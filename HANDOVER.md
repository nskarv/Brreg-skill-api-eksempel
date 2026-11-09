# HANDOVER.md

State of the Brreg tool for whoever works on it next, person or Claude Code. Read this first. Keep it a snapshot: replace outdated statements, remove resolved items, keep the log short. History is in git.

Last updated: 2026-11-09

## What this is

The example from part 4 of the KI series on nskarv.no (/notater/bygg-et-ki-verktoy/): search and lookup in Enhetsregisteret at Brønnøysundregistrene, used from a Claude project with a linked folder, and at the end turned into a skill with /skill-creator.

The files live in two places:

- The website repository, folder `tools/brreg/`. The website shows and serves `skill/brreg-oppslag/SKILL.md`, `skill/brreg-oppslag/scripts/brreg_oppslag.py` and `instructions.md` in step 2 of the article, and its live lookup card (src/components/BrregDemo.astro) mirrors the script's logic.
- The public repository, github.com/nskarv/Brreg-skill-api-eksempel (MIT).

Edit in the website repository, then copy the folder to the public repository and push. The two must stay identical. Nothing private may go into this folder.

## Files

| File | Purpose |
| --- | --- |
| `skill/brreg-oppslag/SKILL.md` | The recipe Claude follows, written as a skill (`name: brreg-oppslag`; the folder name must match for an upload to Claude). |
| `skill/brreg-oppslag/scripts/brreg_oppslag.py` | About 120 lines, standard library only. Search with `navnMetodeForSoek=FORTLOEPENDE` and `sort=antallAnsatte,DESC` (five hits, `viser` says which), lookup by number with a modulus 11 check first, one status field, short JSON, errors as `{"feil": ...}` with exit code 1, a network message that names Settings, Capabilities, and a small injection check with `advarsel`. |
| `skill/brreg-oppslag/scripts/brreg_oppslag.ps1` | The same script for Windows PowerShell 5.1 and 7. Must stay UTF-8 with BOM. Output identical to the Python version (compared as JSON on six queries, 2026-11-09). |
| `instructions.md` | Project instructions pasted under Instructions in a Claude project with the folder linked. |
| `README.md` | Public description in Norwegian. |

## Data terms

Enhetsregisteret is open data under NLOD 2.0; name Brønnøysundregistrene as the source. Brreg asks that entities answering 410 are removed from copies; the script stores nothing.

## Before every push

1. Run both scripts on: a nonsense name («Xyzzqqv Bakeri AS», expect «ingen treff»), «telenor» (expect Telenor Norge AS first), «Equinor», 923609016, 923609017 (check digit error, no request) and 938096392 (injection warning). Compare the Python and PowerShell output as JSON.
2. Check that the website card gives the same answers.
3. Nothing private in the folder: no names other than the owner's copyright line, no server details.

## Log

- 2026-11-09: the folder now holds only what part 4 uses. Removed the MCP server `brreg.py` with its tests (`test_brreg.py`, `smoke_test.py`), the UTCP manual, the text-only `SKILL.md`, `SYSTEMPROMPT.md` and `actions/brreg.yaml` (owner: UTCP and MCP are no longer needed; delete the old files). The script now searches the whole name and sorts by employees, after the owner's /skill-creator run in the Claude app found single-word matches, poorly chosen hits and a misleading network message. README rewritten for the new set.
- 2026-11-01: first version, with the MCP server, the text skill, the system prompt and ChatGPT Actions (in the public repository's history).
