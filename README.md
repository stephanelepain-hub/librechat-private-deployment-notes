# LibreChat Private Deployment Notes

Public, vendor-neutral notes on running [LibreChat](https://www.librechat.ai/), the open-source AI chat platform, as a self-hosted private interface layer: the decisions you have to make, and the pitfalls that aren't obvious until you hit them.

This is general notes on the public LibreChat project, written from and linking to its own documentation. It names no customer, no specific deployment, and no CPLT delivery details — it's background reading, not a case study.

## What LibreChat is

LibreChat is an open-source platform that gives a team one chat interface in front of several model providers — hosted APIs (Anthropic, OpenAI, Azure and others) or a model you run yourself — instead of one interface per provider. You host it; the data path for each piece (model calls, document extraction, embeddings, logs) depends on how you configure it, not on the fact that the interface itself is self-hosted.

Official docs: [librechat.ai/docs](https://www.librechat.ai/docs)

## Decisions to make before you deploy

### 1. How you'll run it

The [local installation guide](https://www.librechat.ai/docs/local) covers three paths: Docker Compose, npm, and a Helm chart for Kubernetes. The [remote hosting guide](https://www.librechat.ai/docs/remote) covers cloud options (DigitalOcean, Railway, HuggingFace and others) plus the networking pieces most of those guides assume you already have: a reverse proxy (Nginx, Traefik or Cloudflare), TLS, and DNS. LibreChat itself doesn't manage any of that for you — it's a separate decision, not a default.

### 2. Which models answer, and whose account pays for it

Set up endpoints under `endpoints.custom` in `librechat.yaml`: hosted providers through API keys, or a self-hosted model server (Ollama, vLLM, or anything exposing an OpenAI-compatible API) through a `baseURL` pointing at it. The [custom endpoints guide](https://www.librechat.ai/docs/quick_start/custom_endpoints) has the exact fields (`name`, `baseURL`, `apiKey`, `models`). All four are required — an incomplete block is dropped silently, which is a confusing way to discover a typo.

### 3. Where documents and their indexes live

If you want LibreChat to answer from uploaded documents, that's the separate RAG API component, not the chat model. Per the [RAG API docs](https://www.librechat.ai/docs/configuration/rag_api), it indexes files into a PostgreSQL + pgvector store and supports OpenAI, Hugging Face or Ollama as the embeddings provider. The default "lite" Docker image only supports remote embeddings — if you want the embedding step to run locally too, you need the full image, which is a decision worth making before deployment, not after you've already uploaded documents expecting them to stay fully local.

### 4. Who can get in

The [authentication docs](https://www.librechat.ai/docs/configuration/authentication) cover `ALLOW_REGISTRATION` (and `ALLOW_SOCIAL_REGISTRATION`) for turning public sign-up on or off, and a `create-user` script for adding accounts directly when registration is disabled. Per the [Docker installation guide](https://docs.librechat.ai/docs/local/docker), the first account registered on a single-tenant deployment is automatically granted admin privileges — worth knowing before you leave registration open on a box you're about to walk away from, even briefly.

## Pitfalls worth knowing in advance

- **Apple Silicon and MongoDB.** The default MongoDB image doesn't run on M-series Macs (missing AVX support). The [Docker guide](https://docs.librechat.ai/docs/local/docker) documents the fix: a `docker-compose.override.yml` pointing at an older, compatible MongoDB image.
- **Silent config drops.** A custom endpoint block in `librechat.yaml` missing a required field, or with bad indentation, doesn't error — it's just dropped. If a model you configured isn't showing up, check the YAML before you check the API key.
- **"Lite" vs. full RAG image.** The lite image works for remote embeddings only. If the documents going through the RAG pipeline are exactly the ones you didn't want leaving your infrastructure, confirm which image you're actually running.
- **First account = admin, by default.** True for single-tenant deployments per the Docker guide above. Disable `ALLOW_REGISTRATION` once you're past initial setup, or you're leaving an open door to an admin account.
- **A reverse proxy is your job, not LibreChat's.** The [remote hosting guide](https://www.librechat.ai/docs/remote) is explicit that you need Nginx, Traefik or Cloudflare in front of it, plus your own TLS certificates, for anything beyond local testing.

## Scope of these notes

These notes describe the public LibreChat project's own documented behaviour as of the dates the linked pages were read. They are not a guarantee that a given configuration is private, compliant, or production-ready for your workload — see the companion [private-ai-stack-checklist](https://github.com/stephanelepain-hub/private-ai-stack-checklist) for the broader architecture questions (OCR, embeddings, logging, backups) that a chat interface alone doesn't answer, and [running a self-hosted model past a handful of concurrent users](https://cplt.tech/blog/vllm-vs-llamacpp-vs-ollama-25-users) for what changes once more than a few people use it at once.

## Sources

- [LibreChat documentation](https://www.librechat.ai/docs)
- [Local installation](https://www.librechat.ai/docs/local) / [Docker guide](https://docs.librechat.ai/docs/local/docker)
- [Remote hosting](https://www.librechat.ai/docs/remote)
- [Custom endpoints](https://www.librechat.ai/docs/quick_start/custom_endpoints)
- [RAG API configuration](https://www.librechat.ai/docs/configuration/rag_api)
- [Authentication configuration](https://www.librechat.ai/docs/configuration/authentication)
- CPLT's own field notes on using LibreChat as the interface layer of a private stack: [What Is a Private AI Stack vs ChatGPT Enterprise?](https://cplt.tech/blog/what-is-a-private-ai-stack)

## License

CC BY 4.0 — reuse, adapt and republish, with attribution. See [LICENSE](LICENSE).

Built and maintained by [Stéphane Lepain](https://cplt.tech/about), [CPLT](https://cplt.tech).
