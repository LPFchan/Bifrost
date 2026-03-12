# Bifrost GitHub Actions setup

Use this guide in the repo that **runs** the sync workflow. In this repo (Bifrost), the workflow is `.github/workflows/bifrost-sync.yml` and syncs these five repos into one Vertex AI Search data store:

- **Immogen**, **Whimbrel**, **Pipit**, **Quicksilver**, **Bifrost** (all `LPFchan/*` on GitHub).

Your GCP key is stored only as an encrypted **GitHub Secret** in that repo—never in the repo or in logs.

## Why the key is safe

- **GitHub Secrets** (Settings → Secrets and variables → Actions) are encrypted at rest and only injected into the workflow at run time.
- They are not shown in the UI after you save them, not printed in logs, and not in the repo.
- This is the [recommended way](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to use credentials in Actions.

**Do not** commit the key file or paste it into any file in the repo. Only paste it into the secret value in GitHub Settings.

## One-time setup (in the repo that runs the workflow)

1. Open that repo on GitHub → **Settings** → **Secrets and variables** → **Actions**.
2. Add these **Repository secrets**:

  | Name                 | Value                                                 | Notes                                                                              |
  | -------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------- |
  | `GCP_SA_KEY`         | **Entire contents** of your service account JSON file | Open the key file in a text editor, select all, copy, paste into the secret value. |
  | `GCP_PROJECT_ID`     | Your GCP project ID (e.g. `lostplusfound`)            |                                                                                    |
  | `BIFROST_GCS_BUCKET` | Your GCS bucket (e.g. `bifrost-bucket`)               | Optional; defaults to `bifrost-bucket` if unset.                                   |

3. Save each secret. You won’t be able to read them again—only update or delete.

4. **If the synced repos are private:** add a `GH_PAT` secret (see [If your repos are private](#if-your-repos-are-private) below).

5. Push to `main` to trigger a sync, or run the workflow manually: **Actions** → **Bifrost sync** → **Run workflow**.

## Auto-sync when target repos get new commits

To run Bifrost whenever **Immogen**, **Whimbrel**, **Pipit**, or **Quicksilver** (or Bifrost itself) has a push to `main`:

1. **Bifrost** already triggers on `repository_dispatch` (event type `sync`), so no change needed here.

2. **In each target repo** (Immogen, Whimbrel, Pipit, Quicksilver):
   - Create a **Personal Access Token** (GitHub → Settings → Developer settings → Personal access tokens) with `repo` scope (or a fine-grained token with **Actions: Read and write** on `LPFchan/Bifrost`).
   - In that repo, add a secret: **Settings** → **Secrets and variables** → **Actions** → **New repository secret** → name `BIFROST_DISPATCH_TOKEN`, value = the token.
   - Copy the workflow from Bifrost’s `docs/trigger-bifrost-on-push.yml` into that repo as `.github/workflows/trigger-bifrost-sync.yml` (create `.github/workflows/` if needed).

After that, a push to `main` in any of those repos will trigger the Bifrost sync workflow. (Bifrost already runs on its own pushes to `main`.)

## What gets synced

The workflow (e.g. `.github/workflows/bifrost-sync.yml` in the orchestrator repo) runs on push or manually and syncs the configured repos into one Vertex AI Search data store (`github-repo-store`). Which repos are synced is defined in that workflow.

### How to set which repos to sync

- **GitHub Actions (orchestrator repo):**  
Edit the workflow file (e.g. `.github/workflows/bifrost-sync.yml`) in the repo that runs the sync. In that file you will:
  1. Clone each repo you want to sync (e.g. with `actions/checkout` or `git clone` steps).
  2. Run Bifrost and pass the **local paths** of those cloned repos as arguments.
  So the list of repos is whatever you clone in the workflow; add or remove clone steps (or a matrix/list of repo URLs) to change which repos get synced.
- **Running Bifrost locally:**  
Pass the repo paths on the command line:
  ```bash
  ./bifrost.sh /path/to/repo1 /path/to/repo2
  ```
  or with `main.py`: `python main.py /path/to/repo1 /path/to/repo2`.

## If your repos are private

If the workflow clones other repos and they are **private**:

1. Create a **Personal Access Token** (GitHub → Settings → Developer settings → Personal access tokens) with `repo` scope.
2. Add it as a secret in the workflow repo: name `GH_PAT`, value = the token.
3. In the workflow, use `secrets.GH_PAT` in the clone URL (e.g. `https://x-access-token:${{ secrets.GH_PAT }}@github.com/ORG/REPO.git`).

## Rotating the key

If you ever rotate the service account key:

1. Update the **`GCP_SA_KEY`** secret in the repo that runs the workflow (Settings → Secrets → Actions → GCP_SA_KEY → Update).
2. No code or file changes needed.

