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

# 2. Drop a set of real, intentionally-vulnerable demo apps into ./aegis-demo
aegis demo init

# 3. Configure an AI provider for dynamic run-and-attack synthesis
cd aegis-demo
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

## License & support

Aegis is free to use. This is the open distribution of the Aegis engine;
enterprise capabilities are offered separately. Found a bug or need a platform
build? Open an issue on this repository.
