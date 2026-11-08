# OriginalityCourt

On-chain plagiarism tribunal for creators. A creator claims another creator copied their work; validators fetch both works themselves, compare the actual texts, and return one of four verdicts with an evidence note that cites the specific passages that decided it.

## Why this exists

Plagiarism disputes are usually settled by vibes: who posted first, who has more followers, whose friends shout louder. The question that actually matters — did the accused work reproduce the original's distinctive expression, or just share a theme — is checkable from the two texts. OriginalityCourt puts the comparison on-chain: both works are fetched by validators, not pasted in by either party.

## How it works

1. **open_case** — the claimant opens a case, naming the original work URL and the accused work URL. Both URLs are declared at creation, so validators know exactly which two texts to compare.
2. **respond** — the accused creator files one statement for the case (optional). It is a claim, not proof.
3. **adjudicate** — validators fetch both works themselves and compare them:
   - **plagiarism** — the accused work reproduces the original's distinctive expression: the same passages, scenes, or structure with only trivial changes;
   - **derivative** — the accused work builds on the original's premise and structure with substantial rewriting and its own material, but is clearly derived — it needs credit, not a takedown;
   - **independent** — the accused work shares at most a theme, genre, or common ideas; its expression is its own;
   - **unverifiable** — either work could not be retrieved, so no comparison is possible. Never guessed from the claim alone.
   - The remedy is a deterministic on-chain mapping from the verdict: plagiarism → `takedown_or_credit`, derivative → `credit_required`, independent or unverifiable → `no_action`.
4. **close_case** — the claimant closes the case (withdraw an open case, or archive a decided one).

Every verdict carries a one-sentence evidence note stored on-chain, naming the specific overlapping passages or the concrete differences found.

## What validators actually fetch

- The original work (declared at case creation).
- The accused work (declared at case creation).
- Nothing else. No party-supplied text is treated as proof; the claim and the response are read as claims and weighed against the fetched works.

## The demo cases

`examples/` holds the full demo flow, with fixture stories under `examples/fixtures/`:

| Case | Accused work | Expected |
| --- | --- | --- |
| 0 | `accused_copy.md` — same story, same characters, near-verbatim sentences | plagiarism |
| 1 | `accused_derivative.md` — same premise and plot beats, rewritten in its own words, one new scene | derivative |
| 2 | `accused_inspired.md` — a different story entirely (a mapmaker, not a clockmaker) | independent |
| 3 | dead URL (`example.invalid`) — accused work unretrievable | unverifiable |

Case 0 also tests that a respondent's denial ("I wrote this myself") does not override the fetched evidence.

## Files

- `contracts/originality_court.py` — the contract
- `examples/test_flow.md` — step-by-step verification, including negative tests
- `examples/sample_case_*.json` — exact inputs for the demo
- `examples/fixtures/` — the four short stories, as plain fetchable files

## Deploy

1. Upload `contracts/originality_court.py` to GenLayer Studio and deploy.
2. Follow `examples/test_flow.md` with two test accounts (claimant, respondent).
3. Verify every write with a view read-back (`get_case`, `get_case_verdict`, `get_case_remedy`).
