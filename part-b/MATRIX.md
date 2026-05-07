# Impact vs Effort Matrix

## The Matrix

|                   | Low Effort                                                     | High Effort                                               |
|-------------------|----------------------------------------------------------------|-----------------------------------------------------------|
| **High Impact**   | Problem 5: Multi-Day Time Display                               | Problem 1: Tatkal Virtual Queue, Problem 2: Search Filters, Problem 3: Seat Hold, Problem 4: Auto-Refresh, Problem 6: Waitlist Alerts |
| **Low Impact**    | None of the six solutions fit cleanly here                      | None of the six solutions fit cleanly here                |

## How I Scored Each Dimension

### Impact Scoring (1–5)
I scored Impact based on:
- Number of users affected (from Part A frequency analysis)
- Whether the problem is in the core booking flow
- Severity of consequence for the user

### Effort Scoring (1–5)
I scored Effort based on:
- Number of system components touched
- Whether new infrastructure is required
- Risk of breaking existing flows
- Railway API dependencies

---

## Placement Justifications

### Problem 1: Tatkal Virtual Queue System — High Impact / High Effort
Part A says this breaks for 20–40 lakh users in the 10:00 AM Tatkal window, and it hits the core booking flow at the exact moment users are trying to complete payment. The spec shows this needs a backend queue service, Redis or equivalent, WebSocket or polling support, booking claim logic, and changes to the booking and payment handoff flow, so the implementation effort is clearly high. This belongs in the high-impact/high-effort quadrant because it is the most severe problem, but it also needs the most new infrastructure and the highest coordination risk.

### Problem 2: Search Filter Persistence and Server-Validated Results — High Impact / High Effort (Updated)
Part A says this affects nearly everyone who searches for trains, and it directly wastes time and trust by showing filters that reset or leak stale quota data. Peer review surfaced critical complexity: multi-tab session synchronization with collision detection, 30-minute session expiry logic, rate-limit fallback strategies, and A/B-test measurement methodology add significant coordination overhead. The spec requires careful backend and frontend integration to prevent session leakage and handle Railway API rate limits correctly, so effort rises from low to high. High impact remains because the user pain is broad and search is the entry point to booking.

### Problem 3: Seat Selection Persistence with Temporary Hold — High Impact / High Effort
Part A says this affects 15–25% of seat-selection sessions and is especially harmful for families, elderly passengers, and accessibility-sensitive users who need a specific berth. The spec requires a booking draft store, temporary seat holds, backend release logic, draft restoration APIs, and resilient frontend state across navigation and orientation changes. This is high impact and high effort because it changes a critical booking path and must coordinate seat inventory safety with durable client and server state.

### Problem 4: Auto-Refresh Availability with Freshness Indicator — High Impact / High Effort (Updated)
Part A says this impacts about 12 lakh daily users and creates repetitive refresh work. Peer review challenged the low-effort claim by asking about Railway API rate limits, network degradation, and 2G fallback strategies. The updated spec now includes adaptive polling (15s on good connections, 30s on poor, disabled on 2G), request coalescing, page-visibility detection, and precise freshness SLA (never > 20s stale). This complexity requires careful instrumentation, monitoring, and integration testing, so effort rises to high. The critical refinement: measurement methodology now requires A/B testing and connection-quality segmentation to isolate the auto-refresh impact from other conversion factors.

### Problem 5: Multi-Day Time Display Clarification — Low Impact / Low Effort
Part A says this affects a large number of overnight searches, but the consequence is confusion and detours rather than direct booking failure or data loss. The spec is mainly a formatting and display consistency change across search and details responses, with next-day labels and clearer timestamps. This is low impact/low effort because it is important for clarity, but it is still mostly a presentation fix rather than a system-wide workflow change.

### Problem 6: Waitlist Status Alerts and Confirmation Notifications — High Impact / High Effort
Part A says this affects roughly 1.2–2.4 lakh users per day when waitlisted bookings are in play, and the consequence is missing a confirmation that can change travel plans. The spec needs a notification event pipeline, subscription storage, dispatch logic, and front-end opt-in controls, plus integration with SMS or push providers. This is high impact/high effort because it solves a serious user problem, but it depends on a new outbound notification system and careful event handling to avoid duplicate or false alerts.

## Recommended Sprint Order
1. Problem 5: Multi-Day Time Display Clarification — fastest high-impact win with minimal infrastructure cost; safe to ship immediately and build team momentum.
2. Problem 2: Search Filter Persistence and Server-Validated Results — high impact but now recognized as higher-effort due to multi-tab session handling and rate-limit awareness; prioritize after the quick win to prove more complex search work.
3. Problem 4: Auto-Refresh Availability with Freshness Indicator — large search-layer effort with complex rate-limit, degradation, and measurement logic; pair with Problem 2 to consolidate search infrastructure investments.
4. Problem 6: Waitlist Status Alerts and Confirmation Notifications — notification infrastructure is new; schedule after search wins to avoid context switching and to establish team delivery patterns.
5. Problem 3: Seat Selection Persistence with Temporary Hold — critical booking integrity work with backend coordination; schedule after notification system exists to leverage similar event patterns.
6. Problem 1: Tatkal Virtual Queue System — largest operational and infrastructure lift; only after the team has shipped 4-5 features to prove reliability and velocity.
