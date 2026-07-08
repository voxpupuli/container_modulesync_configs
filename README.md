# Container ModuleSync Configs

This repository contains the shared
[ModuleSync](https://github.com/voxpupuli/modulesync) configuration for
Vox Pupuli container repositories. It manages common GitHub workflows, release
automation, Renovate configuration, linting defaults, and release support files
for the container projects listed in `managed_modules.yml`.

For the detailed template reference, see `TEMPLATES.md`. For release
maintenance of this repository, see `RELEASE.md`. User-visible changes are
tracked in `CHANGELOG.md`.

## Managed Repositories

The default target organization is configured in `modulesync.yml`:

```yaml
---
namespace: voxpupuli
git_base: 'git@github.com:'
branch: modulesync
message: "Update from voxpupuli/container_modulesync_config"
```

`managed_modules.yml` contains the Vox Pupuli container repositories managed by
default:

- `container-commitlint`
- `container-onceover`
- `container-r10k`
- `container-r10k-webhook`
- `container-renovate`
- `container-semantic-release`
- `container-test`
- `container-voxbox`

`managed_modules_openvox.yml` contains the OpenVoxProject repositories that can
be synced with an explicit namespace override:

- `container-openvoxserver`
- `container-openvoxdb`
- `container-openvoxagent`
- `container-openbolt`

## Repository Layout

| Path | Purpose |
| --- | --- |
| `modulesync.yml` | Default ModuleSync options such as namespace, branch, Git base URL, and commit message |
| `managed_modules.yml` | Default list of Vox Pupuli container repositories |
| `managed_modules_openvox.yml` | Optional list for OpenVoxProject container repositories |
| `config_defaults.yml` | Global and file-specific defaults consumed by templates |
| `moduleroot/` | ERB templates rendered into target repositories |
| `TEMPLATES.md` | Detailed reference for all templates and supported `.sync.yml` overrides |
| `RELEASE.md` | Release process for this configuration repository |
| `CHANGELOG.md` | Generated changelog for released versions |

## What ModuleSync Manages

The files in `moduleroot/` are rendered into each target repository with the
`.erb` suffix removed. The current templates manage:

- GitHub Actions workflows for container CI, image publishing, security
  scanning, release creation, markdownlint, ShellCheck, and pull request
  labeling.
- GitHub repository metadata such as `CODEOWNERS`, release note categories,
  Dependabot, and labeler configuration.
- Repository-level defaults such as `.gitignore`, markdownlint configuration,
  `Gemfile`, `Rakefile`, `RELEASE.md`, and `renovate.jsonc`.

Some historical files are removed through `config_defaults.yml` even though they
do not have templates in `moduleroot/`, for example `.markdownlint.json` and
`.github/workflows/ci.yaml`.

## Configuration

`config_defaults.yml` provides defaults for templates. Global values live under
`:global:` and apply to multiple workflows, for example:

- `main_branches`
- `matrix_command`
- `matrix_requires_yq`
- `build_runner`
- `build_context`
- `build_file`
- `image_tag`
- `build_platforms`
- `build_args`

File-specific values use the target path as the key, for example
`.github/workflows/ci.yml:` or `.github/workflows/build_container.yml:`.

Target repositories can override defaults with a `.sync.yml` file that uses the
same keys as `config_defaults.yml`. A typical override looks like this:

```yaml
---
:global:
  main_branches:
    - main
  build_args:
    - "OPENVOX_VERSION=${{ matrix.openvox_version }}"

.github/workflows/ci.yml:
  general_ci_scan_dir: "."
  test_commands:
    - "docker run --rm ci/test:${{ github.sha }} --version"

.github/workflows/security_scanning.yml:
  fail_build: true
  severity_cutoff: high
```

See `TEMPLATES.md` for the complete list of templates, variables, defaults, and
example overrides.

## Running ModuleSync

Install the Ruby dependencies first:

```shell
bundle config set --local path vendor/bundle
bundle install
```

ModuleSync needs a GitHub token with permission to push branches and open pull
requests in the target repositories:

```shell
export GITHUB_TOKEN="your_github_pat"
export GITHUB_BASE_URL="https://api.github.com"
```

Run ModuleSync for one repository:

```shell
bundle exec msync update \
  -f container-test \
  --pr \
  --pr-labels modulesync \
  --pr-title "chore: Modulesync ($(git describe --always))" \
  --message "chore: Modulesync update"
```

Run ModuleSync for all repositories in `managed_modules.yml`:

```shell
bundle exec msync update \
  --pr \
  --pr-labels modulesync \
  --pr-title "chore: Modulesync ($(git describe --always))" \
  --message "chore: Modulesync update"
```

Run ModuleSync for the OpenVoxProject repository list:

```shell
bundle exec msync update \
  --namespace OpenVoxProject \
  --managed-modules-conf managed_modules_openvox.yml \
  --pr \
  --pr-labels modulesync \
  --pr-title "chore: Modulesync ($(git describe --always))" \
  --message "chore: Modulesync update"
```

`modulesync.yml` already defines the default namespace, branch, Git base URL,
and commit message. Command-line options override those defaults when needed.

## Special ModuleSync Options

A target repository can opt out of a managed file by marking it as unmanaged in
`.sync.yml`:

```yaml
---
.github/workflows/ci.yml:
  unmanaged: true
```

ModuleSync can also remove a file from target repositories:

```yaml
---
.github/workflows/old-workflow.yml:
  delete: true
```

Use unmanaged files sparingly. Shared behavior should usually be represented in
the templates or in supported `.sync.yml` variables so all container
repositories stay aligned.

## Releasing This Repository

Releases are prepared on a branch named `release-vX.Y.Z`. The branch name is
used by the changelog task to determine the future release version. After the
release pull request is merged, maintainers tag `main` with `vX.Y.Z` and push
the tag.

The exact commands are documented in `RELEASE.md`.
