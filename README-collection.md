# About this collection

A working checklist for auditing API security. Covers REST (mapped to OWASP API Top 10), GraphQL, gRPC, plus deep dives on the specific issues that show up most often: BOLA/IDOR, mass assignment, authentication, and rate limiting / hardening.

## Files

1. REST API Security — OWASP API Top 10 Checklist
2. GraphQL Security Checklist
3. gRPC Security Checklist
4. BOLA / IDOR Deep Dive
5. Mass Assignment Checklist
6. Authentication Checklist
7. Rate Limiting, Schema Disclosure, and Hardening

## How to use

For a new API: walk all seven. For an existing API: use file 1 as a quick triage, then drill into specific files when issues surface. The BOLA, mass-assignment, and authentication checklists are the highest-value to run as automated tests rather than manual reviews.

This is a baseline. High-stakes APIs (payments, health, identity) need additional regulator-specific controls beyond what's here.
