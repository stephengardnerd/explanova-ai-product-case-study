# 01 — Ideation in Google Stitch

> Why I started in **Google Stitch**, what nine screens I shipped from it, and what the UI-first approach unlocked downstream.

← [Back to README](../README.md)

---

## The decision: UI-first, not code-first

A lot of solo founders build the wrong thing for six months because they wrote the code before they pressure-tested the experience. I wanted to skip that mistake.

So **before any code**, I prototyped the entire surface area of Explanova in Google Stitch — Google's AI-assisted UI design tool. The goal was a single forcing question: **could a parent-avatar tutor feel warm, simple, and trustworthy enough for a kitchen-table moment?**

If the answer in the mockups was "no," there was nothing to build.

## What I shipped from Stitch

Nine screens covering the entire end-to-end journey. Each one was a real product decision, not a layout exercise:

| Screen | Decision it forced |
|---|---|
| **Landing page** | What's the one-sentence promise to a parent? |
| **Pricing (with community discounts)** | Subscription model + community-pricing posture (military, teacher, low-income) before any payment integration |
| **Parent dashboard** | What does the parent see *first* every day? Children, lessons, archive — in that order. |
| **Avatar setup intro** | How do you onboard someone to the idea that their face is about to teach their child? |
| **Avatar recording process** | How granular is the recording flow? Few minutes, no studio gear, child-supervised. |
| **Accessible homework upload** | Photo + typed text + accessibility-first input |
| **Processing lesson** | What does the parent see while the AI pipeline runs? Confidence-building, not a blank spinner. |
| **Avatar whiteboard player** | The core delivery surface — picture-in-picture parent + full-screen whiteboard |
| **Learning archive** | The receipt — every lesson reviewable later, by parent or child |

## What this approach unlocked

By the time I opened a code editor:

- **Subscription tiering was already designed** — no mid-build pricing flip-flop
- **Child-safety / consent UX was already in the journey** — not bolted on later for App Store review
- **The whiteboard surface had a hard contract** — parent in PiP, math fills the rest of the screen
- **The "warm parent voice" tone** was visible in the mockups, which gave the avatar-script LLM prompt a concrete reference point

## The throughline to today

The Stitch screens are not 1:1 with the production app — production iterates on every one of them — but the **journey** they describe (landing → pricing → dashboard → avatar → upload → processing → whiteboard → archive) is exactly the journey the live app delivers today.

That's the dividend of pressure-testing the experience before writing the code.

→ Next: [02 — AI prototyping in Google AI Studio](02-ai-prototyping-studio.md)
