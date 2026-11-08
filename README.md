# tde-apt

A signed apt repository for extra Trinity Desktop Environment packages on Debian 13 (trixie, amd64), published with GitHub Pages.

## Packages

All packages follow the `tde-<name>-trinity` naming scheme.

| Package | Source | Description |
|---|---|---|
| `tde-systemd-trinity` | [tde-systemd](https://github.com/Yjlion/tde-systemd) | systemd manager and journal viewer |
| `tde-flatpak-trinity` | [tde-flatpak](https://github.com/Yjlion/tde-flatpak) | Flatpak manager |
| `tde-packagekit-trinity` | [tde-packagekit](https://github.com/Yjlion/tde-packagekit) | PackageKit software manager |
| `tde-refresh-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Full Trinity Desktop Environment, curated (metapackage) |
| `tde-refresh-desktop-base-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Ceratopsian theme |
| `tde-refresh-win9x-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Windows 9x look |
| `tde-refresh-macclassic-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Mac OS Classic look |
| `tde-refresh-macosx-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Mac OS X look |

Renamed in November 2026: `tde-systemd` → `tde-systemd-trinity`, `tdeflatpak` → `tde-flatpak-trinity`, `tdepackagekit-trinity` → `tde-packagekit-trinity`. Installing the new package removes the old one.

## Use it

You also need the official Trinity repository configured.

```bash
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://yjlion.github.io/tde-apt/tde-apt.asc -o /etc/apt/keyrings/yjlion-tde.asc
echo "deb [signed-by=/etc/apt/keyrings/yjlion-tde.asc] https://yjlion.github.io/tde-apt trixie main" \
  | sudo tee /etc/apt/sources.list.d/yjlion-tde.list
sudo apt update
```

## How it works

No packages are stored in this repository. The workflow in `.github/workflows/publish.yml`:

1. downloads the `.deb` assets from the **latest GitHub release** of each source repo,
2. builds the repository with `reprepro` (configuration in `conf/distributions`) and signs it,
3. deploys the result to GitHub Pages.

It runs daily, on pushes to `conf/`, `site/` or the workflow, on manual dispatch, and on a `new-deb` repository dispatch.
The repository is rebuilt from scratch every time, so there is no state to lose. Only the newest release of each source repo is published.

## One-time setup (maintainer)

1. **Signing key** (dedicated to this repository, no passphrase):
   ```bash
   gpg --batch --passphrase '' --quick-gen-key "Yjlion TDE apt <you@example.com>" ed25519 sign 5y
   gpg --armor --export-secret-keys "Yjlion TDE apt" | gh secret set APT_GPG_KEY --repo Yjlion/tde-apt
   ```
   Back the private key up somewhere safe, offline. Key expiry is 5 years; extend or rotate before then.
2. **Pages source**: Settings → Pages → Build and deployment → Source: **GitHub Actions**.
3. **Releases**: each source repo must attach its `.deb` file(s) to a GitHub release (tag `v*`).
4. Run the workflow once from the Actions tab.

### Optional: publish immediately after a release

Add to a source repo's release workflow, with a fine-grained token (`APT_DISPATCH_TOKEN`, repository `Yjlion/tde-apt`, permission *Contents: read and write*):

```yaml
- run: gh api repos/Yjlion/tde-apt/dispatches -f event_type=new-deb
  env:
    GH_TOKEN: ${{ secrets.APT_DISPATCH_TOKEN }}
```

## Troubleshooting

- `No section and no priority`: the `.deb` has no `Section`/`Priority` in `debian/control`; add them in the source repo.
- `.deb` for another architecture: add it to `Architectures:` in `conf/distributions`.
- Replacing a package with the same version but different contents is rejected by reprepro; bump the version.
