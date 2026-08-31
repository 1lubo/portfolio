---
title: "Leave the Package Sealed"
description: "Learning Rust from Java, I almost compared Option and Result to rabbits and grenades. The better analogy is calmer: a sealed package with a contents label — and the habit of not opening it until you mean both answers."
pubDate: 2026-08-31
tags: ["rust", "java", "option", "result", "learning-in-public", "self-taught"]
draft: false
---

I almost used the wrong metaphors.

A rabbit pulled from a hat — `Option` as theatre, something that might materialise if the magician is kind. A live grenade passed down a line of callers — `Result` as danger you palm off on the next person with a muttered *here, you deal with it*. Both instincts were reaching for the same picture. Both were wrong in the same way.

They make the wrapper feel tricky or risky. What I actually landed on, building a small bookmark service in Rust after years of Java, is the opposite. The wrapper is calm. It is a sealed package with a contents label.

An `Option<T>` is a parcel whose label reads: *may contain a Bookmark — or may be empty*. A `Result<T, E>` reads: *contains the answer, or a note explaining what went wrong*.

While it is sealed you can still carry it, stack it, hand it between functions. The label travels with it. You only open it when you are ready to act on the contents — and at that moment you deal with both possibilities the label already named. The tension is not danger versus defusal. It is **labeled and deferred** versus **opened too early**.

## The one place it clicked

I am working through a test-driven Rust course I set myself: a personal read-later service — save a URL, let a worker fetch a title and excerpt, list and search and mark read. Familiar domain on purpose, the same trick that made [the Spring piece](/blog/what-spring-was-doing-for-me/) work. When the *what* is boring, every surprise left is a surprise about the *how*.

The look-up signature is not "Bookmark or boom." It is nested honesty:

```rust
async fn get(&self, id: &str) -> Result<Option<Bookmark>, AppError>
```

Outer label: the lookup itself can fail — storage poisoned, a database error. Inner label: the row might simply not be there. Neither is a scorpion hiding in an unlabeled box. Both are written on the type, before anyone runs anything.

So a handler does not tear either package open on contact. It *moves* them. The `?` operator is the calm sentence: if this package is the failure note, send that note upstairs; otherwise keep carrying what remains. The `Option` can stay closed until the HTTP edge, where `None` becomes a 404 only because you are finally writing a response. Opening early is optional. Carrying is the default.

That is the habit I did not have. Coming from Java I reached for the contents the moment a function returned — force the happy path into existence so I could keep typing. Rust kept asking me to wait.

## What `.unwrap()` actually is

`.unwrap()` is tearing the box open without reading the label and hoping.

Sometimes there is nothing inside. Faceplant. Useful in a test, or in a branch the type system has already made unreachable. As a reflex everywhere else it means you stopped believing the label and decided surprise was cheaper than two arms of a `match`. It rarely is. The compiler offered you the honest fork for free; unwrap is how you throw the fork away and call the crash a personality trait of the language.

## The unlabeled box

Java often hands you a box with no label. The type says `Bookmark`. At runtime it might be `null`. Or the call throws, and the stack is the first honest label you get — after you have already opened it, sometimes only in production, under load, on a path no test exercised.

`Optional<T>` is real progress. So are checked exceptions, on the days anyone still uses them. Neither is the default texture of everyday Spring code the way `Option` and `Result` are the default texture of this Rust service. In the bookmark store, absence and failure are not conventions you remember to honour. They are the return type. The compiler will not let you toss a sealed package aside as if it were already a `Bookmark`.

I am not scorning Java. I write it for a living. I am naming the trade: a language that lets you leave the label blank will let you ship faster until the blank becomes a null pointer at two in the morning. A language that prints the label on every doorframe will feel pedantic until you notice you have stopped being surprised by empty boxes.

## Leave it sealed

What changed in my head was smaller than "Rust is safer." It was a change of timing.

Carry the package. Let the label do its job across layers. Open at the boundary where both outcomes have a real answer — a status code, a bookmark marked `Failed`, a title that stays `None` until the worker fills it. Do not open in the middle just to quiet the anxiety of not knowing. The not-knowing is already documented. That is what the seal is for.

This is the same lesson as the Spring post, wearing a different costume. There, Rust made ownership and wiring explicit. Here, it makes absence and error explicit — and then trusts you to keep them sealed until the moment acting on them *is* the work.

Not magic. Not a grenade. A label, a seal, and a compiler that asks whether you read them.
