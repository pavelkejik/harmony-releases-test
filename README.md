# harmony-releases-test

Test channel for signed iNELS Harmony releases. Servers install from here with
`harmony-update` (`RELEASES_REPO` in `/etc/harmony/harmony.conf`).

## New server

```sh
curl -fsSL https://github.com/pavelkejik/harmony-releases-test/releases/latest/download/install.sh \
  | sudo HARMONY_RELEASES_REPO=pavelkejik/harmony-releases-test bash
```

## Files of a release

| File | What it is |
|---|---|
| `harmony-X.Y.Z.tar.gz` | install/upgrade package |
| `harmony-X.Y.Z.manifest.json` | version, source commit, sha256 of the package |
| `harmony-X.Y.Z.manifest.json.sig` | signature of the manifest (`ssh-keygen -Y`, namespace `harmony-release`) |
| `install.sh` | installer for new servers |

Verify by hand:

```sh
ssh-keygen -Y verify -f allowed_signers -I harmony-release -n harmony-release \
  -s harmony-X.Y.Z.manifest.json.sig < harmony-X.Y.Z.manifest.json
sha256sum harmony-X.Y.Z.tar.gz   # must equal artifact.sha256 in the manifest
```

## Workflows

| Workflow | Started by | What it does |
|---|---|---|
| Publish candidate | the source repo's Release run | signs the build, publishes it as a candidate (pre-release) for staging, removes older candidates |
| Promote release | a person, after testing on staging | checks signature and checksum, makes the candidate the latest release |
| Withdraw release | a person, if a release turns out bad | servers stop offering it; servers running it keep it until the next release |

All run in the protected `production` environment (main only). Published
versions are never replaced and never go backwards.

Settings in environment `production`:

- secret `SIGNING_KEY`: the private signing key
- secret `RELEASE_APP_PRIVATE_KEY`, variable `RELEASE_APP_CLIENT_ID`: the release
  GitHub App (owned by the account/org, not a person), used to read the build from
  the source repo
- variable `SOURCE_REPO` (default `pavelkejik/harmony-v2-dev`)

`allowed_signers` lists the public keys servers trust (main and backup key).
