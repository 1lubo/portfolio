---
title: "Fuhgeddaboudit and the Borrow Checker"
description: "Donnie Brasco has a whole monologue about how 'fuhgeddaboudit' means five different things depending on context. Learning Rust's ownership rules, I kept hearing it in Lefty's voice — and it turned out to be a better guide than I expected."
pubDate: 2026-09-21
tags: ["rust", "ownership", "borrowing", "learning-in-public", "self-taught"]
draft: false
---

There's a bit in *Donnie Brasco* where Lefty explains "fuhgeddaboudit" to Donnie like it's a whole dialect on its own. Say it one way, it means *forget about it, no problem*. Say it another way, in another context, and it means the exact opposite — *don't even think about it*. Same two words. The meaning was never in the phrase. It was always in what was happening around it.

![The Godfather saying fuhgeddaboudit](https://media1.tenor.com/m/keY9zNtLuvEAAAAC/the-god-father-marlon-brando.gif)

I did not expect to think about that monologue while learning Rust. Then I wrote my third `&mut` in a row and got told, flatly, no.

## The literal meaning

Start with the easy one, because Rust has a rule that is not a metaphor at all — it is just called "forget about it."

```rust
struct Bookmark {
    id: String,
    url: String,
}

fn archive(bookmark: Bookmark) {
    // takes ownership; the caller's binding is done after this
    println!("archiving {}", bookmark.url);
}

fn main() {
    let b = Bookmark { id: "1".into(), url: "https://example.com".into() };
    archive(b);
    // println!("{}", b.url); // error: value borrowed here after move
}
```

`archive` takes `bookmark` by value. Once `b` is handed over, `main` is done with it — not "should probably stop using it," not "please be careful," but *cannot compile if you touch it again*. That last commented-out line isn't a style complaint. The compiler enforces the forgetting. Coming from Java, where a reference just... keeps existing, still pointing at the same object, no matter how many places have "moved on" from it, this was the first time forgetting was a rule rather than a discipline.

That part of the metaphor is almost too literal to be interesting. What kept it in my head for weeks was the other half of Lefty's speech — the part where the same phrase flips meaning depending on who's in the room.

## The contextual meaning

Here's the borrow checker doing the exact trick the monologue is about.

```rust
fn print_two(a: &Bookmark, b: &Bookmark) {
    println!("{} and {}", a.url, b.url);
}

fn rename(bookmark: &mut Bookmark, new_url: String) {
    bookmark.url = new_url;
}

fn main() {
    let mut b = Bookmark { id: "1".into(), url: "https://example.com".into() };

    let r1 = &b;
    let r2 = &b;
    print_two(r1, r2); // fuhgeddaboudit — fine, no problem

    // let r3 = &mut b;   // fuhgeddaboudit — no, don't even think about it
    // rename(r3, "https://example.org".into());
}
```

`&b` twice is nothing. Two people looking at the same thing at once is not a crime; nobody's changing it, so there's nothing to protect against. That's the "fuhgeddaboudit, no problem" reading — the context (two *shared*, read-only borrows) makes the phrase mean *go ahead*.

Ask for `&mut b` while `r1` and `r2` are still alive, though, and it's the other reading: *fuhgeddaboudit, don't even think about it*. Same syntax — `&mut` isn't a scarier keyword than `&`, it's one character different. The verdict flips entirely because of what else is going on around it: whether anyone else might be looking at the same data while you plan to change it out from under them. Rust isn't reacting to the phrase. It's reacting to the context the phrase is spoken in, exactly like Lefty's monologue insists it should.

![Donnie Brasco](https://media1.tenor.com/m/01vxQOXxA-0AAAAC/donnie-brasco.gif)

That reframed the compiler error for me. `error[E0502]: cannot borrow ... as mutable because it is also borrowed as immutable` isn't the compiler being precious about a keyword. It's the compiler asking the same question Lefty is asking: *given everyone else who's already listening, what does this particular thing you're about to do actually mean right now?*

## Where I stopped stretching it

Lifetimes are where I made myself stop reaching for the bit. It's tempting — `'a` looks like it wants a punchline. But lifetimes aren't really about a phrase changing meaning by context. They're closer to the compiler keeping a ledger of how long you're still allowed to be remembering something at all, independent of what you're doing with it in the moment. That's a different kind of bookkeeping than "same words, different room, different verdict." Forcing it into the same joke would have meant writing a punchline I didn't actually believe, and the whole reason the [package post](/blog/leave-the-package-sealed/) worked was refusing to do that with `Option` and grenades. Same discipline here: two solid parallels are worth more than three where the third is padding.

## What it actually taught me

I've spent most of this series translating Rust into things I already understood — a sealed package, a labeled contract, wiring made explicit. This one's different, because the metaphor didn't just decorate the concept, it predicted my mistakes. Every time I got a borrow-checker error that felt unfair in the moment, it was because I was still thinking of `&` and `&mut` as fixed labels with fixed meanings, instead of asking what else was in the room. Lefty already told me the answer. I just wasn't listening for it in the right accent.
