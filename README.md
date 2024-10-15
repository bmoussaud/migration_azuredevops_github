# Migration from ADO to GH

## Migrated Items

Mmigrating the following repository data from Azure DevOps to GitHub Enterprise Cloud.

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

Install GitHub CLI

Install ado2gh plugin for GitHub CLI

## Create a Personel Access Token

Assign the permission depending of the role (Enterprise owner, Organization owner, Migrator).
Use personal access token (classic) Only.
Permissions: `admin:org, repo, workflow`

```
$env:GH_PAT="ghp_V42LfhyBO0BenoitxYs8PVyHoPper42Yd0qn"
````

## Grant Importer Role

Allow `bmoussaud` user to migrate repository to the `moussaudms` github organization

```
$gh ado2gh grant-migrator-role --github-org bmoussaudms --actor bmoussaud --actor-type USER
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

```
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

```
gh ado2gh migrate-repo --ado-org "mseng" --ado-team-project "Typescript" --ado-repo "Typescript" --github-org "bmoussaudms" --github-repo "Typescript-Typescript" --queue-only --target-repo-visibility private
```

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

```
gh ado2gh wait-for-migration --migration-id $RepoMigrations["mseng/Typescript-Typescript"]
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

```
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

## Option --create_team

With this options activated, the powershell script include for each migrated repository the creation of 2 teams
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
Includes lock-ado-repo scripts that lock repos before migrating them.

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

![Migration Logs](img/Migration_log.png "Migration Logs")

### Teams

![Teams](img/teams.png "teams")


