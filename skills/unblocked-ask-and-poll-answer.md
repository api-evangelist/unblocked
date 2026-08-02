---
name: Ask Unblocked a question and poll for the answer
description: Submit a codebase question to the Unblocked Answers API and poll until the answer is complete, respecting quotas and the async contract.
api: openapi/unblocked-public-api-openapi-original.json
operations: [askQuestion, getAnswer, listAnswers, deleteAnswer]
generated: '2026-07-21'
method: generated
---

# Ask Unblocked a question and poll for the answer

Base URL `https://getunblocked.com/api/v1`. Authenticate every request with
`Authorization: Bearer <token>` (personal or team API token from Settings ->
API Tokens).

## Steps

1. Generate a globally unique UUID for the question — this is the
   `questionId` and your idempotency handle; retrying the same PUT with the
   same UUID is safe.
2. `askQuestion` — `PUT /answers/{questionId}` with body
   `{"question": "..."}`. Expect `204 No Content` (queued).
3. `getAnswer` — `GET /answers/{questionId}` and poll. While processing the
   body is `{"state": "processing"}`; when done, `state` is `complete` and
   `result` carries the Markdown `answer` plus `references[].htmlUrl`.
4. Optionally `listAnswers` (`GET /answers`, cursor pagination via `limit`,
   `after`, `before` and the RFC 8288 `link` response header) to review past
   Q&A, or `deleteAnswer` (`DELETE /answers/{questionId}`) to permanently
   remove one — irreversible.

## Rules

- Quota: 1,000 questions/day per organization; `429` means the quota is
  exhausted (resets midnight PST). Personal Access Tokens are further limited
  to 1,000 API calls/day and only see their own questions.
- Errors return `{"status": <code>}` — see `errors/unblocked-problem-types.yml`.
- Answers are Markdown; always surface the `references` so users can verify
  sources.
