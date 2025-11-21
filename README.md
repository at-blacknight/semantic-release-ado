# semantic-release-ado

[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)
[![Build Status](https://klluch.visualstudio.com/semantic-release-ado/_apis/build/status/semantic-release-ado-CI?branchName=master)](https://klluch.visualstudio.com/semantic-release-ado/_build/latest?definitionId=10&branchName=master)

Semantic release plugin for automatic builds on Azure DevOps pipelines.

| Step             | Description |
|------------------|-------------|
| `analyzeCommits` | If configured to do so, stores the current version as an Azure DevOps pipeline variable. Optionally exposes properties from the last release as individual variables. |
| `verifyRelease`  | Stores the next version as an Azure DevOps pipeline variable availabe to downstream steps on the job. Optionally exposes individual properties from the next release as separate variables. |
| `generateNotes`  | Optionally writes release notes to a markdown file with configurable behavior (prepend, append, or overwrite). Runs during the generateNotes stage and works in dry-run mode. |

## Install

```bash
$ npm install -D semantic-release-ado
```

## Usage

The plugin can be configured in the [**semantic-release** configuration file](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/configuration.md#configuration):

`YAML`:
```yaml
plugins:
  - @semantic-release-ado"
```

`JSON`:
```json
{
  "plugins": [
    "semantic-release-ado",
  ]
}
```

The generated version number will be stored on a variable availabe to downstream steps on the job.
By default this variable is named *nextRelease*, but the name can be configured in the plugin options.
The behavior when no new release is available can be configured with *setOnlyOnRelease*.

## Configuration

### Options

#### Core Options

| **Options**      | **Desctiption**                                       |
|------------------|-------------------------------------------------------|
| varName          | Name of the variable that will store the next version. Defaults to *nextRelease*. |
| setOnlyOnRelease | `Bool`. Determines if the variable with the new version will be set only when a new version is available. <br> If set to `false`, the next version variable will store the last released version when no new version is available.<br> Defaults to *true*. |
| isOutput         | `Bool`. Determines whether the version will be set as an output variable, so it is available in later stages.<br> Defaults to *false*. |

#### Release Object Variables (Next Release)
These options enable exposing individual properties from the next release as separate variables in the `verifyRelease` hook.

| **Options**              | **Description**                                       |
|--------------------------|-------------------------------------------------------|
| setReleaseObjectVariables | `Bool`. Enables individual next release property variables. Defaults to *false*. |
| releaseVersionVarName     | Name of the variable that will store the next release version. Defaults to *releaseVersion*. |
| releaseTypeVarName        | Name of the variable that will store the semver type (major, minor, patch). Defaults to *releaseType*. |
| releaseGitHeadVarName     | Name of the variable that will store the git commit SHA. Defaults to *releaseGitHead*. |
| releaseGitTagVarName      | Name of the variable that will store the git tag. Defaults to *releaseGitTag*. |
| releaseNotesVarName       | Name of the variable that will store the release notes. Defaults to *releaseNotes*. |
| releaseChannelVarName     | Name of the variable that will store the distribution channel. Defaults to *releaseChannel*. |

#### Last Release Object Variables
These options enable exposing individual properties from the last release as separate variables in the `analyzeCommits` hook.

| **Options**               | **Description**                                       |
|---------------------------|-------------------------------------------------------|
| setLastReleaseObjectVariables | `Bool`. Enables individual last release property variables. Defaults to *false*. |
| lastReleaseVersionVarName     | Name of the variable that will store the last release version. Defaults to *lastReleaseVersion*. |
| lastReleaseGitHeadVarName     | Name of the variable that will store the last release git commit SHA. Defaults to *lastReleaseGitHead*. |
| lastReleaseGitTagVarName      | Name of the variable that will store the last release git tag. Defaults to *lastReleaseGitTag*. |
| lastReleaseChannelVarName     | Name of the variable that will store the last release distribution channel. Defaults to *lastReleaseChannel*. |

#### Release Notes File Generation

These options enable writing release notes to a markdown file during the `generateNotes` stage.

| **Options**               | **Description**                                       |
|---------------------------|-------------------------------------------------------|
| writeNotesFile            | `Bool`. Enables writing release notes to a markdown file. Defaults to *false*. |
| notesFilePath             | Path where the release notes file will be written. Defaults to *CHANGELOG-latest.md*. |
| notesFileBehaviour        | `String`. How to handle existing file: *prepend* (new notes at top), *append* (new notes at bottom), or *overwrite* (replace entire file). Defaults to *prepend*. |

### Examples

#### Basic Configuration

The following example stores the generated version number in a variable named *version*.

`YAML`:
```yaml
plugins:
  - - "semantic-release-ado"
    - varName: "version"
      setOnlyOnRelease: true
      isOutput: true #defaults to false
```

`JSON`:
```json
{
  "plugins": [
    ["semantic-release-ado", {
      "varName": "version",
      "setOnlyOnRelease": true,
      "isOutput": true
    }]
  ]
}
```

#### Using Release Object Variables

The following example enables individual release properties as separate variables.

`YAML`:
```yaml
plugins:
  - - "semantic-release-ado"
    - varName: "nextRelease"
      setReleaseObjectVariables: true
      setLastReleaseObjectVariables: true
      isOutput: true
```

`JSON`:
```json
{
  "plugins": [
    ["semantic-release-ado", {
      "varName": "nextRelease",
      "setReleaseObjectVariables": true,
      "setLastReleaseObjectVariables": true,
      "isOutput": true
    }]
  ]
}
```

This will set the following variables when a release is published:

- From `verifyRelease`: `releaseVersion`, `releaseType`, `releaseGitHead`, `releaseGitTag`, `releaseNotes`, `releaseChannel`
- From `analyzeCommits`: `lastReleaseVersion`, `lastReleaseGitHead`, `lastReleaseGitTag`, `lastReleaseChannel`

#### Writing Release Notes to File

The following examples show how to enable release notes file generation with different behaviors.

**Prepend mode (latest releases at top):**

`YAML`:
```yaml
plugins:
  - - "@at-blacknight/semantic-release-ado"
    - writeNotesFile: true
      notesFilePath: "CHANGELOG-latest.md"
      notesFileBehaviour: "prepend"
```

`JSON`:
```json
{
  "plugins": [
    ["@at-blacknight/semantic-release-ado", {
      "writeNotesFile": true,
      "notesFilePath": "CHANGELOG-latest.md",
      "notesFileBehaviour": "prepend"
    }]
  ]
}
```

**Append mode (chronological order):**

`YAML`:
```yaml
plugins:
  - - "@at-blacknight/semantic-release-ado"
    - writeNotesFile: true
      notesFilePath: "CHANGELOG.md"
      notesFileBehaviour: "append"
```

`JSON`:
```json
{
  "plugins": [
    ["@at-blacknight/semantic-release-ado", {
      "writeNotesFile": true,
      "notesFilePath": "CHANGELOG.md",
      "notesFileBehaviour": "append"
    }]
  ]
}
```

**Overwrite mode (keep only latest release):**

`YAML`:
```yaml
plugins:
  - - "@at-blacknight/semantic-release-ado"
    - writeNotesFile: true
      notesFilePath: "LATEST-RELEASE.md"
      notesFileBehaviour: "overwrite"
```

`JSON`:
```json
{
  "plugins": [
    ["@at-blacknight/semantic-release-ado", {
      "writeNotesFile": true,
      "notesFilePath": "LATEST-RELEASE.md",
      "notesFileBehaviour": "overwrite"
    }]
  ]
}
```

## Azure DevOps build pipeline YAML example:

Using the variable on the seme job:
```yaml
jobs:
- job: Build
  pool:
    vmImage: 'vs2017-win2016'
  steps:

  - script: >
      npx -p semantic-release
      -p @semantic-release/git
      -p semantic-release-ado
      semantic-release
    env: { GH_TOKEN: $(GitHubToken) }
    displayName: 'Semantic release'

  - script: echo $(nextRelease)
    displayName: 'Show next version'
```
### Using the variable on a later job:
### Configuration:
Below is the configuration for setting `isOutput` to true, which will allow the variable to be referenced from other jobs/stages

`JSON`: 
```json
{
  "plugins": [
    ["semantic-release-ado", {
      "varName": "version",
      "setOnlyOnRelease": true,
      "isOutput": true //sets version as output for later use
    }],
  ]
}
```

### In another job:

```yaml
jobs:
- job: Job1
  pool:
    vmImage: 'vs2017-win2016'

  steps:
  - script: >
      npx -p semantic-release
      -p @semantic-release/git
      -p semantic-release-ado
      semantic-release
    env: { GH_TOKEN: $(GitHubToken) }
    displayName: 'Semantic release'

- job: Job2
  dependsOn: Job1
  pool:
    vmImage: 'vs2017-win2016'
  variables:
    versionNumber: $[ dependencies.Job1.outputs['setOutputVar.versionNumber'] ]

  steps:
  - script: echo $(versionNumber)
    displayName: 'Show next version'
```

### In another stage:

```yaml
stages: 
  - stage: Stage1
    jobs:
    - job: Job1
      pool:
        vmImage: 'vs2017-win2016'

      steps:
      - script: >
          npx -p semantic-release
          -p @semantic-release/git
          -p semantic-release-ado
          semantic-release
        env: { GH_TOKEN: $(GitHubToken) }
        name: semantic-release
        displayName: 'Semantic release'

  - stage: Stage2
    dependsOn: Stage1
    #want to make sure variable is set before we continue to run the stage
    condition: and(succeeded(), ne(dependencies.Stage1.outputs['Job1.semantic-release.version'], ''))
    jobs:
    - job: Job2
      variables:
          versionNumber: $[ stageDependencies.Stage1.Job1.outputs['semantic-release.version'] ]
      pool:
        vmImage: 'vs2017-win2016'
      variables:
        versionNumber:
      steps:
      - script: echo $(versionNumber)
        displayName: 'Show next version'
```

