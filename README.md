# Echoes of the Monarch — Presentation Site

The public site for the game — the whole thing, including the insider "State of
the Game" overview and the build Ledger.

> **The gate came off on 2026-08-18.** This site used to publish as ciphertext
> behind an AES-GCM passphrase gate; it now publishes in the clear, and anyone
> with the link reads everything. The gate machinery is still in `site/build.mjs`
> and is one environment variable away (`SITE_PASSWORD="…" node site/build.mjs`)
> — it was kept rather than deleted so re-sealing is a decision, not a rebuild.

> **Do not hand-edit `presentation/index.html`** — it is a generated artifact and
> the build overwrites it. Edit `site/source.html`.

> **It is still `noindex, nofollow`.** Removing the passphrase opened the site to
> anyone with the link; inviting the crawlers is a separate decision and has not
> been made. The tag lives in `site/source.html`'s head, with a note on why it is
> the one change here that is hard to take back.

## Where the source lives

| File | Purpose |
|------|---------|
| `site/source.html` | The editable site — hero, classes, loop, realms, the **State of the Game** overview. Holds a `LEDGER_MARKER` token. |
| `site/build.mjs` | Reads `source.html` + `planning/RELEASE_NOTES.md`, renders the Ledger into the marker, and writes `presentation/index.html`. |
| `presentation/index.html` | **Generated, public.** The only HTML that ships. |
| `presentation/assets/**` | Brand SVGs + class sprites, incl. `assets/sprites/rot/<sprite>/<dir>.png` (8-direction frames for the hover-to-spin portraits). |
| `site/sync-sprites.mjs` | Copies the 8 rotation frames per class (+ the Blightwarden's Wild/Grove forms) from `art/` into `assets/sprites/rot/`. Re-run with `node site/sync-sprites.mjs` whenever the source art changes. |

`site/` is outside `publish_dir`, so the source file itself is not pushed to the
public repo — but its **content now is**, since the built page is plaintext.

## Build it

```bash
node site/build.mjs                                  # OPEN — no gate (the default)
SITE_PASSWORD="Something Else" node site/build.mjs   # SEALED behind that word again
```

Each branch asserts its own failure mode. The sealed build round-trips the
ciphertext, confirms a wrong password is rejected, and aborts if any plaintext
term leaks into the output. The open build asserts the opposite — that the
Ledger and the State of the Game **are** there and no gate markup survived,
because a silently empty page published in the clear is the one way this branch
can go wrong.

## Preview locally

`file://` works for the open build, but serve it over `localhost` anyway so
relative asset paths and fonts behave the way they will in production:

```bash
npx serve presentation
# or
python -m http.server -d presentation 8080
```

Then open the printed `http://localhost:…` URL.

## Publish — auto-synced to a public repo

This (private) repo keeps the working source here. A GitHub Action
(`.github/workflows/deploy-presentation.yml`) pushes the **contents** of this
folder to a separate **public** repo on every change to `main`, which serves
GitHub Pages. You never copy files by hand.

### One-time setup

1. **Create the public site repo** — `echoesofthemonarch-presentation` (must be
   **Public**; empty, no README). If you rename it, update `external_repository`
   in the workflow.
2. **Generate a deploy key** (Git Bash, anywhere):
   ```bash
   ssh-keygen -t ed25519 -C "eotm-site-deploy" -f eotm_deploy -N ""
   ```
3. **Register the key (two places):**
   - Public repo → **Settings → Deploy keys → Add deploy key** → paste
     `eotm_deploy.pub` → **check "Allow write access"**.
   - This repo → **Settings → Secrets and variables → Actions → New repository
     secret** → name `SITE_DEPLOY_KEY` → paste the **private** key (`eotm_deploy`).
4. **Enable Pages** on the public repo → **Settings → Pages** → Source:
   **Deploy from a branch** → `gh-pages` / root.
5. **Custom domain** → in the same Pages settings, set Custom domain to
   `echoes-of-the-monarch.com`. The workflow already writes the matching `CNAME`
   file on each deploy (`cname:` input). At your DNS registrar, point the apex at
   GitHub Pages:
   - `A` → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - (optional `AAAA` → `2606:50c0:8000::153`, `…8001::153`, `…8002::153`, `…8003::153`)
   Then enable **Enforce HTTPS** once the certificate provisions.
6. Delete the local key files: `rm eotm_deploy*`.

It goes live at `https://echoes-of-the-monarch.com/`, open to anyone with the
link. After setup, every push to `main` that touches `presentation/**` republishes
within ~1 minute. Because the Ledger is built from `planning/RELEASE_NOTES.md`,
the flow is: **update the notes → `node site/build.mjs` → commit the regenerated
`presentation/index.html`**. `/echo` does this automatically.

> **Note:** editing `site/source.html` alone changes nothing live — the published
> `presentation/index.html` only updates when you re-run the build and commit it.

## Files

| File | Purpose |
|------|---------|
| `index.html` | **Generated** page (the published artifact). Built by `site/build.mjs`; don't hand-edit. |
| `assets/logo-primary.svg` | Hero wordmark (mirror of `/branding`). |
| `assets/emblem-icon.svg` | Favicon + finale emblem (mirror of `/branding`). |
| `assets/sprites/*.png` | Class portrait sprites shown on the page. |

The editable source is in **`site/`** (see the table up top). If the brand assets
in `/branding` change, re-copy them into `assets/`.
