# Bifrost GitHub Actions setup

Use this guide in the repo that **runs** the sync workflow (e.g. Quicksilver or another orchestrator that clones Bifrost and runs it). Your GCP key is stored only as an encrypted **GitHub Secret** in that repo—never in the repo or in logs.

## Why the key is safe

- **GitHub Secrets** (Settings → Secrets and variables → Actions) are encrypted at rest and only injected into the workflow at run time.
- They are not shown in the UI after you save them, not printed in logs, and not in the repo.
- This is the [recommended way](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to use credentials in Actions.

**Do not** commit the key file or paste it into any file in the repo. Only paste it into the secret value in GitHub Settings.

## One-time setup (in the repo that runs the workflow)

1. Open that repo on GitHub → **Settings** → **Secrets and variables** → **Actions**.

2. Add these **Repository secrets**:

   | Name | Value | Notes |
   |------|--------|--------|
   | `GCP_SA_KEY` | **Entire contents** of your service account JSON file | Open the key file in a text editor, select all, copy, paste into the secret value. |
   | `GCP_PROJECT_ID` | Your GCP project ID (e.g. `lostplusfound`) | |
   | `BIFROST_GCS_BUCKET` | Your GCS bucket (e.g. `bifrost-bucket`) | Optional; defaults to `bifrost-bucket` if unset. |

3. Save each secret. You won’t be able to read them again—only update or delete.

## What gets synced

The workflow (e.g. `.github/workflows/bifrost-sync.yml` in the orchestrator repo) runs on push or manually and syncs the configured repos into one Vertex AI Search data store (`github-repo-store`). Which repos are synced is defined in that workflow.

## If your repos are private

If the workflow clones other repos and they are **private**:

1. Create a **Personal Access Token** (GitHub → Settings → Developer settings → Personal access tokens) with `repo` scope.
2. Add it as a secret in the workflow repo: name `GH_PAT`, value = the token.
3. In the workflow, use `secrets.GH_PAT` in the clone URL (e.g. `https://x-access-token:${{ secrets.GH_PAT }}@github.com/ORG/REPO.git`).

## Rotating the key

If you ever rotate the service account key:

1. Update the **`GCP_SA_KEY`** secret in the repo that runs the workflow (Settings → Secrets → Actions → GCP_SA_KEY → Update).
2. No code or file changes needed.
