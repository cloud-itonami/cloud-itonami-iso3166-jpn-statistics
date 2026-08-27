# Operator Guide

## First Deployment

1. Confirm the client already uses (or has completed the equivalent of)
   `cloud-itonami-iso3166-jpn` for general Japan market-entry; this repo is
   an agency-specific supplement, not a substitute.
2. Register the client's intake: business type, the specific
   Statistics Japan-regulated activity involved, prior filing/compliance
   history in Japan if any.
3. Run the advisor in read-only mode against Statistics Japan's
   (総務省統計局) published guidance.
4. Compare the checklist against the client's current documentation.
5. Enable gated filing/compliance-draft assistance once the
   Statistics-Reporting Compliance Governor contract is trusted; actual submission always
   requires human sign-off.

## Minimum Production Controls

- client-owned data store for compliance documents
- clear provenance (official Statistics Japan source citation) for every
  requirement surfaced — the citable set is [`facts.edn`](../facts.edn),
  re-checkable against the live authorities with
  `nbb scripts/verify-facts.cljs`. A requirement whose basis is not in that
  register has no provenance. Note in particular that the register does
  **not** establish whether a given survey is a 基幹統計調査, a 一般統計調査
  or a 届出統計調査, nor whether a named operator is a designated respondent
  for it: 統計法 and 統計法施行令 are cited as instruments, not read
- approval workflow for any filing, registration, or compliance-program
  submission
- named referral relationship with Japan-licensed counsel or a registered
  agent for anything beyond checklist/draft assistance
- monthly audit export

## Certification

Certified operators must prove data provenance, audit traceability, that
automated actions cannot bypass the Statistics-Reporting Compliance Governor, and a working
referral relationship with Japan-licensed counsel or a registered agent for
whatever licensed representation Japanese law requires for actual
Statistics Japan filings.
