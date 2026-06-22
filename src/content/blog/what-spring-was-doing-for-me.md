---
title: "What Spring Was Doing For Me All Along"
description: "I tried to learn Rust by porting a service I actually maintain — and found out how much of my Java career a framework had been quietly handling. A story about ownership, singletons, routes, and a compiler that made me do the work by hand."
pubDate: 2026-06-22
tags: ["rust", "java", "spring", "learning-in-public", "ownership", "self-taught"]
draft: false
---

My first attempt to learn Rust did not survive contact with reality.

I'd picked a Tor proxy project — something genuinely interesting, networking and crypto and async all at once. And it was too much. Every line introduced three new things simultaneously, and I couldn't tell which of them was *Rust* and which was just *the problem being hard*. The differences from Java were too large to see clearly, because everything was different at the same time. I bounced off it.

So I did something deliberate the second time. Instead of learning Rust through an unfamiliar problem, I learned it through a familiar one: I took a thin slice of a service I actually maintain in Java and Spring — an in-memory CRUD API over "media fragments" — and rebuilt it in Rust. Same domain, same endpoints, same shapes I already understood cold. The only unknown left in the room was the language itself.

That single change is what made everything visible. When the *what* is familiar, every surprise that's left is a surprise about the *how*. And what surprised me, over and over, was the same thing wearing different costumes: **Spring had been doing an enormous amount of work on my behalf, and I had stopped being able to see it.**

## The seduction of magic

This is the thing about a mature framework: it's so good that it disappears.

In Spring I write `@RestController`, `@Repository`, `@Autowired`, and a machine I never look at conjures the rest — instantiates beans, wires them together, maps routes, manages lifecycles, hands every request a shared instance of the right thing. I don't decide who owns an object. I don't decide whether something is shared. I don't write down where my routes live; I scatter annotations and trust they're collected. For years I called this "writing the application." Rust taught me it was mostly *signing for deliveries I never opened*.

Rust ships with no such machine. There's no container deciding things for you. So the work doesn't go away — it just becomes yours, explicitly, on the screen, where the compiler can see it and refuse to continue until you've made up your mind.

Here are the four moments where I felt that most.

## Scene 1 — Ownership: the compiler made me answer "who owns this?"

The repository was the first wall. In Java it's the most boring class imaginable: a `HashMap` and some methods. In Rust the same `HashMap` made me account for every value that crossed the boundary.

```rust
pub fn insert(&mut self, fragment: Fragment) {
    self.fragments.insert(fragment.id.clone(), fragment);
}

pub fn get(&self, id: &str) -> Option<Fragment> {
    self.fragments.get(id).cloned()
}
```

Look at `insert`: it takes the fragment *by value*. The repository now **owns** it; the caller has given it away and can't use it again. Look at `get`: I had to consciously `.clone()` to hand back a copy, because otherwise I'd be lending out a reference into a map I might later mutate — and the borrow checker will not allow that ambiguity to exist.

In Java I have never once thought about this. The garbage collector is the silent partner that makes "who owns this object and for how long?" a question I'm allowed to ignore. Rust doesn't have that partner, so it asks me directly, on every line. At first this felt like harassment. Then it felt like the first time anyone had ever made me *say out loud* something I'd been hand-waving my entire career.

## Scene 2 — Building my own singleton bean

In Spring, a `@Repository` is a singleton. The framework creates one instance and injects that same instance into every controller, on every request, across every thread, and the thread-safety is mostly the database's problem. I never write any of that down. It is, quite literally, an annotation.

In Rust I had to construct that guarantee by hand:

```rust
pub type SharedState = Arc<Mutex<FragmentRepository>>;

pub fn new_shared_state() -> SharedState {
    Arc::new(Mutex::new(FragmentRepository::new()))
}
```

Two wrappers, and each one is a decision I was previously allowed to skip. `Arc` — atomic reference counting — says *many handlers may share ownership of this one repository*. `Mutex` says *only one of them may mutate it at a time*. Together they spell out, in the type itself, "one shared, thread-safe instance." That's the singleton bean. I just had to *write the sentence* Spring had been saying for me silently for years. Concurrency stopped being a property of the framework and became a thing I could point at.

## Scene 3 — Spelling out my own routes

```rust
pub fn build_router(state: SharedState) -> Router {
    Router::new()
        .route("/healthz", get(health))
        .route("/fragments", get(list_fragments).post(create_fragment))
        .route("/fragments/{id}", get(get_fragment).delete(delete_fragment))
        .with_state(state)
}
```

In Spring my route table doesn't exist as an object — it's an emergent property of `@GetMapping`/`@PostMapping` annotations sprinkled across methods, assembled by reflection at startup. To answer "what are all my routes?" I grep, or I boot the app and read logs.

Here the route table is a *value* I build and return. The whole API is in one place I can read top to bottom. Even attaching the shared state is explicit — `.with_state(state)` — the dependency injection I'd never had to perform myself. It was more typing. It was also the first time I could *see* my own API.

## Scene 4 — Declaring my own packages

The smallest one, and somehow the one that reframed it all. In Java a folder *is* a package; the directory layout silently is the structure. In Rust I had to declare it:

```rust
pub mod api;
pub mod error;
pub mod model;
pub mod repository;
pub mod state;
```

Nothing is a module until I say it is. The crate root is a table of contents I write by hand. A convention I'd taken for granted as a law of nature turned out to be one more thing a tool had been quietly doing for me.

## The reframe

For a while I told myself Rust was hostile — pedantic, slow, allergic to getting anything done. That's the wrong frame, and porting a service I already understood is what corrected it. Because I knew exactly what the app was *supposed* to do, I couldn't blame the friction on the problem. The friction was just the bill, finally itemised.

None of `Arc<Mutex>`, ownership, the explicit router, or the module tree is Rust inventing difficulty. Every one of them is a real decision that exists in my Spring app too — Spring just made it, on my behalf, out of sight. Rust dragged each one into the light and refused to compile until I'd genuinely decided. **The reason "it compiles" means so much more in Rust is that compiling required me to answer questions Java let me leave blank.**

I'm not abandoning Spring; its magic is good magic, and most days I want it. But I understand my Java far better now for having spent a week without it. Knowing what a framework is doing for you is a different kind of knowledge from knowing how to invoke it — and you mostly only get it by taking the framework away.

So I did the same thing I always do: I picked something I already understood, rebuilt it in public, and let the compiler break me until I understood it for real. The Tor proxy was too big a leap to learn anything from. A boring CRUD API I already knew by heart turned out to be exactly the right size to learn almost everything.
