# Quick Intensity — Example

## Scenario

User asks: "Should we use PostgreSQL or MongoDB for our new analytics dashboard? We have ~10M events/day, need sub-second query response, 3-person backend team with SQL experience."

Gate classifies as: **Design** → Critique mode.
Model recommends: **Quick** (well-scoped decision, known domain).
User selects: Quick.

---

## Phase 1: Build (2 frameworks)

### Framework 1: PostgreSQL + materialized views

**Core hypothesis**: Use PostgreSQL with pre-computed materialized views refreshed every 5 minutes. Query performance comes from pre-computation, not raw scan speed.

**Why it holds**: 10M events/day = ~115 writes/second peak. PostgreSQL handles this easily on modest hardware. The team already knows SQL — no learning curve for queries, indexing, or view management.

**Strongest advantage**: Zero new tooling. Team uses existing skills, existing ORM, existing deployment. The fastest path to a working dashboard.

**Most likely failure point**: Materialized view refresh becomes a bottleneck as event volume grows beyond ~50M/day. At that point, refresh locks contend with write load, and query freshness drops below acceptable thresholds.

### Framework 2: MongoDB + aggregation pipeline

**Core hypothesis**: Use MongoDB's aggregation pipeline to compute analytics on-demand from a denormalized event collection. Flexibility comes from schema-less documents adapting to evolving analytics needs.

**Why it holds**: 10M documents/day with well-designed indexes can support sub-second aggregations. The schema-less model means adding new event types or dimensions doesn't require migrations.

**Strongest advantage**: Schema flexibility — adding a new analytics dimension is a code change, not a schema migration. This matters if the dashboard requirements are evolving rapidly.

**Most likely failure point**: The team has SQL experience, not MongoDB. Index design, aggregation pipeline optimization, and operational management of MongoDB are different skills. The learning curve could delay launch by 4-8 weeks.

---

## Phase 2: Debate (Round 1 of 1)

### Skeptic attacks

**On Framework 1**: Builder claims "zero new tooling" — but materialized views ARE new tooling relative to the team's current ORM-based workflow. The team uses Django ORM for everything today; materialized views require raw SQL management, migration coordination with application code, and a refresh strategy that the ORM doesn't natively support. This is a hidden complexity, not zero-tooling.

**On Framework 2**: Builder acknowledges the learning curve but frames it as a "4-8 week delay." That's optimistic. The real risk isn't learning the query API — it's operational. MongoDB requires different backup strategies, different monitoring, different scaling patterns. When something breaks at 3 AM, the team's PostgreSQL instincts (VACUUM, EXPLAIN ANALYZE, pg_stat_activity) are useless. The hidden cost is operational fragility, not just initial learning.

### Builder responds

**To Framework 1 attack**: Accepted and revised. Skeptic is right — materialized views introduce operational overhead I understated. Revised claim: the team already knows the QUERY language (SQL), which is the harder part. Managing materialized view refreshes (a cron job + a few stored procedures) is incremental operational work, not a paradigm shift. The gap between "using Django ORM" and "using PostgreSQL with views" is smaller than the gap between "using PostgreSQL" and "using MongoDB."

**To Framework 2 attack**: Partially accepted. The operational fragility concern is real. But the 4-8 week estimate is for production readiness, not basic competence. Mitigation: use MongoDB Atlas (managed) to eliminate backup/monitoring/scaling operational burden. This shifts the learning curve to index design and aggregation pipeline optimization only — which the team can learn in 2-3 weeks. The operational argument weakens significantly with a managed service.

### Information Increment Gate

Skeptic self-check after Round 1: 3/5 NO → No further rounds needed. Gate triggers. Proceed to Phase 4a.

---

## Phase 4: Converge (Editor only, Quick mode)

### Editor compression

**[Established]** PostgreSQL is the lower-regret choice for this specific context. The team's SQL expertise, existing tooling, and managed PostgreSQL availability make it the faster, safer path to production.

**[Probable]** The materialized view approach scales to ~50M events/day before requiring architectural change. At current volume (10M/day), that gives 12-18 months of runway assuming 2-3x annual growth.

**[Contested]** The MongoDB flexibility advantage is real — if analytics requirements change frequently, PostgreSQL schema migrations will create friction. Skeptic and Builder agree MongoDB handles evolving schemas better, but disagree on how much this matters: Builder thinks requirements are stable enough to not justify the operational risk; Skeptic thinks this is an unknown that should be measured before deciding.

---

## Quick Output

### Conclusion
**[Established]** PostgreSQL with materialized views is the lower-regret choice — team expertise and existing tooling outweigh MongoDB's schema flexibility for this volume and team composition.

### Key Risk
The materialized view refresh strategy has not been tested under this team's concurrent write load. If refresh locks contend with ingestion writes, query freshness degrades, and the "sub-second" requirement fails. Mitigate by benchmarking refresh WITH production-like write load before committing to the refresh interval.

### Boundary
This recommendation holds for event volumes up to ~50M/day with the current team. If volume grows beyond that, or if the team hires MongoDB expertise, re-evaluate. If analytics requirements are discovered to change weekly (not monthly), MongoDB's flexibility becomes more valuable than PostgreSQL's familiarity.
