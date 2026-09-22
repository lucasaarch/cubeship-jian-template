# Jian on Cubeship

[Jian](https://github.com/lucasaarch/jian) is a self-hosted agent gateway that
gives AI agents a persistent identity. A profile keeps its own instructions,
model, memory and tools, and every session and channel that belongs to it —
the web panel, WhatsApp, Telegram, its API — works from that same context.

This template installs the gateway on a Cubeship instance, with its web panel
on a domain and everything it keeps in a managed PostgreSQL.

## What it creates

- **jian** — Jian `1.0.1`, from `ghcr.io/lucasaarch/jian-gateway`. The API
  answers on the domain you choose, and the panel under `/ui/` on the same
  domain. It holds no state of its own: a redeploy loses nothing.
- **jian-db** — PostgreSQL 17 with a database named `jian`, attached to the
  gateway. Profiles, sessions, history, memories, the run queue and the
  encrypted vault are all in it. Jian migrates it on start.

It needs Cubeship 0.6.0 or newer.

## What you are asked

| Input | What to give |
| --- | --- |
| Where the gateway and its panel answer | A domain you control, pointed at your instance. |
| The token you sign in with | Nothing — the instance generates it and shows it once. **Keep a copy.** |
| The key the vault encrypts credentials with | Nothing — the instance generates it and shows it once. **Keep a copy, away from the database backups.** |

No model provider key is asked for. Add one in the panel instead: it is
encrypted into the vault, and a key the template set would be an empty
variable on the app whenever you left it blank.

## After installing

1. Open `https://<your domain>/ui/` and sign in with the token.
2. Under **Providers**, add a key for Anthropic, Gemini or OpenAI, or sign in
   with ChatGPT.
3. Create a profile and talk to it.
4. To reach it from a chat platform, open **Channels** on the profile.
   WhatsApp pairs by scanning a QR code. Telegram takes a BotFather token and
   shows a webhook URL and secret once; point the bot at them with Telegram's
   `setWebhook`. A stranger's first message becomes a contact request you
   approve.

## The token is the whole installation

Jian has one owner and one credential. Whoever has the token can read every
conversation, change every profile and run anything the agents can. It is the
only thing in front of the panel on a public domain. Treat it like a root
password. To rotate it, change `JIAN_API_TOKEN` on the app and redeploy: that
also signs every panel session out.

## The vault key

Provider keys, MCP tokens and channel credentials are encrypted with the key
in `JIAN_MASTER_KEYS` and never read back. The key lives on the app, not in
the database, so **a database backup without it cannot open a single stored
credential**. Keep the generated value somewhere of its own.

To rotate it, add a second entry and make it the active one — keep the old
one until every credential has been sent again from the screen that
configures it:

```text
JIAN_ACTIVE_KEY_ID=v2
JIAN_MASTER_KEYS={"v1":"<old key>=","v2":"<44 base64 characters for 32 new bytes>"}
```

## What Jian can reach

Outbound calls go to public HTTPS only. Private addresses — including every
other app and database on this instance — are refused unless named, exactly,
in `JIAN_ALLOW_PRIVATE_ORIGINS`: for an Ollama on the same instance,
`http://cubeship-<project>-<environment>-ollama:11434`. Metadata and
link-local addresses stay blocked whatever it says.

## What does not work here

- **Per-client rate limiting.** Jian limits requests by connection address and
  does not trust `X-Forwarded-For`, so behind the instance's proxy every
  client shares one limit.
- **More than one copy.** Jian migrates on start and copies would race for it.
  Keep `scale` at 1; split API and worker with `JIAN_ROLE` only once you know
  why.

## Updating Jian

The Jian version is the image `tag` in `template.yaml`. Jian migrates its
database on start and a migration does not undo itself: back the database up
before moving to a newer release of this template. Going back to an older
image does not go back on the schema.

## Resources

The gateway is limited to 1 CPU and 1 GiB of memory, and the database to the
same. Raise `limits` in `template.yaml` if you need more.

---

<!-- cubeship-crosslink -->

## About Cubeship

This is a template for [**Cubeship**](https://github.com/cubeshipd/cubeship) —
a PaaS you run on your own server: `docker push`, and it is live, with HTTPS,
a database beside it, and a second machine when one stops being enough.

Browse every template at [cubeship.dev/templates](https://cubeship.dev/templates).
