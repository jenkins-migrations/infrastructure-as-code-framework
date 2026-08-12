# Jenkins to GitHub Actions migration report

## Source analysis

The scripted Jenkins pipeline from `Jenkinsfile` has been archived as
`.github/ci-archive/Jenkinsfile`. It contains no shared-library calls,
credential bindings, parameters, or source-defined triggers.

The `Deploy` stage incorrectly uses declarative `when` and `steps` syntax in
the scripted pipeline. Its apparent intent, deployment only from `master`, is
implemented as a GitHub Actions job-step condition.

## Workflow mapping

| Jenkins behavior | GitHub Actions implementation |
| --- | --- |
| `checkout scm` | `actions/checkout` |
| `bat` commands | `cmd` steps |
| `env.BUILD_NUMBER` | `github.run_number` in `PRODUCT_VERSION` |
| `tool 'MSBuild'` and `tool 'VSTest'` | `MSBuild` and `VSTest` on the self-hosted Windows runner |
| `publishTestResults '**/*.trx'` | `test-results` artifact |
| `archiveArtifacts` | `build-artifacts` artifact |
| `publishHTML` | `build-report` artifact containing `index.html` |
| deploy on `master` | push to `refs/heads/master` condition |

The replacement workflow is `.github/workflows/dotnet-build.yml`. It runs for
pushes to `master` and manual dispatches. Pull request and other branch push
runs are intentionally excluded: this workflow executes repository code on a
self-hosted runner. All
third-party actions are pinned to immutable commits.

## Required runner configuration

The workflow requires a self-hosted Windows runner. It must provide `nuget`,
`MSBuild`, and `VSTest` on `PATH` and have write access to
`C:\Deploy\ProjectName\`. The source pipeline uses sample names
(`SolutionName.sln`, `ProjectName`, and `ProjectName.Tests`); replace these
with the repository's actual solution and project names before enabling runs.

No GitHub secrets or variables are required by the source pipeline. Any
Jenkins SCM trigger configured outside the Jenkinsfile was not available to
migrate. GitHub Actions artifacts retain build outputs, test result files, and
the HTML report, but do not provide Jenkins HTML Publisher's persistent latest
report page or Jenkins artifact fingerprint history.

## Validation

- Verified the workflow's referenced action release commits against their
  upstream tags.
- `actionlint .github/workflows/dotnet-build.yml` passed.
- This repository has no solution or project files, so the .NET commands
  cannot be executed here.
