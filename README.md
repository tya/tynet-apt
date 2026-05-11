# tynet-apt

GitHub Pages-hosted apt repository for `tynet` packages.

The `gh-pages` branch is what apt actually consumes:

```
deb [signed-by=/etc/apt/trusted.gpg.d/tynet.gpg] https://tya.github.io/tynet-apt stable main
```

## How it works

Each consumer repo (currently just [`tynet-cloud-init`](https://github.com/tya/tynet-cloud-init)) appends a step to its release workflow that fires `repository_dispatch` here when a tag is published. The `ingest.yml` workflow downloads the released `.deb`, drops it into `pool/main/<first-letter>/<package>/`, regenerates `Packages` /
`Packages.gz` with `dpkg-scanpackages`, regenerates the `Release` file with
`apt-ftparchive`, and signs `Release` / `InRelease` with the tynet apt key
(GPG fingerprint `47AA8444945F450A`).

The signed result is committed back to `gh-pages` and served via Pages.

## Layout (gh-pages branch)

```
pool/main/s/serve-cloud-init/serve-cloud-init_<ver>_arm64.deb
dists/stable/main/binary-arm64/Packages
dists/stable/main/binary-arm64/Packages.gz
dists/stable/Release
dists/stable/Release.gpg
dists/stable/InRelease
```

## Manual ingest

```sh
gh workflow run ingest.yml \
  -f repo=tya/tynet-cloud-init \
  -f tag=v0.4.0 \
  -f package=serve-cloud-init \
  -f asset_url=https://github.com/tya/tynet-cloud-init/releases/download/v0.4.0/serve-cloud-init_0.4.0_arm64.deb
```

## Required secrets

- `TYNET_APT_GPG_KEY` — ASCII-armored private key for `47AA8444945F450A`.
  In 1Password as `APT_REPO_GPG_KEY` (same value the `apt-mirror` Ansible role uses).
- `TYNET_APT_GPG_PASSPHRASE` — empty if the key has no passphrase; otherwise the passphrase.
