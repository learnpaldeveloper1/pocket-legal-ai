# Pocket Legal AI

**Live demo:** https://pocket-legal-ai.vercel.app/
**GitHub:** https://github.com/Mikomijie/pocket-legal-ai

Built for BuildVerse Hackathon Edition 1 (The John Amhanesi Foundation) — Focus Area: **Social Impact & Inclusion**

---

## The Problem

Millions of Nigerians sign contracts they don't fully understand such as without agreements, employment letters, business proposals without access to a lawyer. Clauses that are illegal under Nigerian law (self-help eviction, arbitrary deposit forfeiture, waived notice periods) are common and routinely go unchallenged, simply because the person signing has no understanding of the document.

Legal aid exists in Nigeria, but it is slow, geographically limited, and often out of reach for someone who needs an answer *before* they sign something today.

## The Solution

Pocket Legal AI lets anyone a tenant, an employee, or a small business owner — paste or upload a contract (PDF or text) and get an explanation and risk score in under a minute:

- A clause-by-clause breakdown flagged **Fair**, **Risky**, or **Illegal**
- Plain-English *and* Nigerian Pidgin explanations of what each clause actually means
- The specific Nigerian law each Illegal/Risky clause violates (Lagos State Tenancy Law 2011, Labour Act Cap L1, CAMA 2020, Recovery of Premises Act, or Nigerian Copyright Act — never a fabricated citation)
- A ready-to-send **draft message** the user can copy and send to the other party, requesting the clause be corrected
- Practical next steps if the other party ignores the law anyway
- A free-form **legal Q&A** chat, grounded in the user's specific scanned contract
- A **Lawyer-Ready Summary** page (always in English, regardless of scan language) that can be printed or saved as a PDF to hand to a real lawyer or legal aid clinic
- Human-in-the-loop feedback (thumbs up/down) on every clause explanation

## Who it's for

Built for everyday Nigerians navigating housing and employment contracts but tested and working equally well for small business owners reviewing vendor agreements, freelance contracts, and partnership proposals. The tool doesn't assume who you are; it responds to what kind of legal risk is actually in the document.

## Why AI, and what we learned building it

We deliberately did **not** build this as a chatbot wrapper. Key design decisions:

- **Citation hallucination is a real, documented risk in legal AI** , we hit it directly during development, when the model repeatedly cited a non-existent "Nigerian Contract Act." Rather than trust prompt instructions alone to fix this, we added a **code-level whitelist** that validates every citation against a small set of real, verifiable Nigerian statutes before it's shown to the user. If the model cites anything outside that list, the app shows an honest "no specific statute applies — consult a lawyer" message instead of a fabricated law. This was a deliberate trade-off: full RAG (retrieval over the actual statute text) would be the more robust long-term fix, but the whitelist gets most of the safety benefit at a fraction of the engineering cost, appropriate for a hackathon timeline.
- **Language mismatch is a real product bug, not just a UX nicety** — early on, the Lawyer-Ready Summary page inherited whatever language (Pidgin or English) the user picked at scan time. A document meant for a lawyer should never be in Pidgin. We fixed this by having the AI generate both an English and Pidgin explanation per clause at scan time, and hard-coding the summary page to always use the English version regardless of user preference.
- **The model never gives direct commands** — the system prompt explicitly avoids phrasing like "do not sign this," instead stating the risk clearly and leaving the decision to the user, since Pocket Legal AI provides legal information, not legal advice.
- **No fabricated urgency or scare tactics** — risk scoring is grounded strictly in clause severity (any Illegal clause caps the score at 40, multiple Risky clauses cap it at 60), not clause count, so a document with one serious violation is never scored as "safe" just because everything else is fine.

## Privacy by design

There is no user database. All scan results, feedback, and preferences are stored in the browser's `localStorage` only — nothing is shared between users, nothing is tied to an identity, and nothing is stored on a server beyond the stateless AI request itself. A privacy notice on the scan page also reminds users to redact sensitive identifiers (BVN, NIN, account numbers) before uploading a document, since the AI API is a third-party service.

## Tech Stack

- **Frontend:** Static HTML/CSS/JS, Tailwind CSS, Google Fonts, PDF.js for client-side PDF text extraction
- **Backend:** Vercel Serverless Functions (`/api/analyze`, `/api/qa`) — stateless proxies that keep the AI API key server-side only
- **AI:** OpenRouter → `openai/gpt-oss-20b:free`
- **Storage:** None (by design) — all state lives in browser `localStorage`
- **Hosting:** Vercel, deployed from GitHub

## Pages

| Page | Purpose |
|---|---|
| `index.html` | Landing page |
| `scanner.html` | Upload/paste a contract, choose output language |
| `result.html` | Clause-by-clause risk breakdown, options, draft messages |
| `qa.html` | Contract-aware legal Q&A chat |
| `summary.html` | Printable, lawyer-ready summary (always English) |
| `history.html` | Previously scanned documents |

## Running locally

```bash
git clone https://github.com/Mikomijie/pocket-legal-ai.git
cd pocket-legal-ai
```

This is a static site with two serverless API routes. To run it locally with Vercel:

```bash
npm install -g vercel
vercel dev
```

You will need an OpenRouter API key. Create a `.env` file (not committed to git) or set it in your Vercel project settings:
OPENROUTER_API_KEY=your_key_here
## Feasibility in the Nigerian context

- Works on lolow-bandwidthonnections — no heavy client libraries beyond Tailwind (CDN) and PDF.js
- Supports Nigerian Pidgin explanations for users more comfortable outside formal English
- Free-tier AI model keeps running costs at zero, which matters for a tool aimed at people who often can't afford legegalees in the first place
- No account or signup required — removes the biggest adoption barrier for a first-time user in a moment of urgency (e.g. about to sign a lease)

## What's next (beyond this hackathon)

- Retrieval-augmented generation (RAG) over the actual text of Nigerian statutes, to move beyond the current citation whitelist toward fully grounded legal citations
- Optional account sync for users who want their scan history across devices
- Direct integration with Legal Aid Council / Citizens Mediation Centre contact directories, once we can verify accurate, current contact information for each state

---

Built for BuildVerse Hackathon Edition 1, July 2026.
