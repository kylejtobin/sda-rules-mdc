# 🧠 Semantic Domain Architecture Rules (.mdc)

This repository defines a canonical ruleset for designing software systems with **Semantic Domain Architecture (SDA)**—a high-integrity architectural approach where your **domain models are the source of truth** for both computation and meaning.

If you’ve ever asked *“Why is all the business logic smeared across ten services?”*—this is your answer.

> **Semantic Domain Architecture** treats domain models not as dumb containers, but as intelligent agents of meaning—self-validating, self-explaining, and structurally aware.  
>  
> It combines the rigor of type systems with the elegance of declarative semantics, giving you a system where knowledge lives where it belongs.

---

## ✅ What This Is

This is a `.mdc` ruleset designed for use with [Cursor](https://www.cursor.sh), a development environment that can **enforce your architecture as you type**. It captures the refined lessons of years spent in both traditional application design and modern LLM-native systems.

It encodes rules for:

- Project structure and folder semantics
- Domain model integrity and composition
- Boundary enforcement between domain, infrastructure, and application layers
- Protocol- and Pydantic-first programming
- Immutable, declarative, testable domain logic

And it does it without dogma. These rules were battle-tested in systems where **agents reason, graphs retrieve, and business logic must never lie**.

---

## 🧩 Why It Works

Semantic Domain Architecture solves a few timeless problems:

- Business logic scattering → **Logic lives in the model**
- Testing complexity → **Models are testable in isolation**
- LLM integration overhead → **Structured output comes for free**
- Onboarding pain → **Read the model, understand the system**
- Drift across services → **One source of semantic truth**

It also makes your architecture **modular, explainable, and correct by construction**.

---

## 🧱 Core Ideas

Here are the foundations the `.mdc` enforces:

- **Value objects all the way down** – Use Pydantic models for every semantic unit (`Price`, `ChunkId`, `RiskScore`, etc).
- **Domain = Computation + Meaning** – Business logic belongs in computed fields on immutable models.
- **Protocols define boundaries** – Every integration is described via explicit contracts, not hard-wired classes.
- **No logic in services** – Application services orchestrate; they don’t calculate.
- **Infrastructure is plug-and-play** – Adapters live behind protocols and can be swapped without touching the domain.
- **Init files define public surface** – Every layer exports models, enums, and protocols clearly and cleanly.

If you want a codebase that explains itself, scales cleanly, and plays perfectly with agents or AI systems, these rules will feel like home.

---

## 🚀 Getting Started

To use these rules in your own repo:

1. Create a `.cursor/` directory in your project root
2. Copy `rules.mdc` into `.cursor/rules.mdc`
3. Use Cursor to develop against the spec—or just treat it as an executable architectural manifesto

You don’t need to adopt every rule at once. But when you’re ready to **stop writing services that don’t serve anyone**, start here.

---

## 🧠 Want to Know More?

The full technical philosophy is published in  
[**The Semantic Layer We Never Knew We Were Building**](the_semantic_layer.md)

It walks through:
- The origins of SDA in cloud-native and agent systems
- Why service-oriented thinking falls short for knowledge-dense domains
- How Pydantic, protocols, and immutable models can redefine your architecture
- How AI systems benefit from semantic precision you were already going to need

---

## 🛠 Maintainer

Kyle ([@kylejtobin](https://github.com/kylejtobin)) – CTO, semantic application architect, sysadmin.  
Former barbecue overlord. Occasional woodworker.

---

## 📜 License

MIT – but don’t just copy this. Understand it. Live it. Then write better software.
