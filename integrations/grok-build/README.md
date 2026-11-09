# Phone Harness for Grok Build

A [Grok Build](https://x.ai/build) plugin that lets Grok drive a **Phone Harness
Cloud** phone through the hosted MCP server at `https://mcp.phone-harness.com/mcp`.
Grok can list phones, start one, read the screen, tap, type, swipe, and open apps.

This is the Grok Build plugin catalog format (a directory of skills plus
`.mcp.json`). It is not a `phone_exec` tool. Hermes and OpenClaw run model-written
Python on the machine that hosts the agent, and that Python can reach a personal
iPhone or a desk Android. This plugin does not.

## What it can drive

| Phone | Through this plugin |
| --- | --- |
| Cloud Android emulator | Yes. Metered while the session is up (the hosted server describes this as 5 cents a minute). |
| Cloud iPhone | Yes. A flat monthly subscription, not a per-minute charge. A different device from the iPhone in someone's pocket. |
| Personal iPhone (iPhone Mirroring) | No. Use the Phone Harness CLI, or the Hermes or OpenClaw plugin, on a Mac. |
| Android on the desk (adb) | No. Same as above: the CLI or those plugins, on the machine the phone is plugged into. |

The hosted server is cloud-only today. Local iPhone and Android control is a
separate effort and is not part of this plugin. There is no local fallback and
no stdio server in `.mcp.json`.

## Install

Install [Grok Build](https://docs.x.ai/build/overview), then sign in with
`grok login`.

Once this plugin is in the [Grok Build plugin marketplace](https://x.ai/news/grok-plugin-marketplace):

```bash
grok plugin install phone-harness --trust
```

Or open `/marketplace` in Grok Build and install **phone-harness**.

To load this directory from a checkout before it is listed:

```bash
grok --plugin-dir /path/to/phone-harness/integrations/grok-build
```

`--plugin-dir` is the documented way to load a plugin directory. Installing
Grok Build and loading this directory does not publish the plugin.

The first phone call needs a Phone Harness Cloud sign-in. Grok runs its MCP
OAuth flow in the browser. Approve it with the account whose cloud phones Grok
should drive. Do not paste an API key into the chat.

## Authentication

`.mcp.json` names one HTTP server and no headers, so the client uses OAuth
instead of a key baked into the plugin.

Live discovery on the hosted server (no credential sent):

- Protected resource: `https://mcp.phone-harness.com/.well-known/oauth-protected-resource/mcp`
- Resource: `https://mcp.phone-harness.com/mcp`
- Authorization server: `https://clerk.phone-harness.com`
- Scope advertised: `phone:drive`
- Bearer method: `Authorization` header

`phone:drive` lets that account list, start, stop, and drive its own Cloud
phones. It does not grant any other account's phones.

The authorization server publishes authorization-code with PKCE (`S256`),
refresh, and device-code grants, plus a registration endpoint. Which grant
Grok's client picks is up to that client. This plugin does not collect, store,
or forward a token itself.

An API key (`PHONE_HARNESS_API_KEY`, minted from a signed-in CLI with
`phone-harness cloud keys create`) is an optional substitute for OAuth when
nobody can finish a browser sign-in. Put it in the user's own MCP settings for
this server, not in this repository. The key is a bearer for the same cloud
account. Anyone who can use the key can drive that account's cloud phones.

## Network

The plugin ships no code. Grok's MCP client, because of `.mcp.json` and the
OAuth discovery above, talks to:

| Host | Why |
| --- | --- |
| `mcp.phone-harness.com` | The hosted MCP (`/mcp`) and its protected-resource metadata. |
| `clerk.phone-harness.com` | Sign-in. Authorize, token, registration, and revocation endpoints. |
| `https://api.phone-harness.com` | Not called by the plugin. The hosted MCP calls it with the user's bearer to list sessions and run phone operations. |
| `phone-harness.com` | The product and dashboard. The skill sends the user here to take over for a PIN, password, or 2FA prompt. The plugin does not fetch it. |

A Cloud Android's adb session stays inside Phone Harness Cloud. Grok's machine
does not open an adb port. A Cloud iPhone is driven over HTTPS by the hosted
server, not by this computer.

No other hosts are declared. There is no telemetry client in this plugin, and
it does not run the Phone Harness CLI (whose CLI telemetry is a separate
setting). Phone Harness Cloud still records the cloud sessions the account
starts.

## Least privilege

- One remote MCP server. No `command`, no stdio, no args, no hooked shell.
- No hooks, no agents, no slash-command scripts, no LSP, no filesystem tools.
- No `phone_exec`. The model does not get a Python or shell runner from this
  plugin.
- `phone_save_screenshot` writes a file on the machine running a local stdio
  server. Hosted mode does not register it, and this plugin does not run that
  server.
- No API key, refresh token, or OAuth client secret in the plugin files.
- OAuth scope is the one the server advertises (`phone:drive`), not a broader
  grant this plugin invents.
- The skill tells the model to ask before sending, posting, buying, deleting,
  or changing settings, and to stop rather than type a PIN, password, or 2FA
  code.

Cloud Android time costs money. The skill says to ask before starting one and
to stop it when the task is done.

## Skill

`skills/phone-harness/SKILL.md` is generated from the repo-root
[SKILL.md](../../SKILL.md). The root file remains the only copy of the consent
rule, the scroll-versus-swipe distinction, and the personal-phone versus
cloud-phone rule. The generated skill maps those rules onto the hosted MCP
tools and states the limits above. Rebuild it after changing the root guide:

```bash
python3 integrations/grok-build/scripts/build-skill.py
```

The hosted tool names match [phone-harness/mcp](https://github.com/phone-harness/mcp)
`server.py` at `b0572ff90388c8e62e17fe08484a072eb594ccf6`. They were not read
back from a signed-in `tools/list` call. If that server adds or renames a hosted
tool, update `HOSTED_TOOLS` in `scripts/build-skill.py` and rebuild.

`phone_start_device`'s `minutes` defaults to 30 on that server. That is not
the CLI's 15-minute default. `phone_long_press_on_screen_at_coordinates` takes
milliseconds. `phone_swipe_on_screen`'s `distance` is pixels, not the CLI's
fraction.

## Marketplace

Submission is a pull request to [xai-org/plugin-marketplace](https://github.com/xai-org/plugin-marketplace)
that adds one remote entry pointing at this repo, subdirectory
`integrations/grok-build`, pinned to a full commit SHA. The prepared entry and
the steps that are still waiting live in [submission/](submission/). Do not
open that pull request until the maintainer named there approves it. This
directory does not submit itself.

## License

MIT. The license file is [LICENSE](../../LICENSE) at the repository root.
Use of Phone Harness Cloud is governed by Phone Harness's own terms at
https://phone-harness.com.
