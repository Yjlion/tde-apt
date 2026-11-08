# tde-apt

A signed apt repository for extra Trinity Desktop Environment packages on Debian 13 (trixie) and Debian 12 (bookworm), amd64, published with GitHub Pages.

## Packages

All packages follow the `tde-<name>-trinity` naming scheme, except `tde-youtube`, which keeps its upstream package name.

| Package | Source | Description | Debian |
|---|---|---|---|
| `tde-systemd-trinity` | [tde-systemd](https://github.com/Yjlion/tde-systemd) | systemd manager and journal viewer | trixie, bookworm |
| `tde-flatpak-trinity` | [tde-flatpak](https://github.com/Yjlion/tde-flatpak) | Flatpak manager | trixie |
| `tde-packagekit-trinity` | [tde-packagekit](https://github.com/Yjlion/tde-packagekit) | PackageKit software manager | trixie |
| `tde-firewalld-trinity` | [tde-firewalld](https://github.com/Yjlion/tde-firewalld) | firewalld manager (Security & Privacy → Firewall) | trixie, bookworm |
| `tde-refresh-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Full Trinity Desktop Environment, curated (metapackage) | trixie |
| `tde-refresh-desktop-base-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Ceratopsian theme | trixie |
| `tde-refresh-win9x-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Windows 9x look | trixie |
| `tde-refresh-macclassic-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Mac OS Classic look | trixie |
| `tde-refresh-macosx-trinity` | [tde-refresh](https://github.com/Yjlion/tde-refresh) | Mac OS X look | trixie |
| `tde-youtube` | [tde-youtube](https://github.com/Yjlion/tde-youtube) | YouTube browser and player | trixie, bookworm |

Renamed in November 2026: `tde-systemd` → `tde-systemd-trinity`, `tdeflatpak` → `tde-flatpak-trinity`, `tdepackagekit-trinity` → `tde-packagekit-trinity`. Installing the new package removes the old one.

## Use it

You also need the official Trinity repository configured for the same Debian release.

Add the signing key:

```bash
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://yjlion.github.io/tde-apt/tde-apt.asc -o /etc/apt/keyrings/yjlion-tde.asc
```

Then add the list for your Debian release and update.

Debian 13 (trixie):

```bash
echo "deb [signed-by=/etc/apt/keyrings/yjlion-tde.asc] https://yjlion.github.io/tde-apt trixie main" \
  | sudo tee /etc/apt/sources.list.d/yjlion-tde.list
sudo apt update
```

Debian 12 (bookworm):

```bash
echo "deb [signed-by=/etc/apt/keyrings/yjlion-tde.asc] https://yjlion.github.io/tde-apt/bookworm bookworm main" \
  | sudo tee /etc/apt/sources.list.d/yjlion-tde.list
sudo apt update
```

Only packages whose source repo releases a bookworm build are in the bookworm suite (see the table above).

## How it works

No packages are stored in this repository. The workflow in `.github/workflows/publish.yml`:

1. downloads the `.deb` assets from the **latest GitHub release** of each source repo,
2. sorts them by Debian release: a file with `bookworm` in its name (`1.0~bookworm` version or `_bookworm.deb` suffix) goes to the `bookworm` suite, every other file to `trixie`,
3. builds the repository with `reprepro` (configuration in `conf/distributions`) and signs it. Each suite has its own pool, so builds for both releases may share a version number: `trixie` is at the site root, `bookworm` under `/bookworm`,
4. deploys the result to GitHub Pages.

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
- Package missing from bookworm: the source repo must attach a separate bookworm build whose file name contains `bookworm`; unmarked files are published for trixie only.
- Replacing a package with the same version but different contents is rejected by reprepro; bump the version.
