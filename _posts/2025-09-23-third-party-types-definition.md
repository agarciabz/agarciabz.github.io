---
layout: post
title: Bringing Type Safety to Cronofy Elements with TypeScript Definitions
categories: [TypeScript, React]
---

One of the things I appreciate most about TypeScript is how it transforms the developer experience. Autocomplete, inline documentation, and type checking make it harder to shoot yourself in the foot with something as simple as a typo. But that benefit only exists when the libraries you use ship with type definitions, or when someone takes the time to create them.

Recently, I had the chance to do exactly that.

## The Problem: No Types for a Critical Library

In one of my projects, we used [Cronofy Elements](https://docs.cronofy.com/developers/elements/), a UI library for time slot management. It was central to our user flow: users interact with these components to schedule availability, which is business-critical.

There was only one problem: **Cronofy Elements shipped with no TypeScript definitions**.

This meant every time we wrote code against it, we were vulnerable to small but impactful mistakes like misnaming a property or forgetting which options were available. In JavaScript, that might only surface as a runtime error. In TypeScript, it would’ve been caught immediately.

For someone like me (who’s occasionally typo-prone) or for someone with a condition like dyslexia, the lack of autocomplete and type checking isn't just an inconvenience: it was a source of real bugs.

## The Solution: Writing My Own Type Definitions

After hitting a couple of bugs caused by these gaps, I had a _“Fine, I’ll do it myself”_ moment.

I started by writing a local `cronofy-elements.d.ts` file in my project. That gave me type safety for my own use case and immediately improved developer productivity. Instead of second-guessing myself while typing, the editor guided me with autocompletion and warnings.

But stopping there felt incomplete. If my team and I were struggling with this, surely others were too. So I decided to polish the type definitions and submit them to **DefinitelyTyped**, the community repository that powers `@types/\*` packages on npm.

## The Outcome: A Contribution to Open Source

A few weeks later, the pull request was accepted, and now anyone installing `@types/cronofy-elements` can enjoy full TypeScript support out of the box.

What started as a local fix became a small but meaningful contribution to the ecosystem. It’s the kind of thing that doesn’t always get headlines but quietly improves the daily lives of developers. And honestly, that’s one of the most satisfying parts of open source.

## Takeaways

- If a library doesn’t ship with type definitions, don’t wait. Write your own.
- Submitting them to **DefinitelyTyped** is a low-barrier, high-impact way to give back to the community.
- Small contributions like this can eliminate entire classes of bugs and improve the developer experience for countless others.

Always proud when I can turn a local hack into something useful for the wider community. 😃

👉 You can find the published type definitions [here](https://lnkd.in/db4mAEFs)
