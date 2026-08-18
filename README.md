# MLB Game Center & Combo Tracker v40 — PWA Push Edition

## v40 highlights

- Adds a `Pin Status` button to every Combo. A Pin is tied to the signed-in user and can be removed at any time.
- Adds an installable PWA manifest, app icons, and Service Worker so iPhone/Android Home Screen installations can receive Web Push after the page closes.
- Stores only pinned Combo copies and device PushSubscriptions in two new RLS-protected Supabase tables; the existing `tracker_data` schema, OCR, pitcher model, recommendations, and team model remain intact.
- Adds authenticated `push-subscribe` and cron-protected `push-check` Edge Functions. The checker reuses the tracker’s MLB boxscore settlement logic for player props and game bets.
- Compares stable progress snapshots and sends nothing when there is no change. A notification is sent only when a leg value/status or overall Combo status changes.
- Reuses one notification tag per Combo, deep-links back to the Combo page, and automatically stops a Pin after final `MATCHED` or `FAILED` settlement.
- Keeps VAPID private keys, service-role access, and the cron secret server-side. No secret is included in the web files.

## v39 highlights

- Preserves a valid capital `L.` player initial, fixing `L. Hicks` so it resolves to Liam Hicks instead of being rewritten as `I. Hicks`.
- Repairs OCR symbol insertions between a player initial and period, including `S$. McClanahan` to `S. McClanahan`, without changing dollar amounts, targets, or spread values elsewhere in a row.
- Adds a fixed seven-leg regression for the August 17 Robinhood screenshot and verifies all seven names, markets, targets, and official player matches.

## v38 highlights

- Repairs Tesseract's digit/letter confusion in abbreviated player names: a leading `0. Lopez` is normalized to `O. Lopez` and can match Otto Lopez.
- Limits the `0.` to `O.` repair to the start of a player-name structure, optionally after a Robinhood `Yes/No` marker, so game scores, spread lines, and prop targets are untouched.
- Extracts a player target from the number adjacent to the detected market before considering any other number on the row. For example, `0. Lopez 3+ Strikeouts` now keeps target `3` rather than misreading the initial as zero.
- Adds fixed parsing and in-game player-pool regression coverage for `PIT vs MIA / 0. Lopez 1+ HRR`, including Otto Lopez's MLB player ID `672640`.

## v37 highlights

- Corrects Robinhood team-spread `No` settlement using the paired matchup: `No · CLE by 1.5` in `DET vs CLE` now becomes `DET +1.5`, matching Robinhood's expanded result.
- Keeps `Yes · CLE by 1.5` as `CLE -1.5` and keeps straight moneyline `No` as the opponent moneyline.
- Refuses to guess an opponent when a spread `No` row has no valid matchup heading, preventing a silently reversed leg.
- Adds a fixed six-leg regression for `CWS ML / DET +1.5 / PIT +1.5 / WSH +2.5 / NYY +1.5 / BOS +1.5`.

## v36 highlights

- Fixes mixed-role pitcher comparisons: a pitcher with only a few starts no longer uses those starts as an unqualified full-season baseline while ignoring relief work.
- Keeps start-only outings for workload and role context, but regresses ERA, WHIP, K/9, BB/9, HR/9, pitch efficiency, and strike rate toward the pitcher's complete pregame season when the starting sample is below 8 starts or 40 innings.
- Separates per-inning quality, expected innings, projected strikeout volume, and starting experience/stability in the matchup score.
- Adds a visible sample-quality note to each pitcher card, including starts, starting innings, and the effective start-role weight for limited samples.
- Changes the matchup verdict to three levels: similar, slight edge, or clearer edge. Limited start-role samples also lower the displayed confidence.
- Preserves strict pregame cutoffs by constructing both the starting-role and full-season baselines only from appearances before the selected game.
- Adds regression coverage for the Tanner Bibee vs. Drew Anderson case: Anderson's three-start 0.82 ERA is pulled toward his complete-season performance, while Bibee receives the workload and established-starter stability edge.

## v35 highlights

- Repairs OCR initial errors such as `|. Herrera` and `l. Herrera` as `I. Herrera` before separator cleanup.
- Accepts OCR bullet substitutions such as `«`, `»`, `‹`, and `›`, including rows such as `Yes « J. Walker 1+ HRR`.
- Replaces blind previous-line borrowing with a guarded two-line-name rule. A previous row is reused only when it is a standalone player name rather than another completed leg, matchup, number, or interface label.
- Carries matchup headings such as `BAL vs MIN` and `PHI vs STL` into every following player leg, then limits abbreviated-name matching to that game's player pool.
- For an otherwise ambiguous SO abbreviation inside a known matchup, uses the announced probable pitchers only as a narrow tie-breaker.
- Adds fixed regression coverage for the original eight-leg screenshot OCR output and verifies all eight player legs retain the correct game context.

## v34 highlights (superseded by v37 for spread `No` rows)

- Earlier behavior retained the named team for `No · TEAM by X`; Robinhood's expanded result later proved that interpretation incorrect, so v37 replaces it with opponent `+X`.
- Keeps straight moneyline behavior separate: a row such as `No · SF` still means the opposing team moneyline.
- The older regression is replaced by v37's expanded-result fixture.

## v33 highlights

- Recognizes Robinhood team-combo rows that pair a matchup such as `BOS vs TOR` with a following choice such as `Yes · BOS`.
- Supports `Yes` and `No` for both straight winner and `by 1.5` team contracts instead of discarding the answer marker before interpreting the leg.
- Earlier v33 behavior converted a negative spread answer to the opponent side; v34 supersedes that interpretation for Robinhood's displayed team-spread contract format.
- Keeps matchup and choice rows paired, preventing unrelated three-letter interface text from being mistaken for a team selection.
- Adds regression coverage for the five-leg `BOS / ATL / LAD / BAL +1.5 / STL +1.5` screenshot format.

## v32 highlights

- Expands every Game Metric Picks hitter card from the top three to the top five confirmed starters.
- Displays `H+R+RBI` explicitly instead of the ambiguous `HRR` abbreviation while preserving the existing tracker market and calculation.
- Adds a two-card starting-pitcher SO forecast above the hitter picks. Each card shows projected SO, a reasonable single-game range, expected innings, opponent-lineup strikeout rate, pitcher role, and the data basis.
- Reuses cached pitcher logs and the existing role-aware workload model, so opening pitchers and bullpen games are not treated like full starters.
- Adjusts the central SO forecast modestly for the confirmed opposing lineup's season strikeout rate and widens the range for opener/swingman uncertainty.
- Keeps the v31 strict pregame team model, weather, schedule, bullpen, probability, and backtest behavior unchanged.

## v31 highlights

- Uses one strict **data cutoff** (the day before first pitch) for team hitting, team pitching, individual hitter pools, and pitcher pools. Every cache key includes the as-of date, and the model page shows the cutoff explicitly.
- Replaces raw Last-10 scoring with a shrunk offense blend: 60% season, 25% last 30 days, and 15% last 14 days. Recent windows shrink toward the season baseline according to games played.
- Estimates bullpen quality from relief-share innings and checks six high-leverage relievers individually for prior-day pitches, two-day load, consecutive appearances, and three-game workload.
- Adds schedule context from actual start-time spacing, venue changes, and eastward time-zone movement. Weekend/weekday is recorded as context only and carries no direct model weight.
- Upgrades MLB weather handling with roof type, retractable-roof uncertainty, temperature, wind direction/speed, and rain/delay risk. Indoor or closed-roof games ignore outdoor weather; uncertain retractable roofs receive a reduced weather effect.
- Removes season head-to-head from the formula. H2H remains visible for context with an explicit zero weight. Historical BvP is also neutralized because the public endpoint cannot be reliably truncated to the selected date.
- Uses a heavier-tail, correlated score simulation and more conservative probability shrinkage. Output is capped at 35%–65%; matchups below 54% show **No clear lean** instead of forcing a confident team pick.
- Separates **Data completeness** from **Model edge** and preserves the v29 opener/workload correction.
- On a strict 45-game regression set from 2026-08-07 through 2026-08-09, forced-direction accuracy improved from v30's 48.9% to 62.2%, margin MAE improved from 3.60 to 3.34, and Brier improved from 0.276 to about 0.250. This is a small validation window, not full-season training.

## v30 highlights

- Adds a lazy-loaded **Team Model** to every Dashboard game card and a full breakdown tab in Game Details. One game is analyzed at a time to avoid an all-day API burst.
- Estimates each team’s scoring rate from season and last-10 offense, confirmed-lineup OPS, the opposing starter’s role-aware Last-5 projection, estimated bullpen ERA, and last-three-game relief workload.
- Adds small, capped adjustments for BvP, home/venue run environment, available weather data, and the current season series. Head-to-head cannot override the current pitcher, lineup, and bullpen layers.
- Runs 10,000 deterministic Poisson score simulations and shows calibrated win probability, a reference score, three common score paths, expected total, a 60% total-run range, first-five scoring, late scoring, pace, and confidence.
- Shrinks simulated win percentages toward 50% to avoid presenting ordinary MLB matchups with false single-game certainty.
- Every factor is explained in the interface. New `?` guides cover Win%, expected runs, total range, R/G, bullpen estimation, and the head-to-head adjustment.
- Historical-date models use only completed games before the selected first pitch and exclude the selected game itself, preventing result leakage during backtesting.
- Validated against the 2026-08-10 BOS @ TOR pregame snapshot, including Sonny Gray vs. Jameson Taillon and Toronto’s 6-3 season-series lead.

## v29 highlights

- Makes pitcher matchup scoring role-aware. A probable pitcher with no meaningful recent start history is labeled **Opener / Bullpen Game** instead of being compared as a full starter.
- Shrinks Last 3/Last 5 rate stats toward the pregame season baseline according to actual innings, preventing a 1–3 inning relief sample from dominating ERA, WHIP, K/9, or HR/9.
- Adds workload-aware **SO/G** and **Projected SO**, so strikeout frequency and likely game strikeout volume are shown separately.
- The pitching-plan edge now blends run prevention, baserunner control, strikeouts, expected innings, and projected strikeout volume. BvP remains sample-weighted and cannot override role/workload on a tiny sample.
- Historical analysis uses only appearances before the selected game, avoiding postgame or later-season leakage into the Season view.
- Adds touch-friendly `?` markers for ERA, WHIP, K/9, BB/9, HR/9, IP/G, P/IP, Strike%, SO/G, Projected SO, pitch-mix terms, and BvP abbreviations. Each guide explains the formula, whether high or low is generally better, and the main caveat.
- Validated against the 2026-08-09 ATH @ BOS pregame case: J.T. Ginn is treated as a full starter, while Erik Miller is treated as an opener after zero prior starts.

## v28 highlights

- Adds a game-level **Starting Pitcher Analysis** tab with side-by-side Last 3, Last 5, and Season views.
- Compares ERA, WHIP, K/9, BB/9, HR/9, IP/start, pitches/inning, and strike rate.
- Loads season pitch usage, velocity, and spin from Baseball Savant and calculates recent pitch usage, velocity, spin, and whiff rate from MLB pitch-by-pitch game feeds.
- Highlights changeup usage and other pitch-mix changes before the selected game.
- Aggregates five-year BvP for each confirmed opposing lineup and down-weights small samples.
- Produces a transparent rules-based starter edge, game-flow lean, confidence label, and pitcher-specific notes.
- Keeps v27 half-inning ending markers, subbed-out Combo settlement, and hitter recommendation tools.

## v27 highlights

- Game Overview now records the last batter of each completed half-inning. The latest completed half is highlighted and up to eight recent half-innings remain visible in a horizontal history strip.
- A tracked batter who has been removed from the active batting order is marked **SUBBED OUT / 已下场**. If the target was not reached, that leg settles immediately as `FAILED` instead of staying `LIVE` until the game ends.
- Combo status now recognizes when the required number of remaining successful legs is mathematically impossible. `ALL`, `ANY`, and `AT_LEAST` modes therefore stop showing `LIVE` at the correct time.
- The Starting Lineup tab adds **Game Metric Picks / 本场指标推荐** after both nine-player lineups are posted.
- Picks rank the top three starters for `H`, `HR`, `TB`, `RBI`, `R`, and `HRR`, blending per-game performance from the last 3, last 5, and season.
- Smart mode uses metric-specific weights: low-frequency HR results receive more season weight, while H/TB/HRR react more to recent form. A small batting-order opportunity adjustment is applied to the final score.
- Custom mode accepts independent Last 3, Last 5, and Season weights. Values are normalized automatically, so they do not need to total exactly 100.
- Recommendation cards show score, weighted per-game rate, team, batting order, and the three period rates. Recent logs are cached and loaded in controlled batches.

## v26 highlights

- Replaced one-off OCR name rules with scored alias matching across every player candidate for the selected date.
- Supports full names, first-initial abbreviations, multiple initials, multi-word surnames, suffixes, accents, apostrophes, and hyphen/space variations. Examples include `E. De La Cruz`, `J.P. Crawford`, `L. O'Hoppe`, `A. Smith-Shawver`, and `J. Chisholm Jr.`.
- Handles OCR-dropped separators such as `CDeLauter`, `AJudge`, `JP Crawford`, and `TJ Friedl` without splitting ordinary full names such as `Chase` or `Yandy`.
- Allows only conservative one-character OCR correction for longer name segments. A match must be both high-confidence and clearly ahead of the next candidate; ties and near-ties remain unresolved for manual review.
- If current Active Rosters do not contain a player, the app now checks the actual player pool for each selected game before falling back to the season player pool. This improves historical dates, roster moves, and game-day call-ups.
- Game player-pool requests are cached and loaded four at a time to avoid unnecessary repeated API bursts.

## v25 highlights

- Fixed full English names being misread as abbreviated names. `Chase DeLauter` is no longer split into `C. hase`, and `Yandy Diaz` is no longer split into `Y. andy`.
- Full-name OCR matching is accent-insensitive, so `Yandy Diaz` and `Yandy Díaz` resolve to the same MLB player.
- Abbreviated forms remain supported only when the first initial has a real dot or space separator: `C. DeLauter`, `C.DeLauter`, `C DeLauter`, and `Y. Diaz`.
- Full-name fallback matching compares the real first name and surname while continuing to ignore suffixes such as `Jr.`, `Sr.`, `II`, `III`, and `IV`.
- Ambiguous matches are still left unresolved for manual review rather than assigned automatically.

## v24 highlights

- Robinhood OCR now removes leading result markers such as `Yes ·`, `No ·`, `Yes -`, and `No:` before parsing player names.
- Rows such as `Yes · D. Susac 1+ HRR` are parsed as `D. Susac` instead of incorrectly treating `Yes` as the player name.
- Standalone `Yes` or `No` lines created by OCR line breaks are ignored safely.
- Team contracts such as `SF Yes` remain supported because only leading result markers are removed.
- If an abbreviated player is missing from a game's Active Roster, OCR performs one season-player fallback lookup and limits candidates to teams playing on the selected date.
- Ambiguous initials and surnames still remain unmatched for manual review rather than being guessed.

## v23 highlights

- The Starting Lineup tab now supports **Last 3**, **Last 5**, and **Season** ranges.
- Two batting metrics are shown side by side. The primary metric controls ranking; the secondary metric provides direct comparison.
- Choose **Stat ranking** for a descending leaderboard or **Batting order** to preserve the official 1–9 lineup.
- Every metric badge shows the player's team rank, with the top three highlighted in gold, silver, and bronze.
- Recent ranges use each player's actual MLB hitting game log and exclude the selected game itself. Doubleheader games are distinguished by game ID.
- Recent-stat requests are loaded in small batches and cached in the browser session to avoid unnecessary repeated API calls.
- The comparison controls and two-metric player rows are optimized for narrow mobile screens.

## v22 highlights

- Fixed Robinhood OCR matching for abbreviated player names with suffixes, including `J. Chisholm Jr.` → `Jazz Chisholm Jr.`.
- Also accepts variants such as `J Chisholm Jr`, `J.Chisholm Jr.`, and suffixes `Sr.`, `II`, `III`, and `IV`.

## v21 highlights

- **Check Now follows the date on screen.** If the previous day is selected, one manual check refreshes only that date and can settle stale `LIVE` games as final.
- **Automatic checks remain restricted to today** and skip already settled Combos, so historical dates are never scanned in the background.
- Final-state normalization also recognizes MLB results reported as `Game Over`, `Completed Early`, or `Forfeit`.
- On phones, tapping the player field opens a full-screen, touch-friendly player picker with both-team filters and search by name, number, or position.
- Desktop player autocomplete and the existing team batch-add workflow are unchanged.

## v20 highlights

- Robinhood OCR recognizes abbreviated player names such as `B. Harper`, `B.Harper`, `A. Bohm`, and `A.Bohm` across every team playing that day.
- OCR treats `HR` as Home Runs, `TB` as Total Bases, and `HRR` as Hits + Runs + RBI. These markets remain separate during parsing and settlement.
- Player rows such as `1+ Hits` and `2+ HRR` can be on the same line as the player or immediately below the abbreviated name.
- Team rows with an explicit result such as `DET by 3.5` create a reviewed `DET -3.5` spread leg; a clear `SF Yes` creates an SF moneyline leg.
- Ambiguous, duplicate, or unmatched OCR results are listed for manual review instead of being silently assigned.

## v19 highlights retained

- Automatic refresh requests data only for today's active Combos; manual Check Now refreshes only the selected date.
- Free local Robinhood screenshot OCR creates a review draft using Tesseract.js. Images are processed in the browser and are not uploaded.
- Hitter details now compare season totals with the previous game and the last 3/5 games, including H, HR, TB, RBI, BB, and SO.
- Mobile layout uses a compact app-style header, sticky controls, a floating Add button, full-screen editors, and touch-friendly horizontal tabs.

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

## Pin Combo Status / background notifications

v40 includes all web and Edge Function source, but background Push does not become active until the one-time Supabase deployment below is completed. Foreground tracking continues to work before that setup.

### 1. Publish the PWA files

Publish these files together at the same HTTPS site root:

- `index.html` and `mlb_combo_tracker.html`
- `manifest.webmanifest`
- `service-worker.js`
- the complete `icons/` folder

Do not rename or omit the Service Worker, manifest, or icon paths. GitHub Pages HTTPS is sufficient.

### 2. Create the tables

Run the complete `SUPABASE_SETUP.sql` once in Supabase SQL Editor. It safely preserves the existing `tracker_data` table and adds:

- `push_subscriptions`
- `pinned_combos`
- RLS policies that restrict browser access to the signed-in user

### 3. Generate and store server secrets

Generate a VAPID pair on a trusted computer:

```bash
npx web-push generate-vapid-keys --json
```

Generate a separate cron secret:

```bash
openssl rand -hex 32
```

Set the four Edge Function secrets. Never put the private key or cron secret in GitHub Pages:

```bash
supabase secrets set \
  VAPID_SUBJECT=mailto:YOUR_EMAIL \
  VAPID_PUBLIC_KEY=YOUR_PUBLIC_KEY \
  VAPID_PRIVATE_KEY=YOUR_PRIVATE_KEY \
  CRON_SECRET=YOUR_LONG_CRON_SECRET
```

### 4. Deploy both Edge Functions

From the folder containing `supabase/config.toml`:

```bash
supabase functions deploy push-subscribe
supabase functions deploy push-check --no-verify-jwt
```

`push-subscribe` validates the signed-in user's JWT. `push-check` deliberately uses `--no-verify-jwt` because the scheduled request is authenticated with the separate `x-cron-secret`; requests without the matching secret receive `401`.

### 5. Schedule the checker

Open `SUPABASE_CRON_SETUP.sql`, replace the project URL and cron-secret placeholders, then run it in Supabase SQL Editor. It invokes `push-check` every three minutes using `pg_cron`, `pg_net`, and Vault.

The job checks every three minutes but does **not** notify every three minutes. It sends Push only when the stored snapshot differs from the newly calculated snapshot.

### 6. Enable on iPhone Chrome

1. Sign in to the tracker.
2. In Chrome's Share menu, choose **Add to Home Screen**.
3. Launch the tracker from its Home Screen icon, not an ordinary browser tab.
4. Open the desired date and tap **Pin Status** on a Combo.
5. Allow notifications when iOS asks.

The Pin button creates the PushSubscription and initial no-notify snapshot. Later progress changes appear on the Lock Screen even when the PWA is closed. iOS can still delay or group notifications according to Focus, battery, and notification settings; this is change-triggered Web Push, not an ActivityKit Live Activity.

If permission was blocked, open iOS Settings > Notifications, select the installed web app, enable notifications, and try Pin again.

## Important behavior

- MLB game data is requested directly from the MLB Stats API.
- Login and synchronized tracker data are handled directly by Supabase.
- No request is sent to the company server.
- The app contains no PHP and does not require a local start script.
- No Ask AI or external weather service is required. The team model uses MLB feed weather/roof context only when MLB provides it, so no cloud function or weather API key is needed.
- GitHub Pages publishes repository files publicly even when repository visibility and Pages availability vary by GitHub plan. Do not commit private keys or internal-only files.

## Updating both deployments

When a newer version is ready, update the company-server copy and replace the GitHub repository files with the same tested front-end version. Keep `SUPABASE_SETUP.sql` only as setup documentation; do not rerun it unless the schema or policies change.
