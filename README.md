# AI Engineering 2026

**11 weeks · 22 curriculum days · ~77 real days · 2–3 hrs/day**

Built by Girish Watwani — documenting the journey from Senior Software Engineer to Senior AI/GenAI Engineer.

---

## Goals

### 1. Master the Core AI Engineering Stack
Build production-grade skills across the full AI stack: LLMs, embeddings, vector databases, RAG pipelines, agentic systems, and LLMOps. Go beyond tutorials — every week ships a real project.

### 2. Build a Public Portfolio of 10 AI Projects
Each project solves a real problem (ENBD wealth domain, PropAgent real estate, general finance). By Week 11, 10 repos are pinned on GitHub — each with a clear problem → solution → impact story, architecture diagrams, and measured eval scores.

Key projects:
- **ENBD Wealth FAQ Semantic Search** — embeddings + Qdrant + FastAPI (Week 2)
- **RAG chatbot over mutual fund PDFs** — LangChain + RAGAS eval ≥ 0.75 (Week 2)
- **ReAct wealth agent** — multi-step tool calling (Week 3)
- **LangGraph KYC Onboarding Agent** — stateful, persistent, HITL (Week 3)
- **Deployed RAG on AWS App Runner** — LangSmith tracing, live HTTPS URL (Week 4)
- **ENBD MCP Server** — 3 tools, connected to Claude Desktop (Week 5)
- **Multi-agent stock research system** — 3 parallel agents, < 30s end-to-end (Week 6)
- **Privacy-first local chatbot** — Phi-3 mini + Ollama + Qdrant, zero cloud calls (Week 7)
- **PropAgent Voice Bot** — real-time STT → LLM → TTS, < 800ms (Week 8)
- **ENBD Wealth AI Agent (Capstone)** — multimodal RAG + LangGraph + MCP + voice (Week 11)

### 3. Land a Senior GenAI Engineer Role
Target roles at **Delphi** and **Cleveland Clinic Abu Dhabi** (and 3+ more UAE Senior GenAI Engineer positions). Week 22 is dedicated interview prep: system design walkthroughs, STAR stories, Loom demo, and live mock interviews.

---

## Study Plan

| Week | Theme | Key Skills |
|------|-------|------------|
| 1 | Foundations — LLMs & the AI Stack | Python async, FastAPI, OpenAI API |
| 2 | LLMs, Embeddings & VectorDBs | Qdrant, RAG, LangChain, RAGAS |
| 3 | Agentic AI Foundations | ReAct, tool calling, LangGraph |
| 4 | LangSmith, AWS & Evals in Production | Tracing, DeepEval, Docker, AWS |
| 5 | MCP & Context Engineering | FastMCP, prompt caching, semantic cache |
| 6 | Multi-Agent Systems & Multimodal AI | Supervisor pattern, CLIP, ColPali |
| 7 | Text-2-SQL & Local AI | SQL agents, Ollama, QLoRA |
| 8 | Deep Work + Voice AI | Deepgram, ElevenLabs, LiveKit |
| 9 | Advanced AI Engineering & Fine-Tuning | Model routing, LoRA, synthetic data |
| 10 | RL + AI Security | GRPO, red-teaming, OWASP LLM Top 10 |
| 11 | Capstone + Interview Showcase | Full-stack deploy, mock interviews |

Full interactive study plan: [ai_engineering_study_plan.html](ai_engineering_study_plan.html)

---

## Daily Notes

Study notes are in the [`notes/`](notes/) folder, one file per study session.

| Date | Topics |
|------|--------|
| [2026-05-05](notes/2026-05-05.md) | Jupyter notebooks, venv, requests/httpx, pandas (Series, DataFrames, filtering, groupby, time series) |
| [2026-05-06](notes/2026-05-06.md) | LLM foundations, transformers, tokens/context/cost, LLM APIs, pretraining datasets, FineWeb, CommonCrawl, BPE tokenization, gradient descent, SFT, RLHF, RLHF vs RL |

---

## Stack

- **LLMs:** OpenAI GPT-4o, Anthropic Claude 3.5 Sonnet, Phi-3 mini (local)
- **Frameworks:** LangChain, LangGraph, LangSmith, FastMCP
- **Vector DB:** Qdrant
- **Evals:** RAGAS, DeepEval
- **Deployment:** AWS App Runner, Docker
- **Local inference:** Ollama, vLLM
- **Voice:** Deepgram STT, ElevenLabs TTS, LiveKit

---

*Progress saved in the browser via the interactive HTML plan. Start → Ship.*
