My name is Allan Jay Badua, and I’m an incoming senior at the Shidler College of Business at the University of Hawai‘i at Mānoa, where I’m double majoring in Management Information Systems and Finance. I’m passionate about understanding how technology and financial strategy come together to shape modern business, and I enjoy working on projects that challenge me to think analytically, solve problems creatively, and build systems that make processes more efficient.

As I move into my final year, I’m focusing on strengthening my technical skills, expanding my knowledge of global markets, and preparing for a career at the intersection of tech and finance. In Fall 2026, I’ll be studying abroad at Yonsei University in Seoul, South Korea. I’m excited to immerse myself in a new culture, learn from an international academic environment, and experience one of Asia’s most innovative business and technology hubs.

I’m driven by curiosity, continuous learning, and the goal of creating meaningful, practical solutions in every project I take on.

---

## Project: FX Transaction Hedging Model — U.S. Pharmaceutical Exporter (FIN-321)

An end-to-end treasury analysis for a EUR 8,000,000 receivable due in 365 days, built in five stages — framing, specification, AI-assisted build + audit, live market-data population, and independent validation — to compare four FX hedge strategies (no hedge, forward, money-market, put option) and deliver a CFO-ready recommendation.

| Stage | Artifact |
|---|---|
| 1 — Framing | [`docs/decisions/2026-07-31{Badua}-{scenario-slug}-hedge-framing.md`](docs/decisions/2026-07-31%7BBadua%7D-%7Bscenario-slug%7D-hedge-framing.md) |
| 2 — Technical specification | [`docs/specs/2026-08-07-Badua-pharma-exporter-spec.md`](docs/specs/2026-08-07-Badua-pharma-exporter-spec.md) |
| 3 — AI-assisted build + audit | [`models/builds/2026-08-07-Badua-pharma-exporter-model.xlsx`](models/builds/2026-08-07-Badua-pharma-exporter-model.xlsx) · [`analysis/2026-08-07-Badua-build-audit.md`](analysis/2026-08-07-Badua-build-audit.md) |
| 4 — Live market data | [`data/2026-08-07-Badua-market-data.md`](data/2026-08-07-Badua-market-data.md) |
| 5 — Independent validation + recommendation | [`analysis/2026-08-14-Badua-pharma-exporter-validation.md`](analysis/2026-08-14-Badua-pharma-exporter-validation.md) (+ [raw LLM output](analysis/2026-08-14-Badua-pharma-exporter-validation-appendix.md)) · [`docs/decisions/2026-08-14-Badua-pharma-exporter-hedge-recommendation.md`](docs/decisions/2026-08-14-Badua-pharma-exporter-hedge-recommendation.md) |
| Full build history | [`prompt-log.md`](prompt-log.md) |

**Bottom line:** a fresh LLM session, given only the spec and market-data memo, reproduced the workbook's core figures almost exactly but anchored its sensitivity grid to a stale placeholder value buried in the spec's worked example — silently erasing the put option's upside story and skewing its recommendation toward the forward hedge. The final recommendation, informed by the corrected live-data model, is a put option on the full notional: the only strategy of the four that satisfies the spec's stated dual objective of downside protection *and* upside retention. Full reasoning in the Stage 5 documents above.
