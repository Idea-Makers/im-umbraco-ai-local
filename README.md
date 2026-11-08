# Umbraco AI Local LLM Provider

[![NuGet](https://img.shields.io/nuget/v/IM.Umbraco.AI.Local.svg)](https://www.nuget.org/packages/IM.Umbraco.AI.Local)

This package adds a *Local LLM* provider to
[Umbraco.AI](https://marketplace.umbraco.com/package/umbraco.ai). It connects the backoffice AI
features (prompts, agents, Copilot and embeddings) to a self-hosted server with an
OpenAI-compatible API, such as [Ollama](https://ollama.com), [LM Studio](https://lmstudio.ai) or
[vLLM](https://github.com/vllm-project/vllm). It needs no cloud account and no API key. Requests
go only to the server you configure.

> **Source availability:** the source repository is private. The package is distributed through
> [NuGet.org](https://www.nuget.org/packages/IM.Umbraco.AI.Local). Report bugs and request features in
> the [issue tracker](https://github.com/Idea-Makers/im-umbraco-ai-local/issues) and ask questions in
> [Discussions](https://github.com/Idea-Makers/im-umbraco-ai-local/discussions).

## Why a separate provider

The cloud providers bundled with Umbraco.AI assume a hosted API. Local servers differ in three
ways, and this provider handles each of them:

1. **No external verification call.** Cloud providers validate API keys against vendor
   endpoints. This provider only needs an endpoint URL. The API key field is optional, because
   Ollama and similar servers ignore authentication.
2. **No model id filtering.** Cloud providers filter the model list to known id shapes, which
   hides local models like `qwen3:14b` or `llama3.3:70b-instruct-q4_K_M`. This provider lists
   every model the server returns from its `/models` endpoint, for both chat and embeddings.
3. **Tool-call schema sanitization.** .NET's tool schema generation emits boolean JSON Schema
   subschemas (`"x": true`). Ollama's OpenAI-compatible endpoint rejects them, which breaks
   Umbraco.AI agents and Copilot. The provider rewrites them into empty object schemas and keeps
   boolean values in `enum`, `const` and `default` positions.

## Requirements

- .NET 10
- Umbraco CMS 17.8 or later within 17.x, using the standard `Umbraco.Cms` host package
- [Umbraco.AI](https://www.nuget.org/packages/Umbraco.AI) 17.6.1 or later within 17.x
- An OpenAI-compatible server reachable from the website (Ollama, LM Studio, vLLM or similar)

## Installation

```bash
dotnet add package IM.Umbraco.AI.Local --version 17.1.0
```

Umbraco discovers the provider automatically. No composer or configuration code is needed.

## Upgrading to 17.1.0

Version 17.1.0 changes how the provider talks to the server. No settings or code changes
are needed, but check these points:

- The provider connects directly and ignores system and environment proxy settings
  (`HTTP_PROXY`, `HTTPS_PROXY`). If your server is only reachable through a proxy, stay on
  17.0.0 and contact Idea Makers.
- Failed requests are no longer retried. 17.0.0 used the SDK default of three retries, which
  could run the same generation up to four times.
- A request without a complete response fails after 10 minutes. 17.0.0 allowed 100 seconds
  per attempt. Connecting to the server times out after 10 seconds.
- The endpoint must be an absolute `http` or `https` URL. Other values fail with a clear error.
- The structured-output fallback also recognises error bodies that are not OpenAI-shaped. It
  remembers a downgrade only after the `json_object` retry succeeds.

## Upgrading to 17.0.0

Version 17.0.0 targets the Umbraco AI 17.x release line and requires
`Umbraco.AI.Core >=17.6.1,<18.0.0`. It replaces the 1.x package's Core dependency
range, which caused NU1608 when used with Umbraco AI 17.x. Umbraco 18.x is not supported.

The OpenAI adapter dependencies are aligned with Microsoft.Extensions.AI 10.7 and
OpenAI 2.11. The minimum CMS version is now 17.8: its dependency graph supplies patched
MessagePack, Microsoft.OpenApi, ImageSharp and System.Security.Cryptography.Xml/Pkcs.
This package requires the standard `Umbraco.Cms` host; custom hosts assembled from a
subset of CMS packages are not supported by this release. ImageSharp remains an indirect
CMS dependency; the provider does not use its API or alter its license validation.

The provider ID (`local`), connection settings, capability aliases and static asset path
(`/App_Plugins/IMUmbracoAILocal`) are unchanged. This release adds no database migrations
or migration states. Existing endpoint and API key settings are retained; no reconfiguration
is required. To roll back to 1.0.1, restore the previous compatible Umbraco AI 1.x package set
together with the provider; do not mix 1.0.1 with AI 17.x.

## Configuration

1. In the Umbraco backoffice, open the **AI** section and add a **Connection**.
2. Pick **Local LLM** as the provider.
3. Set the **Endpoint** to your server's OpenAI-compatible base URL:

   | Server    | Typical endpoint                  |
   |-----------|-----------------------------------|
   | Ollama    | `http://localhost:11434/v1`       |
   | LM Studio | `http://localhost:1234/v1`        |
   | vLLM      | `http://localhost:8000/v1`        |

4. Leave **API Key** empty unless a gateway in front of the server requires one.
5. Create profiles as usual. The model dropdowns list every model your server reports.
   With no model selected, chat defaults to `qwen3:14b` and embeddings to `bge-m3`.

## Capabilities

| Capability | Notes                                                                  |
|------------|------------------------------------------------------------------------|
| Chat       | OpenAI chat completions, including streaming. Agent and Copilot tool calls work against Ollama because of the schema sanitization. |
| Embeddings | OpenAI `/embeddings` endpoint (for example `bge-m3`, `nomic-embed-text`) |

## Network behaviour

- Connects directly to the endpoint. System and environment proxy settings are ignored, so
  prompts do not pass through a proxy.
- Does not follow redirects and does not send or store cookies.
- Does not retry failed requests.
- Connection attempts time out after 10 seconds and requests after 10 minutes. For streaming
  calls the limit applies to each read, so long streamed answers are not cut off.
- SDK request logging and distributed tracing are turned off, so the SDK does not write
  prompts or API keys to logs.
- Accepts only absolute `http` or `https` endpoint URLs. Use `https` when the API key travels
  over a network you do not control.
- A 404 from `/models` produces an error that points at the missing `/v1` suffix.

> **Note on naming:** the NuGet package id uses the `IM.*` vendor prefix (Idea Makers) because
> the bare `Umbraco.*` prefix is reserved on NuGet.org for Umbraco HQ packages.

## License

Free for private and commercial use under the [IM.Umbraco.AI.Local License](https://github.com/Idea-Makers/im-umbraco-ai-local/blob/main/LICENSE.md):
no modification, distribution only as part of your website or application, and a commercial
website or application shows "Uses IM.Umbraco.AI.Local by Idea Makers" with a link to https://ideamakers.pl.
Versions up to 17.1.1 are MIT licensed.

© Idea Makers
