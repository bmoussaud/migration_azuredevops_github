# Migration from ADO to GH

## Migrated Items

Migrating a repository data from Azure DevOps to GitHub Enterprise Cloud includes:

* Git source (including commit history)
* Pull requests
* User history for pull requests
* Work item links on pull requests
* Attachments on pull requests
* Branch policies for the repository (user-scoped branch policies and cross-repo branch policies are not included)

## Links
* https://github.com/tjcorr/github-migration
* https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-from-azure-devops-to-github-enterprise-cloud/about-migrations-from-azure-devops-to-github-enterprise-cloud


## Install CLI

Install [GitHub CLI](https://cli.github.com/)

Install ado2gh plugin for GitHub CLI

```
gh extension install github/gh-ado2gh
```

## Create an ADO Token

Create an ADO personal access token (PAT). See [instructions](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows#create-a-pat). Necessary permission vary for different steps:
  - Most operations require basic access: `work item (read)`, `code (read)`, and `identity (read)` scopes.
  - Additional functionality in ADO2GH also requires: `code (read/write/manage)`, `security (manage)`, `build (read)`, and `service connection (read)` 
  - Full control is needed for performing the inventory report and to integrate boards.

```ps1
$env:ADO_PAT="EMl5wguBt0UjUa1qyD1YGHDG3KHVPKN4eX9wwLcaOU0RmTbf7GHuJQQJ99AJACAAAAAAArohAAASAZDOnp9r"
````
![PAT](img/ADO_PAT.png "ADO PAT")

## Create a GitHub Personel Access Token (PAT)

Assign the permission depending of the role (Enterprise owner, Organization owner, Migrator).

Use personal access token __classic__ Only.

The Permissions are `admin:org, repo, workflow`

```ps1
$env:GH_PAT="ghp_V42LfhyBO0BenoitxYs8PVyHoPper42Yd0qn"
````

## Grant Importer Role (GitHub)

Allow `bmoussaud` user to migrate repository to the `moussaudms` GitHub organization.

The targeted user is one from which you have created the PAT.

```ps1
gh ado2gh grant-migrator-role --github-org bmoussaudms --actor bmoussaud --actor-type USER
```

```
[2024-10-15 15:08:18] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 15:08:18] [INFO] GITHUB ORG: bmoussaudms
[2024-10-15 15:08:18] [INFO] ACTOR: bmoussaud
[2024-10-15 15:08:18] [INFO] ACTOR TYPE: USER
[2024-10-15 15:08:18] [INFO] Actor type is valid...
[2024-10-15 15:08:18] [INFO] Granting migrator role ...
[2024-10-15 15:08:19] [INFO] Migrator role successfully set for the USER "bmoussaud"
```

## Generate the global migration script

```ps1
$env:GH_PAT="ghp_V42LfhyBO0BenoitxYs8PVyHoPper42Yd0qn"
$env:ADO_PAT="A2jd4EHC4POKcRAp9BenoitWWLYaMb9G0VYCjAnt4W8YwAR7EFRQsJQQJ99AJACAAAAAAArohAAASAZDODzi8"
gh ado2gh generate-script --ado-org mseng --github-org bmoussaudms --output migrate.ps1
```

This action reads the information in ADO and generate a huge powershell script the include the different command to migrate the repository and the team. No action on the Github Organization.

```
[2024-10-15 15:14:04] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 15:14:04] [INFO] GITHUB ORG: bmoussaudms
[2024-10-15 15:14:04] [INFO] ADO ORG: mseng
[2024-10-15 15:14:04] [INFO] OUTPUT: migrate.ps1
[2024-10-15 15:14:04] [INFO] Generating Script...
[2024-10-15 15:14:11] [INFO] ADO Org: mseng
[2024-10-15 15:14:11] [INFO]   Team Project: Typescript
[2024-10-15 15:14:11] [INFO]     Repo: Typescript
[2024-10-15 15:14:11] [INFO]   Team Project: DevDivDesign
[2024-10-15 15:14:11] [INFO]   Team Project: dotnetcore
[2024-10-15 15:14:11] [INFO]   Team Project: Project Wasabi
[2024-10-15 15:14:11] [INFO]   Team Project: InternalTools
[2024-10-15 15:14:11] [INFO]     Repo: ActionableTelemetry
[2024-10-15 15:14:11] [INFO]     Repo: AutoWatson
[2024-10-15 15:14:11] [INFO]     Repo: DataModelMapper
[2024-10-15 15:14:11] [INFO]     Repo: DataPlatform
.....
```

## Migrate a repository (details)

For each repository the previous command generate a command like this one

```ps1
gh ado2gh migrate-repo --ado-org "mseng" --ado-team-project "Typescript" --ado-repo "Typescript" --github-org "bmoussaudms" --github-repo "Typescript-Typescript" --queue-only --target-repo-visibility private
```

Note: the `--queue-only` means it is an asynchronous job.

The ouput is indicating the action to trigger the migration using an *asynchronous* task

```
[2024-10-15 15:17:25] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 15:17:25] [INFO] ADO ORG: mseng
[2024-10-15 15:17:25] [INFO] ADO TEAM PROJECT: Typescript
[2024-10-15 15:17:25] [INFO] ADO REPO: Typescript
[2024-10-15 15:17:25] [INFO] GITHUB ORG: bmoussaudms
[2024-10-15 15:17:25] [INFO] GITHUB REPO: Typescript-Typescript
[2024-10-15 15:17:25] [INFO] QUEUE ONLY: true
[2024-10-15 15:17:25] [INFO] TARGET REPO VISIBILITY: private
[2024-10-15 15:17:25] [INFO] Migrating Repo...
[2024-10-15 15:17:28] [INFO] A repository migration (ID: RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg) was successfully queued.
```

Then later the script will wait for the end of the migration

```ps1
gh ado2gh wait-for-migration --migration-id $RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg
```

```
[2024-10-15 15:49:52] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 15:49:52] [INFO] MIGRATION ID: RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg
[2024-10-15 15:49:52] [INFO] Waiting for migration (ID: RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg) to finish...
[2024-10-15 15:49:53] [INFO] Waiting for migration of repository Typescript-Typescript to finish...
[2024-10-15 15:49:53] [INFO] Migration RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg succeeded for Typescript-Typescript
[2024-10-15 15:49:53] [WARNING] 82 warnings encountered during this migration
[2024-10-15 15:49:53] [INFO] Migration log available at https://objects-origin.githubusercontent.com/octoshiftmigrationlogs/bmoussaudms/Typescript-Typescript-4bf49ab6-72ad-4b7b-960a-f308db926648-logs.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=***%2F20241015%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241015T133115Z&X-Amz-Expires=432000&X-Amz-Signature=38c6ca4de896e599833af9fc224449d23e11afa4306cff7617ecda8a13984af0&X-Amz-SignedHeaders=host or by running `gh ado2gh download-logs`
``` 

To get the details or logs for a given repository

```ps1
gh ado2gh download-logs --github-org bmoussaudms --github-repo Typescript-Typescript
```

```
[2024-10-15 15:52:43] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 15:52:43] [INFO] GITHUB ORG: bmoussaudms
[2024-10-15 15:52:43] [INFO] GITHUB REPO: Typescript-Typescript
[2024-10-15 15:52:43] [WARNING] Migration logs are only available for 24 hours after a migration finishes!
[2024-10-15 15:52:43] [INFO] Downloading migration logs...
[2024-10-15 15:52:44] [INFO] Downloading log for repository Typescript-Typescript to migration-log-bmoussaudms-Typescript-Typescript-RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg.log...
[2024-10-15 15:52:45] [INFO] Downloaded Typescript-Typescript log to migration-log-bmoussaudms-Typescript-Typescript-RM_kgDaACQ0YmY0OWFiNi03MmFkLTRiN2ItOTYwYS1mMzA4ZGI5MjY2NDg.log.
```

At the end of the migration script , it displays

```
=============== Summary ===============
Total number of successful migrations: $Succeeded
Total number of failed migrations: $Failed
```

## Option --sequential

With this options activated, the generated powershell script includes synchronous calls only.
For each repository, the command looks like:

```ps1
gh ado2gh migrate-repo --ado-org "mseng" --ado-team-project "DevTest" --ado-repo "ArtifactsForTesting" --github-org "bmoussaudms" --github-repo "DevTest-ArtifactsForTesting" --target-repo-visibility private
```
```
[2024-10-17 10:45:05] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-17 10:45:05] [INFO] ADO ORG: mseng
[2024-10-17 10:45:05] [INFO] ADO TEAM PROJECT: DevTest
[2024-10-17 10:45:05] [INFO] ADO REPO: ArtifactsForTesting
[2024-10-17 10:45:05] [INFO] GITHUB ORG: bmoussaudms
[2024-10-17 10:45:05] [INFO] GITHUB REPO: DevTest-ArtifactsForTesting
[2024-10-17 10:45:05] [INFO] TARGET REPO VISIBILITY: private
[2024-10-17 10:45:05] [INFO] Migrating Repo...
[2024-10-17 10:45:07] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: PENDING_VALIDATION. Waiting 10 seconds...
[2024-10-17 10:45:18] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: QUEUED. Waiting 10 seconds...
[2024-10-17 10:45:28] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:45:38] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:45:49] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:45:59] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:46:09] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:46:20] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:46:30] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:46:40] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:46:51] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:01] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:11] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:21] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:32] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:42] [INFO] Migration in progress (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg). State: IN_PROGRESS. Waiting 10 seconds...
[2024-10-17 10:47:52] [INFO] Migration completed (ID: RM_kgDaACRkZjI3OTNjNC00ZTFlLTQ1NTQtODc4Yy04YTI3MWE5OWZkODg)! State: SUCCEEDED
[2024-10-17 10:47:52] [INFO] Migration log available at https://objects-origin.githubusercontent.com/octoshiftmigrationlogs/bmoussaudms/DevTest-ArtifactsForTesting-df2793c4-4e1e-4554-878c-8a271a99fd88-logs.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=***%2F20241017%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241017T084752Z&X-Amz-Expires=432000&X-Amz-Signature=7f75d5c66945e3388639340d4a86da3f44c658ce1e631161f89dc959c183c14a&X-Amz-SignedHeaders=host or by running `gh ado2gh download-logs --github-org bmoussaudms --github-repo DevTest-ArtifactsForTesting`
```

## Option --create_team

With this options activated, the powershell script includes for each migrated repository the creation of 2 teams
* One for Mainteners
* One for Admins

That can be linked to an Identity Provider if `--link-idp-groups xxxx` provided.
Doc: https://docs.github.com/en/enterprise-cloud@latest/organizations/organizing-members-into-teams/synchronizing-a-team-with-an-identity-provider-group

```
gh ado2gh create-team --github-org "bmoussaudms" --team-name "Typescript-Maintainers" 
gh ado2gh create-team --github-org "bmoussaudms" --team-name "Typescript-Admins" 
```

```
[2024-10-15 16:18:54] [INFO] You are running an up-to-date version of the ado2gh CLI [v1.8.0]
[2024-10-15 16:18:54] [INFO] GITHUB ORG: bmoussaudms
[2024-10-15 16:18:54] [INFO] TEAM NAME: Typescript-Maintainers
[2024-10-15 16:18:54] [INFO] Creating GitHub team...
[2024-10-15 16:18:55] [INFO] Successfully created team
[2024-10-15 16:18:55] [INFO] No IdP Group provided, skipping the IdP linking step
```

## Option --lock-ado-repos         

Includes lock-ado-repo scripts that locks repositories before migrating them.

## Option  --rewire-pipelines        

Includes share-service-connection and rewire-pipeline scripts that rewire Azure Pipelines to point to GitHub repos.

## Output 

### Repository Migration

![step 1](img/step_1.png "Step 1")

![step 2](img/step_2.png "Step 2")

![step 3](img/step_4.png "Step 3")

### Pull Request Migration

![Azure Devops PR](img/PR_ADO.png "ADO")
![GitHub PR](img/PR_GH.png "ADO")


### Migration Logs

![Migration Logs](img/Migration_log_2.PNG "Migration Logs")

### Teams

![Teams](img/teams.png "teams")


