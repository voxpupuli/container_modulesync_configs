# Templates

This document describes the templates in `moduleroot/`. ModuleSync renders each
`*.erb` template into the target repository and removes the `.erb` suffix. For
example, `moduleroot/.github/workflows/ci.yml.erb` becomes
`.github/workflows/ci.yml`.

Many templates are static and only contain the shared ModuleSync header. Dynamic
templates read values from `@configs`. These values come from
`config_defaults.yml`, from `.sync.yml` in the target repository, or from global
ModuleSync options such as `namespace` and `puppet_module`.

## Configuration

Global defaults are defined under `:global:` in `config_defaults.yml` and apply
to multiple templates:

| Variable | Default | Usage |
| --- | --- | --- |
| `main_branches` | `['main']` | Branch filters for CI, publishing, and security scanning |
| `matrix_command` | empty | Enables a dynamic GitHub Actions matrix |
| `matrix_requires_yq` | `false` | Installs `yq` before generating the matrix |
| `build_runner` | `ubuntu-latest` | Runner for build and scan jobs |
| `build_context` | `.` | Docker/container build context |
| `build_file` | `Containerfile` | Containerfile/Dockerfile used for builds |
| `image_tag` | `ci/test:${{ github.sha }}` | Local test tag for CI and scanning |
| `build_platforms` | empty | Optional platforms for `docker/build-push-action` |
| `build_args` | `[]` | Build arguments for container builds |

File-specific defaults are defined under the target path, for example
`.github/workflows/ci.yml:`. Overrides in a target repository's `.sync.yml` use
the same path.

Example:

```yaml
---
:global:
  main_branches:
    - main
    - stable
  matrix_command: "yq -o=json '.builds' build_platforms.yaml"
  matrix_requires_yq: true
  build_args:
    - "OPENVOX_VERSION=${{ matrix.openvox_version }}"

.github/workflows/ci.yml:
  test_commands:
    - "docker run --rm ci/test:${{ github.sha }} --version"
```

## Template Overview

| Template | Target path | Purpose | Dynamic |
| --- | --- | --- | --- |
| `moduleroot/.github/CODEOWNERS.erb` | `.github/CODEOWNERS` | Requests reviews from the container maintainers | yes |
| `moduleroot/.github/labeler.yml.erb` | `.github/labeler.yml` | Labeler rule for release branches | no |
| `moduleroot/.github/release.yml.erb` | `.github/release.yml` | GitHub release note categories | no |
| `moduleroot/.github/renovate.jsonc.erb` | `.github/renovate.jsonc` | Renovate configuration for container repositories | no |
| `moduleroot/.github/workflows/build_container.yml.erb` | `.github/workflows/build_container.yml` | Builds and publishes containers | yes |
| `moduleroot/.github/workflows/ci.yml.erb` | `.github/workflows/ci.yml` | Pull request CI for container builds and tests | yes |
| `moduleroot/.github/workflows/labeler.yml.erb` | `.github/workflows/labeler.yml` | Pull request labeler workflow | no |
| `moduleroot/.github/workflows/markdownlint.yml.erb` | `.github/workflows/markdownlint.yml` | Markdownlint workflow | no |
| `moduleroot/.github/workflows/release.yml.erb` | `.github/workflows/release.yml` | GitHub release on tags | no |
| `moduleroot/.github/workflows/security_scanning.yml.erb` | `.github/workflows/security_scanning.yml` | Container scan with Anchore Grype | yes |
| `moduleroot/.github/workflows/shellcheck.yml.erb` | `.github/workflows/shellcheck.yml` | Differential ShellCheck workflow | no |
| `moduleroot/.gitignore.erb` | `.gitignore` | Shared ignore rules | no |
| `moduleroot/.markdownlint-cli2.yaml.erb` | `.markdownlint-cli2.yaml` | Ignore rules for markdownlint-cli2 | no |
| `moduleroot/.markdownlint.yaml.erb` | `.markdownlint.yaml` | Markdownlint rules | no |
| `moduleroot/Gemfile.erb` | `Gemfile` | Release gems for changelog generation | no |
| `moduleroot/RELEASE.md.erb` | `RELEASE.md` | Release process for maintainers | no |
| `moduleroot/Rakefile.erb` | `Rakefile` | Rake task for changelog generation | yes |

## Dynamic Templates

### `.github/CODEOWNERS`

Requests a review from the container maintainer team for every change.

| Variable | Type | Description |
| --- | --- | --- |
| `namespace` | String | GitHub organization, for example `voxpupuli` or `OpenVoxProject` |

Example output for `namespace: voxpupuli`:

```text
* @voxpupuli/container-maintainers
```

### `.github/workflows/ci.yml`

Creates the pull request CI workflow. The workflow builds the container image,
optionally with a matrix, runs optional test commands, and can automatically
merge Dependabot and Renovate pull requests after successful CI.

| Variable | Default | Description |
| --- | --- | --- |
| `main_branches` | `['main']` | Target branches for pull requests |
| `matrix_command` | empty | Shell command that outputs JSON for `strategy.matrix` |
| `matrix_requires_yq` | `false` | Installs `yq` in the matrix setup job |
| `general_ci_scan_dir` | empty | Enables the reusable `general_ci` workflow with ShellCheck |
| `build_job_name` | `Build ci container` | Display name of the build job |
| `build_runner` | `ubuntu-latest` | Runner for the build job |
| `build_context` | `.` | Build context |
| `build_file` | `Containerfile` | Containerfile/Dockerfile |
| `image_tag` | `ci/test:${{ github.sha }}` | Tag for the locally built image |
| `build_platforms` | empty | Optional build platforms |
| `build_args` | `[]` | List of build arguments |
| `test_repository` | empty | Optional repository checked out before tests |
| `test_commands` | `[]` | Shell commands for the `Test image` step |
| `post_test_commands` | `[]` | Shell commands for diagnostics after tests |
| `automerge` | `true` | Enables auto-merge for Dependabot and Renovate pull requests |

Example:

```yaml
---
.github/workflows/ci.yml:
  general_ci_scan_dir: "."
  test_commands:
    - "docker run --rm ci/test:${{ github.sha }} --help"
  post_test_commands:
    - "docker image inspect ci/test:${{ github.sha }}"
```

### `.github/workflows/build_container.yml`

Creates the publish workflow for container images. The workflow reacts to
pushes to `main_branches`, tags, optional schedules, and manual starts. It can
create pre-build steps, matrix builds, attestations, multi-arch manifests,
diagnostic jobs, and Docker Hub description updates.

| Variable | Default | Description |
| --- | --- | --- |
| `main_branches` | `['main']` | Branches that trigger publish builds |
| `publish_tag_patterns` | `['*']` | Tags that trigger publish builds |
| `publish_schedules` | `[]` | Cron expressions for scheduled builds |
| `matrix_command` | empty | Matrix for build jobs |
| `matrix_requires_yq` | `false` | Installs `yq` in the setup job |
| `publish_manifest_matrix_command` | empty | Matrix for manifest jobs |
| `publish_setup_outputs` | `{}` | Additional outputs from the setup job |
| `publish_setup_steps` | `[]` | Additional setup steps |
| `publish_build_job_name` | `Build and publish container` | Display name of the build job |
| `publish_runner` | empty | Runner for publish builds; falls back to `build_runner` |
| `publish_pre_build_steps` | `[]` | Steps before the build-and-publish action |
| `build_args` | `[]` | Build arguments |
| `publish_build_arch` | `linux/amd64,linux/arm64` | Architectures for the publish action |
| `build_context` | `.` | Build context |
| `build_file` | `Containerfile` | Containerfile/Dockerfile |
| `publish_tags` | `[]` | Tags for the publish action |
| `publish_attestation_subject` | empty | Enables `actions/attest` for the built image |
| `publish_manifest_steps` | `[]` | Enables the job that creates multi-arch manifests |
| `publish_info_container` | empty | Enables a diagnostic job in this container |
| `publish_info_commands` | `[]` | Commands for the diagnostic job |
| `dockerhub_repository` | empty | Enables updating the Docker Hub description |

Structure for `publish_setup_steps`, `publish_pre_build_steps`, and
`publish_manifest_steps`:

```yaml
---
.github/workflows/build_container.yml:
  publish_pre_build_steps:
    - name: "Prepare build metadata"
      id: "metadata"
      commands:
        - "echo VERSION=${GITHUB_REF_NAME#v} >> $GITHUB_ENV"
  publish_manifest_steps:
    - name: "Create manifest"
      if: "github.ref_type == 'tag'"
      commands:
        - "docker buildx imagetools create ghcr.io/voxpupuli/example:${{ github.ref_name }}"
```

Example for scheduled builds and tags:

```yaml
---
.github/workflows/build_container.yml:
  publish_schedules:
    - "0 3 * * 1"
  publish_tags:
    - "ghcr.io/voxpupuli/container-example:latest"
    - "ghcr.io/voxpupuli/container-example:${{ github.ref_name }}"
  dockerhub_repository: "voxpupuli/container-example"
```

### `.github/workflows/security_scanning.yml`

Creates the security scanning workflow. The image is built, loaded locally, and
scanned with Anchore Grype. The SARIF report is uploaded afterwards.

| Variable | Default | Description |
| --- | --- | --- |
| `main_branches` | `['main']` | Branches for push and pull request events |
| `matrix_command` | empty | Matrix for scan jobs |
| `matrix_requires_yq` | `false` | Installs `yq` in the setup job |
| `scan_job_name` | `Scan ci container` | Display name of the scan job |
| `build_runner` | `ubuntu-latest` | Runner for scan jobs |
| `build_context` | `.` | Build context |
| `build_file` | `Containerfile` | Containerfile/Dockerfile |
| `image_tag` | `ci/test:${{ github.sha }}` | Tag for the image to scan |
| `build_platforms` | empty | Optional build platforms |
| `build_args` | `[]` | Build arguments |
| `fail_build` | `false` | Allows Grype to fail the build |
| `severity_cutoff` | empty | Minimum severity for `fail-build`, for example `high` |
| `sarif_category` | empty | Category used when uploading the SARIF report |

Example:

```yaml
---
.github/workflows/security_scanning.yml:
  fail_build: true
  severity_cutoff: high
  sarif_category: container-image
```

### `Rakefile`

Defines an optional `rake changelog` task through
`github_changelog_generator`. If the gem is not installed, the task is not
registered.

| Variable | Type | Description |
| --- | --- | --- |
| `namespace` | String | GitHub organization for the changelog generator |
| `puppet_module` | String | Repository/module name for the changelog generator |

The task sets `future_release` from the current branch name. This turns
`release-v1.2.3` into `v1.2.3`.

## Static Templates

### `.github/labeler.yml` and `.github/workflows/labeler.yml`

The labeler configuration marks pull requests from `release-*` branches with
`skip-changelog`. The workflow runs on `pull_request_target` and is restricted
to repositories under `voxpupuli` or `OpenVoxProject`.

### `.github/release.yml` and `.github/workflows/release.yml`

`release.yml` defines categories for automatically generated GitHub release
notes and excludes labels such as `duplicate`, `modulesync`, and
`skip-changelog`. The release workflow creates a GitHub release with generated
release notes on tags or manual starts.

### `.github/workflows/markdownlint.yml`

Runs `DavidAnson/markdownlint-cli2-action` for `**/*.md` on pull requests
against `main` and on manual starts.

### `.github/workflows/shellcheck.yml`

Runs Differential ShellCheck on pull requests against `main` and on manual
starts. The workflow scans the repository root and uploads SARIF as an
artifact.

### `.gitignore`

Ignores local Bundler/vendor directories, `Gemfile.lock`, and `.DS_Store`.

### `.markdownlint.yaml` and `.markdownlint-cli2.yaml`

`.markdownlint.yaml` enables the default rules, sets `line-length` to 210,
allows inline HTML for `br` and `img`, and disables `descriptive-link-text`.

`.markdownlint-cli2.yaml` ignores generated or external paths such as
`.github/**`, `.vendor/**`, `vendor/**`, `test/**`, and `CHANGELOG.md`.

### `Gemfile`

Sets the gem source from `GEM_SOURCE` or `https://rubygems.org` and defines the
optional `release` group with `faraday-retry` and
`github_changelog_generator`.

### `RELEASE.md`

Documents the release process: create the `release-vX.Y.Z` release branch,
generate the changelog, merge the pull request, then tag on `main` and push the
tags.

### `.github/renovate.jsonc`

Enables Renovate with `config:recommended`, immediate pull requests, auto-merge
by pull request, and security labels for vulnerability alerts. Additional regex
managers read versions from `build_versions.yaml`, `build_platforms.yaml`, and
`Containerfile` for Rubygems dependencies.

## Delete Defaults

`config_defaults.yml` marks some historical files for removal:

```yaml
---
.markdownlint.json:
  delete: true
.github/workflows/ci.yaml:
  delete: true
```

These entries do not have templates in `moduleroot/`; they only make ModuleSync
remove the files from target repositories.
