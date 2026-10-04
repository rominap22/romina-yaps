# YAPS — Yet Another Personal Site

A [Hugo](https://gohugo.io/) site using the [Hextra](https://github.com/imfing/hextra) theme,
developed inside a Dev Container and deployed to GitHub Pages at
<https://rominap22.github.io/romina-yaps/>.

## Running the site locally

Hugo and Go are **not** installed on the host machine. They live inside the Dev Container
defined in [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json). So the
`hugo` command only works from *inside* the container — running `hugo` in a plain host
terminal gives `command not found`.

The container mounts this repo at `/workspaces/romina-yaps` and publishes the Hugo dev
server on host port **1313** (via `appPort`), so once the server is running you can open:

**<http://localhost:1313/romina-yaps/>**

> The site serves under the `/romina-yaps/` subpath to match the deployed GitHub Pages URL.
> The site root (`http://localhost:1313/`) will 404 — that is expected.

### Prerequisites

- [Docker](https://www.docker.com/) installed and running.
- For the CLI workflow: the Dev Containers CLI —
  `npm install -g @devcontainers/cli`.

---

### Option A — VS Code (Dev Containers extension)

VS Code ships the official **Dev Containers** extension, which gives you a one-click flow.

1. Install the **Dev Containers** extension (`ms-vscode-remote.remote-containers`).
2. Open this folder in VS Code.
3. Click **Reopen in Container** in the popup (or Command Palette → `Cmd+Shift+P` →
   **Dev Containers: Reopen in Container**). VS Code builds the container and reloads the
   window attached to it.
4. Open an integrated terminal (`` Ctrl+` ``). It is now *inside* the container.
5. Start the dev server:
   ```bash
   hugo server --bind 0.0.0.0 --port 1313 --baseURL http://localhost:1313/romina-yaps/ --appendPort=false
   ```
6. Open <http://localhost:1313/romina-yaps/> (VS Code auto-forwards the port).

With this option you do **not** prefix commands with `devcontainer exec` — the terminal is
already inside the container.

---

### Option B — Kiro IDE (or any editor without the Dev Containers extension)

Kiro is built on VS Code but does **not** include the Microsoft Dev Containers extension,
so there is no "Reopen in Container" button. Drive the container from the terminal using
the Dev Containers CLI instead.

1. Open a terminal in Kiro (**Terminal → New Terminal**, or `` Ctrl+` ``). This is a *host*
   terminal.
2. (First time, or after a reboot) build/start the container:
   ```bash
   cd ~/romina-yaps
   devcontainer up --workspace-folder .
   ```
3. Open a shell inside the container:
   ```bash
   devcontainer exec --workspace-folder . bash
   ```
   Your prompt is now inside the container at `/workspaces/romina-yaps`.
4. Start the dev server:
   ```bash
   hugo server --bind 0.0.0.0 --port 1313 --baseURL http://localhost:1313/romina-yaps/ --appendPort=false
   ```
5. Open <http://localhost:1313/romina-yaps/>.

#### One-liner (host terminal, no interactive shell)

```bash
devcontainer exec --workspace-folder ~/romina-yaps \
  hugo server --bind 0.0.0.0 --port 1313 --baseURL http://localhost:1313/romina-yaps/ --appendPort=false
```

---

## Troubleshooting

### `hugo: command not found`
You are running `hugo` on the host. It only exists inside the container — use Option A or
Option B above to get a shell inside the container first.

### `devcontainer: command not found`
The Dev Containers CLI is installed under your nvm Node version, but your current shell
did not load nvm (common when the terminal is `bash` and nvm is configured only in
`~/.zshrc`). Fixes:

- Use zsh (nvm is wired into `~/.zshrc`): run `zsh`, or set zsh as the default terminal
  profile (Command Palette → **Terminal: Select Default Profile** → **zsh**).
- Or add the Node bin dir to PATH for the session:
  ```bash
  export PATH="$HOME/.nvm/versions/node/$(node -v 2>/dev/null || echo current)/bin:$PATH"
  ```
- Or install the CLI globally again: `npm install -g @devcontainers/cli`.

### `localhost:1313` doesn't load / connection refused
- Make sure the container was started with the port published. This repo sets
  `"appPort": ["1313:1313"]` in `devcontainer.json`. If you changed that, recreate the
  container: `devcontainer up --workspace-folder . --remove-existing-container`.
- Remember the site is under `/romina-yaps/` — the bare root 404s by design.

### `port in use` when starting the server
Another Hugo server is already bound to 1313. Stop it, or start on a different port
(e.g. `--port 1314` and open `http://localhost:1314/romina-yaps/`).

### `fatal: detected dubious ownership` during container create
Git inside the container doesn't trust the host-owned mount. Fix once inside the container:
```bash
git config --global --add safe.directory /workspaces/romina-yaps
git submodule update --init --recursive
```

---

## Project layout

| Path | Purpose |
|------|---------|
| `content/` | Markdown content (blog, book-reviews, health-fitness, business). |
| `layouts/` | Project template overrides (e.g. `blog/list.html`, `musings/list.html`). |
| `layouts/_partials/custom/` | Custom partials, including the browse-by-tag cloud. |
| `layouts/_shortcodes/` | Custom shortcodes (e.g. `tag-cloud`). |
| `assets/css/custom.css` | Site styles. |
| `themes/hextra/` | Hextra theme (git submodule). |
| `hugo.toml` | Site configuration. |
| `public/` | Generated output (not committed). |

### Content: single pages vs. page bundles
- **Page bundle** — a folder with `index.md` inside (e.g. `content/blog/my-first-post/index.md`).
  The folder name is the URL slug and you can co-locate images next to the post. The
  markdown file **must** be named `index.md`.
- **Single page** — a standalone `whatever.md` file; the filename is the slug, no bundle.
- `_index.md` (underscore) marks a **section/list** page, not a single post.

### Tags
Tags are a site-wide taxonomy. Hugo generates a page per tag at `/tags/<name>/` and a
browse-all page at `/tags/`. The browse-by-tag cloud appears on the home page, the blog
landing, and the musings landing; individual tags are also rendered on each post card.
