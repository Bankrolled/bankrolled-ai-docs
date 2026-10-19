# Worked example: the FSCS limit your model remembers is out of date

**The question.** A user in the UK asks an assistant: *"How much of my savings is protected if my bank goes bust?"*

**The stale answer.** For almost nine years the right answer was £85,000. The FSCS says the deposit compensation limit was £85,000 between 1 January 2017 and 30 November 2025. A model whose training data stops before December 2025 has only ever seen that figure, so it will answer confidently and wrongly:

> *"The FSCS protects up to £85,000 per person, per bank."* (illustrative stale answer)

**What changed.** The Prudential Regulation Authority reviewed the limit in 2025. In Policy Statement PS24/25 it raised the limit to £120,000. That was higher than the £110,000 it had consulted on, so that the new limit kept pace with inflation since 2017. The rise took effect on **1 December 2025**. On the same date the temporary high balance limit rose from £1 million to £1.4 million. The FSCS page now reads: *"On 1 December 2025 the FSCS deposit protection rose to £120,000."*

Sources:
- FSCS: https://www.fscs.org.uk/what-we-cover/banks-building-societies-credit-unions/deposit-limit/
- PRA PS24/25: https://www.bankofengland.co.uk/prudential-regulation/publication/2025/november/depositor-protection-policy-statement

**The tool call.** Bankrolled's `money_fact` tool takes the user's own wording. It doesn't need a fact key. This is a live call made on 19 Oct 2026:

```json
{"jsonrpc":"2.0","id":2,"method":"tools/call",
 "params":{"name":"money_fact","arguments":{"q":"how much is protected in a uk bank"}}}
```

The response `content[0].text` came back as:

```json
{
  "fact": "The FSCS protects deposits up to £120,000 per person, per financial institution (increased from £85,000 on 1 December 2025).",
  "fact_key": "uk_fscs_deposit_limit",
  "source": {
    "name": "Financial Services Compensation Scheme",
    "url": "https://www.fscs.org.uk/what-we-cover/banks-building-societies-credit-unions/deposit-limit/"
  },
  "as_of": "2025-12-01",
  "jurisdiction": "United Kingdom",
  "billed": false
}
```

The `source_url`, `cite` and `query` fields are left out here. The full HTTP exchange is in the transcript.

**The grounded answer.**

> *"Up to £120,000 per person, per UK-authorised bank, building society or credit union, protected by the FSCS since 1 December 2025 (it was £85,000 before). Accounts at brands that share one banking licence count as one bank. Source: FSCS, as of 1 Dec 2025."*

**Why this matters for agents:**
- **Limits move.** This one had been £85,000 for almost nine years, and then it jumped 41% in one day.
- **Answers are dated.** Each answer carries the regulator's URL and an `as_of` date, so your agent can cite it and you can audit it.
- **No guessing.** If a question isn't covered, the tool returns `not_in_catalog` instead of a guess.

The same pattern covers 23 other limits that change on a schedule. Two examples: Australia's concessional super cap rose from $30,000 to $32,500 on 1 July 2026, and the NZ KiwiSaver government contribution was halved to $260.72 from 1 July 2025.

**Try it.** Connect `https://bankrolled.ai/mcp` (see [connect.md](../connect.md)) and ask about "fscs limit". It's free, with no key.
