# 24metrics Bot

The 24metrics Bot is an automated agent operated by [24metrics](https://24metrics.com), an
ad-tracking and traffic-quality platform. It fetches individual web pages on behalf of
24metrics customers, so that those customers can be told whether their own advertising
links work correctly, and what kind of pages their traffic is arriving from.

This repository is **documentation only**. It contains no source code. It exists so that
site operators — and Cloudflare's verified-bot programme — can find out what our bot is,
how to recognise it, and how to reach us.

## What the bot fetches, and why

The bot fetches pages for two reasons, both of them on behalf of a customer:

**Verifying a customer's own advertising links.** 24metrics customers register their
advertising and affiliate tracking links with us. The bot follows those links end to end
and reports back whether each one resolves to the destination the customer expects. This is
how a customer finds out that one of their links is broken, has been diverted into an
unexpected redirect chain, or is serving something other than what it should.

**Describing the pages a customer's traffic arrives from.** Customers send us the referrer
URLs recorded against their traffic. The bot visits those pages so that we can tell the
customer what kind of content is on them.

In both cases the bot is acting on a URL that a customer has handed us, in order to answer
a question that customer asked about that URL.

## What the bot does not do

- It does **not** crawl the open web. It follows redirect chains in order to resolve a
  submitted link, but it does not extract links out of page content and queue them for
  further fetching. There is no crawl frontier.
- It does **not** index or republish your content. What it retrieves is used to answer the
  question the customer asked about the URL they submitted, and for nothing else.
- It does **not** attempt to bypass paywalls, log in, submit forms, or reach anything an
  ordinary anonymous visitor could not.

Every request the bot makes is to a specific URL a 24metrics customer submitted to us, in
order to answer a question that customer asked about that URL. A request is closer in kind
to a single visitor clicking a link than to a crawler enumerating a site, and the volume
per host reflects that.

If our traffic is unwelcome on your site, see [Blocking the bot](#blocking-the-bot) below.
We would rather hear from you than have you discover us in your logs.

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

We would prefer that you allow it, since every fetch is made on behalf of a customer who
has asked us a question about a link or a referrer pointing at your site. But it is your
site.

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
us to trace a specific request back to what prompted it.
