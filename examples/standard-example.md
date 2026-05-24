# Standard Intensity — Example

## Scenario

User asks: "We need to redesign our authentication system. Currently it's session-based with server-side storage. We're moving to a multi-service architecture with 12 services. Should we use JWT, centralized session service, or stick with sticky sessions + shared session store?"

Gate classifies as: **Design** → Critique mode.
Model recommends: **Standard** (multi-service architecture decision, moderate complexity, multiple valid approaches).
User selects: Standard.

---

## Phase 1: Build (3 frameworks)

### Framework 1: JWT with short-lived access tokens + refresh tokens

**Core hypothesis**: Eliminate session storage entirely. Each service validates JWTs independently using a shared public key. Short-lived access tokens (5 min) limit exposure; refresh tokens (24h, stored in an opaque table) allow re-issuance.

**Why it holds**: This is the standard pattern for multi-service architectures. No service needs to call a central session store on every request — token validation is stateless and local. The 5-minute access token window means revocation is at most 5 minutes delayed, which is acceptable for most applications.

**Strongest advantage**: Zero inter-service auth latency. Each service validates independently. No single point of failure for request-time auth.

**Most likely failure point**: Token revocation is fundamentally asynchronous. If an account is compromised, the attacker has up to 5 minutes of access after revocation. For a system handling financial data, this may be unacceptable. The refresh token table becomes a single point of failure for token renewal.

### Framework 2: Centralized session service (Redis-based)

**Core hypothesis**: A dedicated session service (Redis cluster) holds all session state. Every service calls it on each request. Sessions are instantly revocable. The session service is the single source of truth for auth state.

**Why it holds**: Instant revocation — delete the session, all 12 services see it on the next request. No token window vulnerability. The operational pattern is well-understood: Redis is battle-tested for session storage, and the request-volume latency is predictable (~1ms for local Redis).

**Strongest advantage**: Instant revocation. No window of vulnerability. If a session is terminated, every service sees it immediately.

**Most likely failure point**: Every request adds a Redis call. At 10K requests/second across 12 services, that's 10K Redis calls/second. If Redis is unreachable (network partition, failover), ALL 12 services reject ALL authenticated requests. The session service becomes a hard dependency that can take down the entire system.

### Framework 3: Sticky sessions + shared session store (hybrid)

**Core hypothesis**: Use load balancer sticky sessions to route each user to the same service instance. Each instance maintains a local session cache backed by a shared Redis store. On cache miss, fetch from Redis. On session write, write-through to Redis.

**Why it holds**: Best-case latency (local cache hit) with eventual consistency to shared store. If Redis is down, existing sticky sessions continue to work — only new sessions and cross-instance moves are affected. Degrades gracefully rather than failing hard.

**Strongest advantage**: Graceful degradation. Redis being down doesn't break existing user sessions — only affects new logins and instance migrations.

**Most likely failure point**: Sticky sessions break when instances scale up/down or are replaced (deployments, crashes). During a rolling deploy, every user is re-routed to a new instance, triggering 100% cache miss rate and Redis load spike. The graceful degradation property is weakest precisely during the most common operational event (deployment).

---

## Phase 2: Debate (Round 1 of 2)

### Skeptic attacks (Round 1)

**On Framework 1**: Builder claims "zero inter-service auth latency" but ignores token introspection overhead. When a service needs to check "is this token still valid?" beyond expiry (e.g., after a password change), it MUST call the auth server. So the claim of zero network calls only holds for the happy path. The moment you add real-world revocation requirements, Framework 1 converges on Framework 2 — you build a centralized auth service anyway.

**On Framework 2**: The "hard dependency" failure mode is worse than Builder states. It's not just "Redis is down → auth fails." It's "Redis has a GC pause / network blip → auth latency spikes → all 12 services time out → cascading failure." This is a known anti-pattern in microservice architectures. Builder's 1ms Redis latency assumption is p50 — p99 can be 50-100ms under load, which adds unacceptable latency to every authenticated request.

**On Framework 3**: The "graceful degradation" property is weaker than Builder claims for a different reason: sticky sessions break when USERS move. Mobile users switching between WiFi and cellular change IP addresses. Users on laptops dock/undock. Every network transition defeats sticky sessions. In a mobile-heavy user base (which Builder didn't specify but should have), sticky sessions provide ~70% hit rate, not ~95%.

### Builder responds (Round 1)

**To Framework 1 attack**: Accepted and revised. Skeptic is right — real-world revocation (password change, admin lockout, suspicious activity detection) requires a centralized check. Revised Framework 1: JWT for the happy path + a lightweight revocation cache (Redis, read-only, updated on revocation events). Most requests validate locally; revocation-critical requests (after password change, admin action) check the cache. This keeps the zero-latency property for >99% of requests while closing the revocation window. The revocation cache is simpler than a full session service — no writes on the auth path, no session creation, just a key lookup.

**To Framework 2 attack**: Partially accepted, partially defended. The p99 latency concern is valid — a loaded Redis cluster can spike. But this is a solved operational problem: Redis Sentinel/Cluster with local read replicas in each service's network segment brings p99 under 5ms. The real question is whether the team has the operational expertise to run Redis at this reliability level. If they don't, Framework 2 is riskier than I estimated. Revision: Framework 2 is recommended ONLY if the team has production Redis experience OR uses a managed Redis service with a >99.99% uptime SLA.

**To Framework 3 attack**: Accepted. I missed the mobile user impact on sticky session hit rate. Given that the user base is mobile-heavy (assumption now explicit), the 70% hit rate means 30% of requests hit Redis anyway — close to Framework 2's overhead, but with the added complexity of sticky session management. This framework weakens significantly for mobile-heavy deployments.

### Skeptic attacks (Round 2)

**On revised Framework 1**: The revocation cache solution is clever, but it reintroduces the exact dependency Builder was trying to avoid. "Lightweight" Redis is still Redis. If the revocation cache is down, do tokens fail open (security risk) or fail closed (availability risk)? Builder's revision dodges this question — it says "most requests validate locally" but doesn't specify the failure mode for the cache lookup path.

**On revised Framework 2**: Builder's concession is appropriate — Redis at >99.99% is achievable but requires either a managed service (vendor lock-in for auth) or deep operational expertise. The deeper issue: even with managed Redis, the network hop adds latency. In a multi-region deployment (which 12 services implies), the Redis call might cross availability zones, adding 5-10ms per request. That's tolerable for user-facing requests but problematic for service-to-service calls where auth is checked on internal APIs.

**On Framework 3**: (Framework 3 was already weakened in Round 1 — Builder accepted the mobile weakness. Skeptic notes this but doesn't add new attacks.)

### Builder responds (Round 2)

**To Framework 1, Round 2**: Accepted and addressed. The failure mode question is critical. Specification: if the revocation cache is unreachable, fail CLOSED for write operations (POST/PUT/DELETE), fail OPEN for read operations (GET). Rationale: reads are informational — stale data is better than no data. Writes are mutating — stale auth is worse than denied writes. This is a trade-off, not a solution, and it should be explicitly configured per-endpoint based on sensitivity.

**To Framework 2, Round 2**: Defended. Cross-AZ latency is 1-2ms in modern cloud providers (AWS us-east-1 AZ-to-AZ is typically <1ms), not 5-10ms. The 12-service architecture doesn't necessarily imply multi-region — it could be single-region multi-AZ, which is the standard pattern. Multi-region adds a different problem (session replication across regions) that none of the three frameworks handle well. If multi-region is a hard requirement, we need a Framework 4.

### Information Increment Gate

Round 2 check: "Did I introduce new attack angles?" 
- Framework 1: YES — challenged the failure mode of the revocation cache.
- Framework 2: YES — challenged cross-AZ latency assumptions and multi-region implications.
- Framework 3: NO — no new attack surface.

3/5 NO overall → Gate triggers. Debate ends after Round 2.

---

## Phase 3: Audit (Analyst + Operator — parallel)

### Analyst audit (summary)

**Framework 1 (revised JWT + revocation cache)**:
- Logical chain: JWT validation is stateless ✓. Revocation cache adds a conditional dependency — the logic holds if and only if the fail-open/fail-closed policy is correctly configured per endpoint. This is a configuration risk, not a logical flaw.
- Skeptic meta-audit: Skeptic's second-round attack (failure mode of cache) was the strongest challenge across all frameworks. Builder's fail-open-for-reads/fail-closed-for-writes response is principled but requires per-endpoint discipline — a single misconfigured endpoint creates a security hole.
- Verdict: **Viable with caveats** — strongest overall but the per-endpoint configuration discipline is non-trivial.

**Framework 2 (revised centralized session service)**:
- Logical chain: Sound. Revocation is instant by definition. The dependency concern is mitigated by managed Redis but not eliminated — a cloud provider outage is rare but not impossible.
- Skeptic meta-audit: Skeptic's multi-region question exposes the real scope limit of Framework 2. If multi-region is required, Framework 2 needs a session replication strategy that Builder hasn't provided.
- Verdict: **Viable with caveats** — best for instant revocation needs, operationally heavier.

**Framework 3 (sticky sessions hybrid)**:
- Logical chain: Weakened by the mobile user discovery. The 70% hit rate means the complexity of sticky sessions isn't justified by the benefit.
- Verdict: **Weakened** — still functional but not competitive with Frameworks 1 or 2 for mobile-heavy deployments.

### Operator audit (summary)

**Framework 1**:
- Vagueness flags: "short-lived" — specify 5 minutes. "fail closed for writes" — enumerate which endpoints are writes.
- Precondition: Team needs to implement and manage a per-endpoint auth configuration system. This doesn't exist yet.
- First step: Deploy a Redis instance (or use managed Redis) as a revocation cache. Before any JWT work, verify Redis p99 latency under simulated load.

**Framework 2**:
- Preconditions: Team MUST have managed Redis or Redis operational expertise. If neither, this framework is not executable.
- First step: Run a Redis benchmark at projected peak load (10K req/s) to establish p99 latency baseline before making architecture decisions.
- Reversibility: Hard to reverse — every service integrates the session service call. Switching away requires changing auth logic in all 12 services.

**Framework 3**: Actionability verdict — Not actionable for mobile-heavy deployments. The sticky session approach doesn't justify its complexity at 70% hit rate.

---

## Phase 4: Converge

### Editor compression

**[Established]** JWT + short-lived access tokens (5 min) + revocation cache (Redis, fail-open for reads, fail-closed for writes) is the recommended architecture for this context, assuming single-region multi-AZ deployment.

**[Probable]** The centralized session service (Framework 2) is a better choice IF instant revocation is a hard requirement AND the team has Redis operational expertise or managed Redis. The trade-off is operational complexity vs. revocation immediacy.

**[Contested]** Whether the 5-minute revocation window in Framework 1 is acceptable depends on the application's security requirements. For most consumer applications, 5 minutes is acceptable. For financial, healthcare, or enterprise admin applications, it may not be. Builder and Skeptic agree on the facts but the user must decide based on their risk tolerance.

**[Speculative]** If the system grows to multi-region, none of the three frameworks handle cross-region session management well. A global session replication layer (not designed here) would be needed.

### Expander (partial output)

**Adjacent applications**: The JWT + revocation cache pattern applies to any multi-service system with stateless auth needs — API gateways, service meshes, CDN edge functions.

**Open frontier**: The single most impactful unknown is the per-endpoint auth configuration system that Framework 1 requires. If building this is a 2-week task, Framework 1 wins decisively. If it's a 2-month task, Framework 2 becomes more attractive.

---

## Standard Output (abbreviated — full 6 fields as per template)

### 1. Main Conclusion
**[Established]** JWT with 5-minute access tokens + Redis revocation cache (fail-open reads, fail-closed writes) provides the best latency/security/operational balance for a single-region 12-service deployment.

### 2. Reasoning Basis
1. **[Established]** Stateless JWT validation eliminates per-request auth latency for >99% of requests — only revocation-critical events hit the cache.
2. **[Probable]** The 5-minute revocation window is acceptable for most consumer applications; financial/healthcare apps may need Framework 2's instant revocation.
3. **[Established]** Managed Redis at >99.99% uptime makes the revocation cache failure risk acceptable for reads (fail-open), with writes protected (fail-closed).
4. **[Speculative]** If multi-region deployment is a future requirement, all three frameworks need significant revision — cross-region session management is unsolved here.

### 3. Main Counterexamples or Risks
1. Per-endpoint auth configuration discipline — a single misconfigured endpoint creates a security hole if fail-open is incorrectly applied to a write endpoint.
2. If instant revocation is a hard requirement (financial, healthcare, admin tools), Framework 1's 5-minute window is unacceptable → use Framework 2.
3. If the team lacks Redis experience AND managed Redis is not an option, both Framework 1 and 2 carry operational risk that Framework 3 (despite its other weaknesses) partially avoids.

### 4-6. (continued per output schema — abbreviated for example)
