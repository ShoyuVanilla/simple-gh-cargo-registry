# simple-gh-cargo-registry

A minimal, GitHub-powered [Cargo](https://doc.rust-lang.org/cargo/) registry.  
No servers, no backend, no auth infrastructure — just static files and GitHub APIs.

---

## 🚀 What is this?

This is a proof of concept (PoC) for a "poor man's" private Cargo registry.

If you've ever had to share internal crates across a private codebase, you've likely resorted to:

```toml
our-secret-crate = { git = "..." }
```

...but that quickly turns into a mess, especially when juggling versions and transitive dependencies.

Of course, the "proper" way is to run a real Cargo registry with a web server, authentication layer, and a formal blessing from your infrastructure and security teams.
But let’s be honest — those folks tend to get nervous when you mention anything homebrewed, and they don’t like it when your registry starts asking for TLS certificates and persistent storage.

So instead, this repo leans on GitHub like an old barn in the woods:

- Crate files are served via raw file downloads
- Index is committed as static content
- Auth (if needed) is handled by GitHub tokens

It doesn’t have luxuries like a publish API, but it works — and more importantly, it keeps the infra guys from calling the cops.

---

## 🔧 Quickstart

### 👉 Try it out

An example registry is available here:  
📦 [ShoyuVanilla/simple-gh-cargo-registry-example](https://github.com/ShoyuVanilla/simple-gh-cargo-registry-example)

---

### 🛠 Repository Setup

1. Create a new repository from this one (template, fork, or clone).
2. Edit the `config.json` at the repo root:

```jsonc
// config.json
{
  "dl": "https://raw.githubusercontent.com/<USER>/<REPO>/crates/crates/{crate}/{version}/download",
                                        // ↑ Replace with your username and repo name

  "auth-required": true // Set to `true` for private registries
}
```

---

## 📦 Using the Registry

Refer to [The Cargo Book](https://doc.rust-lang.org/cargo/reference/registries.html) for in-depth usage.

1. Add the registry to your project’s `.cargo/config.toml`:

```toml
[registries.<your-registry-name>]
index = "https://github.com/<USER>/<REPO>"
```

2. (Optional, for private registries)  
   Create a [GitHub Personal Access Token (PAT)](https://github.com/settings/personal-access-tokens/new) and add it to your Cargo credentials:

```toml
# ~/.cargo/credentials.toml
[registries.<your-registry-name>]
token = "Bearer ghp_..."
```

> ⚠️ Token should be prefixed with `"Bearer "`

If you encounter authentication issues fetching the registry, you may need:

```toml
# .cargo/config.toml
[net]
git-fetch-with-cli = true
```

---

## 📤 Publishing Crates

### Option 1: Manual (via your machine)

Requires [`cargo-index`](https://github.com/ehuss/cargo-index) installed.

Run the helper script:

```bash
./_scripts/publish.sh
```

Or do it manually:

1. Run `cargo package` in your crate project.
2. Checkout your registry repo’s `crates` branch and place the `.crate` file at:
   ```
   crates/<crate-name>/<version>/download
   ```
3. Commit the change.
4. Switch to `main` branch and run:
   ```bash
   cargo index add --index <path> --index-url <github-index-url> --crate <path-to-*.crate-output-above>
   ```
5. You don't need to commit this time, as `cargo-index` does it.
6. Push both branches

---

### Option 2: GitHub Actions

If using Actions:

- [Enable `read/write` permissions on the `GITHUB_TOKEN` for the workflow](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#setting-the-permissions-of-the-github_token-for-your-repository)
- Trigger the `"Publish crate"` action with desired inputs

If publishing a crate from a **private** repo, ensure the workflow is granted permission to read it.
