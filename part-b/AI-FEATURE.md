# AI Feature Specification: Waitlist Confirmation Forecast

## Problem It Solves
This feature addresses Part A Problem 6, where waitlisted users must manually check PNR status because IRCTC does not proactively tell them when a WL ticket is likely to confirm. The AI layer does not replace notifications; it helps users understand how likely confirmation is and whether they should turn on alerts immediately.

## Proposed Feature — User Perspective
When a user opens the PNR Status page or the Booked Tickets page for a waitlisted ticket, they see a small forecast card above the status table. The card says things like “Confirmation likely in the next 6–12 hours” or “Low chance of confirmation before departure.” The user can then decide whether to enable alerts, change travel plans, or keep checking manually.

The feature is most useful for users who already have a WL ticket and need a quick, plain-language estimate without reading raw booking history or repeatedly polling the site.

## Model or API Choice
Use an XGBoost classification model exposed through a Google Vertex AI prediction endpoint.

Why this choice:
- The task is structured tabular prediction, not free-form text generation.
- XGBoost is strong on mixed categorical and numerical booking features and is easier to calibrate than a large language model.
- Vertex AI gives a managed online endpoint, model monitoring, and version rollback if accuracy drops.

## Training or Input Data
The model needs historical waitlist outcome data with features such as:
- PNR booking class, quota, route, train number, source station, destination station, and journey date.
- Current WL position, booking timestamp, charting time proximity, and ticket age.
- Historical cancellation rate for the route and train type.
- Remaining seat inventory and past WL-to-CNF transitions for the same route.
- Aggregated demand signals from IRCTC booking history, not raw personal content.

Data sources:
- IRCTC historical booking DB for past PNR status transitions.
- Railway availability/status APIs for live WL position and inventory context.
- Existing user booking history only as aggregated features, not as personalized profiling.

Availability:
- Most of the required data already exists in booking and status systems, but the status-transition history must be retained in a training dataset if it is not already stored for long enough.
- If historical labels are incomplete, the first release can train on routes with the cleanest WL-to-CNF history and expand later.

## How Output Is Shown to the User
On the PNR Status page and Booked Tickets page, add a compact forecast card above the existing status table. The card uses the same visual placement as the alert control shown in [assets/wireframes/waitlist-alert-subscription.png](../assets/wireframes/waitlist-alert-subscription.png).

Example output:
- “Confirmation likely: 82%”
- “Most likely window: 6–12 hours before charting”
- “Reason: similar routes on this train usually clear late cancellations”

The card has three states:
- Green: high confidence, with a clear probability and recommended action to enable alerts.
- Amber: medium confidence, with a softer estimate and a prompt to keep alerts on.
- Hidden: no trustworthy prediction, so only the standard status table and alert toggle appear.

The forecast never replaces the real PNR status table. It sits above it as a helper, and the user can still subscribe to the normal status alert from the same screen.

## Confidence Threshold and Fallback
Show the forecast only when the calibrated confidence is at least 0.75 and the model score is stable across the last few retraining checks.

If confidence is between 0.60 and 0.75, show an amber “uncertain” note rather than a precise promise.

If confidence is below 0.60, the endpoint times out, or the model is unavailable, hide the forecast card entirely and fall back to the standard deterministic experience:
- show current WL/CNF status,
- keep the alert opt-in control visible,
- let the user manually refresh PNR status.

If the model confidence is low but the user has already opted in to alerts, the notification system still runs normally. The AI layer only changes how much guidance is shown, not whether the alert pipeline works.

## Success Metrics
- Users who enable PNR alerts after seeing the forecast increase by at least 20%.
- Manual PNR refresh checks per waitlisted booking drop by at least 30%.
- The forecast calibration error stays below the agreed threshold after launch.
- False high-confidence predictions remain below 5% on monitored routes.

## Limitations and Risks
- WL confirmation patterns vary by route, season, holidays, and charting rules, so the model can drift quickly if it is not retrained.
- A bad prediction could create false hope or unnecessary anxiety, so the UI must avoid language that sounds guaranteed.
- Routes with sparse historical data will produce weaker predictions and may never be suitable for the forecast card.
- The model can inherit bias from routes that have more historical data or more frequent cancellations, which could make some corridors look more certain than they really are.
- If the live Railway data is delayed or incomplete, the forecast should disappear rather than guess.

