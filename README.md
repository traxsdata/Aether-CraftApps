# Aether-CraftApps

æther builds of the Craft apps from [github.com/storytold](https://github.com/storytold),
for the æther OS at [github.com/traxsdata/Aether](https://github.com/traxsdata/Aether).

The apps are untrusted on æther: they are built from their upstream sources
without changes, run inside æther's sandboxed Rust runner, and are installed
and updated by the OS's App Store from this repository's releases.

## Building and publishing

Actions → **Build Craft apps for æther** → Run workflow
([.github/workflows/build-craft-apps.yml](.github/workflows/build-craft-apps.yml)).

| Input | Default | Meaning |
|---|---|---|
| `apps` | every app with a shell | space- or comma-separated app names, e.g. `gridcraft photocraft` |
| `aether_ref` | `main` | the Aether branch, tag or commit to build against |
| `release_tag` | `craft-apps-latest` | the release to publish to |
| `publish_release` | on | publish the packages, or only keep them as workflow artifacts |

The workflow checks out Aether, clones each app's source, builds it against
Aether's `runner/eframe` with `tools/build-craft.sh`, and publishes to the
release:

- `<app>-<version>-aether-x86_64.aeapp`: the æther package, byte for byte.
  App Store installs this one. (GitHub refuses release assets ending in `.app`.)
- `<app>-<version>-aether-x86_64.app.zip`: the same package as a `.app` file,
  for manual download.
- `craft-apps.json` and `<app>.json`: each package's version, upstream source
  commit, Aether commit, size and SHA-256. A run that rebuilds some apps keeps
  the other apps' entries, and removes older versions of the apps it rebuilt.

The build tooling and the per-app shells (`tools/craft-shells/<app>.rs`) live
in the Aether repository, next to the runner they build against. Today
GridCraft and PhotoCraft have shells; add a shell there before adding an app
here.

## Using the packages

- **App Store** on æther reads the `craft-apps-latest` release and requires
  GitHub's SHA-256 digest for each asset; the kernel re-checks the package
  header, SHA-256 and ELF image before installing it.
- **The ISO**: in an Aether checkout, `make craft-apps && make` downloads the
  packages, checks their SHA-256, and puts them in the ISO's `/apps`.

SHA-256 ties a download to this release; it does not prove who published it.
Package signing is on Aether's roadmap.
