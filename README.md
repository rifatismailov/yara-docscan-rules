# DocScan rules

YARA rules used by [DocScan](https://github.com/rifatismailov/yara-docscan) — a YARA 4.6 fork with a `docscan`
module that parses Office (OLE / OOXML / RTF / ODF), PDF and LNK files, finds attack vectors (macros, DDE,
remote templates, embedded objects, exploits) and disarms documents on Windows workstations.

The set is **curated from public sources**: no rules are written here for anybody's specific files, and nothing
organisation-specific is in this repository. Critical-information rules (data-leak detection) are deployed
separately by each organisation and are never part of this repository.

## Layout

| Path | What |
|---|---|
| `docscan-index.yar` | entry point: `docscan-policy.yar` + `file-index.yar` (what DocScan compiles with `yarac`) |
| `docscan-policy.yar` | DocScan policy: which `docscan` module findings are a detection, their severity and the human-readable text of an alert; what is disarmed or quarantined |
| `docscan.yar` | minimal rule set for the `docscan` module alone |
| `file-index.yar` | includes of the vendor rules below |
| `maldoc/<source>/` | document-focused vendor rules (macros, RTF, PDF, OLE …) |
| `general/` | rules for executables, scripts and archives (droppers, loaders, RATs, hack tools, vulnerable drivers …) |
| `tools/` | `build_vendor_rules.py` (rebuilds `general/`, `maldoc/`, `file-index.yar` from pinned upstream commits), `vendor-selection.txt` (which upstream files are used), `vendor-report.md` (every rule taken or dropped and why), `make_candidates.py`, `fp_scan.ps1` (false-positive scan on a clean Windows) |
| `LICENSE-*` | licenses of the upstream sources |

## Sources and licenses

Rules keep their original `meta` (author, reference, license) and come from:
[YARA Forge](https://github.com/YARAHQ/yara-forge) (core), [signature-base](https://github.com/Neo23x0/signature-base),
[Elastic protections-artifacts](https://github.com/elastic/protections-artifacts),
[ReversingLabs](https://github.com/reversinglabs/reversinglabs-yara-rules), [InQuest](https://github.com/InQuest/yara-rules),
[ditekshen](https://github.com/ditekshen/detection), [bartblaze](https://github.com/bartblaze/Yara-rules),
[Yara-Rules community](https://github.com/Yara-Rules/rules). Each source is under its own license (`LICENSE-<source>`);
DocScan's own files (`docscan*.yar`, `file-index.yar`, `tools/`) follow the license of the DocScan repository.

What is dropped when a source is taken (see `tools/build_vendor_rules.py`): rules that can never fire on a Windows
workstation (ELF / Mach-O / Android / server-side webshells), duplicates of other sources, slow rules, rules with
false positives on clean Windows systems and on the public test corpora; indicator-only rules get the tag
`docscan_info` (reported, but they do not raise the verdict).

## Use

The rules need DocScan's `yara64` / `yarac64` (the `docscan` module and `--docscan-*` options):

```
yarac64 -w -d filename= -d filepath= -d extension= -d filetype= -d owner= docscan-index.yar rules.yarc
yara64 -w --docscan-json -C rules.yarc <file>
```

### On a PC

Download the rules bundle of a release **before** installing DocScan and give it to the installer:

```powershell
# https://github.com/rifatismailov/yara-docscan-rules/releases/latest/download/DocScan-rules-bundle.zip
powershell -ExecutionPolicy Bypass -File DocScan-Install.ps1 -Rules C:\Temp\DocScan-rules-bundle.zip   # or a URL / UNC path
```

The installer checks the DocScan CI signature and every file, compiles the rules and only then installs; without
rules DocScan is not installed. Afterwards the DocScanUpdater service keeps the rules up to date on its own (GitHub,
an organisation mirror or an agent marker) and rolls back when the watcher does not start with new rules.

### Releases

`vX.Y.Z` (own numbering): `DocScan-rules-bundle.zip` (manual install), `DocScan-rules.zip` + `rules-manifest.json` +
`rules-manifest.sig` (the updater). Signed with ed25519 by the DocScan CI; compiled with DocScan's `yarac` before
publishing.

## Status

This repository is the home of the DocScan rule set; signed releases and rule-only updates on the PCs work. For a
transition period the DocScan program package still carries a copy of the rules (`docscan-project/rules` of
[yara-docscan](https://github.com/rifatismailov/yara-docscan)); next, the program comes without rules and CI tests
the rules on the corpora before each rules release.
