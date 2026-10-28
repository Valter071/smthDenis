# AI Basketball Scouting — Review, Market Scan & Build Plan

_Date of scan: 2026-10-28. Companion to `00-original-spec.md`._

---

## 1. Verdict in one paragraph

The idea is sound and the spec is unusually disciplined (buy-don't-build, start with one game, "prove it with video"). **But the premise that nothing like this exists is wrong — at least for US college basketball.** In 2026 several products already ship AI fit scores, natural-language queries over film, and video-linked evidence. What remains open is (a) **European / lower-tier basketball**, where the AI players are thin and Synergy/InStat coverage is patchy, and (b) the **"find players who *play* this way, proven with clips" query** across a broad multi-league library — which nobody has convincingly delivered yet because it needs rights to a large video library. The startup should be built around winning that rights/data problem in one region, not around the AI (which is becoming a commodity).

---

## 2. Market scan (Oct 2026)

### 2a. Direct / near-direct competitors

| Product | What it does | Market | Overlap with spec |
|---|---|---|---|
| **ShotTracker Scout** | Mar 2026: *AI Fit Score* matching portal prospects to a program's roster, system and style; every data point tied to video; voice/text natural-language queries returning clips + stats; NIL benchmarks, roster-gap analysis. | NCAA (men's & women's) | **Very high.** This is essentially MVP v3 for US college. |
| **Synergy Sports (Sportradar)** | AI game/player/team scouting summaries with follow-up prompts (2026); Synergy Basketball API with *Player Comps, Offensive Player Roles, Shot Quality, Player Projections*; an **official MCP server** so any AI assistant can query Synergy. | NCAA first, pro leagues worldwide | **High**, and they own the data layer the spec wants to build on. |
| **Ballin AI** | Learns a program's roster/style, recommends recruits (HS, JUCO, international, portal); athletes turn film into profiles. ~$1M raised. | US recruiting | High on "system fit", lighter on video evidence. |
| **Portal Vision, magisterial, The Portal Report, Portal Sniper, Basketball Quant** | Portal search, AI assistants, comparisons, fit analysis — mostly on top of public stats (Torvik, EvanMiya, CBBD). | NCAA portal | Medium — stats-only, little or no video. |
| **ScoutIQ, AirPLAi (PLAi Chat)** | Upload your film, ask questions in plain language, get clips back. | HS / youth / club | Medium — natural-language film search, but on your own team's film only, no cross-player discovery. |
| **SportsVisio** | 8-model CV pipeline: box score, shot chart, possession log, highlights from phone footage (claims 95% shot detection, 92% player attribution). | Youth / amateur | Low on scouting; relevant as a CV benchmark/partner. |

### 2b. Europe / international

| Product | What it does | Gap left |
|---|---|---|
| **Hudl InStat** | Huge competition coverage, video, filters by position/style/level. No AI player-finder found for basketball. | Old-school UI, no NL search. |
| **Scouting4U** | 50+ leagues (EuroLeague, ACB, BSL, etc.), 10k+ players, video tagging, *AI-generated scouting reports*. Founded by ex-EuroLeague player. | Report generation ≠ "find me players who play like X". |
| **Scoutbasketball** | 270k players, 390+ competitions, 55+ countries incl. Balkans/Baltics. Stats search & comparisons. | Stats only, no video evidence. |
| **PNR Scout** | EuroLeague player intelligence, AI projections, beta 2026-27. | Top-tier only. |

### 2c. What this means

- **US college is a red ocean.** Competing with ShotTracker + Synergy (who own the tracking data or the tagged possessions) as a wrapper on their own API is a weak position — they can copy you or cut your access.
- **"Synergy gives you the data, we give you the AI scout" is now Synergy's own pitch** (AI summaries + MCP server). Drop that positioning.
- **The genuine white space:** European second-tier / developmental basketball (ABA 2, national 2nd divisions, Balkan/Central-European leagues, women's leagues, youth academies) where:
  - clubs have small budgets and no analysts,
  - pro clubs recruit *from* these leagues and currently do it by watching hours of film,
  - the AI-native competitors are all US-focused.
- **The moat is not the model.** It is: (1) licensed video + play-by-play for a region nobody else indexes well, (2) a human-verified labelled dataset of possessions built through usage, (3) relationships with clubs/agents in that region.

---

## 3. Critique of the spec — things to change

1. **Small-sample trap in the flagship query.** "38% on catch-and-shoot when drifting to the right corner" — a guard might get 10–20 of those per *season*. Any filter that narrow is noise. The product must show **attempts + confidence interval** next to every percentage and fall back to broader buckets (all corner C&S) automatically. This is a feature, not a footnote — it is what makes a coach trust it.
2. **Player identification — there's a cheaper path than CV tracking.** Most leagues (ABA included) publish **play-by-play with game clock**. OCR the scoreboard clock in the broadcast → map game-clock ↔ video-time → every PBP event (made 3, turnover, foul, rebound, assist — already attributed to a player by the official scorer) becomes a correctly-labelled clip **with no jersey recognition at all**. Video AI is then only needed to *describe* each short clip (spot-up vs P&R vs transition, corner vs wing, contested or not). This removes the spec's #1 technical risk for offensive events.
3. **Defense is the hard part, and the spec underrates it.** "Holds opponents under 30% in isolation" needs knowing *who was guarding whom* — not in play-by-play, and hard for video AI. Make defense a later phase; launch offense-first and say so honestly.
4. **Long-video Gemini calls are the wrong unit.** Default sampling is ~1 frame/sec — a catch-and-shoot is over in under a second. Send **short clips (8–15 s) at higher fps**, cut via the PBP alignment above. Cheaper, more accurate, parallelisable.
5. **Rights vs. cold-start tension.** "Teams upload their own footage" solves rights but gives you no library to search *across*. "Find players like X" needs a library. Resolve it explicitly: (a) team-uploaded film for the single-team product, (b) a **league-level licence** (ABA League / a national federation) for the searchable library — pitch the league a revenue share or free analytics in exchange. ABA publishes full games on its YouTube channel, but YouTube's terms prohibit downloading; you need written permission.
6. **Too much cloud for an MVP.** Cloud Run + Tasks + Pub/Sub + Cloud SQL + IAM is weeks of plumbing. Start with one Postgres (with pgvector), one object store, one worker queue. Move to managed GCP pieces when there's load.
7. **Validation step is missing before code.** The spec's Week 1 question ("is the output useful to a basketball person?") can be answered in 2 days by hand: take one ABA game, run clips through Gemini manually, show the report to 5 coaches/scouts. Do that before building anything.

---

## 4. Recommended positioning

> **"The AI scout for European basketball's next tier. Describe the player you need — get ranked players from leagues your scouts don't have time to watch, each claim backed by clips."**

Customers, in order of likely willingness to pay:
1. Pro clubs in ABA / national top divisions recruiting from lower tiers (budget: low-thousands €/season).
2. Player agents (very motivated, want to surface clients, will pay per report).
3. National federations / youth academies.
4. US colleges recruiting European internationals — a bridge into the US market where competitors don't have European film.

---

## 5. Build plan

### Phase 0 — Validate (1–2 weeks, near-zero code)
- [ ] Interview 10–15 people: club coaches/sporting directors, scouts, agents in the ABA region. Key question: *"Last time you signed a player, how did you find him and how many hours of film did it cost?"*
- [ ] Hand-run one ABA game: pick 1 player, cut 20 clips manually, ask Gemini to classify them, write the report, show it.
- [ ] Email ABA League / one national federation about a data + video licence for a pilot.
- [ ] Confirm PBP availability & format for target leagues.

**Gate:** ≥3 people say "I'd pay / I'd pilot this" and one viable legal video source exists.

### Phase 1 — Single-game player report (weeks 3–6)
Pipeline:
```
Game video (legally obtained) + official play-by-play
  → scoreboard OCR (game clock ↔ video time alignment)
  → FFmpeg: one short clip per PBP event, per player
  → Video LLM on each clip → structured JSON (play type, shot zone, contest, outcome)
  → Postgres: events, clips, labels
  → Player report page with stats + clickable evidence clips
  → "Correct this label" button (every correction = training/eval data)
```
Deliverable: upload a game, pick a player, get a report with clips in < 15 min.
Also: a hand-labelled "gold" game to measure label accuracy per play type.

### Phase 2 — Multi-game fingerprints + natural-language search (weeks 7–12)
- Ingest a season of one league (≈ 200–250 games).
- Player fingerprint = per-play-type frequency, efficiency, and attempt counts, normalised per 100 possessions and by league strength; stored as a vector in pgvector.
- NL query → LLM with structured output → filter spec (with sample-size thresholds) → SQL + vector ranking.
- "Find players like X" via fingerprint similarity.
- Results page: ranked players, why they match, sample sizes, clip playlists.

### Phase 3 — System fit & defense (months 4–6)
- Team style profile (pace, play-type mix) → fit score.
- "Like our PG but more athletic, less ball-dominant" → fingerprint deltas.
- Defensive assignments via video AI on clips; ship only when the gold-set accuracy is acceptable.

### Proposed MVP stack
| Layer | Choice | Why |
|---|---|---|
| Frontend | Next.js (React, TypeScript) | As in spec. |
| API + workers | Python, FastAPI, a simple job queue (e.g. RQ/Celery on Redis, or Postgres-based) | Python owns video/CV/ML tooling. |
| DB | PostgreSQL + pgvector | One database for stats, events, vectors. |
| Auth / storage | Supabase (Postgres + auth + storage in one) **or** GCP equivalents | Fewer moving parts for a solo founder. |
| Video | FFmpeg; OCR via a vision LLM or PaddleOCR for the scoreboard clock | Clock OCR is the alignment linchpin. |
| Video understanding | Gemini (per-clip) first; benchmark Twelve Labs in Phase 2 for library search | Clip-level calls are cheap & accurate. |
| Query orchestration | Any strong LLM with structured outputs / tool use | Keep it provider-swappable. |
| Hosting | Cloud Run (or similar) for API/workers, Vercel for frontend | Cheap at low volume. |

### Unit economics to measure in Phase 1
- LLM cost per game (≈ 150–250 clips per game per both teams).
- Storage per game (source video + clips).
- Human correction minutes per game needed to reach target accuracy.

---

## 6. Open decisions for the founder
1. **Region/market:** Europe lower tiers (recommended) vs US college?
2. **Video source for the pilot:** league licence, club partner footage, or own recordings?
3. **Budget & timeline** for Phase 0–1.
4. **Who is building:** founder + Claude only, or additional engineers?

---

## Sources (scan of 2026-10-28)
- ShotTracker AI transfer portal tool — https://shottracker.com/shottracker-ai-transfer-portal-tool-scout/
- Synergy Team Site release notes — https://support.synergysports.com/support/solutions/articles/77000591657-synergy-team-site-release-notes
- Synergy Basketball API advanced endpoints — https://developer.sportradar.com/sportradar-updates/changelog/synergy-basketball-api-advanced-endpoint-package
- Sportradar MCP server — https://developer.sportradar.com/getting-started/docs/mcp-server
- HEART conference + Synergy AI scouting — https://heartofamericaconference.com/general/2025-26/releases/20260526meqh49
- Ballin AI — https://www.ballin.ai/ ; funding: https://www.blackenterprise.com/nyla-pollard-500k-funding-ballin-ai-platform/
- Portal Vision — https://portalvision.io/ ; magisterial — https://magisterial.ai/ ; Portal Sniper — https://portalsniper.com/
- PortalPoint (Berkeley capstone) — https://www.ischool.berkeley.edu/programs/mids/capstone/2026b-summer/portalpoint-transfer-portal-intelligence-platform-college
- ScoutIQ — https://www.scoutiq.org/ ; AirPLAi — https://airplaisports.com/
- SportsVisio technical guide — https://www.sportsvisio.com/stories/how-ai-basketball-analysis-works
- Scouting4U — https://basketball.scouting4u.com/ ; Scoutbasketball — https://scoutbasketball.com/ ; PNR Scout — https://pnrscout.com/
- Hudl Instat basketball scouting — https://www.hudl.com/blog/hudl-instat-basketball-scouting-recruitment
- Twelve Labs sports pipeline — https://www.twelvelabs.io/blog/sports-analysis
- ABA League YouTube channel — https://www.aba-liga.com/news/35325 ; ABA broadcasters — https://www.aba-liga.com/news/40604
