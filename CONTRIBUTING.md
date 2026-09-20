# Contributing to corp-lab

Contributions are welcome. This project is maintained by the Cyberanzen Security Club at SRM IST-Trichy.

## What to Contribute

**Good contributions:**
- New AD attack surfaces with setup script + attack command + expected output
- Additional misconfiguration patterns found in real-world AD environments
- Bug fixes in existing setup scripts
- Documentation improvements and clarifications
- Translations of docs to other languages

**Not accepted:**
- Techniques for attacking unauthorized systems
- Content that violates the legal disclaimer in SECURITY.md
- Scripts that require internet access from the lab VMs (defeats isolation)

## How to Contribute

1. Fork the repository
2. Create a branch: `git checkout -b feature/new-attack-surface`
3. Make your changes
4. Test your changes against a fresh DC01 build
5. Commit with a clear message: `git commit -m "Add: PrintNightmare misconfiguration"`
6. Push and open a Pull Request

## Script Standards

All PowerShell scripts must:
- Start with `#Requires -RunAsAdministrator`
- Include a comment block explaining what the script does and WHY it creates a vulnerability
- Echo `[OK]`, `[ERROR]`, or `[WARN]` prefixes on all Write-Host output
- Handle errors with `-ErrorAction Stop` where failure would break downstream scripts
- Tell the user what to run next at the end

All Bash scripts must:
- Start with `#!/bin/bash`
- Use `set -e` for error handling
- Print a summary at the end with output file locations

## Testing

Before submitting, verify:
- Script runs cleanly on a fresh Windows Server 2022 Standard (Desktop Experience) install
- All expected attack commands from Kali return the described output
- `attacks/verify-lab.sh` still passes all checks after your changes

## Questions

Open a GitHub Discussion for questions about the lab or specific attack techniques.
