# Config

One central place for the settings, assets and credentials the project needs.

## What lives here

| Thing | Where | Committed to git? |
|---|---|---|
| Settings that aren't sensitive | `.dadai/config/*.yaml`, `*.json`, `*.toml` | Yes |
| Assets — logos, fonts, design tokens, templates | `.dadai/config/assets/` | Yes |
| The list of secrets the project expects | `.dadai/config/secrets.env.example` | Yes — values are blank |
| **The actual secret values** | `.dadai/config/secrets.env` | **No — gitignored** |

## Secrets

`secrets.env` is listed in `.gitignore`, so it stays on this machine and is never
uploaded to GitHub. It is the only file here that behaves that way.

To set up on a new machine:

```
cp .dadai/config/secrets.env.example .dadai/config/secrets.env
```

then fill in the values.

**When you add a new secret,** add the key — and only the key — to
`secrets.env.example` too, with the value left blank. That file is the record of
*what* the project needs; `secrets.env` is the only place that holds *the actual
values*.

Never paste a real key into `secrets.env.example`, into a doc, or into a chat
message. Once a secret reaches git history, removing it from the current version
does not remove it from the history.
