# MLB Game Center & Combo Tracker v18 — GitHub Pages Edition

This folder can be deployed directly to GitHub Pages. It is a static HTML/CSS/JavaScript app and does not depend on the company server, PHP, or VPN.

## Included features

- Live MLB scores, current batter vs. pitcher, count, and outs
- Batting and pitching box scores, including Total Bases (TB)
- All pitchers used in a game, with SP/RP/LIVE and decisions
- Batter-vs-pitcher history with season comparisons
- Combo tracking and settlement
- Supabase account login and cross-device synchronization
- Chinese and English interfaces
- Browser notifications while the page remains open

## Deploy to GitHub Pages

1. Create a new GitHub repository. A private repository is recommended while testing.
2. Upload every file from this folder to the repository root. `index.html` must remain at the root.
3. In GitHub, open **Settings > Pages**.
4. Under **Build and deployment**, select **Deploy from a branch**.
5. Select the `main` branch and `/ (root)`, then click **Save**.
6. Wait for GitHub to publish the site, then open the HTTPS URL shown on the Pages screen.

Typical project-site URL:

```text
https://YOUR-GITHUB-USERNAME.github.io/YOUR-REPOSITORY-NAME/
```

If the repository is named `YOUR-GITHUB-USERNAME.github.io`, the URL is:

```text
https://YOUR-GITHUB-USERNAME.github.io/
```

## Supabase configuration

The app already contains the same Supabase Project URL and Publishable Key as the company-server version. A Supabase Publishable/Anon Key is intended for browser use; access protection depends on Row Level Security (RLS).

Before public use:

1. Run `SUPABASE_SETUP.sql` once in the Supabase SQL Editor if the database table and RLS policies have not already been created.
2. In **Supabase Dashboard > Authentication > URL Configuration**, set **Site URL** to the GitHub Pages address or your preferred production address.
3. Add the exact GitHub Pages URL to **Redirect URLs**. It is also useful to add a wildcard for pages below that path, for example:

```text
https://YOUR-GITHUB-USERNAME.github.io/YOUR-REPOSITORY-NAME/**
```

4. Keep RLS enabled. Never put a `service_role` key, secret key, database password, Gemini key, or weather API key in this repository.

The company-server and GitHub Pages deployments can use the same Supabase project. Combos and user settings will then synchronize after signing in with the same account.

## Notifications

GitHub Pages is served over HTTPS, so Chrome can request notification permission. Click **Enable alerts / 开启通知** on the site and choose **Allow**.

Current notifications are page-based: the browser tab must remain open for the tracker to continue checking results. Closing the page stops monitoring because this edition does not include a push-notification server or background service worker.

If permission was previously blocked, open the lock/site-controls icon beside the GitHub Pages URL, reset Notifications, reload, and enable alerts again.

## Important behavior

- MLB game data is requested directly from the MLB Stats API.
- Login and synchronized tracker data are handled directly by Supabase.
- No request is sent to the company server.
- The app contains no PHP and does not require a local start script.
- Ask AI and weather are not present in this v18 source package, so no cloud function is required for this release.
- GitHub Pages publishes repository files publicly even when repository visibility and Pages availability vary by GitHub plan. Do not commit private keys or internal-only files.

## Updating both deployments

When a newer version is ready, update the company-server copy and replace the GitHub repository files with the same tested front-end version. Keep `SUPABASE_SETUP.sql` only as setup documentation; do not rerun it unless the schema or policies change.
