---
title: "Postcard: Sharing My Notes Only With the People I Choose"
date: 2026-10-10
summary: "Building Postcard - a small command line tool that publishes my Obsidian notes to my own server as private links, shared only with the people I choose."
description: "How I built Postcard, a small tool that publishes my Obsidian notes to my own server as private links I can share with the people I choose, and what building it with AI taught me about the engineer's role."
toc: true
readTime: true
autonumber: false
math: false
tags: ["Personal Project", "Obsidian", "Python", "Nginx", "Self-Hosting", "AI", "Utility"]
showTags: true
hideBackToTop: false
categories: ["Engineering"]
weight: 1
draft: false
---

Markdown is my favourite markup language. The skill carries over everywhere: Notion, Obsidian, Confluence, Azure Boards. Add Mermaid for flow diagrams and it's hard to beat.

In 2024 I discovered Notion and started using it. It felt good and it has a lot of great features, but there were some things I wanted to keep on my own machine.

The feature I liked most was how easily I could publish a note and share it with exactly the people I wanted to.

## Why I wanted this

I write small micro blogs, mostly about the places I travel. Nothing fancy, just simple notes: what I did each day, what I ate and where, what I saw, and how much I spent. No long text.

I don't want to share these with everyone. But people do ask me for my itinerary. And now that I'm obsessed with Obsidian, all of these notes live there in Markdown. The same goes for my personal learning notes.

So I was stuck. Either I copy and paste everything into Notion just to get a shareable link, or I send the file over email. And if I email it, the moment I edit the note, the other person no longer has the latest version.

## What you need to put something on the internet

Writing code has become a lot easier these days, and I already had most of what it takes to make something available online:

1. A domain name. I have one.
2. A server. I have one.
3. A web server running on it.
4. A way to push things from my machine to that server.

I already write public posts on `pwnchaurasia.dev`, and I have a DigitalOcean server that costs me about 600 INR a month.

## Planning it

Next I had to work out what I actually needed. I prefer the terminal and the keyboard, so I wanted a command line tool.

I always SSH into my server myself to set up nginx and anything else it needs. My priority is keeping the monthly bill low, and my blog gets built and deployed through GitHub Actions.

For these micro blogs and notes, though, I wanted publishing to be as simple as running one command.

What I needed was an engine that renders my Markdown exactly the way I see it in Obsidian, with the same setup and the same theme I use (Dark Moss). Build everything locally, then push it to the server.

I added a subdomain in Namecheap and moved on to building.

## Building it with Claude

I described what I needed to Claude and asked what the different ways of doing it were. After checking the options, I got it built.

Then I realised I'd also want to push a whole folder, with every note inside it. Once we had finalised how that should work, Claude built that too.

After that I made a few more improvements, and one of them is important to point out.

Claude didn't think of adding pages for 404s or other errors. When I opened `postcard.pwnchaurasia.dev`, it showed the default nginx page. There was no index page either.

That tells you something: AI does what it's asked to do. That's why engineers and thinkers still hold their place. Not just as coders, but as the people who think through the system and design it. That part matters more than ever.

## What it does today

These are the commands Postcard currently supports:

```text
publish    Publish a note or a whole folder, or update it if it's already live.
preview    Render a note or folder exactly as it would be published, and open it locally. Nothing is uploaded.
unpublish  Take a page down. Its link stops working for everyone.
rotate     Give a live page or folder a new link. The old link dies immediately.
list       Everything you've published (read from the server, so it works from any machine).
status     Is this note or folder live, and has it changed since it was published?
open       Open a note's or folder's live page in the browser.
init       Create .postcard/config.toml in your vault and install the Obsidian plugin.
doctor     Check vault, plugin, SSH, server, DNS and HTTPS.
version    Show the version.
themes     Obsidian themes to publish with.
plugin     The Postcard Bridge plugin inside Obsidian.
server     Server setup helpers.
```

## See it for yourself

Here is a sample I published with Postcard, a small folder of notes rendered just like my Obsidian:

[postcard.pwnchaurasia.dev/uKGcSNxMxm2e/00-overview](https://postcard.pwnchaurasia.dev/uKGcSNxMxm2e/00-overview)

If you want to use it yourself, the code and setup steps are on GitHub: [github.com/pwnchaurasia/postcard](https://github.com/pwnchaurasia/postcard).
