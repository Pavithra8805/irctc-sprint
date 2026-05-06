# IRCTC Problem Discovery — Part A

---

## Problem 1: Tatkal Booking Crashes at 10:00 AM [Given]

**What is broken:**
The IRCTC server crashes or becomes unresponsive at exactly 10:00 AM every morning when Tatkal quota opens. Users who successfully reach the payment page often have their sessions dropped, their selected seats re-released, and their OTPs delayed — causing the booking to fail at the final step despite completing every earlier step. The system provides zero feedback during the freeze, leaving users unable to determine if their request is queued, failed, or succeeded.

**Affected users:**
Every Indian trying to book a Tatkal ticket — roughly 20–40 lakh active users in the 9:58–10:05 AM window. This disproportionately impacts people in Tier 2 and 3 cities who depend on train travel and cannot afford to miss a booking window.

**Frequency:**
Daily. Every single morning at 10:00 AM. The pattern has existed for years and is a known, documented failure that IRCTC has attempted to patch multiple times without solving the root cause.

**Current flow — step by step:**
1. User opens IRCTC at 9:50 AM, logs in, searches for train and selects a train from the search results
2. User selects Tatkal quota — page shows availability as "Available 12" at 9:55 AM
3. User fills passenger details (pre-filled if saved) and clicks "Book Now" at 9:59:45
4. At 10:00:00 — page freezes. Loading spinner appears. No progress feedback is displayed
5. After 15–45 seconds: either HTTP 502 error, session timeout, or CAPTCHA reset occurs
6. User refreshes — logged out or session expired. Logs back in. Train shows "Tatkal WL 1" — quota is gone
7. User has no way to know if their payment was attempted or not — checks bank statement in panic

**Where exactly it breaks:**
Step 4–6: The critical failure is that the system provides zero feedback during the freeze. The moment 10:00 AM hits, 40 lakh concurrent requests hit a single stateless endpoint designed to handle a fraction of that traffic. The session state is not persisted, so when the connection times out, the user is logged out without knowing whether their booking succeeded, was queued, or failed entirely.

**Impact:**
Users lose booking opportunities within seconds. The lack of feedback creates panic and repeat clicks, compounding the server load. Users verify payment status manually through bank statements and SMS, adding 5-10 minutes of anxiety post-failure.

---

## Problem 2: Search Filters Do Not Work Reliably [Given]

**What is broken:**
The train search results page has filters for quota type, class, availability, and departure time. These filters frequently either do not apply correctly, reset when the page refreshes, or show trains that do not match the selected filter criteria. The quota filter is the most unreliable — users see "WL 34" seats even after filtering for "Available" only.

**Affected users:**
Every user who searches for trains — all 8 crore registered users. Senior citizens and first-time users who rely on filters to find accessible coaches or specific departure times are disproportionately affected because they do not know to distrust the filter output.

**Frequency:**
Inconsistent — the filters work correctly roughly 60–70% of the time. The failure rate increases during high-traffic periods (morning and evening hours). The "quota" filter is the most unreliable, failing 35–40% of the time.

**Current flow — step by step:**
1. User enters source (e.g., Delhi), destination (e.g., Mumbai), and date in search form — clicks "Search Trains"
2. Results appear showing 20–40 trains on the page — overwhelming without filtering
3. User selects "Sleeper Class" and "Available" from the filter panel on the left sidebar
4. Page reloads — some trains that are "WL" (waitlisted) still appear in the filtered results
5. User clicks on a train — the fare page shows class as "Sleeper WL 34". Filter said "Available only"
6. User clicks back — filter has reset to "All Classes" and "All Quotas" — must reapply manually
7. User abandons filtering, manually scans all 40 trains — adds 8–15 minutes to the search process

**Where exactly it breaks:**
Step 3–6: Filters are applied client-side on a cached result set that may already be stale. The availability data is fetched synchronously from a Railway backend API with a 2–5 minute cache. When the page refreshes to show "live" availability, the filter state object is not persisted in local storage, forcing users to reapply filters after each refresh. The quota filter fails because the backend sometimes returns old quota data mixed with fresh data.

**Impact:**
Users give up on filtering and resort to manual scanning. This adds 5–10 minutes to search time and increases bounce rate. Accessibility is harmed — users unable to quickly find their preferred class/quota may miss bookings.

---

## Problem 3: Seat Selection Resets Randomly [Given]

**What is broken:**
During the booking flow, when a user selects a specific seat in the seat map, the selection is sometimes lost when they proceed to the next step. They arrive at the passenger details page with either a different seat assigned or the "Auto" option selected — meaning they may end up in any seat on the train, including upper berths they did not want.

**Affected users:**
Users booking for families (especially those with elderly or children who need lower berths), users with physical disabilities who require specific berths, and anyone who has paid extra attention to selecting a preference. Estimated 30–40% of all booking attempts involve a seat preference.

**Frequency:**
Occurs in approximately 15–25% of sessions involving seat map interaction. The rate is significantly higher on mobile (35%) than desktop (12%) due to re-renders during state transitions.

**Current flow — step by step:**
1. User selects a train, class, and quota from search results — proceeds to seat selection page
2. Seat map loads — shows available (white), booked (grey), selected (blue) berths clearly marked
3. User clicks a lower berth for an elderly passenger — berth turns blue (selected) and highlights as "Selected"
4. User clicks "Proceed to Passenger Details" — page loads passenger form with berth selection area
5. Seat preference field shows "Auto" or a different berth number (e.g., "Upper 37") — not what was selected
6. User clicks back to reselect — seat map reloads, and the previously selected seat now shows as "Booked" (grey)
7. User proceeds with auto-assignment — boards train to find they have an upper berth despite preferring lower

**Where exactly it breaks:**
Step 3–5: The seat selection state is stored in component-level React state or memory. When the page navigation occurs from the seat map component to the passenger form component, the state is not passed through URL parameters or Redux/session storage. On mobile, a screen orientation change or any re-render triggers a component unmount, clearing local state entirely. The seat map and form are two separate API calls — between them, the user's selected seat can be booked by another user, causing the system to ignore their selection on commit.

**Impact:**
Users end up with unwanted seats (upper berths when they needed lower for elderly parents, middle seats when they prefer aisles for accessibility). Users must rebook or request seat changes after boarding — adding post-purchase friction and dissatisfaction.

---
