# AI Basketball Scouting & Transfer Portal Matching — MVP Spec

## 1. Product concept

Build an AI-assisted basketball scouting platform that lets a coach describe the type of player/system fit they need in natural language, searches structured basketball data, analyzes relevant video, and returns ranked candidates with evidence clips.

### Core thesis

Do **not** build a new basketball AI model from scratch.

Instead, orchestrate existing:
- Basketball data APIs
- Licensed video/data providers
- LLMs
- Video-understanding APIs
- Computer-vision models
- Cloud infrastructure

The proprietary layer should become the basketball ontology, player fingerprints, system-fit scoring, ranking methodology, and accumulated scouting intelligence.

---

# 2. Example user experience

Coach enters:

> Find me a mid-major guard who holds opponents to under 30% shooting in isolation, runs the floor hard in transition, and shoots above 38% on catch-and-shoot opportunities when drifting to the right corner.

The system should translate this into structured requirements:

```text
Position: PG/SG
Competition: Mid-major
Isolation defense: < 30% opponent FG
Transition: above-league-average frequency/impact
Catch-and-shoot: > 38%
Location: Right corner
```

Then:

1. Query structured player statistics.
2. Eliminate most players.
3. Analyze/search video for remaining candidates.
4. Build/refresh each candidate's player fingerprint.
5. Calculate system-fit scores.
6. Return ranked candidates.
7. Show timestamped video evidence explaining why each candidate matches.

---

# 3. Strategic positioning

The product should initially be positioned as:

> **An AI scouting layer on top of existing basketball data and video.**

Do not initially try to replace Synergy, Hudl/InStat, or other established data providers.

Potential positioning:

> "Synergy gives you the data. We give you the AI scout."

A second possible market is underserved basketball:

- European lower divisions
- ABA/ABA 2
- NCAA DII/DIII
- NAIA
- academies
- women's basketball
- smaller professional leagues

These markets may have substantial video but less sophisticated analytical infrastructure than NBA-level organizations.

---

# 4. Initial geographic/market opportunity

## Europe

Potential sources include:

- ABA League
- ABA 2
- EuroLeague
- EuroCup
- national leagues
- club/team websites
- YouTube
- Hudl/InStat

The ABA ecosystem is particularly attractive for a prototype because full games are publicly published on league properties and can provide a contained basketball ecosystem for experimentation.

### Important legal distinction

Being able to watch a game does **not** mean a startup can download, process, redistribute, or commercially exploit the footage.

Before production use, establish:
- video rights
- API/data licensing
- commercial processing rights
- storage rights
- clip redistribution rights
- customer access rights

Do not build a commercial business around scraping consumer streaming services without checking their terms and obtaining appropriate rights.

## North America

Potential sources include:

- NBA
- G League
- NCAA
- BallerTV
- college programs
- Hudl
- Synergy
- other amateur/club ecosystems

College basketball is particularly interesting because established video-exchange systems already exist.

---

# 5. Existing technologies to leverage

## Basketball data

### Synergy Sports / Sportradar

Potentially provides:
- games
- teams
- players
- possessions
- play types
- advanced statistics
- tracking-related information
- video relationships

There is a Synergy basketball API ecosystem.

**Recommendation:** investigate commercial API access before attempting to recreate this data layer.

### Hudl InStat

Potentially provides:
- 900+ basketball competitions
- historical data
- video
- player statistics
- team statistics
- tactical/event information
- player movement information
- API access for external partners

**Recommendation:** investigate InStat API/commercial access.

---

# 6. Video AI

## Google Gemini

Potential capabilities:
- long-form video understanding
- visual analysis
- timestamps
- audio
- multiple videos
- video questions
- clipping/segment-oriented analysis
- agentic exploration of long videos

Good candidate for the first prototype because it allows testing the concept without building a complete computer-vision stack.

Possible workflow:

```text
Game video
    ↓
Gemini
    ↓
"Find all possessions where Player #7 attacks a closeout"
    ↓
Timestamped evidence
    ↓
Clip extraction
```

## Twelve Labs

Potentially useful for:
- video indexing
- semantic video search
- video embeddings
- timestamped segmentation
- video analysis

Potential workflow:

```text
Thousands of games
       ↓
Video index
       ↓
Semantic search
       ↓
"Find clips where player rejects P&R"
       ↓
Timestamps / segments
```

### Recommendation

Prototype with Gemini and/or Twelve Labs rather than building video search infrastructure immediately.

---

# 7. Computer vision building blocks

## Ultralytics YOLO

Useful for:
- player detection
- ball detection
- object detection
- tracking
- pose
- custom model training

Tracking options include:
- ByteTrack
- BoT-SORT
- OC-SORT
- Deep OC-SORT

## Existing basketball CV projects

There are open-source projects demonstrating combinations of:
- YOLO
- player tracking
- ball tracking
- jersey classification
- court keypoint detection
- possession detection
- homography
- tactical mapping

**Recommendation:** use existing projects as learning/prototyping references. Do not assume their models are production accurate.

---

# 8. What to build vs. buy/use

| Component | Strategy |
|---|---|
| Web application | Build |
| User accounts | Use existing service |
| Database | Use PostgreSQL |
| Video storage | Use GCS/S3 |
| Video ingestion | Build |
| Video understanding | Use Gemini/Twelve Labs |
| Basketball statistics | License Synergy/InStat |
| Player tracking | Hybrid: existing CV + custom work |
| Court detection | Hybrid |
| Jersey/player identification | Hybrid |
| Shot detection | Hybrid |
| Possession detection | Hybrid |
| Natural-language search | Use LLM + structured output |
| Query → SQL/statistics | Build |
| Player ranking | Build |
| System-fit score | Build |
| Player fingerprint | Build |
| Video clip extraction | Build with FFmpeg |
| Semantic video search | Use Twelve Labs/embeddings |
| Authentication | Use existing service |
| Cloud infrastructure | Use GCP/AWS |
| Payments | Use Stripe |

---

# 9. Core proprietary concept: Player Fingerprint

Each player should eventually have a structured behavioral representation.

```text
PLAYER FINGERPRINT

ROLE
├── Primary ball handler
├── Secondary creator
└── Off-ball guard

OFFENSE
├── P&R ball handler
├── Isolation
├── Closeout attack
├── Catch & shoot
├── Pull-up
├── Transition
├── Rim finishing
└── Passing

DEFENSE
├── Point-of-attack
├── P&R
├── Isolation
├── Closeouts
├── Help defense
├── Switching
└── Transition defense

PHYSICAL
├── Speed
├── Acceleration
├── Lateral movement
├── Size
└── Athleticism

CONTEXT
├── League strength
├── Competition level
├── Team system
└── Role
```

The product should eventually search for:

> "Players whose behavioral fingerprint resembles the fingerprint our team needs."

rather than simply:

> "Find good players."

---

# 10. Agent architecture

The LLM should be the orchestrator, not the entire scouting engine.

```text
                     COACH
                       |
                       v
              Natural-language request
                       |
                       v
                   LLM Agent
                       |
          +------------+------------+
          |            |            |
          v            v            v
     Player DB      Synergy      Video DB
                       API
          |            |            |
          +------------+------------+
                       |
                       v
                 Candidate pool
                       |
                       v
              Video AI / CV layer
                       |
                       v
              Player fingerprint
                       |
                       v
                Ranking engine
                       |
                       v
             Evidence + video clips
                       |
                       v
                  Coach UI
```

---

# 11. Recommended technical stack

## Frontend
- Next.js
- React

## Backend
- Python
- FastAPI

## Database
- PostgreSQL
- pgvector

## Storage
- Google Cloud Storage

## Video processing
- FFmpeg

## Video AI
- Gemini
- Twelve Labs

## Computer vision
- YOLO
- ByteTrack / BoT-SORT

## Basketball data
- Synergy API
- Hudl InStat API

## Cloud
- Google Cloud Run
- Cloud Tasks
- Pub/Sub
- Cloud SQL
- Secret Manager
- IAM/service accounts

## Authentication
- Clerk or equivalent

## Payments
- Stripe

---

# 12. MVP strategy

Do NOT start with:

> "Analyze 10,000 games and reconstruct every player's career."

Start with one game.

## MVP v1

Input:

- One legally obtained full game
- Player/team identification

Output:

```text
PLAYER #7

OFFENSE
- Catch-and-shoot opportunities
- Closeout attacks
- P&R possessions
- Transition possessions
- Isolation possessions

DEFENSE
- P&R defensive possessions
- Isolation possessions
- Closeouts

EVIDENCE
- Timestamp 00:04:32
- Timestamp 00:11:08
- Timestamp 00:19:44
...
```

The first goal is not statistical perfection.

The first goal is to determine:

> **Does the AI produce scouting insight that a basketball person actually finds useful?**

---

# 13. MVP v2

Add:

> "Find players like this."

Pipeline:

```text
Player A
   ↓
Player fingerprint
   ↓
Search player database
   ↓
Candidate similarity
   ↓
Video verification
   ↓
Ranked alternatives
```

Example:

```text
Player B — 91% match
Player C — 87%
Player D — 83%
Player E — 81%
```

---

# 14. MVP v3

Add team/system intelligence.

Example:

> "Find me someone like our current starting PG, but more athletic, a better shooter, and less ball dominant."

Agent workflow:

```text
Interpret request
      ↓
Understand team/player profile
      ↓
Query player database
      ↓
Filter candidates
      ↓
Compare fingerprints
      ↓
Analyze video
      ↓
Rank candidates
      ↓
Generate evidence
```

This is the point where the product becomes a real recruiting/scouting tool.

---

# 15. Major technical risk

The hardest problem is probably not the LLM.

It is **reliable player identification and tracking**.

Example:

Player #7 makes a three.

The AI correctly identifies:
- shot
- location
- outcome

But after a camera cut it accidentally assigns another player to #7.

Now all subsequent statistics become contaminated.

For a trusted commercial product, identity persistence, roster mapping, camera cuts, jersey recognition and tracking quality are critical.

This is one reason established providers such as Synergy/InStat are potentially valuable partners rather than competitors to replace immediately.

---

# 16. Skills required

The founder/operator does NOT need to become a computer vision researcher.

## 1. Python — essential

Learn:
- functions
- classes
- lists/dictionaries
- JSON
- REST APIs
- async processing
- pandas
- debugging
- virtual environments
- package management

Importance: 8/10

## 2. SQL/database fundamentals

Learn:
- SELECT
- JOIN
- WHERE
- GROUP BY
- ORDER BY
- CTEs
- window functions
- indexes
- relational modeling

This project is ultimately a large data problem disguised as an AI problem.

## 3. APIs

Learn:
- REST
- JSON
- authentication
- API keys
- OAuth
- webhooks
- pagination
- rate limits
- retries

Existing experience connecting business software APIs transfers directly.

## 4. Computer vision fundamentals

Understand:
- object detection
- tracking
- OCR
- pose estimation
- embeddings
- segmentation
- homography
- re-identification
- optical flow

You do not need to train everything yourself. You need to understand what the systems do and how to evaluate failures.

## 5. Cloud architecture

Recommended:
- Cloud Run
- Cloud Storage
- Cloud Tasks
- Pub/Sub
- Cloud SQL
- Secret Manager
- IAM
- service accounts

## 6. LLM/agent architecture

Learn:
- structured outputs
- function calling
- tool use
- RAG
- embeddings
- vector databases
- agent orchestration
- workflow state
- evaluation
- hallucination control

---

# 17. Vibe-coding difficulty

## Concept/demo

**4/10**

A surprisingly convincing prototype is possible using:
- Next.js
- Python
- Postgres
- Gemini
- FFmpeg
- basic CV

## Useful MVP

**7/10**

Difficulty rises because of:
- reliable player identification
- timestamps
- automatic clips
- repeatable statistics
- multiple games
- player databases
- background processing
- cloud infrastructure
- error handling

## Commercial product

**9/10**

Additional challenges:
- video rights
- data licensing
- player identification
- CV accuracy
- league normalization
- infrastructure cost
- security
- QA
- support

---

# 18. Recommended 30-day exploration plan

## Week 1 — prove the concept

Build:

```text
Game video
    ↓
LLM/video AI
    ↓
Player scouting report
    ↓
Timestamped evidence
```

Use one game.

Question to answer:

> Is the output genuinely useful to someone who understands basketball?

## Week 2 — evidence and clips

Build:

```text
Game
 ↓
Detected event
 ↓
Timestamp
 ↓
FFmpeg
 ↓
Short video clip
```

Now the report can prove its claims.

## Week 3 — structured player model

Create the first version of:

```text
Player
 ├── Role
 ├── Offense
 ├── Defense
 ├── Shooting
 ├── Transition
 └── Context
```

Start building the player fingerprint.

## Week 4 — candidate search

Build:

```text
Player fingerprint
        ↓
Similarity search
        ↓
Top candidates
        ↓
Video verification
```

At the end of 30 days, you should have a prototype that demonstrates the core product rather than a huge unfinished platform.

---

# 19. Business questions to answer before serious development

The biggest uncertainty is not whether the technology exists.

It is whether the required data/video can be obtained at commercially viable cost and under appropriate rights.

Investigate:

1. Synergy API pricing/licensing
2. Hudl InStat API pricing/licensing
3. Commercial video-processing rights
4. Whether ABA/ABA 2 footage can be licensed
5. Whether teams can provide their own footage
6. Cost per analyzed game
7. Cost per player fingerprint
8. Cost of storing video
9. Cost of repeated AI analysis
10. Whether target customers would pay enough

---

# 20. Key hypothesis

The strongest version of the idea is NOT:

> "AI watches basketball."

It is:

> **"Describe the player your system needs, and the AI finds the players who actually play that way — then proves it with video."**

That is the product to validate.

---

# 21. Immediate next research questions

Before committing to architecture, investigate:

### Data
- What exactly does Synergy expose through its API?
- What exactly does InStat expose?
- What are commercial prices?
- Can video and data be linked programmatically?

### Video
- Which European leagues provide legally licensable footage?
- Can ABA/ABA 2 footage be used commercially?
- Which North American leagues provide accessible footage?
- Can teams upload their own footage?

### AI
- Gemini vs Twelve Labs for basketball video
- Accuracy of generic video models on basketball
- Existing basketball-specific CV models
- Player identification accuracy
- Tracking accuracy through broadcast camera cuts

### Economics
- Cost to process one game
- Cost to maintain a 10,000-game library
- Cost per candidate search
- Likely customer pricing

### Product
- What does a coach actually need?
- Which scouting questions are easiest to automate?
- Which outputs are trusted?
- What evidence makes a coach believe the AI?

---

# 22. Bottom line

This is technically feasible.

You do **not** need to build:
- a foundation model
- a video search engine
- a basketball database
- an entire computer vision stack
- cloud infrastructure from scratch

You can assemble existing components.

The hardest parts are:

1. **Rights/data access**
2. **Reliable player identification**
3. **Basketball-specific event interpretation**
4. **Creating a trustworthy player/system fingerprint**
5. **Turning that into a useful ranking methodology**

The most promising initial approach is:

> **ABA/European basketball → one-game analysis → player fingerprint → similarity search → evidence clips → expand into licensed data APIs.**

This is a strong candidate for an AI-assisted/vibe-coded prototype and is unusually aligned with an operator who already understands business processes, APIs, cloud automation and agentic AI.
