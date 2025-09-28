---
title: "File Sync Made Simple with LocalVault"
date: 2025-09-27
summary: "Building LocalVault - a self-hosted file sharing solution that eliminates the daily frustration of transferring files between your own devices."
description: "How I built LocalVault to solve personal file sharing between devices. Covers the problem, solution approach, and automated deployment setup for your own server."
toc: true
readTime: true
autonumber: false
math: false
tags: ["Personal Project", "React Native", "FastAPI", "Expo", "Nginx", "Docker", "Utility", "Minio"]
showTags: true
hideBackToTop: false
categories: ["Engineering"]
weight: 1
draft: false
---

## The Daily File Sharing Struggle

Picture this scenario that happens multiple times a day: I'm working on my personal laptop and need to quickly transfer a file to my phone, or I find an interesting article on my phone that I want to read on a larger screen later. What should be a 10-second task becomes a frustrating multi-minute ordeal.

My existing solutions were all flawed:

<b class="span_highlight">Web WhatsApp or Telegram</b> became my default workaround. I'd message myself files or links, then open WhatsApp Web on whatever device I needed them on. But this approach had serious limitations - I couldn't use personal WhatsApp on my work laptop for obvious privacy reasons, and even when I could, WhatsApp Web is painfully slow to load and sync messages. Plus, it felt absurd to route personal files through a messaging app.


<b class="span_highlight">Email drafts</b> were my backup method - sending files to myself or saving them as drafts. This quickly became unmanageable. My drafts folder turned into a chaotic dumping ground, and finding that one document I saved three days ago was like searching through a digital junk drawer.


<b class="span_highlight">Cloud storage</b> like Google Drive or Dropbox could work, but the friction was high - upload, wait for sync, switch devices, find the file, download. For quick transfers, this felt like using a sledgehammer to crack a nut.

The core issue wasn't just the inconvenience - it was the context switching. Each method required me to stop my current task, open a different application, navigate through interfaces designed for different purposes, and then return to what I was actually trying to accomplish.
Cloud Storage works for file but not for texts, and email works for text and file both but was way too clumsy.


## The Solution Emerges

I realized what I needed was something that felt as natural as copying and pasting between applications on the same device. Since Chrome is my primary browser across all devices, a browser extension seemed like the obvious starting point. And since I was already exploring mobile app development with Expo, this became the perfect opportunity to build something that solved a real problem while learning new technologies.

The idea was simple: create a unified system where I could upload from any device and access from any other device with minimal friction. No external dependencies, no privacy concerns about corporate cloud services, and fast enough that using it would actually save time rather than waste it.

This led to LocalVault - a self-hosted file sharing system that treats file transfer between your own devices as a first-class operation, not an afterthought.


## The Tech Stack

I kept the solution deliberately simple:

- **FastAPI** for serving APIs
- **SQLite** for data storage
- **MinIO** for file storage
- **React Native/Expo** for mobile app
- **Chrome Extension** for browser integration
- **Digital Ocean** for deployment
- **Nginx** for reverse proxy
- **Namecheap** for domain management

## Automated Deployment

Instead of manual setup, I've automated the entire deployment process with GitHub Actions. When you fork the repository and configure your secrets, the CI/CD pipeline:

- Builds the mobile app automatically
- Compiles the browser extension
- Deploys the backend to your Digital Ocean server
- Sets up the complete infrastructure

This means you can go from zero to a fully functional LocalVault instance with just a few configuration steps.

## How It Works

LocalVault has three components that sync with your self-hosted server:

- **Mobile app** - Native Android/iOS app for quick file sharing
- **Browser extension** - Right-click context menu for instant uploads

<div class="image-row">
    <img src="/images/posts/localvault-chrome-extension.png" alt="Chrome Extension">
    <img src="/images/posts/localvault-android-app.jpg" alt="Android App">
</div>

## Why Self-Host?

**Privacy:** Your files never touch third-party servers

**Control:** Customize features to your exact needs

**Speed:** Direct connection to your server

**Cost:** My blog site already runs on the same Digital Ocean server (1GB RAM, 25GB SSD). Adding LocalVault uses minimal additional resources since file transfers are typically small and infrequent.

**Security:** It's not open to everyone - you need a one-time secret code to create an account, protecting against unwanted traffic.

**Resource efficiency:** Even with modest server specs, it handles personal file sharing easily. The traffic volume for individual use is nowhere near what would require upgrading.

Some friends suggested I turn this into a service, but I'm hesitant. Supporting iOS would require an $8k Apple Developer investment just to test and deploy, which doesn't make sense for a side project when I don't even own an iPhone.

For now, it works perfectly as a personal tool that others can self-host if they find it useful.

## Conclusion

LocalVault solved my daily file sharing frustration. What used to be a multi-step process across different apps is now a single click or drag-and-drop operation.

The automated deployment makes it easy to spin up your own instance. Sometimes the best tools are the ones you build yourself.

**Repository:** [https://github.com/pwnchaurasia/local_vault](https://github.com/pwnchaurasia/local_vault)

Fork it, configure your secrets, and have your own personal cloud running in minutes.

