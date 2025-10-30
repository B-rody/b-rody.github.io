---
layout: single
author_profile: true
sidebar:
  nav: "projects"
permalink: /projects/assetforge/
title: AssetForge
classes: wide
---

## Summary
AssetForge is a desktop application that automates the creation of complete digital product bundles for online marketplaces. Instead of manually researching niches, designing assets, and formatting metadata for platforms like Etsy or Gumroad, users click a button and get a production-ready folder containing assets, listing copy, pricing strategy, and a QA summary.  

I built it as a hands-on way to learn more about AI SDKs and agent orchestration in the context of end-to-end automation.

## Role & Scope
I led the entire design and implementation of the system, from concept to deployment. Throughout development, I used GitHub Copilot to explore design tradeoffs and accelerate implementation, but every architectural decision, schema design, and model configuration was my own.

## System Overview
The app runs as a Tauri desktop client (Rust + React + TypeScript) with a Python backend compiled to a native binary using Nuitka. The frontend communicates with the backend over stdio using JSON-Lines, which keeps the architecture simple and avoids the complexity of managing a local HTTP server or port conflicts. The Python backend orchestrates a four-stage pipeline using OpenAI's API, with each agent handling a distinct phase of bundle creation:

- **Researcher**: Analyzes marketplace trends and identifies profitable digital product niches
- **Planner**: Designs the bundle structure, including asset types, pricing strategy, and target personas
- **Maker**: Generates the actual digital assets (templates, guides, etc.) and writes marketplace metadata
- **Packager**: Organizes everything into a final output folder with store-ready listings and file structure

Each stage writes structured outputs and checkpoints to an SQLite database. The user's API key is encrypted with Fernet (AES-128) and stored locally — no keys or data ever leave the machine except for OpenAI API calls. The build process compiles Python to C using Nuitka, making the final binary harder to reverse-engineer.

Here's a high level view of the system architecture. (Techincally there's more happening under the hood, such as long handling, event streaming, retries and schema validation, but this captures the core flow)

![System architecture diagram](/assets/images/assetforge-architecture.png)

## Key Challenges
- **Error recovery:** API calls can fail mid-pipeline due to rate limits or malformed responses. I implemented checkpointing so each stage writes outputs before advancing, allowing retries without restarting earlier steps.  
- **Cross-process communication:** Initially considered HTTP, but managing ports and firewalls was brittle. Given the local nature, switching to stdio with JSON-Lines simplified communication and removed external dependencies.  
- **Structured output handling:** OpenAI responses often included citation artifacts or inconsistent formatting. I added recursive cleanup and schema validation to ensure every output matched strict downstream expectations.

## Outcomes
The final build produces a single executable that runs fully offline after setup.  
Each pipeline run generates a complete bundle folder in ~20 minutes, containing all assets, metadata, and documentation ready for upload. I could have made it faster, but this model confgiuration provided the most reliable and highest quality results. Plus, reducing draft time for a digital asset bundle from multiple hours to 20 minutes felt like a pretty good improvement to me.

Building AssetForge taught me a lot about multi-agent orchestration, resilient retry logic for LLM APIs, AI SDKs, configuring tool calls, ephemeral containers and file creation and management.

## What This Project Highlights
- Multi-agent pipeline design (structured outputs, schema validation, streaming)  
- Python backend architecture (async orchestration, persistence, reliability)  
- Rust/Tauri desktop packaging with Nuitka-compiled Python  
- Secure local credential storage and offline-first encryption  
- React + TypeScript UI with real-time IPC streaming  
- Data modeling lifecycle (idea → bundle → packaged artifact)