# {{PROJECT_NAME}}

<!-- template:start -->

> This repository was created from the `java-starter` template. Do this one-time setup before anything else:
>
> 1. Fill in the placeholders. The script reads your `git remote` to guess the slug and asks for the rest:
>    ```sh
>    ./scripts/init.sh
>    ```
> 2. Install the git hooks. This needs the lefthook binary, see [Getting started](#getting-started):
>    ```sh
>    lefthook install
>    ```
> 3. Review the diff, then commit and push it, for example:
>    ```sh
>    git add -A
>    git commit -m "chore: fill in the template placeholders"
>    git push
>    ```
> 4. Do the GitHub steps under [Repository settings](#repository-settings).
>
> `init.sh` rewrites the placeholders (`{{PROJECT_NAME}}`, `{{REPO_SLUG}}`, `{{DESCRIPTION}}`, `{{AUTHOR}}`, `{{OWNER}}`) across the repo, moves the `org.example.app` package to your groupId, strips this banner and deletes itself.

<!-- template:end -->

[![Verify](https://github.com/{{REPO_SLUG}}/actions/workflows/verify.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/verify.yml)
[![CodeQL](https://github.com/{{REPO_SLUG}}/actions/workflows/scan-codeql.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/scan-codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/{{REPO_SLUG}}/badge)](https://securityscorecards.dev/viewer/?uri=github.com/{{REPO_SLUG}})
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/{{REPO_SLUG}})](https://github.com/{{REPO_SLUG}}/releases)

# {{DESCRIPTION}}

## Stack

Maven multi-module (Java 25, `mvnw` pinned), Spring Boot app module, Spotless with the Eclipse formatter, Checkstyle, JUnit 5 with a JaCoCo coverage gate, lefthook hooks, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

## Getting started

Prerequisites: Java 25 on PATH (Maven comes through the wrapper) and the [lefthook](https://lefthook.dev) binary for the git hooks (`apt install lefthook`, `brew install lefthook`, or a release download). The shared git hooks need `python3`, `gitleaks`, `actionlint`, `hadolint` and `shellcheck` on PATH. A missing tool fails the commit. actionlint, hadolint and shellcheck are only called when a workflow file, Dockerfile or shell script is staged.

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

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, every line of the message stays at most 100 characters.

To cut a release, check out the commit on `main` that you want to release, tag it `vX.Y.Z` and push the tag. Example for version 1.2.3:

```sh
git tag v1.2.3
git push origin v1.2.3
```

The tag starts `release.yml`, see [Workflows](.github/workflows/README.md#workflows).

## Repository settings

GitHub copies only the files of a template, not its settings or rulesets. Do these steps once, in this order, after `scripts/init.sh`.

| Step | Setting | Where and what to do | Why |
| --- | --- | --- | --- |
| 1 | Branch ruleset on `main` | Settings → Rules → Rulesets → New ruleset → New branch ruleset. Name `default`, enforcement status Active, target the default branch. Switch on every rule listed under [Gates on `main`](.github/workflows/README.md#gates-on-main). Type the names of the two required checks exactly as written there | Keeps unreviewed, unsigned or broken code off `main` |
| 2 | Tag ruleset on `v*` | Same screen, New tag ruleset. Name `release-tags`, enforcement status Active, target the pattern `v*`. Switch on restrict creations, restrict updates and restrict deletions | Releases are tag-driven, without the rule anyone with push access can cut one |
| 3 | Bypass list of both rulesets | Open each ruleset and add to its Bypass list. In an organisation: Organization admin on both, the Maintain role on `release-tags` as well. Under a personal account: Repository admin on both | A personal account has no organisation admin. Without Repository admin on the list the only maintainer cannot merge their own pull requests, because 1 approval and a code owner review are required, and nobody can push a release tag |
| 4 | Workflow permissions | Settings → Actions → General → Workflow permissions. Select Read repository contents and packages permissions, leave Allow GitHub Actions to create and approve pull requests unticked | Every workflow declares its own `permissions`, the release workflow requests `contents: write` itself |
| 5 | Fork PR approval | Settings → Actions → General → Approval for running fork pull request workflows from contributors. Select Require approval for all external contributors | Keeps untrusted fork code from running workflows before a maintainer has looked |
| 6 | Pull request creation | Settings → General. In the dropdown under Pull requests select Collaborators only | Only people who were given access to the repository can open pull requests |
| 7 | Merge settings | Settings → General → Pull Requests. Allow squash merging only, default to the pull request title with a blank message, always suggest updating pull request branches, auto-delete head branches | One conventional subject per PR lands on `main`, which is what git-cliff reads for the release notes |
| 8 | Commit sign-off | Settings → General, require contributors to sign off on web-based commits | Commits made in the browser carry the same attribution as local ones |
| 9 | Immutable releases | Settings → General → Releases. Select Enable release immutability | A published release and its files cannot be changed afterwards |
| 10 | Code scanning | Settings → Advanced Security. Keep the CodeQL "Default setup" off, the scan runs from `scan-codeql.yml` | Default setup conflicts with the workflow |
| 11 | Secret scanning + push protection | Settings → Advanced Security, on by default for public repos | Blocks committed credentials |
| 12 | Dependabot alerts | Settings → Advanced Security, enable Dependabot alerts and Dependabot security updates | `dependabot.yml` only schedules version updates, the vulnerability alerts and the pull requests that fix them are separate toggles |
| 13 | Labels | Issues → Labels. Delete `duplicate` and `invalid`, add `critical` (`B60205`, "Blocking release or core functionality") and `minor` (`FEF2C0`, "Low impact issue") | Same triage labels in every repo |
| 14 | Template toggle | Leave "Template repository" off if this repo was made from the template | Prevents re-templating |
| 15 | Housekeeping | Settings → General. Set description and topics, untick Wikis and Projects if unused | Discoverability, fewer places for stale content |

### Optional branch `dev`

Do this only if features should collect on `dev` before a release lands on `main`. `main` then mirrors the released state.

1. Create `dev` from `main` and push it:
   ```sh
   git switch -c dev main
   git push -u origin dev
   ```
2. Create a second branch ruleset as in step 1 of the table. Name `dev`, target the pattern `dev`, the rules listed for `dev` under [Gates on `main`](.github/workflows/README.md#gates-on-main), the same bypass list as `default`.
3. Edit the ruleset `default`. Target the pattern `main` instead of the default branch and change the merge method to the one that section names for a repository with `dev`.
4. Settings → General → Pull Requests. Also allow merge commits.
5. In `.github/dependabot.yml` add `target-branch: dev` to both entries. Example for the first entry:
   ```yaml
   - package-ecosystem: maven
     directory: /
     target-branch: dev
     schedule:
       interval: weekly
   ```

### Private repository

A private repository needs two things that cost extra:

| Needs | For | Without it |
| --- | --- | --- |
| GitHub Code Security | CodeQL and the dependency review | `review / review` fails and CodeQL delivers no result, so the ruleset `default` blocks every pull request |
| GitHub Enterprise Cloud | The attestations in `release.yml` | The job `build` fails and no release is published |

To release without attestations, add the input `attest: false` to the job `build` in `.github/workflows/release.yml`. The build then skips the SBOM and the attestations, and the release holds only the jar:

```yaml
  build:
    needs: verify
    permissions:
      contents: read
      id-token: write
      attestations: write
    uses: aktin/aktin-github-scripts/.github/workflows/build-java.yml@main
    with:
      attest: false
```

## How the infra works

| Component | What it does |
| --- | --- |
| Maven multi-module | Parent pom + `app` module, add modules via `<module>` entries |
| mvnw wrapper | Maven version pinned in `.mvn/wrapper/maven-wrapper.properties` |
| `${revision}` versioning | Single version property in the parent pom. It stays at `0.0.0`, the release workflow sets the version |
| Spotless | Eclipse formatter (`eclipse-formatter.xml`: 2-space indent, 150 columns), `check` bound to the build, `apply` in the pre-commit hook |
| Checkstyle | Correctness and naming rules only, no formatting rules |
| JUnit 5 + JaCoCo | 80% line gate on `verify`, Boot entry point excluded |
| lefthook | All hooks come from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts#git-hooks), see `lefthook.yml`. pre-commit formats and lints staged Java files, scans for secrets with gitleaks, lints staged workflows, Dockerfiles and shell scripts and rejects data files, large files and oversized commits, commit-msg enforces Conventional Commits, pre-push runs `mvnw verify` |
| GitHub workflows and rules | Checks on pull requests, CodeQL, dependency review, Scorecard, releases, Dependabot and the rulesets on `main` and release tags. Details in [.github/workflows/README.md](.github/workflows/README.md) |

## Project structure

```
app/                   Spring Boot application module (package renamed by init.sh)
.github/workflows/     verify, CodeQL, dependency review, Scorecard, release, described in the README there
checkstyle.xml         correctness rules (formatting lives in Spotless)
eclipse-formatter.xml  Spotless formatter settings (2-space indent, 150 columns)
SECURITY.md            security policy
scripts/init.sh        one-time template initializer (removed after first run)
```

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
