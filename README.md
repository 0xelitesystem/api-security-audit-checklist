# api-security-audit-checklist

Seven checklists covering the audit surface of HTTP, GraphQL, and gRPC APIs. Each checklist is structured as audit questions paired with concrete tests and mitigations, not abstract guidance.

## Checklist structure

Every checklist has three parts:

1. **Audit questions** — what to ask the team or determine from the codebase
2. **Tests** — specific requests, payloads, or fuzzing patterns
3. **Mitigations** — controls to add, ordered by impact

## Contents

| # | Checklist | Focus |
|---|-----------|-------|
| 01 | [REST OWASP API Top 10](checklists/01-rest-owasp-top-10.md) | The full OWASP API Security Top 10 mapped to audit questions |
| 02 | [GraphQL](checklists/02-graphql.md) | Introspection, query depth, alias abuse, batched queries, field auth |
| 03 | [gRPC](checklists/03-grpc.md) | Reflection, message-size limits, metadata trust, streaming abuse |
| 04 | [BOLA / IDOR](checklists/04-bola-idor.md) | Object-level authorization gaps; the most-exploited API class |
| 05 | [Mass assignment](checklists/05-mass-assignment.md) | Over-posting, allow-list vs deny-list, auto-binding pitfalls |
| 06 | [Broken authentication](checklists/06-broken-auth.md) | JWT issues, session fixation, OAuth misuse, credential stuffing |
| 07 | [Rate limiting and introspection](checklists/07-rate-limiting-and-introspection.md) | Quotas, abuse cost, leaked debug endpoints, error verbosity |

## Intended use

Use this in code review, design review, or a focused audit window. Each checklist runs about 15–60 minutes depending on API size. The tests are written assuming you have access to the API and a couple of test accounts; nothing requires a full lab environment.

The OWASP API Top 10 is referenced because the categories are useful, not because the project is endorsed. Where the OWASP wording is vague, this checklist substitutes a more specific question.

## Contributing

If a test produces false positives in your environment, file an issue with the framework or platform involved. PRs that add language-specific notes (e.g. Rails strong-params, Django serializers, Spring `@RequestBody`) under each checklist are welcome.

## Related repositories

Part of a 10-repo security audit set.

Browser-based audit tools:
- [iam-policy-analyzer](https://github.com/0xelitesystem/iam-policy-analyzer)
- [terraform-security-linter](https://github.com/0xelitesystem/terraform-security-linter)
- [kubernetes-manifest-security-scanner](https://github.com/0xelitesystem/kubernetes-manifest-security-scanner)
- [session-cookie-auditor](https://github.com/0xelitesystem/session-cookie-auditor)
- [regex-redos-checker](https://github.com/0xelitesystem/regex-redos-checker)

Reference collections:
- [incident-response-runbooks](https://github.com/0xelitesystem/incident-response-runbooks)
- [ai-llm-security-audit](https://github.com/0xelitesystem/ai-llm-security-audit)
- [secrets-leak-response-runbook](https://github.com/0xelitesystem/secrets-leak-response-runbook)
- [threat-modeling-worksheets](https://github.com/0xelitesystem/threat-modeling-worksheets)

## License

MIT. See [LICENSE](LICENSE).
