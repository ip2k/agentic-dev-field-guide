# Cloud: DigitalOcean for always-on, OpenRouter for model APIs

Default rule: develop and test on your own machines and in GitHub's CI. Rent cloud only for
what must be **always on and reachable** (a public site, an API, a bot, a scheduled job) or
for compute you do not have (a big GPU for a few hours). Prices below were checked
2026-11-21. Verify them on the linked pages.

## DigitalOcean

Why it suits small projects: simple flat-priced VMs ("Droplets", from about $4/month),
a usable CLI (`doctl`), managed Postgres, and GPU Droplets by the hour when needed.
Pricing: <https://www.digitalocean.com/pricing>.

### The pattern that has worked

A single small Droplet runs the public part of a project. The private part (source,
heavy processing, raw data) stays at home or in CI.

- **The private side pushes, the cloud box never pulls.** Home or CI connects out to the
  Droplet over SSH to deliver updates (a built site, a database snapshot, media files). The
  Droplet gets **no credentials or tunnel back** to your home network or private repos. If
  the Droplet is compromised, the attacker gets the public data and nothing else.
- Push to a restricted receiver: a dedicated SSH user, ideally a forced command and a
  separate port, that can only drop files in one place. It is not a general login.
- The receiver verifies an upload before it goes live (checksums, a database snapshot that
  is activated only after it loads), so a failed transfer leaves the last good version serving.
- **Caddy** in front: automatic HTTPS certificates, static files, and a reverse proxy to
  the app. The app and the database listen on loopback only.
- When home is down, the site keeps serving what it last received.
- Write down a recovery procedure, and test a restore at least once. An untested backup is a hope.

### Basics checklist

- Create the Droplet with an **SSH key**, not a password. Disable password login.
- Use a **DigitalOcean Cloud Firewall**: 80/443 open, SSH only from your IP or over a
  VPN/overlay (WireGuard, Tailscale). This is free and sits outside the VM. GitHub-hosted
  runners have no fixed IP, so a CI job cannot pass an IP allow-list. Deploy from home, from
  a self-hosted runner, or over the VPN.
- Unattended security updates on. Enable Droplet backups or snapshot before risky changes.
- `doctl` with a **scoped, expiring API token**. You run `doctl auth init`, and the agent
  never sees the token.
- Set a **billing alert**. Droplets (GPU ones included) are billed while powered off, so
  *destroy* a temporary one when done, do not just stop it. Snapshot first if you need the disk.
- Keep the Droplet's configuration in the repo (Ansible, cloud-init, or at least a
  documented script) so it can be rebuilt. Never hand-edit the box and walk away.

Alternatives with the same shape: Hetzner (cheaper in the EU), Fly.io (containers),
Render, Railway. For a pure static site, GitHub Pages or Cloudflare Pages are free.

## OpenRouter

One API key and one bill for hundreds of models from many providers (open-weight and
closed), with an OpenAI-compatible endpoint: `https://openrouter.ai/api/v1/chat/completions`.
Use it when **your own code** needs a model (summarize, classify, transcribe, embed) and you
want to try or switch models without new accounts. Docs: <https://openrouter.ai/docs>.

Practices that matter:
- **Choose the provider deliberately for sensitive data.** The request's `provider` object
  lets you pin a provider list, disable fallback to others, and require zero-data-retention
  and no-training endpoints:
  ```json
  "provider": { "order": ["<provider>"], "allow_fallbacks": false,
                "zdr": true, "data_collection": "deny" }
  ```
  Your account's privacy settings can also restrict allowed providers. If a request is
  refused at routing, check that list.
- **Never send someone else's data without the owner's yes.** Recordings, documents and
  messages that are not yours need explicit permission before they go to any API.
- **Prepaid credits are a natural spending cap.** When they run out you get HTTP 402.
  Handle it (retry later, fail one item, not the whole run). Also handle 400 per item, and
  write scripts that resume by skipping finished items.
- Key in the OS keychain or a password manager, never in the repo. **Your script** reads it
  at run time, for example on macOS with
  `subprocess.run(["security","find-generic-password","-s","openrouter","-a","<project>","-w"])`.
  The agent never runs that command in its own shell, where the key would land in the transcript.
- The default `/models` list shows text models only. Add `?output_modalities=all` to
  find speech-to-text, image and other models.
- Record the model, provider, parameters and cost of every run next to its output, so results
  can be compared and reproduced. Small jobs are often extremely cheap (cents).

### Optional: Claude Code itself through OpenRouter

Possible, documented by OpenRouter
(<https://openrouter.ai/docs/guides/coding-agents/claude-code-integration>):
`ANTHROPIC_BASE_URL=https://openrouter.ai/api` (no `/v1`),
`ANTHROPIC_AUTH_TOKEN=<your key>`, `ANTHROPIC_API_KEY=""`; check with `/status`. For daily
interactive use, a Claude subscription is usually much cheaper than per-token billing, and
non-Anthropic models may not work well with Claude Code. Treat this as a niche option.

## Local models (if you have a GPU)

Ollama serves open-weight models on your own hardware with no per-token cost. One trap: use its
**native** API (`/api/chat`, `/api/embeddings`) for anything that uses tool calling. Its
OpenAI-compatible `/v1` path broke tool calling for us, and the model started writing
tool-call JSON as plain text. That looked like a bad model when it was really a bad URL.
That was one setup in 2026. If tool calls come back as text, try the native API first.
