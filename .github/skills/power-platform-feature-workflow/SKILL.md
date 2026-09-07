# Power Platform feature workflow

## When to use this skill

Use this skill whenever an agent implements a feature, fixes a bug, works an Azure DevOps ticket, updates a Power Platform solution, or changes workflow automation for this repository or another repository using the same Power Platform workflow pattern.

This repository uses GitHub Flow for Power Platform solution work. Treat `main` as the deployable source of truth and keep it aligned with Kay's canonical Power Platform environment unless the user explicitly says otherwise.

Canonical environment:

| Setting | Value |
| --- | --- |
| Environment ID | `ef77d261-f915-e0fb-95f4-1cbc09edb6ab` |
| Environment URL | `https://org273306d6.crm11.dynamics.com/` |
| Solution unique name | `HelloWorld` |
| Solution source folder | `power-platform-solution` |
| Base branch | `main` |

Do not make feature changes directly in Kay's canonical environment. Use a feature branch and a dedicated feature Power Platform environment created or reused by the repo workflows.

## Pre-flight checklist

1. Confirm the work item, feature, bug, or user request is understood. If a required work item ID, target environment, app ID, or Studio URL is missing, ask one focused question.
2. Start from latest `main`, not from an unrelated feature branch:
   - Fetch `origin/main`.
   - Create a dedicated branch named `change/<work-item-id>-<slug>` when a work item ID exists.
   - Use `hotfix/<slug>` only for urgent production fixes when the user or incident context calls it a hotfix.
3. Read `.github/power-platform-project.json` before invoking workflows. Respect project defaults unless the user gives explicit overrides.
4. Identify whether the change belongs in this project repository or in the shared workflow/template repository:
   - Project solution, app, and project-specific config changes belong in `KayodeAjayi-veldarr/Hello-World-Canvas`.
   - Reusable workflow/template improvements belong in `KayodeAjayi-veldarr/power-platform-workflows`, then should be propagated to project repositories.
5. Do not expose, print, or commit secrets. Workflows use repository secrets such as `PP_TENANT_ID`, `PP_APP_ID`, `PP_CLIENT_SECRET`, and `AZURE_DEVOPS_PAT`.

## Feature implementation flow

1. Create or reuse a feature development environment with workflow `1. Manage Power Platform Development Environment`.
2. If the feature should start from Kay's current canonical solution, baseline from:
   - `source_environment`: `ef77d261-f915-e0fb-95f4-1cbc09edb6ab` or `https://org273306d6.crm11.dynamics.com/`
   - `source_solution_name`: `HelloWorld`
3. Make Power Platform changes only in the feature environment. For canvas apps, connect Canvas Authoring MCP only to the feature app/environment, not to Kay's canonical environment.
4. Save and publish maker changes in the feature environment before export when required.
5. Export and commit solution changes with workflow `2. Commit Solution Changes`.
6. Open or update a pull request to `main`. Keep it draft until the exported solution changes have been reviewed.
7. Monitor workflow `3. Validate Power Platform Pull Request`. Investigate failures before merging.
8. After review and validation, merge to `main`.
9. After merge, run workflow `4. Build and Deploy Solution` from `main` to Kay's target environment so `main` and Kay's solution remain aligned.
10. Clean up the feature branch and feature environment only after merge and deployment/sync decisions are complete.
11. Update the Azure DevOps work item at notable milestones or completion using workflow `6. Update Azure DevOps Work Item`.

## GitHub Actions workflow usage

### 1. Manage Power Platform Development Environment

Workflow name: `1. Manage Power Platform Development Environment`

Use it to prepare, reuse, or delete a dedicated maker development environment.

Important inputs:

| Input | Guidance |
| --- | --- |
| `workspace_action` | Use `PrepareOrReuse` at feature start. Use `DeleteEnvironment` only after merge/cleanup. |
| `developer_alias` | Use the maker alias, for example `kayode`, or leave blank to use project config. |
| `developer_upn` | Use the maker UPN or leave blank to use project config. |
| `source_environment` | Use Kay's canonical environment ID or URL when baselining from the canonical solution. |
| `source_solution_name` | Use `HelloWorld` unless the user explicitly changes the solution. |
| `solution_folder` | Usually blank, which resolves to `power-platform-solution`. |
| `workspace_environment` | Provide an existing feature environment name, URL, or ID when reusing/deleting. |
| `workspace_environment_type` | `Developer` is normal for individual maker work. |
| `workspace_environment_mode` | `ReuseOrCreate` is normal. `CreateOnly` should be used only when reusing would be unsafe. |
| `commit_baseline_to_branch` | Keep true when the branch needs a committed baseline from the source environment. |
| `deletion_confirmation` | Required for deletion. Use the exact `DELETE:<environment-name>` value shown by the workflow. |

Avoid deleting any environment unless you are certain it is the feature environment for this branch.

### 2. Commit Solution Changes

Workflow name: `2. Commit Solution Changes`

Use it to publish/export from the feature environment, unpack solution artifacts, commit them, push the branch, and optionally create/update a PR.

Important inputs:

| Input | Guidance |
| --- | --- |
| `developer_environment` | Required. Use the feature environment name, URL, or ID, never Kay's canonical environment for feature exports. |
| `developer_alias` | Maker label for generated commit/PR text. |
| `solution_folder` | Usually blank for `power-platform-solution`. |
| `commit_message` | Include useful context and `AB#<id>` when manually linking Azure Boards. |
| `publish_before_export` | Keep true unless a shared dev environment or explicit reason requires otherwise. |
| `azure_work_item_ids` | Add one or more IDs such as `487` or `487,488` to write Azure Boards links. |
| `create_pull_request` | Usually true. |
| `pull_request_title` | Use a clear feature/bug title. |
| `pull_request_draft` | Default true is safest until review is ready. |

### 3. Validate Power Platform Pull Request

Workflow name: `3. Validate Power Platform Pull Request`

It runs automatically for PRs to `main` when `.github/power-platform-project.json` or `power-platform-solution/**` changes. It can also be run manually.

Validation behavior:

- Validates the solution/source in a temporary validation environment.
- Runs Solution Checker when enabled.
- Deletes the validation environment after validation by default.
- Can retain the validation environment on failure for diagnosis when manually dispatched.

Important manual inputs:

| Input | Guidance |
| --- | --- |
| `source_ref` | Branch or commit to validate. Blank uses the selected workflow branch. |
| `solution_folder` | Usually blank for the project default. |
| `validation_environment_type` | `Sandbox` is normal. |
| `retain_validation_environment_on_failure` | Set true only when failure diagnosis needs the temporary environment. |
| `run_solution_checker` | Keep true unless the user explicitly scopes validation differently. |
| `checker_fail_on_errors` | Optional stricter gate for manual runs. |

Do not merge until validation failures are understood and resolved or the user explicitly accepts the risk.

### 4. Build and Deploy Solution

Workflow name: `4. Build and Deploy Solution`

Use it after a PR is merged to deploy from `main` to the target/Kay environment so source control and the Power Platform environment stay aligned.

Important inputs:

| Input | Guidance |
| --- | --- |
| `source_ref` | Usually `main` after merge. |
| `solution_folder` | Usually blank for `power-platform-solution`. |
| `build_environment` | Leave blank unless project config or the user provides one. |
| `target_environment` | Use Kay's canonical target when the project default is blank or the user asks for Kay's environment. |
| `solution_package_type` | `Unmanaged` is this starter's default; use `Managed` only when intended for downstream managed releases. |
| `commit_solution_zip` | Keep true for release traceability unless the user says otherwise. |

### 5. Generate Release Notes

Workflow name: `5. Generate Release Notes`

Use it when the user needs release notes for a PR, merged change, deployment, or work item. Include the PR, commit range, work item IDs, and solution context where available.

### 6. Update Azure DevOps Work Item

Workflow name: `6. Update Azure DevOps Work Item`

Use it for targeted Azure Boards updates through `AZURE_DEVOPS_PAT`. Do not spam routine progress. Good update points include:

- Feature environment prepared when the user asked for status tracking.
- Meaningful commit or PR created.
- Validation passed or failed with a clear diagnosis.
- Merge completed.
- Deployment to Kay's environment completed.
- Work is blocked and needs user action.

Important inputs:

| Input | Guidance |
| --- | --- |
| `work_item_id` | Required Azure Boards ID. |
| `discussion` | Concise milestone/status comment. Blank is allowed when another update is requested. |
| `target_state` | Optional. Validate valid states for the work item type before setting it. |
| `assigned_to` | Optional user display name or email. |
| `tags` | Optional comma or semicolon separated tags to append. |
| `github_pr_url` | Include when a PR is created or materially updated. |
| `github_commit_url` | Include for notable commits. |

Known project state note: Feature work item states discovered earlier were `New`, `Active`, `Resolved`, `Closed`, and `Removed`. `Done` was invalid; completed Feature 487 used `Closed`.

## Canvas Authoring MCP rules

Use Canvas Authoring MCP only after verifying the exact environment ID and app ID from the Power Apps Studio URL or the feature environment. If a co-authoring session is required, ask the user to open the target feature app in Power Apps Studio with co-authoring enabled before connecting.

Before calling Canvas Authoring MCP tools:

1. Confirm the target is the feature app/environment, not Kay's canonical app/environment, unless the user explicitly asks for canonical inspection.
2. Extract and verify the environment ID and app ID from Studio or workflow outputs.
3. Ask a single focused question if the app ID, environment ID, or required open Studio session is missing.
4. Sync the current canvas before editing so local YAML reflects server state.
5. Compile after YAML edits where available.
6. Run accessibility and app checker tools where available.
7. Ask the user to save/publish in Studio when changes require maker-side confirmation.

Never assume the currently connected MCP session points at the correct app. Reconnect or verify before making changes.

## Accessibility requirements for canvas app work

Canvas app changes must be accessible by default:

- Interactive controls need meaningful accessible labels.
- Keyboard users must be able to reach and operate interactive controls in a logical order.
- Focus must be visible and not trapped.
- Screens should use a sensible heading structure and descriptive screen/control names.
- Text and important UI states need sufficient color contrast.
- Error states should be announced or clearly associated with the affected fields.
- Icon-only buttons need accessible names.
- Galleries, forms, filters, dialogs, and navigation should be understandable with a screen reader.

Use compile, accessibility checker, and app checker where available. Treat accessibility defects introduced by the change as blockers unless the user explicitly accepts a documented limitation.

## Azure DevOps ticket rules

- Update tickets only for meaningful milestones, completion, blockers, or explicit status requests.
- Keep comments concise and factual. Include PR or commit links when useful.
- Do not write repetitive "still working" comments.
- Validate valid states before setting `target_state`.
- For this project, do not use `Done` for Feature work items. Use known valid states such as `Active`, `Resolved`, or `Closed` based on the requested workflow and discovered work item type.
- Do not expose the Azure DevOps PAT or any workflow secret.

## Pull request, merge, and cleanup

Before PR:

- Ensure solution changes were exported from the feature environment.
- Ensure generated artifacts are intentional.
- Include work item references, usually `AB#<id>`, in commits or PR text where appropriate.
- Explain manual maker actions still required, if any.

Before merge:

- Confirm PR validation has passed or the user explicitly accepts remaining risk.
- Confirm review feedback is addressed.
- Confirm Azure DevOps state updates are accurate and not premature.

After merge:

1. Delete the remote feature branch when it is no longer needed.
2. Delete the local feature branch if applicable.
3. Delete the feature Power Platform environment with workflow `1. Manage Power Platform Development Environment` using `workspace_action: DeleteEnvironment` and the exact deletion confirmation.
4. Run workflow `4. Build and Deploy Solution` from `main` to deploy/sync Kay's environment.
5. Generate release notes if requested or required.
6. Update the Azure DevOps work item with final PR/deployment links and the correct completed state.

Branch cleanup removes refs, not commit history. Merged branch lines may still appear in the repository graph unless squash or rebase merge was used.

## Common pitfalls

- Do not edit Kay's canonical environment for feature work.
- Do not run workflow `2. Commit Solution Changes` against the canonical environment for a feature branch.
- Do not delete validation environments retained for diagnosis until the diagnosis is complete.
- Do not delete a feature environment before exported changes are committed and the PR is merged or abandoned.
- Do not merge a PR before validation is understood.
- Do not assume `Done` is a valid Azure DevOps state.
- Do not make reusable workflow/template improvements only in one project repo. Make them in `KayodeAjayi-veldarr/power-platform-workflows` when they belong to the shared pattern.
- Local `git push` may use the wrong GitHub account, such as `KayodeAjayi200`, and fail for `KayodeAjayi-veldarr`. If that happens, use the authenticated GitHub CLI/API carefully or ask the user to sign into the Veldarr account. Never expose secrets.
- Deleting branches does not rewrite the visible commit graph. Use squash or rebase merge when a linear graph is required.

## Definition of done

A Power Platform ticket or feature is done only when:

- The work is implemented on a dedicated branch from latest `main`.
- Feature changes were made in a dedicated feature environment.
- Solution artifacts are exported, committed, and pushed.
- A PR to `main` exists and references the relevant work item when applicable.
- PR validation passes or any remaining risk is explicitly accepted by the user.
- The PR is merged.
- The feature branch and feature environment are cleaned up when no longer needed.
- `main` is deployed/synced to Kay's canonical environment when the change should reach Kay's solution.
- Azure DevOps is updated at completion with the correct state, discussion, and links.
- Canvas app changes meet accessibility expectations and compile/checker results are addressed where available.
