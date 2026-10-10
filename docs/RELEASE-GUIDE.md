# GitHub Release Guide (Binary-only)

## Public repository files

Keep only these files in the normal Git repository:

- `README.md`
- `LICENSE`
- `CHANGELOG.md`
- `.gitignore`
- `config.example`
- `whitelist.example`
- `docs/RELEASE-GUIDE.md`

Do **not** commit `main.py`, `integrity.py`, other Python source files, `.venv/`, local `config`, local `whitelist`, logs, runtime cache, quarantine data or crash reports.

## Prepare the release archive locally

Build the binary locally. Then create a clean staging folder. Example commands (adjust paths and confirm that the database may legally be redistributed):

```bash
VERSION=0.1.0
ARCH=amd64
STAGE="semvirsol-linux-${ARCH}-v${VERSION}"
mkdir -p "release/${STAGE}"
install -m 0755 ./semvir-cli "release/${STAGE}/semvir-cli"
cp ./semvir-rules.smdb "release/${STAGE}/semvir-rules.smdb"
cp ./config.example "release/${STAGE}/config"
cp ./whitelist.example "release/${STAGE}/whitelist"
cp ./README.md ./LICENSE "release/${STAGE}/"

tar -C release -czf "${STAGE}.tar.gz" "${STAGE}"
sha256sum "${STAGE}.tar.gz" > "${STAGE}.tar.gz.sha256"
```

The sample config deliberately sets `quarantine.enabled` to `false`. Review `config` on the target system, set watch paths and permissions, and only then enable quarantine if required. Do not package your personal config or whitelist.

## Upload to GitHub

1. Push the documentation/example files to the default branch.
2. Open **Releases** → **Draft a new release**.
3. Create tag `v0.1.0` and title `SemvirSol v0.1.0 — Linux Beta`.
4. Attach the generated `.tar.gz` and `.tar.gz.sha256` files as release assets.
5. Include the database version, changes, known limitations and architecture in the release notes.
6. Publish and verify the assets by downloading them in a clean Linux environment.

Keep the custom `LICENSE` at the repository root and include the same file in each release archive. In the GitHub repository creation/picker UI, do not select MIT/GPL/Apache: add this custom `LICENSE` file manually. GitHub may not classify a custom licence as a recognised open-source licence.

## Before every release

- Check the binary works on a clean compatible Linux system.
- Check `./semvir-cli help` and `./semvir-cli version`.
- Check the DB version in the shipped `.smdb` matches the release notes.
- Check file modes, sample config paths and log/quarantine write permissions.
- Run benign detection tests and review false positives/quarantine behaviour.
- Re-check licensing/redistribution permission for the database and dependencies.
