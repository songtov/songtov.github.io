---
author: Chiho Song
pubDatetime: 2026-11-06T01:45:00Z
title: "Startup #4: Shipping an MVP in Days"
slug: startup-shipping-mvp
featured: false
draft: false
tags:
  - startup
  - saas
  - indie-hacking
description: A solo SaaS MVP should take days, not months. Build the one job users pay for, reuse the plumbing every SaaS needs, and put a deadline on the build.
lang: en
---

Part 4 of the Startup series. [Part 3](/posts/startup-validating-ideas/) argued that when an MVP takes a weekend, the launch itself is the validation.

That only works if you can actually ship in days. This part is about how.

---

## What "Minimum" Means

An MVP is not a smaller version of your dream product. It's the smallest thing that does **one job** someone would pay for, plus a way for them to pay.

If your idea is "invoices for Stripe customers," the MVP generates an invoice. It doesn't have templates, team accounts, or a settings page. One job, done well enough that a stranger would use it.

Everything else is a guess about what users will want. Guesses can wait until real users show up.

---

## Don't Rebuild the Plumbing

Every SaaS needs the same boring parts: login, payments, transactional emails, a landing page, basic SEO. None of them is what users pay for. All of them take time.

If you build those from scratch for every product, most of your build time goes to plumbing, and the actual feature gets squeezed into whatever time is left.

![Two bars comparing build time: from scratch, login, payments, emails, and landing page take most of the time; with a reusable starter, most time goes to the core feature](/images/startup-shipping-mvp.svg)

This is exactly the problem behind ShipFast. Marc Lou kept launching products and kept rebuilding the same pieces, so he packaged them into a Next.js boilerplate. Its site says "Ship your startup in days, not weeks" and adds up the setup work it replaces as "22+ hours of headaches." It lists the stack as MongoDB, Mailgun, Stripe, Next.js, Tailwind, and NextAuth, and claims 8,446 makers have used it (self-reported, as of November 2026).

The lesson isn't "buy ShipFast." It's: **have a starter.** Buy one, use an open-source template, or extract your own from your first project. If you plan to make many small bets, the plumbing should be solved once.

---

## Cut Scope Ruthlessly

A few rules that keep an MVP small:

- **No settings page.** Pick sensible defaults.
- **No team accounts.** One user, one account.
- **No polished dashboard.** A plain list is fine.
- **No features nobody has asked for.** If you're unsure, leave it out.

When you're tempted to add something, ask: "Would anyone refuse to pay without this?" If the answer is no, it goes on a later list.

---

## Put a Deadline on It

Work expands to fill the time you give it. A deadline forces the scope cuts you'd otherwise avoid.

Marc Lou's Ship or Die puts it bluntly: "SHIP your app in 30 days, or die." For a small SaaS with a starter, a weekend to two weeks is a reasonable target.

When the deadline arrives, ship what works. Whatever didn't make it was, by definition, not part of the minimum.

---

## Embarrassing Is Fine

The first version will feel unfinished. That's the point. It exists to answer one question: does anyone use it and pay?

Early users of a small, focused tool are surprisingly forgiving about rough edges if it solves their problem. They're much less forgiving of a product that never ships.

If the answer is yes, you polish with real feedback. If it's no, you've lost days instead of months, and you move on to the next idea.

---

## Takeaways

- An MVP does one paid job, plus payments. Nothing else.
- Solve the plumbing once with a starter, then reuse it.
- Cut anything users wouldn't refuse to pay without.
- Set a deadline and ship what works when it arrives.

---

## Sources

- [ShipFast](https://shipfa.st): tagline, setup-time claim, stack, maker count (accessed November 2026)
- [Marc Lou's site](https://marclou.com): Ship or Die tagline (accessed November 2026)
