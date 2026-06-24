# Digital Narrative Workstation (DNW) -- SaaS Overview

> Logic Pro, for words. With pip install for long-form narrative fiction. And plugins.

## Problem Statement

DNW is a **narrative intelligence workstation** -- a purpose-built environment where the AI works *with* the author, not instead of them.

> Write once, read everywhere.

Writing long-form narrative fiction is one of the most cognitively demanding creative endeavours a human being undertakes. A novelist managing a 120,000-word manuscript is simultaneously tracking plot, character arcs, internal consistency, prose rhythm, vocabulary, world-building, timeline, and the emotional truth of every scene across months or years of work.

The tools available to authors have not kept pace with this complexity. Word processors were built for documents. Dedicated writing tools (Scrivener, Ulysses, Final Draft) manage structure but offer no intelligence. AI writing tools offer intelligence but in entirely the wrong direction -- they want to write *for* you, producing generic prose that sounds like everything and nothing.

And as an author, I don't need a tool that will write for me. I need a tool that will help me *think* about my story and that will help me manage the complexity of my work without getting in the way of my creative flow.

The result: professional authors, screenwriters, and narrative teams spend enormous cognitive overhead on mechanical tasks: continuity checking, consistency enforcement, lexicon management and structural analysis to name just a few. And these mechanical tasks have nothing to do with the act of writing.

---

## Write Once, Publish Everywhere

DNW imports from every major writing tool. Wherever an author is working today -- Scrivener, Word, Final Draft, Fountain, Markdown, legacy formats going back to Adobe Story -- DNW will meet them there.

On the output side, DNW publishes to every platform that puts readers first: every major EPUB retailer (Amazon, Apple, Google, Kobo, B&N), print-ready PDF, audiobook formats for Audible and the major podcast platforms, social serialisation for LinkedIn, Substack, Bluesky, Instagram, Twitter, Facebook, and others, professional production formats including Final Draft, Scrivener, InDesign, Adobe Audition, and Unreal Engine, and HTML output for AO3 and author websites.

Platforms whose primary product is their algorithm rather than their authors are outside DNW's scope. DNW publishes to platforms that serve readers and authors, not platforms that serve engagement metrics and advertising revenue.

---

## The Fault Line

The AI writing tool market has a fault line.

On one side: tools that want to write *for* you. Sudowrite, Jasper, and their equivalents sell the fantasy of the finished manuscript -- the destination without the journey. There is a market for this. It is not my market. This is the market that "wants to have written" without the work of writing. It is a market that is well-served by existing tools.

On the other side: authors who would be offended by the suggestion that an AI should write their book. These are people who have spent years developing a voice. The craft is the point. What they want isn't a ghostwriter -- it is a collaborator who handles the mechanical overhead that has nothing to do with craft. Did I use this word four times in the last chapter? Does this character's backstory contradict what I wrote three books ago? Is the timeline consistent? That's not writing, that's book keeping.

The same fault line runs through publishing platforms. AO3 is author-governed, non-profit, algorithm-light. Authors write what they want to write for readers who sought them out deliberately. The feedback loop is between author and reader. Wattpad is the opposite -- engagement metrics, algorithmic promotion, short chapters engineered for completion rates. The platform's incentives colonise the craft and you can plainly see it in the writing. There's some good writers on Wattpad, there's a lot more bad writers on Wattpad than there are good, and the platform's incentives are a big part of why.

DNW is built for the AO3 side of that fault line. Authors who write because they don't have a choice but to write. DNW is for the authors who want to write, not the authors who want to have written. Authors whose work is worth something to them, and hopefully their readers, and eventually to the publishers, studios, and streaming platforms who need it.

The "write for me" market is well-served. The "write with me" market is devoid of anything. DNW is the second half of that fault line. It is the only tool that exists to serve the authors who want to write, not the authors who want to have written.

## Proposed Solution

The mental model is pair programming, not GitHub Copilot or sudowrite. The author writes. DNW watches, learns, and assists: flagging a character whose eye colour changed between chapters, surfacing a word the author has used fourteen times in the last three scenes, noting that a location described in Act One contradicts the map established in Act Three. It handles the mechanical burden of consistency so the author can stay in the creative space.

DNW is built on three principles:

**1. The manuscript belongs to the author.** DNW never trains on manuscript content. We have no interest in what authors write. Their creative work is private and secure.

**2. The AI learns from the creative process, not the creative output.** As authors work, the system observes the *process* -- the decisions made, the revisions, the structural choices -- through  interaction signals for that specific author in that specific project, and importantly, that data never leaves that project workspace.

**3. No forced upgrades.** Each author's environment is version-pinned. A user on v4 stays on v4 until they choose to upgrade. There is no "we updated the platform and your workflow broke." Professional authors cannot afford that.

**4. No format lock-in.** DNW is designed to be interoperable with existing tools and formats. Authors can import and export their work in standard formats (e.g., FDX, DOCX, SCRIVX, TXT) without losing any of the intelligence or structure that DNW provides. All of the data in DNW is stored as plain-text files in version control, ensuring that authors can always access and manage their work outside of DNW.

---

## Market Overview

The addressable market for DNW spans several overlapping creative communities:

| Segment | Scale |
|---|---|
| Traditionally published authors (English-speaking markets) | ~300,000 |
| Active self-publishing authors (Amazon KDP) | 4M+ |
| Working screenwriters globally | ~150,000 |
| Fanfic and independent fiction communities (AO3 registered users alone) | 5M+ |
| Small studios, game narrative teams, podcast writers, interactive fiction authors | Hundreds of thousands |
| Enterprise: major studios, streaming platforms, publishers | 9,000+ organisations globally |

**Conservative TAM: $500M/yr** based on ~400 enterprise customers out of a potential 9,000+ (Disney, Warner Bros, Universal, Netflix, Amazon Studios, Apple TV+, Sony Pictures, and equivalents), plus the long tail of professional and semi-professional authors.

The individual author market is large, growing, and underserved by tools that take creative work seriously. The enterprise market is concentrated, high-value, and has no credible incumbent.

---

## Market Segmentation

DNW is offered across six tiers designed to serve the full spectrum from hobbyist to enterprise:

**Community Edition -- Free**
The entry point. Full access to all writing tools except those that must be gated due to legal requirements, e.g. likeness cloning. Community and Hobbyist tiers are the acquisition funnel and are never feature-constrained or monetised beyond their tier price. The goal is to be genuinely useful to every author regardless of means.

**Hobbyist Edition -- $12/yr**
Reserved editor with no queuing, more compute than Community. Priced at less than a large coffee and a fancy pastry at a major chain. This is a deliberate ceiling -- the Hobbyist tier will never cost more than that. Authors who love the product should be able to afford it without thinking about it.

**Professional Edition -- $200/yr**
Reserved editor, multiple edit windows, more compute, priority support. Latest features, and version pinning. For working authors who depend on the tool professionally.

**Small Studio Edition -- from $10,000/yr**
Up to 20 users, reserved editors, fleet-grade compute, priority support. For narrative teams, small publishers, game studios, podcast production companies.

**Enterprise Remote Edition -- from $200,000/yr**
Dedicated environment with dedicated compute. Full isolation from other tenants. For major studios and publishers who require contractual separation.

**Enterprise On-Premises Edition -- from $500,000/yr**
DNW deployed on the customer's own infrastructure. For organisations with data sovereignty requirements, existing compute investments, or security postures that preclude cloud SaaS.

---

## Market Differentiator

- **Purpose-built for narrative, not adapted from documents.** Every feature is designed around the specific cognitive demands of long-form fiction and screenplay.
- **Version-pinned environments.** No forced upgrades. A professional workflow established on v4 is not disrupted by a v20 release.
- **The manuscript stays private.** We never see it, never train on it, never store it beyond what's needed to serve the author's session. This is an architectural guarantee, not a policy.
- **"Write with me" not "write for me."** The AI surfaces, flags, and assists. It does not generate prose on the author's behalf unless explicitly asked. The author's voice remains the author's voice.
- **No incumbent.** There is no tool that does what DNW does at any price point. The enterprise market is currently served by a combination of Word, Scrivener, and custom internal tooling. The bar to step over is exceptionally low.

---

## The Knowledge File System

DNW introduces a structured narrative DSL -- a lightweight, human-readable language for describing fictional worlds. Authors write lore the way they think about it, not the way a database expects it:

```
:name Augustus
:aliases Augustus Hearthog, Sir Hearthog, Sir Augustus Hearthog, Charlie
:type Dog
:quirk officially knighted
:quirk loves sticks
:quirk navigates by the feel of long grass on his prodigious testicles
:continuity check Augustus' responses after day 66 whenever Beans leaves the scene
:description A long black dachshund, theoretically owned by Mayhew at Moreton,
             unfortunately attached to Beans & Emily.
```

Every `:aliases` entry is a recognised identity. Every `:continuity` is a standing instruction to the engine. Every `:ref` is a resolvable cross-reference. The lore isn't static documentation about the fictional world but actual executable specification.

### pip install for fictional universes

Knowledge files are versioned, URL-fetchable, and community-maintained:

```
:world Supernatural TV Show
:import https://raw.githubusercontent.com/SupernaturalFan/supernatural-tv-show/refs/heads/main/supernatural-tv-show-canon-lore.knowledge@v3.14
:import Supernatural TV Show Complete Episode Guide Summaries.knowledge@v4.99
:import Supernatural TV Show Complete Episode Scripts S1-S8.knowledge@v1.04
; grab the non-canon fanfic lore too, because the engine can distinguish canon from fanon
:import https://archiveofourown.org/works/000111488289111

```

`:import` is `pip install` for fictional universes. The community builds and maintains knowledge files the same way they maintain wikis -- because they already do that work. DNW gives it a useful format. Github repos emerge as a natural distribution mechanism. Unofficial registries follow. An ecosystem develops without DNW having to build or maintain it.

For authors working in licensed universes -- a writer hired to work on a Star Wars novel, a screenwriter joining an established TV show -- the IP holder maintains the canonical knowledge file. The writer imports it. Canon consistency is enforced automatically. Errors that would require a continuity editor catch get caught before the manuscript leaves the author's desk.

### The lore is a compiler target

When an author defines a new entity type, the LLM reads the lore and generates executable generation tables:

```
:lexicon ~velfire
:description Velfire are elvish vampires corrupted by magic. Only elves can
             become velfire. As all elves are French, all velfire are French.
:detail Hair colour varies, predominantly blonde, occasionally black
:detail Predominantly tall, 180cm to 220cm
:detail Given to hedonism
:detail Usually bored -- they've exhausted every known kink by their 200th year
:note No velfire can predate 514 AD
```

DNW compiles this to weighted generation tables with probability distributions, age formulae, and conditional logic. `CURRENT_YEAR - 514` is derived directly from the `:note`. The author wrote "predominantly blonde, occasionally black" and the engine resolved that to `0.85 / 0.15` as a probability. The JSON is always an artefact of the DSL, never the source of truth. The author wrangles the source of truth and the machine wrangles the compiled output.

### Inline queries

The backtick is an inline query to the knowledge engine without the author leaves their manuscript. The query is resolved immediately, and the answer is inserted inline. For example:

```
` does the impala have rope in the trunk?
```

Queries the Supernatural import. It's a canon answer, provided inline, and immediately available.

```
Ifor: "Vos estes un elfe de France? Que faites vos ci en Engleterre?"

` check ifor's french?
' Mostly correct. Vos and estes are spot-on -- 2nd person plural used as polite
  singular, the medieval equivalent of vous êtes. Que faites vos is the correct
  inverted question form. Ci is perfectly medieval. One note: un elfe uses the
  masculine article -- if Ifor can see she's a woman, une elfe might be more
  natural, unless he's classifying her as a category rather than gendering her.
  That ambiguity may be intentional.
```

Inline grammar check against medieval French. Returned as an annotation. The author accepts, dismisses, or acts on it without breaking the writing session.

```
Emily: "Sors immanis. Et inanis, rota tu volubilis, status malus, vana salus."
` what was the date carmina burana was released? It's pre 14th century, right?
' The Carmina Burana is a collection of medieval Latin poems, likely composed
  and circulated between the 11th and 13th centuries. Though the song and
  music we know today was set to music by Carl Orff in 1936, the original
  poems themselves are indeed pre-14th century.
```

Anachronism check, done inline, at the moment of doubt. Not after I finished the chapter and sent it to my editor. Not after the manuscript is locked and the continuity editor is checking for errors.

### The manuscript is the lore

The DSL is not limited to lore files. It runs through the manuscript itself:

```
:title And You May Ask Yourself
:location The Warm Wet Crotch Tavern
:status Draft
; hard-code references to Bill the donkey in this scene because he hasn't been named yet and the prose is a bit ambiguous
:ref Bill
```

Scene metadata, cast declaration, workflow state is inline and machine-readable and all part of the same language. It's not a Word document with some metadata bolted on. The manuscript is a structured document in a language designed for narrative. It's version-controlled, queryable, and compiles to every output format DNW supports.

One language. Lore files, character files, location files, event files, manuscripts. All the way down.

### The ecosystem play

The knowledge file format is an open specification. Anyone can author a knowledge file and anyone can host a registry. The community that already maintains Supernatural wikis, Star Wars canon databases, and Wookieepedia will maintain knowledge files -- because the format is designed to be written by the people who already enjoy doing this. And the format is designed to be useful to authors, not just fans, and is open and easily accessible.

For IP holders -- studios, publishers, game companies -- the official knowledge file is a new kind of asset. It is the canon, machine-readable and enforceable. Licensed authors import it and continuity gets maintained automatically across every writer working in the universe simultaneously. And the knowledge file outlasts any individual production.

The long-term value of a well-maintained knowledge file for a major IP, be it Marvel, Star Wars, or Tolkien -- is not small.

> Note: This repository is intentionally public to provide a high-level overview of the DNW SaaS, its offerings, and its potential market. The information contained herein is subject to change as development proceeds.



