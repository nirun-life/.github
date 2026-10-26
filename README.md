# Nirun

Nirun supports digital workflows for community and healthcare operations. We build practical digital infrastructure that helps community health programs, care teams, public health implementers, and healthcare service organizations coordinate work and deliver care more effectively.

This repository, `.github`, serves three purposes for the [nirun-life](https://github.com/nirun-life) organization:

1. **Org profile README** — this file renders on the [organization page](https://github.com/nirun-life).
2. **Default community health files** — `CODE_OF_CONDUCT.md`, `SECURITY.md`, `.github/pull_request_template.md`, and `.github/ISSUE_TEMPLATE/` apply org-wide to any repository that does not define its own.
3. **Reusable workflows** — shared CI used by the organization's repositories (see below).

## Reusable Workflows

### `build-image.yml`: build, scan, push, and sign an application image

Builds one Docker image, scans it, pushes it to `ghcr.io/<image>` (optionally mirrored to Docker
Hub) with an SBOM and provenance attestation, signs the digest with keyless cosign, and returns the
digest. Deployments take the image by digest (`image@sha256:…`), never by tag.

```yaml
on:
  push:
    branches: ["16.0"]
    tags: ["16.0.*"]
  pull_request:
    branches: ["16.0"]

jobs:
  image:
    uses: nirun-life/.github/.github/workflows/build-image.yml@<commit-sha> # pin a commit
    permissions:
      contents: read
      packages: write
      id-token: write
    with:
      image: nirun-life/nirun
      dockerhub-image: nirun/nirun # optional mirror
    secrets:
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
```

| Input | Default | Meaning |
| --- | --- | --- |
| `image` | calling repository | Image name without registry, pushed to `ghcr.io/<image>` |
| `context` / `dockerfile` | `.` / `<context>/Dockerfile` | Build context and Dockerfile |
| `build-args` | — | Build arguments, one `KEY=value` per line |
| `platforms` | `linux/amd64` | Platforms of the pushed image (the scan runs on `linux/amd64`) |
| `dockerhub-image` | — | Optional Docker Hub mirror; needs the `dockerhub-username` and `dockerhub-token` secrets |
| `scan-severity` | `CRITICAL` | Fixable vulnerabilities at these severities fail the build |
| `trivyignore` | `.trivyignore` | Exception file in the calling repository |

Outputs: `pushed` (`true`/`false`), `digest` (`sha256:…`), `image` (`ghcr.io/<image>@<digest>`,
the value to deploy), and `tags`.

Behavior:

- **Pull requests** build and scan only: nothing is pushed, signed, or logged in. Other events push.
  Never call the workflow from `pull_request_target`.
- **Vulnerability policy:** fixable CRITICAL findings fail the build. HIGH findings are reported in
  the run summary but do not block. Accepted exceptions go in the calling repository's
  `.trivyignore`, one CVE per line, each with a comment giving the reason and a review date.
- **Tags:** branch name, git tag, pull request number, and `sha-<commit>`. Tags are for people;
  deploy by digest.
- **Signing is always on.** Keyless cosign needs no keys to manage. For private repositories it
  records the repository name, commit, branch, image digest, and run link in Sigstore's public
  transparency log; source code, images, logs, and secrets stay private.
- **Verify a signature:** the signing identity is this workflow, not the calling repository:

  ```bash
  cosign verify ghcr.io/nirun-life/nirun@sha256:… \
    --certificate-oidc-issuer https://token.actions.githubusercontent.com \
    --certificate-identity-regexp '^https://github[.]com/nirun-life/[.]github/[.]github/workflows/build-image[.]yml@'
  ```

`build-image-selftest.yml` checks every change to the workflow: a pull request must not push, and a
push must return a digest that resolves in GHCR and has a valid signature.

## What We Work On

Our work focuses on mission-driven digital infrastructure for care delivery, including operational workflows, service coordination, and tools that support day-to-day implementation.

## Get Involved

If you would like to contribute, please review the `CONTRIBUTING.md` file in the repository you are interested in and contact us at [nirun@nstda.or.th](mailto:nirun@nstda.or.th).

## Contribution Notice

Unless stated otherwise, contributions to repositories in this organization grant copyright to [NSTDA](https://www.nstda.or.th/).
