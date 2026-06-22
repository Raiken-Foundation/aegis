# Publishing Aegis Releases

This repository is the public distribution repo for Aegis binaries. The private
source repository builds release assets, then those assets are uploaded here as
GitHub Release files.

## Build Assets From Source

From the private `aegis-core` checkout:

```bash
dist build --artifacts=host
dist build --artifacts=local --target x86_64-apple-darwin
dist build --artifacts=global
```

Patch generated installers so they download from this public repo:

```bash
python3 - <<'PY'
from pathlib import Path
root = Path("target/distrib")
for path in list(root.glob("*installer.sh")) + list(root.glob("*installer.ps1")):
    text = path.read_text()
    text = text.replace("Raiken-Foundation/aegis-core", "Raiken-Foundation/aegis")
    text = text.replace('"name":"aegis-core"', '"name":"aegis"')
    path.write_text(text)
PY
```

Copy the generated archives/installers into this repo's ignored
`release-assets/` staging directory.

## Publish

From this public repo:

```bash
release-assets/publish.sh
```

The script verifies required staged assets and checksums, then creates the
GitHub Release in `Raiken-Foundation/aegis`.

## Verify

After publishing:

```bash
curl --proto '=https' --tlsv1.2 -LsSf \
  https://github.com/Raiken-Foundation/aegis/releases/latest/download/aegis-cli-installer.sh | sh

aegis --version
aegis doctor
aegis ai configure
aegis ai test
aegis demo init
cd aegis-targets
./run.sh node-command-injection
```
