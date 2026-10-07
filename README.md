# fullstory/ci

Reusable GitHub Actions workflows for the fullstory repositories. These
builds are for development; the official aptosid uploads stay slh's.

`deb.yml` builds a Debian source in a sid container: source and arch:all
plus amd64 on an amd64 runner, arm64 binaries on an arm64 runner when the
source has any, a `debian/*` tag checked against the changelog (DEP-14),
lintian as a report only, every file attested and uploaded as the
artifacts `deb-amd64` and `deb-arm64`. Callers pin `@v1`:

    jobs:
      build:
        uses: fullstory/ci/.github/workflows/deb.yml@v1
        permissions:
          contents: read
          id-token: write
          attestations: write
          artifact-metadata: write
        with:
          ref: refs/tags/${{ github.ref_name }}

Inputs: `ref` (required), `repository` (owner/repo, the caller when
empty), `version` (set with dch before the build, for pre-releases) and
`prepare` (shell run in the source before the build, as root; a
debian-only quilt package such as linux-aptosid fetches and unpacks its
orig there).
