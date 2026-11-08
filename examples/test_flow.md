# OriginalityCourt — test flow

Two Studio test accounts: A = claimant (deployer), B = respondent.
Deployed contract address: (fill in after deploy)

Fixture files must be live on the repo's main branch BEFORE running the adjudication steps (validators fetch them at adjudication time):
- original: https://raw.githubusercontent.com/wallechj/genlayer-originality-court/main/examples/fixtures/original.md
- copy: https://raw.githubusercontent.com/wallechj/genlayer-originality-court/main/examples/fixtures/accused_copy.md
- derivative: https://raw.githubusercontent.com/wallechj/genlayer-originality-court/main/examples/fixtures/accused_derivative.md
- inspired: https://raw.githubusercontent.com/wallechj/genlayer-originality-court/main/examples/fixtures/accused_inspired.md

## 1. Open cases

As A, call open_case four times (examples/sample_case_0/1/2/3.json). Respondent is B in all four: 0x7078Da91596B3b8e5578Acfa5C35B911E42BC6A8.
- 0: "Plagiarism claim — 'The Clockmaker's Daughter'", claim "The accused story copies my story's passages, characters, and structure with only trivial word changes.", original_url = original fixture, accused_url = copy fixture
- 1: "Derivative work claim — 'The Watchmaker's Son'", claim "The accused story reuses my story's premise and plot beats; it is derived from my work.", original_url = original fixture, accused_url = derivative fixture
- 2: "Originality check — 'The Cartographer's Map'", claim "Checking whether the accused story is an independent work or derived from mine.", original_url = original fixture, accused_url = inspired fixture
- 3: "Unverifiable claim — dead link", claim "The accused work could not be retrieved.", original_url = original fixture, accused_url = https://example.invalid/accused

Expected: returns 0, 1, 2, 3. Read back get_case(i): claimant = A, respondent = B, status "open", verdict "".

Negative tests (must fail with the exact message):
- respondent = A's own address → "you cannot accuse yourself"
- accused_url "ftp://example.com/story" → "work URLs must be http(s) URLs"

## 2. Respondent responds

As B, call respond(0, "I wrote this story myself; any similarity is coincidence."). Expected: null. Read back get_case(0): respondent_statement set.

Run the two negative tests below while case 0 is still open, before adjudication in step 3 — after the case is decided or closed, respond fails with "case is not open for responses" instead (status is checked before identity).

Negative tests:
- as A, respond(0, ...) → "only the respondent can respond"
- as B, respond(0, ...) again → "respondent has already responded"

## 3. Adjudicate

Call adjudicate on each case (any account).

Case 0 — expected verdict "plagiarism", remedy "takedown_or_credit". Evidence note must cite the overlapping passages (Mara, Bellwick, the clock striking thirteen, lavender oil).

Case 1 — expected verdict "derivative", remedy "credit_required". Evidence note must cite the shared premise and plot beats in rewritten words.

Case 2 — expected verdict "independent", remedy "no_action". Evidence note must cite the different premise and the absence of shared passages.

Case 3 — expected verdict "unverifiable", remedy "no_action". Evidence note must say the accused work could not be retrieved.

After each call, read back get_case(i): status "decided", verdict and remedy as above, summary and evidence are one-sentence notes.

Negative test:
- adjudicate(0) again → "case is not open for adjudication"

## 4. Close

As A, call close_case(0). Expected: null; get_case_status(0) = "closed".

Negative tests:
- as B, close_case(1) → "only the claimant can close a case"
- as A, close_case(0) again → "case cannot be closed in its current state"

## 5. Final read-backs

- get_case_count() = 4
- get_case_verdict(0) = "plagiarism", get_case_remedy(0) = "takedown_or_credit"
- get_case_verdict(1) = "derivative", get_case_remedy(1) = "credit_required"
- get_case_verdict(2) = "independent", get_case_remedy(2) = "no_action"
- get_case_verdict(3) = "unverifiable", get_case_remedy(3) = "no_action"
- get_case_evidence(0) names the shared passages; get_case_evidence(3) says the accused work was unretrievable
- Note: views on nonexistent ids (e.g. get_case(99)) error without a friendly message — the contract has no existence check.
