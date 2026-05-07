# Impact vs Effort Matrix

## The Matrix

|                   | Low Effort                                                     | High Effort                                               |
|-------------------|----------------------------------------------------------------|-----------------------------------------------------------|
| **High Impact**   | Problem 2: Search Filters, Problem 4: Auto-Refresh Availability | Problem 1: Tatkal Virtual Queue, Problem 3: Seat Hold, Problem 6: Waitlist Alerts |
| **Low Impact**    | Problem 5: Multi-Day Time Display                               | None of the six solutions fit cleanly here                |

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

### Problem 2: Search Filter Persistence and Server-Validated Results — High Impact / Low Effort
Part A says this affects nearly everyone who searches for trains, and it directly wastes time and trust by showing filters that reset or leak stale quota data. The spec mostly needs search API query handling, session storage, URL state, and freshness metadata, which is meaningful work but not a new platform layer. This is high impact with comparatively low effort because the user pain is broad, while the change is mainly about correcting state handling and validation rather than adding major infrastructure.

### Problem 3: Seat Selection Persistence with Temporary Hold — High Impact / High Effort
Part A says this affects 15–25% of seat-selection sessions and is especially harmful for families, elderly passengers, and accessibility-sensitive users who need a specific berth. The spec requires a booking draft store, temporary seat holds, backend release logic, draft restoration APIs, and resilient frontend state across navigation and orientation changes. This is high impact and high effort because it changes a critical booking path and must coordinate seat inventory safety with durable client and server state.

### Problem 4: Auto-Refresh Availability with Freshness Indicator — High Impact / Low Effort
Part A says this impacts about 12 lakh daily users and creates repetitive refresh work across multiple trains and classes, even though the underlying booking flow still exists. The spec is mostly a frontend and search-layer change: batched availability fetches, visibility-aware polling, timestamps, and stale-state messaging. This lands in high impact/low effort because it improves a very common search task with relatively contained changes and no new booking infrastructure.

### Problem 5: Multi-Day Time Display Clarification — Low Impact / Low Effort
Part A says this affects a large number of overnight searches, but the consequence is confusion and detours rather than direct booking failure or data loss. The spec is mainly a formatting and display consistency change across search and details responses, with next-day labels and clearer timestamps. This is low impact/low effort because it is important for clarity, but it is still mostly a presentation fix rather than a system-wide workflow change.

### Problem 6: Waitlist Status Alerts and Confirmation Notifications — High Impact / High Effort
Part A says this affects roughly 1.2–2.4 lakh users per day when waitlisted bookings are in play, and the consequence is missing a confirmation that can change travel plans. The spec needs a notification event pipeline, subscription storage, dispatch logic, and front-end opt-in controls, plus integration with SMS or push providers. This is high impact/high effort because it solves a serious user problem, but it depends on a new outbound notification system and careful event handling to avoid duplicate or false alerts.

---

## Recommended Sprint Order
1. Problem 4: Auto-Refresh Availability with Freshness Indicator — fastest trust win with high daily reach and low implementation risk.
2. Problem 2: Search Filter Persistence and Server-Validated Results — fixes a broad search trust issue before users even enter the booking funnel.
3. Problem 5: Multi-Day Time Display Clarification — a low-cost clarity improvement that can ship quickly alongside search work.
4. Problem 6: Waitlist Status Alerts and Confirmation Notifications — larger than the first three, but it unlocks a meaningful experience for waitlisted users.
5. Problem 3: Seat Selection Persistence with Temporary Hold — critical booking integrity work that needs tighter backend coordination.
6. Problem 1: Tatkal Virtual Queue System — biggest operational lift and highest-risk change, so it should come after the smaller fixes prove the team’s delivery path.
