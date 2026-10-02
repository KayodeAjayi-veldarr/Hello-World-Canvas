# Hello World Canvas solution reference

Last verified: 2026-10-02. Treat the status details below as a snapshot; recheck GitHub Actions, pull requests, branch state, and Power Platform environment IDs before acting.

## Solution identity

| Item | Value |
| --- | --- |
| Solution display name | `Hello World` |
| Solution unique name | `HelloWorld` |
| Solution source folder | `power-platform-solution` |
| Publisher | `Veldarr Solutions` (`VeldarrSolutions`) |
| Publisher prefix | `vls` |
| Canonical source/target environment | Kay's Power Platform environment |
| Canonical environment ID | `ef77d261-f915-e0fb-95f4-1cbc09edb6ab` |
| Canonical Dataverse URL | `https://org273306d6.crm11.dynamics.com/` |

`main` is intended to track Kay's canonical Power Platform environment unless the user explicitly says otherwise.

## Power Platform links

| Target | URL |
| --- | --- |
| Maker home | <https://make.powerapps.com/environments/ef77d261-f915-e0fb-95f4-1cbc09edb6ab/home> |
| Solution overview | <https://make.powerapps.com/environments/ef77d261-f915-e0fb-95f4-1cbc09edb6ab/solutions/316d143f-3faa-f111-aaac-7ced8d9c5c1b/overview> |
| Canvas Studio | <https://make.powerapps.com/e/ef77d261-f915-e0fb-95f4-1cbc09edb6ab/canvas/?action=edit&form-factor=tablet&name=Hello+World&solution-id=316d143f-3faa-f111-aaac-7ced8d9c5c1b&app-id=%2Fproviders%2FMicrosoft.PowerApps%2Fapps%2F30dd9df8-89f2-4cc7-bf6c-2b7fa8c09c96> |
| Dataverse environment | <https://org273306d6.crm11.dynamics.com/> |

## Canvas app identity

| Item | Value |
| --- | --- |
| App display name | `Hello World` |
| App ID in Kay environment | `30dd9df8-89f2-4cc7-bf6c-2b7fa8c09c96` |
| Solution component name | `vls_helloworld_830cb` |

Before using Canvas Authoring MCP, verify the target environment ID and app ID from the live Studio URL or workflow output. Do not assume the MCP connection is already pointed at the right app, and do not edit Kay's canonical app for feature work unless the user explicitly asks for canonical inspection or changes.

## Azure DevOps references

| Item | Value |
| --- | --- |
| Organisation | `VeldarrProjects` |
| Project | `Hello World Canvas` |
| Project URL | <https://dev.azure.com/VeldarrProjects/Hello%20World%20Canvas> |
| Work item URL pattern | `https://dev.azure.com/VeldarrProjects/Hello%20World%20Canvas/_workitems/edit/<id>` |
| Recent feature references | `AB#487` Hello World Text, `AB#488` Implement a darkmode toggle |

Use `AB#<id>` references in commits and pull request text when changes should link back to Azure Boards.

## GitHub repository and branch strategy

| Item | Value |
| --- | --- |
| Repository | `KayodeAjayi-veldarr/Hello-World-Canvas` |
| Default branch | `main` |
| Solution branch pattern | `change/<id>-<slug>` when there is an Azure Boards work item |
| Hotfix branch pattern | `hotfix/<slug>` only for urgent production fixes |
| Shared workflow/template repo | `KayodeAjayi-veldarr/power-platform-workflows` |

Use GitHub Flow: create a short-lived branch from `main`, work in a dedicated Power Platform feature environment, open a PR to `main`, validate, merge, deploy `main` back to Kay's canonical environment, and clean up the feature branch/environment after merge. Reusable workflow or template improvements belong in `KayodeAjayi-veldarr/power-platform-workflows`; project solution and project-specific config changes belong in this repository.

## GitHub Actions workflow map

Run the numbered project workflows. The `Internal - Reusable ...` workflows are implementation details.

| Workflow | When to use it | Main result |
| --- | --- | --- |
| **1. Manage Power Platform Development Environment** | Start a change by preparing/reusing a maker environment, or clean up a feature environment after merge. | Maker-ready environment, optional baseline export/commit, or deleted feature environment. |
| **2. Commit Solution Changes** | After making and publishing changes in the feature Power Platform environment. | Exported solution source committed to the branch, optional PR, and solution artifacts. |
| **3. Validate Power Platform Pull Request** | Automatically on PRs to `main` that change solution source/config, or manually before review. | Temporary validation import, optional Solution Checker output, and validation artifacts. |
| **4. Build and Deploy Solution** | After a PR is approved and merged, usually from `main`. | Release ZIP built from source, optional committed package, and import to the target environment. |
| **5. Generate Release Notes** | When preparing release notes for stakeholders or deployment records. | Markdown release notes from commits, PRs, and Azure Boards references. |
| **6. Update Azure DevOps Work Item** | At meaningful milestones or when explicitly asked to update one Azure Boards item. | One targeted work item discussion/state/assignment/tag/link update. |

Required Power Platform GitHub Actions secrets are names only: `PP_TENANT_ID`, `PP_APP_ID`, and `PP_CLIENT_SECRET`. `AZURE_DEVOPS_PAT` is optional for release note work item verification and Azure DevOps work item updates. Never print, commit, or document secret values.

## Environment lifecycle

1. Create or reuse a dedicated feature environment with **1. Manage Power Platform Development Environment**.
2. Baseline from Kay's canonical environment when the feature should start from the current canonical solution.
3. Make maker and Canvas app changes only in the feature environment.
4. Save and publish Canvas app changes in Power Apps before exporting.
5. Export and commit changes with **2. Commit Solution Changes**.
6. Validate the PR with **3. Validate Power Platform Pull Request**.
7. Merge the PR to `main` after review and validation.
8. Deploy `main` back to Kay's canonical environment with **4. Build and Deploy Solution**.
9. Delete the feature environment after merge/deployment decisions are complete.

Do not do feature work directly in Kay's canonical environment. Do not delete environments unless you have confirmed the exact feature environment name/ID and deletion confirmation string.

## Current known status snapshot

As of 2026-10-02:

- `main` and `origin/main` were aligned at merge commit `3d590ff48218169e5de60baa6db20d0a79adbee5`.
- No open pull requests were returned by `gh pr list`.
- The only remote branch reported for project feature work was `main`.
- **4. Build and Deploy Solution** run `34113244681` completed successfully from `main` at commit `3d590ff48218169e5de60baa6db20d0a79adbee5`: <https://github.com/KayodeAjayi-veldarr/Hello-World-Canvas/actions/runs/34113244681>.
- Feature environments for `AB#487` and `AB#488` were reported deleted by prior project context.

Recheck these before deploying, cleaning up, or making Power Platform changes.

## Related project documentation

- `.github/power-platform-project.json` contains the project key, solution unique name, default solution folder, branch defaults, region/checker defaults, and Azure DevOps organisation/project.
- `.github/skills/power-platform-feature-workflow/SKILL.md` contains agent workflow guidance for feature implementation, Canvas Authoring MCP rules, accessibility expectations, Azure DevOps ticket rules, and cleanup.
- `README.md` documents the reusable Power Platform GitHub workflow template and detailed workflow inputs.
