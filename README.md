# fullstory/ci

Reusable GitHub Actions workflows for the fullstory repositories. These
builds are for development; the official aptosid uploads stay slh's.

`deb.yml` builds a Debian source the way Debian's buildds do: a sid
container makes the source package, then sbuild builds it as a normal
user in a fresh unshare chroot without network, so nothing the workflow
installs can stand in for a missing Build-Depends. Source and arch:all
plus amd64 come from an amd64 runner, arm64 binaries from an arm64
runner when the source has any. A `debian/*` tag is checked against the
changelog (DEP-14); lintian, piuparts and autopkgtest report as warnings
only; every file is attested and uploaded as the artifacts `deb-amd64`
and `deb-arm64`. Callers pin `@v1`:

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
empty), `version` (set with dch before the build, for pre-releases),
`prepare` (shell run in the source as root before the source package is
made; a debian-only quilt package such as linux-aptosid fetches and
unpacks its orig there, and what it installs never reaches the build),
`piuparts` and `autopkgtest` (both on by default; off for sources whose
dependencies are not all in Debian, or too costly to test).

The job container is privileged and lifts the runner's AppArmor
restriction on unprivileged user namespaces, which sbuild's unshare mode
needs; the runner VM is thrown away after the job.
