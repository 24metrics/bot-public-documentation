# 24metrics Bot

The 24metrics Bot is an automated agent operated by [24metrics](https://24metrics.com), an
ad-tracking and traffic-quality platform. It fetches individual web pages on behalf of
24metrics customers, so that those customers can be told whether their own advertising
links work correctly and where their traffic actually comes from.

This repository is **documentation only**. It contains no source code. It exists so that
site operators — and Cloudflare's verified-bot programme — can find out what our bot is,
how to recognise it, and how to reach us.

## What the bot fetches, and why

The bot exists to serve two features of the 24metrics platform:

**Affiliate link checking.** 24metrics customers register their own advertising and
affiliate tracking links with us. The bot follows those links end to end and reports back
whether each one resolves to the destination the customer expects. This is how customers
find out that a link is broken, has been hijacked into an unexpected redirect chain, or is
serving different content than it is supposed to.

**Click referrer tagging.** When traffic arrives at a customer's tracking link, the bot may
fetch the referring page in order to classify the traffic source. This lets customers
distinguish legitimate placements from misrepresented or fraudulent ones.

Both of these are checks a customer has asked us to perform on their behalf, against links
and referrers that already exist in their own account.

## What the bot does not do

- It does **not** crawl the open web. It has no crawl frontier and does not discover pages
  by following links for the purpose of indexing.
- It does **not** collect content for search indexing, for resale, or for training machine
  learning models.
- It does **not** attempt to bypass paywalls, log in, submit forms, or access anything that
  an ordinary anonymous visitor could not reach.

### robots.txt

**The bot does not consult robots.txt.** We would rather state this plainly than claim a
compliance we do not implement.

The reason is that the bot is not a crawler. Every request it makes is to a specific URL
that a 24metrics customer has submitted to us, in order to answer a question that customer
asked about that specific URL. A request is closer in kind to a single user clicking the
link than to a crawler enumerating a site, and the volume per host reflects that.

If our traffic is nonetheless unwelcome on your site, please see
[Blocking the bot](#blocking-the-bot) below — we will honour a block, and we would rather
hear from you than have you discover us in your logs.

## Identifying the bot

### Web Bot Auth signature

The bot identifies itself cryptographically using
[Web Bot Auth](https://developers.cloudflare.com/bots/reference/bot-verification/web-bot-auth/),
the HTTP Message Signatures mechanism for bot verification. This is the reliable way to
confirm a request is genuinely ours: a User-Agent string can be copied by anyone, whereas a
valid signature requires our private key.

Our signing keys are published at:

```
https://bot-auth.24metrics.com/.well-known/http-message-signatures-directory
```

That directory currently publishes a single Ed25519 key:

```json
{
  "keys": [
    {
      "kty": "OKP",
      "crv": "Ed25519",
      "x": "oBA_L3AVl8tZnuY8_EeIxGH0OvyC10dAH1vebRyUWa4"
    }
  ]
}
```

The corresponding key ID — the RFC 7638 JWK thumbprint, which appears as `keyid` in the
signature parameters — is:

```
VHUNe7HlZKjZxFFCgeD7quZ7ziR87RmqXRuPHaxwBWQ
```

The directory response is itself signed with that key, so you can confirm we control it
before trusting anything served from the endpoint:

```console
$ curl -sSD - https://bot-auth.24metrics.com/.well-known/http-message-signatures-directory
HTTP/1.1 200 OK
content-type: application/http-message-signatures-directory+json
signature: sig1=:...:
signature-input: sig1=("@authority";req);created=...;keyid="VHUNe7HlZKjZxFFCgeD7quZ7ziR87RmqXRuPHaxwBWQ";alg="ed25519";expires=...;nonce="...";tag="http-message-signatures-directory"
```

Signatures are Ed25519, carry a nonce, and expire five minutes after issue. The directory
is served with `Cache-Control: max-age=300`.

If you are behind Cloudflare, you do not need to verify any of this yourself — Cloudflare
performs the verification and surfaces the result as a verified bot.

### User-Agent

The bot sends:

```
24metricsBot/1.0 (+https://github.com/24metrics/bot-public-documentation)
```

Treat this as a convenience for reading logs, not as proof of identity. Anything can send
this string. Use the signature if you need certainty.

## Blocking the bot

We would prefer that you allow the bot, because the checks it performs are what tell an
advertiser their traffic to your site is genuine. But it is your site.

To block it, deny requests carrying the User-Agent above, or the Web Bot Auth key ID if
your edge supports matching on it. If you are on Cloudflare, the bot appears as a verified
bot and can be blocked through your existing bot management rules without writing anything
custom.

Blocking will not cause us to retry from other addresses or otherwise attempt to evade the
block.

## Contact

For anything about this bot — unexpected traffic, a block that is not taking effect, a
report of someone impersonating our User-Agent, or a question about what a particular
request was for — write to **dev@24metrics.com**.

Please include the requested URL and an approximate timestamp; that is usually enough for
us to trace a specific request back to the check that caused it.
