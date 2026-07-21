# {{PROJECT_NAME}}

<!-- template:start -->

> **This repo was generated from the `java-starter` template.** Run the one-time
> setup before anything else:
>
> ```sh
> ./scripts/init.sh  # fills in name, slug, author, owner, groupId; strips this banner; self-deletes
> lefthook install   # installs the git hooks (needs the lefthook binary)
> ```
>
> `init.sh` reads your `git remote` to guess the slug, prompts for the rest,
> rewrites the placeholders (`{{PROJECT_NAME}}`, `{{REPO_SLUG}}`,
> `{{DESCRIPTION}}`, `{{AUTHOR}}`, `{{OWNER}}`) across the repo, and moves the
> `org.example.app` package to your groupId. After it runs, review the diff,
> commit, then do the one-time GitHub steps under
> [Repository settings](#repository-settings).

<!-- template:end -->

[![CI](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml)
[![CodeQL](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/{{REPO_SLUG}}/badge)](https://securityscorecards.dev/viewer/?uri=github.com/{{REPO_SLUG}})
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/{{REPO_SLUG}})](https://github.com/{{REPO_SLUG}}/releases)

# {{DESCRIPTION}}

## Stack

Maven multi-module (Java 21, `mvnw` pinned), Spring Boot app module, Spotless
with palantir-java-format, Checkstyle, JUnit 5 with a JaCoCo coverage gate,
lefthook hooks, tag-driven releases via git-cliff, and a hardened GitHub
Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build
attestations).

## Getting started

```sh
./mvnw verify                    # full gate: format check, checkstyle, tests + coverage, build
./mvnw spotless:apply            # format
./mvnw test                      # unit tests
./mvnw -pl app spring-boot:run   # run the app
lefthook install                 # git hooks (one-time, needs the lefthook binary)
```

Commits must follow [Conventional Commits](https://www.conventionalcommits.org)
(enforced on commit). Cut a release by pushing a `vX.Y.Z` tag.

## Repository settings

One-time GitHub settings the code cannot set for you. Do them after `scripts/init.sh`.

| Setting | Where and what to do | Why |
| --- | --- | --- |
| Branch protection on `main` | Settings → Branches. Require a pull request and the `CI` check | Keeps unreviewed or broken code off `main` |
| Actions bypass on `main` | Same screen, add GitHub Actions to the bypass list | The release workflow pushes a `chore(release)` commit back to `main`. Without the bypass that push is rejected and the release stops |
| Workflow permissions | Settings → Actions → General → Read and write | The release workflow creates the GitHub Release and pushes the changelog commit |
| Code scanning | Runs from `codeql.yml`, free on public repos | Private repos need GitHub Advanced Security for the alerts to appear |
| Secret scanning + push protection | Settings → Code security, on by default for public repos | Blocks committed credentials |
| Template toggle | Leave "Template repository" off if this repo was made from the template | Prevents re-templating |

## How the infra works

| Component | What it does | Why |
| --- | --- | --- |
| Maven multi-module | Parent pom + `app` module, add modules via `<module>` entries | Shared plugin config, room for libs later |
| mvnw wrapper | Maven version pinned in `.mvn/wrapper/maven-wrapper.properties` | Same build tool locally and in CI |
| `${revision}` versioning | Single version property in the parent pom | Release bumps one line instead of every pom |
| Spotless | palantir-java-format, `check` bound to the build, `apply` in the pre-commit hook | One formatter of record |
| Checkstyle | Correctness and naming rules only, no formatting rules | Never fights the formatter |
| JUnit 5 + JaCoCo | 80% line gate on `verify`, Boot entry point excluded | Regressions surface locally and in CI |
| lefthook | pre-commit formats staged files and runs gitleaks when installed, commit-msg enforces Conventional Commits via shell check, pre-push runs `mvnw verify` | Catch problems locally, keep history parseable for the changelog |
| CI | `ci.yml` calls the reusable `verify.yml` on push and PR, cancels superseded runs, least-privilege token | Fast, safe feedback |
| Security scans | CodeQL, dependency-review, OpenSSF Scorecard, secret scanning, all surfaced in the Security tab | Catch code and supply-chain issues early |
| Releases | Tag `vX.Y.Z` triggers git-cliff changelog and notes, revision bump committed back, CycloneDX SBOM, SLSA attestations, Boot jar attached to the Release | Reproducible, auditable releases from commit history |
| Dependabot | Weekly grouped maven and github-actions updates | Stay patched, grouping avoids partial bumps that break paired steps such as CodeQL |
| SHA-pinned actions | Every action is pinned to a full commit SHA with a version comment | A moved tag cannot swap the code, Dependabot keeps the SHAs current |

## Project structure

```
app/                Spring Boot application module (package renamed by init.sh)
.github/workflows/  CI, CodeQL, Scorecard, dependency-review, release
checkstyle.xml      correctness rules (formatting lives in Spotless)
scripts/init.sh     one-time template initializer (removed after first run)
```
