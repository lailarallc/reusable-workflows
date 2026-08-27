# reusable-workflows

Org-level reusable GitHub Actions workflows for the Lailara fleet.

## `pages-deploy.yml` — Cloudflare Pages deploy

Owns the shared deploy tail only: credential check → create-project-if-missing
→ `wrangler pages deploy`. The org-level `CLOUDFLARE_API_TOKEN` and
`CLOUDFLARE_ACCOUNT_ID` secrets flow in via `secrets: inherit`. Callers keep
their own build / data-pipeline / gate steps.

### How to convert a repo

1. In the caller, build the site in its own job and upload the output dir:

   ```yaml
   - name: Upload built site
     uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
     with:
       name: site
       path: build          # the dir wrangler would have deployed (e.g. build, web/build, dist)
   ```

2. Replace the deploy steps with a job that calls this workflow:

   ```yaml
   deploy:
     needs: [build]
     uses: lailarallc/reusable-workflows/.github/workflows/pages-deploy.yml@main
     with:
       project_name: slotmath      # the Cloudflare Pages project name, explicit
       artifact_name: site
     secrets: inherit
   ```

The artifact's *contents* become the site root, so `path: web/build` in step 1
deploys `web/build/*` at the root — no `deploy_subdir` needed.

### Reference conversions
- `slotmath-fair-share` (Slot Math) — SvelteKit, `build/`
- `cinderhaven-promo-incrementality` (Lift Math) — Python pipeline + SvelteKit, `web/build/`

Remaining Cloudflare Pages deployers in the fleet are next-touch conversions
(convert opportunistically when a repo is otherwise being edited).
