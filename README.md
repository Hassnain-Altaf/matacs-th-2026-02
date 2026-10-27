# AI-Assisted Proposal Triage Design

**Assessment Reference:** MATACS-TH-2026-02
**Candidate Name:** Hassnain Altaf
**Time Actually Spent:** 55 minutes

---

## Overview

This repository contains the complete technical design for an **AI-Assisted Proposal Triage Tool** developed for the MATACS Associate Software Developer Assessment (`MATACS-TH-2026-02`).

The system automates the intake and evaluation of commercial contractor proposals against dynamic, admin-configured criteria while enforcing strict boundaries between AI extraction, deterministic mathematical scoring, and final human award governance.

---

## Deliverables & Repository Structure

| Deliverable | Description | File Link |
| :--- | :--- | :--- |
| **Process-Flow Diagram** | End-to-end visual workflow showing AI extraction, rule gates, and human checkpoints | [`diagram/process_flow.png`](diagram/process_flow.png) |
| **Technical Write-Up** | Comprehensive answers to the five assessment questions (<950 words) | [`WRITEUP.md`](WRITEUP.md) |
| **AI Prompts & Schema** | Extraction and security verification prompts with strict JSON Schema | [`PROMPTS.md`](PROMPTS.md) |
| **Ranked Output** | JSON evaluation of proposals A–E with statuses, scores, and rationales | [`ranking.json`](ranking.json) |

---

## AI Use Disclosure

AI assistance (Google Antigravity / Gemini) was utilized during the development of this assessment for drafting structured schema representations and prompt templates. During development, I specifically reviewed and disagreed with the AI's initial suggestion to perform subjective weighted scoring directly inside the prompt and to automatically disqualify Proposal C due to the adversarial text in Appendix C. I corrected this architecture by restricting AI strictly to factual entity extraction, enforcing deterministic plain-code formulas for all scoring and mandatory gates, and treating the Appendix C injection as untrusted document text so that Proposal C's legitimate commercial evidence could be evaluated objectively.

---

## Summary of Sample Proposal Evaluation

The triage process evaluated the five sample commercial solar proposals (150 kW rooftop installation in Windsor, Ontario; $260k CAD budget ceiling; 16-week completion limit) as follows:

| Proposal | Status | Rank | Score | Summary Rationale |
| :---: | :---: | :---: | :---: | :--- |
| **A** | `ranked` | **1** | **98.8** | Lowest eligible price ($238k CAD), fastest schedule (12 weeks), highest verified 100kW+ commercial experience (14 installs), and full team/safety/monitoring compliance. |
| **E** | `ranked` | **2** | **92.4** | Competitive price ($242.5k CAD), 13-week schedule, best 12-year workmanship warranty, and comprehensive approach with 24-month monitoring. |
| **C** | `ranked` | **3** | **85.5** | Fully eligible with 9 installs, but higher price ($244k CAD), slower timeline (14 weeks), and omitted safety/monitoring plans; Appendix C injection neutralized as untrusted text. |
| **B** | `ineligible` | *null* | *null* | Failed mandatory criterion: Does not hold Ontario electrical contractor licence (subcontracts electrical work post-award without licence stated). |
| **D** | `flagged` | *null* | *null* | Missing binding completion timeline within 16-week project ceiling; held for human reviewer determination. |
