# anthropic-custom

Hermes model-provider that speaks the **Anthropic Messages protocol**
against a **custom base_url** — default deployment targets the
**CLR Gateway** at `https://llm.ketlu.com`.

## What it does

Subclasses the native Anthropic provider profile. Forces `base_url` to a
configurable endpoint while preserving the full Anthropic wire format:

- `x-api-key` header + `anthropic-version`
- native `/v1/messages` shape
- byte-identical SDK transport, streaming, prompt caching, tool-use
  encoding

The change vs native Anthropic is **only** `base_url` + credential. Auth
tokens for native Anthropic (OAuth from Claude Code) never leak into
proxy-bound clients, and the proxy never has to pretend to be the
native provider.

## Initial concrete deployment

CLR Gateway at `https://llm.ketlu.com`, keyed by `CLR_GATEWAY_API_KEY`.

In `~/.hermes/config.yaml`:

```yaml
providers:
  anthropic_custom:
    name: Anthropic-custom
    base_url: https://llm.ketlu.com
    api_key: ${CLR_GATEWAY_API_KEY}
    api_mode: anthropic_messages
    anthropic_version: '2023-06-01'
    default_model: claude-sonnet-4-6
    models:
      - claude-opus-4-7
      - claude-sonnet-4-6
      - claude-haiku-4-5

model:
  provider: anthropic_custom    # primary
```

Adding a second proxy is a **config-only** change — point `base_url` at
the other endpoint, supply credential, done.

## Why "anthropic-custom" as a separate provider

Encoding "anthropic-protocol + custom endpoint" as a separate provider
profile keeps the SDK setup path honest. Replaces the previous
`clr-gateway` plugin, which was implicitly tied to a single endpoint —
this generalises it.

## Mounting

External plugin: pulled by `sync-external-plugins.sh` into
`sources/hermes-external-plugins/anthropic-custom/`, bind-mounted at
`/opt/data/plugins/model-providers/anthropic-custom/`.

## License

MIT — see LICENSE.
