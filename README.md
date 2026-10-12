# harmony-releases-test

Test channel for publishing and signing iNELS Harmony releases. Servers install
releases from here with `harmony-update` (`RELEASES_REPO` in `/etc/harmony/harmony.conf`).

Every release has three files:

| File | What it is |
|---|---|
| `harmony-X.Y.Z.tar.gz` | install/upgrade package |
| `harmony-X.Y.Z.manifest.json` | version, source commit, sha256 of the package |
| `harmony-X.Y.Z.manifest.json.sig` | signature of the manifest (`ssh-keygen -Y`, namespace `harmony-release`) |

Verify by hand:

```sh
ssh-keygen -Y verify -f allowed_signers -I harmony-release -n harmony-release \
  -s harmony-X.Y.Z.manifest.json.sig < harmony-X.Y.Z.manifest.json
sha256sum harmony-X.Y.Z.tar.gz   # must equal artifact.sha256 in the manifest
```

## Publishing

Actions → **Publish release** → version + sha256 of the candidate tested on the lab.
The workflow runs in the protected `production` environment (main only), downloads
the candidate from the source repo, checks the sha256, signs the manifest and
creates the release. Published releases are never replaced; a newer version is
the only way forward.

Settings this needs: environment `production` with secret `SIGNING_KEY` (private
signing key) and secret `SOURCE_REPO_TOKEN` (fine-grained token, Contents: read-only,
source repo only). Optional variable `SOURCE_REPO` (default `pavelkejik/harmony-v2-dev`).
`allowed_signers` lists the public keys servers trust (main + backup key).
