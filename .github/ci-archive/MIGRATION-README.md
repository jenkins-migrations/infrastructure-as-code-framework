# Jenkins to GitHub Actions Migration Report

## Summary

Migrated the repository's scripted Jenkins pipeline from `Jenkinsfile` to `.github/workflows/dotnet-build.yml`. The original Jenkins configuration has been archived at `.github/ci-archive/Jenkinsfile`.

## Source Pipeline Analysis

- Source file: `Jenkinsfile`
- Pipeline type: Scripted Jenkins pipeline using `node { ... }`
- Platform: Windows/.NET build pipeline
- Shared libraries: None identified
- Jenkins credentials: None identified
- Jenkins plugins/features used:
  - `checkout scm`
  - `bat`
  - Jenkins tool lookup for `MSBuild` and `VSTest`
  - `publishTestResults`
  - `archiveArtifacts`
  - `publishHTML`
  - Branch-gated deploy step for `master`

## GitHub Actions Workflow

- Workflow file: `.github/workflows/dotnet-build.yml`
- Runner: `windows-latest`
- Triggers:
  - `push`
  - `pull_request`
  - `workflow_dispatch`
- Required permissions: `contents: read`

## Stage Mapping

| Jenkins stage | GitHub Actions equivalent |
| --- | --- |
| `Checkout` | `actions/checkout` pinned to commit SHA `fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09` (`v5`) |
| `Restore` | PowerShell step running `nuget restore SolutionName.sln` |
| `Build` | PowerShell step running `msbuild` with Release configuration and run-number product version |
| `Test` | PowerShell step running `vstest.console.exe`, followed by TRX artifact upload |
| `Archive` | `actions/upload-artifact` pinned to commit SHA `ea165f8d65b6e75b540449e92b4886f43607fa02` (`v4`) for build outputs and HTML report |
| `Deploy` | Separate self-hosted Windows job gated to `refs/heads/master`; downloads the `project-release` artifact and runs the original `xcopy` command |

## Secrets and Variables

No Jenkins credentials or secret bindings were found in the source pipeline. No GitHub Actions secrets are required for this migration.

The Jenkins `BUILD_NUMBER` value is mapped to `${{ github.run_number }}`.

## Manual Follow-up

- Confirm the placeholder paths from the source Jenkinsfile (`SolutionName.sln`, `ProjectName.Tests`, and `ProjectName/bin/Release`) match the actual project files when application source is added.
- Confirm whether the `master` deployment copy path (`C:\Deploy\ProjectName\`) should target a persistent self-hosted Windows runner. On GitHub-hosted runners, that path is ephemeral.

## Validation

- Original Jenkinsfile archived under `.github/ci-archive/`.
- GitHub Actions workflow created under `.github/workflows/`.
- Workflow action references are pinned to commit SHAs.
- `actionlint` validation was run for the migrated workflow.
