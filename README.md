# otty-plugin-openai

The official OpenAI plugin of OTTY. OTTY has no code for any agent by
name: this plugin teaches it Codex. Its agent part `codex` names Codex's
default config folder (`~/.codex`, or the one `CODEX_HOME` names; the
user adds other folders, such as `~/.codex-work`, as profiles), tells
whether each profile is logged in, lists the sessions of each profile,
and reads the tokens of every answer, with its session and project, from
the session files, plain or compressed. The plugin never sees a key or a
token: OTTY hands it handles in their place.

Its provider part `openai` finds the OpenAI accounts itself, in the
Codex profile folders OTTY gives it, and reads nothing outside them: the
ChatGPT login of `auth.json` is a subscription known by its account id
(`chatgpt:<account id>`, the same after Codex renews the tokens) and
named by its email, with its plan; an API key is a key known by its
fingerprint (`key:<fingerprint>`, named such as `key ••••a1f3`), and an
admin key an organization. One account logged in to two folders is one
account found in both. A login Codex sends to a gateway is for the
gateway's host and is no OpenAI account.

OpenAI tells nothing of what a key spent, so an API key gets "Your
spend this month" at API prices: the answers of this calendar month that
OpenAI served (`model_provider`) in the Codex sessions of the folders
that log in with it now, priced at the price OTTY gives during
`numbers`, kind by kind. It counts only this computer's Codex sessions,
never OTTY's own login folder.

It reads the numbers of an account any way it can, only in the folders
it was found in: the limits and credits of a ChatGPT login from
`chatgpt.com` with its token; when the server gives no answer, the
limits Codex knows there, from `codex app-server` started with
`CODEX_HOME` of that folder, which it asks only for the limits and never
for the list of sessions, else from the last record of them in the
session files; and the costs of an organization with an admin key, from
`api.openai.com`. With no way that works it reads nothing, which is no
failure. A token whose claims say it expired is not sent. OTTY's own
login (Add account → "Log in with Codex") is read first with
`codex app-server` in its folder, even past its time, so Codex keeps
that login fresh itself; its session files are not read, and a login
refused there needs "Log in again".

What only the user can do, the parts ask for themselves, in OTTY's
language, which they follow when the user picks another: `codex login`
for a profile nobody is logged in with, `codex login` in the folder of
an account whose ChatGPT login expired or was refused (whether it
expired is the plugin's call, from the token's claims), and a field for
the admin key (`sk-admin-…`) when the bill of an organization needs one
or OpenAI refused the one given.

When Codex is nowhere on the `PATH` of the login shell, the manifest's
`install` gives its install script, `curl -fsSL
https://chatgpt.com/codex/install.sh | sh`, which OTTY runs in a
terminal tab for **Install Codex**, and its repository,
`https://github.com/openai/codex`. The repository names no folder the
script puts the program in, so the part has no program place.

## Build

The plugin takes `otty-plugin-sdk` from the `agents` branch of the OTTY
repository (`Cargo.toml`). The first build pins the SDK in `Cargo.lock`
to the newest commit of that branch; commit the lock, and run `cargo
update -p otty-plugin-sdk` to take a newer SDK. Its tests run natively
over the made-up files of `fixtures/`, which stand for the home folder:

```sh
rustup target add wasm32-wasip2
cargo test
cargo build --release --target wasm32-wasip2
```

The component is `target/wasm32-wasip2/release/otty_plugin_openai.wasm`.

To build against an OTTY checkout instead, such as one next to this
folder, point cargo at its SDK for one command, without changing
`Cargo.toml`:

```sh
cargo --config 'patch."https://github.com/otty-shell/otty".otty-plugin-sdk.path="../otty/crates/plugins/sdk"' test
```

## Install

In OTTY, open Plugins mode, press **Add plugin…**, paste
`https://github.com/otty-shell/otty-plugin-openai`, then **Next** and
**Install**. OTTY installs the newest release it can run and offers the
later ones as updates.

To try a build that is not released, put the component as `openai.wasm`
in a folder with `otty-plugin.toml` and the icons (`codex.svg` and
`openai.svg`), and install that folder with **Add plugin from folder…**.
After a change, copy the new build there and press **Reload** on the
plugin's row.

## Release

Set the same `version` in `otty-plugin.toml` and `Cargo.toml`, commit,
and push the tag `v<version>`:

```sh
git tag v0.2.17
git push origin v0.2.17
```

The workflow `.github/workflows/release.yml` checks that the tag equals
`version` of `otty-plugin.toml`, runs the tests, builds the plugin for
`wasm32-wasip2` and attaches to the GitHub release of the tag the files
OTTY installs: `otty-plugin.toml`, `openai.wasm` and the icons. OTTY
refuses a release whose manifest names another version than its tag.
