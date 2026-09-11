# cloud-itonami-iso3166-jpn-statistics

Open ISO 3166 Agency Blueprint for **JPN-STATISTICS**: Statistics Japan
(総務省統計局, Statistics Japan) — a Japan-agency-level LEAF under
the `cloud-itonami-iso3166-jpn` country-level coordinator.

This repository designs a forkable OSS business for an independent
compliance consultant: an already-incorporated operator (typically one
already using `cloud-itonami-iso3166-jpn` for general Japan market entry)
gets a Compliance Advisor + independent **Statistics-Reporting Compliance Governor** to
navigate official-statistics reporting obligations under the Statistics Act (統計法) for an operator whose public-sector contract requires submitting designated statistical surveys or data to Statistics Japan (総務省統計局).

This is the final repo in the Japan agency-level sweep started by
ADR-2607040100 — with this blueprint published, all 19/19 Japan central-
government bodies in `kotoba-lang/iso3166` are `:maturity :blueprint`.

## No robotics premise — digital/data service exemption

Agency-specific compliance navigation is a pure data/software service with
no physical-domain work — the same exemption class as `cloud-itonami-6310`
and `cloud-itonami-gtin-*`. `blueprint.edn` sets
`:itonami.blueprint/robotics false` and `:required-technologies` lists only
real capabilities (`:identity`, `:forms`, `:dmn`, `:bpmn`, `:audit-ledger`),
no `:robotics`.

## Core Contract

```text
operator intake + prior filing/compliance history
        |
        v
Compliance Advisor -> Statistics-Reporting Compliance Governor -> compliance draft, or human sign-off
        |
        v
gated filing / registration / compliance-program submission + audit ledger
```

No automated proposal can submit a filing or registration the governor
refuses, suppress a compliance record, or claim a legal conclusion the
governor has not cleared. `:filing/submit` is never in any phase's `:auto`
set — it always requires human sign-off (mirrors `cloud-itonami-M6910`'s
`filing-submit-never-auto-at-any-phase` invariant).

## What this is NOT

- **Not Statistics Japan (総務省統計局) itself, and not the
  government of Japan.** See [`docs/business-model.md`](docs/business-model.md)
  for the boundary with `com-etzhayyim-ooyake`, `matsurigoto`,
  `com-etzhayyim-toritsugi`, `legal-entity.etzhayyim.com`,
  `cloud-itonami-M6910`, and the country-level `cloud-itonami-iso3166-jpn`.
- **Not legal or tax advice.** Every regulatory claim must cite the
  official Statistics Japan source and route final filings to
  Japan-licensed counsel or a registered agent where the law requires
  licensed representation.

## Regulatory source register

The citation requirement above has a referent: [`facts.edn`](facts.edn), a
register of the statutes and official pages this repository is allowed to
build a requirement on. A law or page that is not in that table has **no
spec-basis here** — extend the table, never invent an id or a URL.

Re-check it against the live authorities:

```bash
kbb --backend sci scripts/verify-facts.cljk
```

Three exit codes, and the third is the point:

| exit | meaning |
|---|---|
| `0` | every source re-fetched and matched |
| `1` | a source did not check out — **the register is wrong** |
| `2` | the run could not answer — **not a pass** |

`2` exists because a check that could not run must not return the same value
as a check that ran and found nothing.

### The authority this repository names is not the one that runs the machinery

`organization.edn` names 総務省統計局 and gives `www.stat.go.jp`. That is the
survey-and-data side. The approval, notification and review machinery under
統計法 — the part an operator with a reporting obligation actually meets — is
run by 政策統括官（統計制度担当）, a different part of the same ministry, and
it is published on `www.soumu.go.jp/toukei_toukatsu/`.

So the register spans two hosts of one ministry plus the statistics portal,
and they do not behave alike (measured 2026-08-27):

| host | charset | in HTTP header? | its 404 page |
|---|---|---|---|
| `www.soumu.go.jp` | Shift_JIS | no | distinctive title |
| `www.stat.go.jp` | Shift_JIS | no | **titled like the front page** |
| `www.e-stat.go.jp` | UTF-8 | yes | distinctive title |

A verifier written against any one of them is wrong about the other two, which
is why `:host/charset` is declared per host and re-checked every run.

### Four measured hazards the exit codes exist for

- **Shift_JIS with no charset in the header, on both ministry hosts.** A body
  read with the default UTF-8 decoding does not throw — it returns stable
  mojibake, so `総務省｜統計制度｜統計法について` reads as
  `�����ȁb���v���x�b���v�@�ɂ���`. Left unhandled that reports every live
  page as a dead citation, or, if an author pins the mojibake, passes forever
  while asserting nothing. Two self-tests decode the same live bytes both ways
  — once on a Shift_JIS host and once on the UTF-8 host — so the per-host rule
  is measured rather than asserted.
- **`www.stat.go.jp` serves its 404 under the front page's own title.** The
  404 body (4,691 B) and the live front page (36,963 B) both carry
  `<title>統計局ホームページ</title>`. Status still separates them, but the
  title check is vacuous for any entry citing a page with that title. The run
  **fails** such an entry before fetching it, which is why the front page is
  deliberately absent from the register and `/data/index.html` is there
  instead. The collision is re-measured every run; if the 404 ever gets a
  title of its own, that is what notices.
- **Each host's 404 body carries the obvious sentinel for that host.**
  `www.stat.go.jp`'s carries `若松町`, `162-8668` and `03-5273-2020` — which
  is what `organization.edn` asserts as the HQ, so confirming the address
  passes on a page that is not there. `www.soumu.go.jp`'s carries `届出`, an
  entry route under 統計法, on the host that publishes the procedure.
  `www.e-stat.go.jp`'s carries `e-Stat` and `政府統計の総合窓口`, the portal's
  own name and its front page's exact title. Needles are cleared against that
  body before the cited page is fetched.
- **`laws.e-gov.go.jp` cannot be checked as a page at all** — `統計法` at
  `/law/419AC0000000053` and an invented `/law/999ZZ9999999999` agree on
  status, final URL, title and length. Statutes are resolved through the law
  API instead, and the indistinguishability is re-measured each run.

### Identity is three fields, because two ordinances share one title

`経済センサス基礎調査規則` names two different instruments:

| law id | law number | status |
|---|---|---|
| `420M60000008125` | 平成二十年総務省令第百二十五号 | **repealed 2019-04-01** |
| `431M60000008046` | 平成三十一年総務省令第四十六号 | in force |

A citer who writes down that title and gets a match has established nothing
about which one they cited — for a designated survey that businesses are
actually enumerated in. So every statute entry declares its law number and its
repeal and revision status, and each of the three has a **real** negative
control: the live ordinance declared with the repealed twin's number, the
repealed twin declared in force with title and number left correct, and
`公文書等の管理に関する法律`, which is not repealed at all yet carries
`PreviousEnforced`.

Not `remain_in_force` — that field reads like the answer and is not. It is
`false` on the live `統計法` too, so keying on it would mark every entry dead.

## Capability layer

Resolves via [`kotoba-lang/iso3166`](https://github.com/kotoba-lang/iso3166)
(code `JPN-STATISTICS`, `:parent "JPN"`, cross-referenced to ooyake's
`gov.jpn.statistics`). Required capabilities:

- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
