# Aegis

**Aegis proves whether a security finding is _actually exploitable_.** Instead of
pattern-matching source code, Aegis runs your app inside a hardened sandbox and
attacks it over loopback. A `Verified` verdict means the vulnerability was
genuinely exploited — not that a regex matched a line of code.

This repository hosts the prebuilt, free Aegis binary and its installers.

---

## Install

**macOS / Linux**

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/Raiken-Foundation/aegis/releases/latest/download/aegis-cli-installer.sh | sh
```

**Windows (PowerShell)**

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/Raiken-Foundation/aegis/releases/latest/download/aegis-cli-installer.ps1 | iex"
```

The installer detects your platform, downloads the matching checksummed binary,
and puts `aegis` on your `PATH`. Verify with:

```bash
aegis --version
```

> **Available prebuilt platforms:** macOS (Apple Silicon + Intel) are published
> today. Linux (x64/arm64) and Windows (x64) are produced by the release
> pipeline — if your platform isn't in the [latest release](https://github.com/Raiken-Foundation/aegis/releases/latest)
> yet, open an issue.

Prefer to grab a binary by hand? Download the `aegis-cli-<target>.tar.xz` asset
from a [release](https://github.com/Raiken-Foundation/aegis/releases), verify it
against the published `.sha256`, extract, and move `aegis` onto your `PATH`.

---

## Quickstart (about a minute)

```bash
# 1. Check your environment (container runtime, etc.)
aegis preflight

# 2. Drop a set of real, intentionally-vulnerable demo apps into ./aegis-targets
aegis demo init

# 3. Configure an AI provider for dynamic run-and-attack synthesis
cd aegis-targets
./configure-ai.sh            # pick DeepSeek / OpenAI / OpenRouter / Ollama / custom

# 4. Let Aegis launch a vulnerable app in the sandbox and attack it
./run.sh node-command-injection
```

Full evidence for each run — the verdict, telemetry, the AI attack plan, and a
replay manifest — lands under `.aegis/artifacts/<run-id>/`.

The bundled demo targets are intentionally vulnerable, self-contained apps
(Node command injection, Python SQL injection, Go path traversal). **Never
deploy them.**

---

## How it works

Aegis takes a security finding, evaluates it against policy, provisions an
isolated sandbox, runs an approved validation routine, collects runtime
evidence, and returns a reproducible verdict: **Verified**, **Not Verified**, or
**Inconclusive**.

- **Static validation** scans code for known-vulnerable patterns. Fast, but it
  can't tell whether the path is actually reachable or exploitable.
- **Dynamic validation** (`aegis validate --dynamic`) asks a model to design a
  *run-and-attack* plan — a start command, a readiness probe, and an attack
  script. Aegis runs that plan in the sandbox and the **engine** (not the model)
  decides the verdict from `AEGIS_VERIFIED` / `AEGIS_NOT_VERIFIED` markers.

That's why dynamic wins on subtle bugs: e.g. a Go path-traversal that uses
`filepath.Join` won't trip static pattern recipes, yet the running app still
leaks `/etc/passwd` — and the dynamic attack proves it.

---

## Command reference

| Command | What it does |
|---|---|
| `aegis preflight` | Check the host: container runtime, available sandbox backends. |
| `aegis demo init` | Write the bundled vulnerable target apps + helper scripts into `./aegis-targets` (no checkout needed). |
| `aegis validate --finding <f.json> --repo <dir>` | Validate a single finding against a repo. Add `--dynamic` to run-and-attack. |
| `aegis scan --repo <dir> --scanner semgrep\|trivy --validate` | Run a scanner in a container, import its findings, and validate them. Add `--dynamic` to attack. |
| `aegis import --sarif <r.sarif> --validate` | Convert any SARIF report into Aegis findings and validate them. |
| `aegis triage --repo <dir> --finding <f.json>` | Map a finding to repo context and suggest validation recipes. |
| `aegis index --repo <dir>` | Infer project type and security context for a repository. |
| `aegis ci --finding <f.json> --repo <dir>` | Validate in CI mode with stable exit codes. |
| `aegis replay --artifact <a.json>` | Re-run a previous validation artifact for reproducibility. |
| `aegis artifacts list \| prune` | Manage stored validation artifacts. |

Run `aegis <command> --help` for the full flag list. Key flags shared by the
validation commands: `--policy <toml>`, `--profile <name>`, `--artifacts-dir
<dir>`, `--ai` (enable AI assistance), and `--dynamic` (run-and-attack; implies
`--ai` and requires `sandbox.allow_app_runtime` in policy).

---

## Requirements

1. **A container runtime** — Docker or OrbStack. Confirm with `aegis preflight`.
2. **An OpenAI-compatible API key** for dynamic run-and-attack synthesis. Aegis
   is provider-agnostic: DeepSeek, OpenAI, OpenRouter, a local Ollama, or any
   custom OpenAI-compatible endpoint all work. `configure-ai.sh` (shipped by
   `aegis demo init`) writes your choice to a gitignored `.env`; your key never
   touches a tracked file.

Static validation needs neither a model nor a network.

---

## Updating & uninstalling

- **Update:** re-run the install command above to fetch the latest release.
- **Uninstall:** remove the installed `aegis` binary (the installer prints its
  location, typically `~/.local/bin/aegis`).

---

## Troubleshooting

- **Installer fails with `404`** — there is no release asset for your platform
  yet. Check the [releases page](https://github.com/Raiken-Foundation/aegis/releases):
  if it's empty, no release has been published; if it exists but lacks your
  platform, that build isn't out yet (see the platform note under *Install*).
- **`aegis preflight` reports no backend** — install and start Docker or
  OrbStack, then re-run. Dynamic runs need a working container runtime.
- **Dynamic run says the plan was invalid / no verdict** — the model must be
  capable enough to emit a valid run-and-attack plan with the
  `AEGIS_VERIFIED` / `AEGIS_NOT_VERIFIED` markers. Use a strong hosted model
  (re-run `./configure-ai.sh` to switch providers); very small local models
  often fail here.
- **`command not found: aegis`** — the install dir isn't on your `PATH`. Open a
  new shell, or add the directory the installer printed (e.g. `~/.local/bin`).
- **macOS "cannot be opened because the developer cannot be verified"** — the
  binary is unsigned for now. Run `xattr -d com.apple.quarantine "$(command -v aegis)"`.

---

## License & support

Aegis is free to use. This is the open distribution of the Aegis engine;
enterprise capabilities are offered separately. Found a bug or need a platform
build? Open an issue on this repository.
