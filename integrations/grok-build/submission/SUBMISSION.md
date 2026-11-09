# Grok Build marketplace submission (do not open yet)

Do not open a pull request against [xai-org/plugin-marketplace](https://github.com/xai-org/plugin-marketplace) until Shawn approves it (PHO-230). This directory is the prepared catalog entry only. Nothing here submits, publishes, or merges that catalog.

This is the Grok Build plugin marketplace announced at https://x.ai/news/grok-plugin-marketplace. It is not a Grok Bot plugin submission.

## Entry

Add this object to the `plugins` array in `.grok-plugin/marketplace.json`. Do not replace the file. Remote source, subdirectory, full commit:

```json
{
  "name": "phone-harness",
  "description": "Drive a Phone Harness Cloud Android emulator or Cloud iPhone from Grok Build through the hosted Phone Harness MCP server. See the screen, tap, type, and open apps. A personal iPhone and a desk Android are not reachable from this plugin.",
  "category": "development",
  "source": {
    "source": "url",
    "url": "https://github.com/phone-harness/phone-harness.git",
    "sha": "071628f0c645dfd3deaeaa9539e1c889f02494ab",
    "path": "integrations/grok-build"
  },
  "homepage": "https://phone-harness.com",
  "keywords": [
    "phone harness",
    "phone-harness",
    "phone harness cloud",
    "phone-harness mcp"
  ],
  "domains": [
    "phone-harness.com",
    "www.phone-harness.com",
    "mcp.phone-harness.com",
    "api.phone-harness.com"
  ]
}
```

The same object is in [marketplace-entry.json](marketplace-entry.json).

`sha` is `071628f0c645dfd3deaeaa9539e1c889f02494ab`, the commit that contains the plugin files this entry describes (`integrations/grok-build` without this `submission/` directory). Grok Build checks `git rev-parse HEAD` against that pin after clone.

## Re-pin before opening the catalog PR

That commit is on the phone-harness branch that added the plugin. It is the right pin only while that commit stays reachable on the public repo.

If the phone-harness change is squash-merged, this SHA will not be the `main` commit. Before opening the marketplace PR:

1. Merge the phone-harness change to `main`.
2. Set `sha` to the full 40-character lowercase `main` commit that contains `integrations/grok-build` (`git rev-parse` of that commit, not a branch or tag).
3. Confirm `git ls-remote https://github.com/phone-harness/phone-harness.git` can see it, and that `integrations/grok-build/.mcp.json` exists at that commit.

Do not leave this branch SHA in the catalog if the branch is about to be deleted.

## Catalog repo steps (after approval and re-pin)

Schema checked against `xai-org/plugin-marketplace` commit `2d9db5ccc539f42be6e13f84eecc2cd7f76d67cf` (`README.md`, `CONTRIBUTING.md`, `scripts/validate-catalog.py`, `scripts/plugin_catalog.py`).

1. Fork `xai-org/plugin-marketplace` and branch from `main`.
2. Append the entry above to `.grok-plugin/marketplace.json`.
3. Regenerate the index. Do not hand-edit it.

   ```bash
   python3 scripts/generate-plugin-index.py
   python3 scripts/validate-catalog.py
   python3 scripts/generate-plugin-index.py --check
   ```

4. Open the PR using that repo's template.

Index generation fetches the pinned commit from GitHub. The commit must already be public. A local-only SHA fails the fetch on purpose.

`plugin_catalog.extract_plugin` on this plugin directory, using that marketplace commit's script, returns:

```json
{
  "version": "0.1.0",
  "components": {
    "mcpServers": [
      { "name": "phone-harness", "description": "http" }
    ],
    "skills": [
      {
        "name": "phone-harness",
        "description": "Drive a Phone Harness Cloud Android or Cloud iPhone. Cloud phones only, via the hosted MCP."
      }
    ]
  }
}
```

The generated index record for a url source also stores `sha`. Keys and skill order follow that script (component names sorted, description truncated at 120 characters).

## Security checklist for the catalog PR

- No `curl | bash`, remote-code download, or postinstall.
- No reading or exfiltration of secrets, tokens, `.env`, or environment variables. The plugin has no executable code.
- No hooks. One HTTP MCP server, `https://mcp.phone-harness.com/mcp`, no stdio command.
- Network endpoints: `mcp.phone-harness.com` (MCP and OAuth protected-resource metadata), `clerk.phone-harness.com` (sign-in), and, from the hosted server rather than from this plugin, `api.phone-harness.com`.
- Credentials: OAuth scope `phone:drive` for the user's own Phone Harness Cloud account, or a bearer API key the user configures outside this repo. The plugin files contain neither.

## What was validated here, and what was not

- Official `scripts/validate-catalog.py` from the marketplace commit above accepts a url source with this path shape and a 40-character sha, and rejects a branch name.
- Official `plugin_catalog.extract_plugin` returns the component record above.
- `grok` is not installed in the environment that prepared this entry, so `grok plugin validate ./integrations/grok-build` was not run. Do that on a machine with the Grok CLI before relying on install UX.

## License

MIT, stated in `.grok-plugin/plugin.json` and the plugin README. The license file is `LICENSE` at the repository root.
