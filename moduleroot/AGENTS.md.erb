# Agent contribution guide

This file defines repository-specific instructions for AI-assisted work.
It supplements the Vox Pupuli and OpenVox Project
[AI usage policy](https://github.com/OpenVoxProject/.github/blob/main/AI_POLICY.md)
and the Vox Pupuli [contribution guidelines](https://github.com/voxpupuli/.github/blob/master/CONTRIBUTING.md).

## Human accountability

- A human contributor must review, understand, and take responsibility for every proposed change before submission.
- Stop and ask for human direction when a decision changes public behavior,
  compatibility, supported platforms, image contents, tags, release metadata,
  registry publication, or security-sensitive behavior.
- Never add a `Signed-off-by` trailer.
  DCO certification is a legal statement that only the human contributor may add.
- Disclose significant AI assistance.
  When asked to commit AI-assisted work, add an appropriate attribution trailer such as:

  ```text
  Co-authored-by: ChatGPT Codex <codex@openai.com>
  ```

- Do not push branches, open pull requests, publish images or releases, or modify remote state unless the human
  explicitly requests it.

## Communication

- Be concise.
  Report the result, relevant evidence, and any remaining risk.
- Avoid long introductions, repeated summaries, and commentary on routine actions.
- Keep comments proportional to the code.
  Do not add lengthy comments that merely restate a simple command or line of code.
- Explain non-obvious constraints and decisions, especially where container lifecycle, portability, or security is
  affected.

## Container design

- Follow the repository's existing `Containerfile`, build scripts, entrypoints, and naming conventions before
  introducing new patterns.
- Keep build contexts small and deterministic.
  Respect `.containerignore` or `.dockerignore` where present, and do not copy credentials, local artifacts, or
  unrelated repository content into an image.
- Pin base images or dependencies when the repository already does so.
  Do not update digests, supported versions, or image tags without checking the resulting compatibility and release
  impact.
- Minimize image layers and installed packages without sacrificing readability.
  Remove package-manager caches and temporary build dependencies in the same layer in which they are created.
- Prefer multi-stage builds when they keep compilers, package caches, or other build-only content out of the runtime
  image.
- Preserve the intended runtime user.
  Do not switch to root, add privileged execution, weaken file permissions, or expose additional ports without a
  demonstrated need.
- Preserve compatibility with mounted volumes, SELinux labels, non-root users, and arbitrary Kubernetes UIDs.
  Avoid recursive ownership changes unless the image cannot operate correctly without them.
- Use `ARG` only for build-time configuration and `ENV` for intentional runtime configuration.
  Never bake secrets, tokens, or credentials into an image, layer, build argument, log, or test fixture.
- Validate JSON, PEM, and other structured runtime configuration before using it.
  Do not print complete certificates, credentials, or other potentially sensitive values to logs.
- Keep entrypoints signal-safe: use exec form where possible, propagate exit codes, and ensure the main process can
  receive termination signals.
- Consider every architecture configured by the build workflow.
  Do not assume an `amd64`-only binary, package, or download URL in a multi-architecture image.
- When the repository supports multiple distributions, keep their packages, copied files, users, environment
  variables, health checks, ports, entrypoints, and commands functionally equivalent.
- OCI labels, image names, and tags are public interfaces.
  Keep them consistent with the existing publication and release strategy.

## Workflows and automation

- Inspect `.github/workflows/ci.yml`, `.github/workflows/build_container.yml`, and
  `.github/workflows/security_scanning.yml` before changing build or test behavior.
- Files marked as managed by ModuleSync must not be edited in a downstream container repository.
  Change the ModuleSync configuration or template upstream, then synchronize it.
- Keep pull request builds separate from publication.
  Untrusted pull request code must not receive registry credentials or other release secrets.
- Preserve the configured build context, build file, build arguments, matrix, target platforms, image tags,
  attestations, and manifest creation unless the task explicitly changes them.
- Treat `.sync.yml`, matrix scripts, `build_platforms.yaml`, and `build_versions.yaml` as one build contract.
  Validate matrix output after changing any of them.
  Ensure every architecture-specific image tag exactly matches the source tag consumed by its multi-architecture
  manifest.
- Treat scripts under `/container-entrypoint.d` as an ordered runtime interface.
  Preserve their lexical execution order, required shell, executable permissions, argument forwarding, and the final
  `exec` of the main process.
- Use least-privilege workflow permissions.
  Pin actions consistently with the repository and do not expose secrets through command output or artifacts.
- Keep security scanning effective.
  Do not suppress findings, lower severity thresholds, or disable SARIF uploads merely to make CI pass.
- Treat registry publication, multi-architecture manifests, Docker Hub
  descriptions, attestations, and GitHub releases as externally visible changes requiring explicit human direction.

## Testing

Add or update tests for behavior changes and bug fixes.
Documentation-only changes may reuse existing coverage when behavior is unchanged.

- Build the image with the same context, file, arguments, target, and platform assumptions used by CI.
- Exercise the image's actual entrypoint or command, not only a successful build.
- Verify relevant runtime behavior, exposed ports, volumes, health checks, permissions, and shutdown handling.
- Run repository-provided linting for shell, Markdown, YAML, and container files when available.
- For architecture-specific changes, test every affected architecture where practical.
  State clearly when validation was limited to one architecture.
- Do not claim that a build, scan, push, or multi-architecture test passed unless that exact operation ran
  successfully.
- Always run:

  ```shell
  git diff --check
  ```

Report commands that were run and relevant limitations.
Do not publish a test image unless explicitly requested.

## Commits and documentation

- Keep logically independent changes in separate commits, including separating refactoring from behavior changes
  where practical.
- Use concise imperative commit subjects.
  Explain motivation and externally visible behavior in the body only when needed.
- Do not commit generated, temporary, commented-out, credential-bearing, or unrelated files.
- Update `README.md` when public image usage, configuration, supported tags, ports, volumes, or runtime behavior
  changes.
- Update release documentation when the build, tagging, registry, or release process changes.
- Do not bump versions, create tags, or prepare releases unless explicitly requested.
- Before committing, inspect the staged diff and ensure unrelated human changes are not included.

## Licensing and provenance

- Check the repository's `LICENSE` before adding or reusing material.
- Ensure added packages, binaries, and copied files may legally be distributed in the published image.
- Do not copy substantial code, Containerfile stages, or workflow definitions from external sources without checking
  their license and recording required attribution.
- Prefer original implementations based on documented interfaces and existing repository patterns.
