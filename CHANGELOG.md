# Changelog

## 17.1.4 - 2026-11-16

- Updated the NuGet README with installation and upgrade instructions that apply across releases and a link to the public changelog.

## 17.1.3 - 2026-11-09

- Chat requests send all system messages as one leading system message, joined in order with a blank line; the other messages keep their order. Fixes HTTP 400 "System message must be at the beginning" from LiteLLM with Jinja-templated models on Umbraco AI prompts.

## 17.1.2 - 2026-11-05

- License change, no code changes. 17.1.2 and later are under the IM.Umbraco.AI.Local License in `LICENSE.md`: free for private and commercial use, no modification, distribution only as part of a website or application, and commercial use shows "Uses IM.Umbraco.AI.Local by Idea Makers" with a link to https://ideamakers.pl. Versions up to 17.1.1 stay MIT.

## 17.1.1 - 2026-11-05

- Metadata only, no code changes. Issues, discussions and releases moved to the public repository https://github.com/Idea-Makers/im-umbraco-ai-local. `RepositoryUrl` points there, and the package no longer records the private source commit.

## 17.1.0 - 2026-11-04

- The provider connects directly and ignores system and environment proxy settings (`HTTP_PROXY`, `HTTPS_PROXY`). Check that the endpoint is reachable directly before upgrading. No settings or code changes are needed for endpoints that are already reachable.
- Failed requests are no longer retried. The previous release used three retries and could run the same generation up to four times.
- Connecting times out after 10 seconds. A request without a complete response fails after 10 minutes. The previous release allowed 100 seconds per attempt.
- The endpoint must be an absolute `http` or `https` URL. Other values fail with a clear error.
- Structured output fallback also handles error bodies that do not use the OpenAI error format. It remembers a downgrade only after the `json_object` retry succeeds.

## 17.0.0

- Targets the Umbraco AI 17.x release line (`Umbraco.AI.Core >=17.6.1,<18.0.0`). The previous 1.x provider dependency range caused NU1608 with Umbraco AI 17.x.
- Requires the standard `Umbraco.Cms` host, version 17.8 or later within 17.x. Its dependency graph supplies patched MessagePack, Microsoft.OpenApi, ImageSharp and System.Security.Cryptography.Xml/Pkcs. ImageSharp is an indirect CMS dependency. The provider does not use its API or alter its license validation.
- Aligns the OpenAI adapter with Microsoft.Extensions.AI 10.7 and OpenAI 2.11.
- Keeps the provider ID (`local`), connection settings, capability aliases and static asset path (`/App_Plugins/IMUmbracoAILocal`). The upgrade retains endpoint and API key settings and requires no database migrations.
- To roll back to provider 1.0.1, restore the previous compatible Umbraco AI 1.x package set together with the provider.
