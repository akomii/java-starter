# {{PROJECT_NAME}}

<!-- template:start -->

> **This repo was generated from the `java-starter` template.** Run the one-time setup before anything else:
>
> ```sh
> ./scripts/init.sh  # fills in name, slug, author, owner, groupId; strips this banner; self-deletes
> lefthook install   # installs the git hooks (needs the lefthook binary)
> ```
>
> `init.sh` reads your `git remote` to guess the slug, prompts for the rest, rewrites the placeholders (`{{PROJECT_NAME}}`, `{{REPO_SLUG}}`, `{{DESCRIPTION}}`, `{{AUTHOR}}`, `{{OWNER}}`) across the repo, and moves the `org.example.app` package to your groupId. After it runs, review the diff, commit, then do the one-time GitHub steps under [Repository settings](#repository-settings).

<!-- template:end -->

[![CI](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml)
[![CodeQL](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/{{REPO_SLUG}}/badge)](https://securityscorecards.dev/viewer/?uri=github.com/{{REPO_SLUG}})
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/{{REPO_SLUG}})](https://github.com/{{REPO_SLUG}}/releases)

# {{DESCRIPTION}}

## Stack

Maven multi-module (Java 25, `mvnw` pinned), Spring Boot app module, Spotless with the Eclipse formatter, Checkstyle, JUnit 5 with a JaCoCo coverage gate, lefthook hooks, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

## Getting started

Prerequisites: Java 25 on PATH (Maven comes through the wrapper) and the [lefthook](https://lefthook.dev) binary for the git hooks (`apt install lefthook`, `brew install lefthook`, or a release download). The shared git hooks are Python scripts and need `python3` on PATH.

```sh
./mvnw verify                    # full gate: format check, checkstyle, tests + coverage, build
./mvnw spotless:apply            # format
./mvnw test                      # unit tests
./mvnw -pl app spring-boot:run   # run the app
lefthook install                 # git hooks (one-time)
```

The coverage report lands in `app/target/site/jacoco/index.html`.

Commits must follow [Conventional Commits](https://www.conventionalcommits.org), enforced by the commit-msg hook:

```
<type>(<optional scope>): <subject>
```

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, the header stays at most 100 characters. Cut a release by pushing a `vX.Y.Z` tag.

## Repository settings

Generating a repo from a template copies files and branches only, never settings, so these one-time steps stay manual. Do them after `scripts/init.sh`.

| Setting | Where and what to do | Why |
| --- | --- | --- |
| Branch ruleset on `main` | Settings → Rules → Rulesets. New branch ruleset `default` targeting the default branch: restrict deletions, block force pushes, require signed commits, require a pull request (1 approval, code owner review, dismiss stale approvals on push, approval of the most recent push, all conversations resolved, squash only), require the status checks `verify / verify` and `review`, require CodeQL code scanning results (security alerts high or higher, alerts errors) | Keeps unreviewed, unsigned or broken code off `main` |
| Tag ruleset on `v*` | Same screen, new tag ruleset `release-tags` targeting `refs/tags/v*`: restrict creations, updates and deletions, bypass for admins and maintainers only | Releases are tag-driven, without the rule anyone with push access can cut one |
| Workflow permissions | Settings → Actions → General → Read repository contents and packages permissions, do not allow GitHub Actions to create or approve pull requests | Every workflow declares its own `permissions`, the release workflow requests `contents: write` itself |
| Fork PR approval | Settings → Actions → General, require approval for outside collaborators | Keeps untrusted fork code from running workflows before a maintainer has looked |
| Merge settings | Settings → General → Pull Requests. Allow squash merging only, default to the pull request title with a blank message, always suggest updating pull request branches, auto-delete head branches | One conventional subject per PR lands on `main`, which is what git-cliff reads for the release notes |
| Commit sign-off | Settings → General, require contributors to sign off on web-based commits | Commits made in the browser carry the same attribution as local ones |
| Code scanning | Runs from `codeql.yml`, keep "Default setup" off in Settings → Code security | Default setup conflicts with the advanced workflow, private repos need GitHub Advanced Security for the alerts to appear |
| Secret scanning + push protection | Settings → Code security, on by default for public repos | Blocks committed credentials |
| Dependabot alerts | Settings → Code security, enable Dependabot alerts and security updates | `dependabot.yml` only schedules version updates, the vulnerability alerts and auto-fix PRs are separate toggles |
| Labels | Issues → Labels. Delete `duplicate` and `invalid`, add `critical` (`B60205`, "Blocking release or core functionality") and `minor` (`FEF2C0`, "Low impact issue") | Same triage labels in every repo |
| Template toggle | Leave "Template repository" off if this repo was made from the template | Prevents re-templating |
| Housekeeping | Settings → General. Set description and topics, untick Wikis and Projects if unused | Discoverability, fewer places for stale content |

## How the infra works

| Component                | What it does                                                                                                                                            | Why                                                                                |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Maven multi-module       | Parent pom + `app` module, add modules via `<module>` entries                                                                                           | Shared plugin config, room for libs later                                          |
| mvnw wrapper             | Maven version pinned in `.mvn/wrapper/maven-wrapper.properties`                                                                                         | Same build tool locally and in CI                                                  |
| `${revision}` versioning | Single version property in the parent pom                                                                                                               | Release bumps one line instead of every pom                                        |
| Spotless                 | Eclipse formatter (`eclipse-formatter.xml`: 2-space indent, 150 columns), `check` bound to the build, `apply` in the pre-commit hook                                                                        | One formatter of record                                                            |
| Checkstyle               | Correctness and naming rules only, no formatting rules                                                                                                  | Never fights the formatter                                                         |
| JUnit 5 + JaCoCo         | 80% line gate on `verify`, Boot entry point excluded                                                                                                    | Regressions surface locally and in CI                                              |
| lefthook                 | pre-commit formats and lints staged files, runs gitleaks when installed and the shared hooks from aktin-github-scripts (data files, large files, commit size), commit-msg enforces Conventional Commits via the shared hook, pre-push runs `mvnw verify` | Catch problems locally, keep history parseable for the changelog                   |
| CI                       | `ci.yml` calls the reusable `verify.yml` on push and PR, cancels superseded runs, least-privilege token                                                 | Fast, safe feedback                                                                |
| Security scans           | CodeQL, dependency-review, OpenSSF Scorecard, secret scanning, all surfaced in the Security tab                                                         | Catch code and supply-chain issues early                                           |
| Releases                 | Tag `vX.Y.Z` builds the tagged commit with the version taken from the tag (`pom.xml` stays at 0.0.0), git-cliff release notes, CycloneDX SBOM, SLSA attestations, Boot jar attached to the Release  | Reproducible, auditable releases from commit history                               |
| Dependabot               | Weekly grouped maven and github-actions updates                                                                                                         | Stay patched, grouping avoids partial bumps that break paired steps such as CodeQL |
| SHA-pinned actions       | Every action is pinned to a full commit SHA with a version comment                                                                                      | A moved tag cannot swap the code, Dependabot keeps the SHAs current                |

## Project structure

```
app/                   Spring Boot application module (package renamed by init.sh)
.github/workflows/     CI, CodeQL, Scorecard, dependency-review, release
checkstyle.xml         correctness rules (formatting lives in Spotless)
eclipse-formatter.xml  Spotless formatter settings (2-space indent, 150 columns)
SECURITY.md             security policy
scripts/init.sh        one-time template initializer (removed after first run)
```
