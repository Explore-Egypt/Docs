# Explore Egypt — Requirements Gathering

**Document version:** 1.0  
**Prepared:** 16 October 2026  
**Project:** Explore Egypt — Tourism Discovery and Trip Planning Platform  
**Status:** Requirements baseline for team review

> **Scope note:** These requirements describe the proposed first release (MVP). The frontend framework and AI provider are not selected yet. Features identified as “Should” or “Could” may be postponed if they threaten delivery of the core workflow.

## 1. Purpose and Product Overview

Explore Egypt is a website that helps people discover Egyptian tourism destinations, particularly lesser-known locations, learn practical information about them, discover tours provided by approved tourism companies, and submit tour booking requests. A planner feature may use stored destination data to suggest a draft itinerary.

The platform is a **managed marketplace and discovery service**, not a tour operator. Tourism companies manage their own tour offerings and booking requests, while platform administrators control publication and company approval.

## 2. Stakeholder Analysis

| Stakeholder | Main needs | Expected interaction | Priority |
|---|---|---|---|
| Visitor / prospective traveler | Browse destinations, search/filter, see practical details, compare tours | Uses public pages without needing an account for browsing | High |
| Registered traveler | Save places, use/save an itinerary, send a booking request, track booking status, write reviews | Uses account and traveler dashboard | High |
| Tourism company representative | Maintain company profile, publish tours and schedules, review booking requests | Uses company dashboard after account/company approval | High |
| Platform administrator | Manage destination data/categories, approve or reject companies, moderate published content | Uses protected admin dashboard | High |
| Project team/developers | Maintain code, database, test data, configuration, and documentation | Uses GitHub, API docs, local development environment | High |
| Project instructor/evaluator | Verify requirements, implementation, team contribution, quality, and demo | Reviews documentation and end-to-end demonstration | High |
| External AI provider (optional) | Receive an appropriately scoped prompt and return itinerary text | Called by the backend only if configured | Medium |

### 2.1 User roles and access
- **Visitor:** view published places, categories, approved companies, and published tours.
- **Traveler:** visitor permissions plus booking requests, favorites, itinerary saving, and reviews.
- **Company user:** manage only the company they own/are assigned to, its tours/schedules, and requests for those tours. Publishing rights depend on company approval.
- **Admin:** manage and moderate the platform. Admin permissions must be restricted to authorized users.

The implementation may use ASP.NET Core Identity roles or an equivalent secure authorization mechanism. Passwords must never be stored as plain text.

## 3. Product Goals and Success Criteria

1. Help users find destinations using search and useful filters.
2. Present destination details, including listed fees and nearby facilities when reliable data exists.
3. Let visitors compare real or clearly labeled demo tour offerings and schedules.
4. Support a traceable booking request and company decision process.
5. Give administrators a practical way to manage published destination/company data.
6. Demonstrate an itinerary-planning feature based on known places rather than invented destinations.
7. Make the project reproducible by following the repository setup guide.

## 4. Functional Requirements

Priorities: **Must** = required for the core MVP; **Should** = important but can be trimmed if schedule pressure occurs; **Could** = optional enhancement.

### 4.1 Accounts and access control

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-01 | The system shall allow a user to register with a name, unique email, and password. | Must | A valid registration creates an account; a duplicate email is rejected with a useful message. |
| FR-02 | The system shall allow registered users to log in and log out. | Must | Valid credentials authenticate; invalid credentials do not reveal sensitive account details. |
| FR-03 | The system shall enforce role-based authorization for Traveler, Company, and Admin actions. | Must | A traveler cannot open protected company/admin actions by calling the URL/API directly. |
| FR-04 | The system shall allow a registered user to view and update basic profile information. | Should | A user can update their own permitted fields but cannot edit another user's data. |

### 4.2 Destination discovery

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-05 | The system shall list published tourism places with a name, image where available, governorate, and category. | Must | Only published records are visible publicly. |
| FR-06 | The system shall support searching places by name or keyword. | Must | Search results match the submitted term without exposing unpublished records. |
| FR-07 | The system shall support filtering places by governorate and category. | Must | A user can combine supported filters and clear them. |
| FR-08 | The system shall support parent/child categories, including religious tourism. | Must | A child category may be connected to a parent category and shown appropriately in browse filters. |
| FR-09 | The system shall provide a place details page with description, location/address, coordinates when available, category, images, and related information. | Must | Selecting a place opens the correct record and its images/details. |
| FR-10 | The system shall flag selected lesser-known destinations as “Hidden Gems.” | Should | A published place marked as a Hidden Gem can be included in a featured list/filter. |
| FR-11 | The system shall display fee records associated with a place when known, including fee type/name, amount, currency, description, and active status. | Should | Missing/unverified fee data is shown as unavailable, not as an invented zero cost. |
| FR-12 | The system shall display nearby facilities such as hospitals, pharmacies, hotels, restaurants, and ATMs when verified information is available. | Should | Each displayed facility has a type and location; distance is shown only when calculated or sourced appropriately. |
| FR-13 | The system shall support place images with an image URL, alt text, and a primary-image indicator. | Must | A place can have multiple images and one image can be selected as the primary image. |

### 4.3 Companies, tours, and schedules

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-14 | The system shall allow a company representative to submit company information for review. | Must | A new company has a pending verification status until an admin decides. |
| FR-15 | The system shall allow an admin to approve or reject a company application. | Must | Only approved companies may publish public tours. |
| FR-16 | The system shall allow an approved company to create and edit its own tour records. | Must | Company A cannot modify Company B's tours. |
| FR-17 | A tour shall include a title, description, price per person, duration, maximum participants, and publication status. | Must | Invalid values (such as a negative price or non-positive capacity) are rejected. |
| FR-18 | A tour shall be able to include multiple places in a defined stop order. | Must | Tour stops appear in ascending order and reference existing place records. |
| FR-19 | A company shall be able to create tour departure schedules with start/end dates, capacity, and status. | Must | Only valid future/allowed schedules are offered for booking; invalid date ranges are rejected. |
| FR-20 | Visitors shall be able to browse published tours and view the company, price, duration, places/stops, and available schedules. | Must | Public results only include tours allowed to be shown. |

### 4.4 Booking workflow

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-21 | A registered traveler shall be able to submit a booking request for a specific departure and participant count. | Must | Booking requires login, a valid departure, and a positive participant count. |
| FR-22 | The system shall store the participant count and a total-price snapshot when a booking request is created. | Must | Later tour-price edits do not silently rewrite an existing booking's stored total. |
| FR-23 | A company shall be able to view and accept or reject booking requests for its own tours. | Must | Companies cannot decide bookings belonging to other companies. |
| FR-24 | The system shall show travelers their booking records and current status. | Must | A traveler sees only their own bookings and the current status. |
| FR-25 | The system shall use clear booking states such as Pending, Confirmed, Rejected, and Cancelled, with controlled transitions. | Must | Invalid state changes are rejected or handled by an explicit policy. |
| FR-26 | The system shall check capacity before confirming a booking to reduce overbooking risk. | Must | A confirmation cannot exceed remaining capacity; the capacity check and update should be handled safely by the backend. |
| FR-27 | The system shall allow cancellation only where the current booking status and cancellation policy permit it. | Should | Disallowed cancellations are not accepted and the reason is communicated. |

**Booking rule:** the first release does not process payments. A booking request is not proof of payment or a guarantee that the trip has been delivered. Payment instructions, if any, are informational and managed outside the website in the MVP.

### 4.5 Itinerary planner

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-28 | The planner shall collect basic preferences such as area, trip duration, interests, and optional budget. | Should (core differentiator) | Required input is validated before generating suggestions. |
| FR-29 | The planner shall select candidate destinations from published records in the platform database. | Should (core differentiator) | Every recommended place can be opened as a place record in Explore Egypt. |
| FR-30 | The planner shall return a day-by-day draft itinerary with places and a suggested order. | Should (core differentiator) | The result clearly identifies itself as a suggestion and does not assert unverified opening times, fees, or tour availability. |
| FR-31 | A logged-in traveler shall be able to save and reopen a generated itinerary. | Should | Saved itineraries belong to the authenticated traveler and can be viewed later. |
| FR-32 | If the external AI provider is unavailable, the application shall show a useful error or use a simple database-based fallback rather than failing unrelated website functions. | Should | Place browsing and booking pages remain available when the AI call fails. |

### 4.6 Reviews and favorites

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-33 | A logged-in traveler shall be able to add a place to or remove it from favorites. | Should | The same user/place pair cannot be duplicated and is private to the owner. |
| FR-34 | A logged-in traveler shall be able to submit a place rating and comment. | Should | Rating must be within the configured range (e.g., 1–5) and input is validated. |
| FR-35 | The system shall allow admins to hide or moderate inappropriate reviews. | Could | A hidden review is not displayed publicly. |

### 4.7 Administration and content management

| ID | Requirement | Priority | Acceptance check |
|---|---|---|---|
| FR-36 | An admin shall be able to create, update, publish/unpublish, and archive tourism place records. | Must | Invalid or incomplete records cannot be published. |
| FR-37 | An admin shall manage governorates and tourism categories. | Must | Place records reference valid governorates/categories. |
| FR-38 | An admin shall manage place images and, where included in the release, fee and nearby-facility records. | Should | Related records are linked to the correct place and are not duplicated unnecessarily. |
| FR-39 | An admin shall be able to review company verification status and moderate company/tour content. | Must | An unapproved company cannot publish tours. |
| FR-40 | The system shall provide a basic admin overview of key counts such as places, companies, tours, and bookings if time permits. | Could | Counts correspond to current database records. |

## 5. Non-Functional Requirements

| ID | Quality attribute | Requirement / target | Verification method |
|---|---|---|---|
| NFR-01 | Security | Passwords are hashed using a standard identity framework; secrets are not committed to GitHub. | Review configuration and authentication implementation. |
| NFR-02 | Authorization | API endpoints enforce ownership and role permissions on the server, not only by hiding UI controls. | Attempt unauthorized API calls in tests. |
| NFR-03 | Data validation | Inputs are validated server-side, including price, rating, date ranges, capacity, and required fields. | Negative/invalid input tests. |
| NFR-04 | Privacy | A user can access only their own profile, bookings, favorites, and saved itineraries, except authorized administrators. | Access-control tests. |
| NFR-05 | Usability | Core navigation is understandable and works on common desktop/mobile viewport sizes. | Team walkthrough and responsive UI check. |
| NFR-06 | Performance | On the team demo environment and seeded dataset, core list/detail API calls should generally respond within 2 seconds, excluding external AI response time. | Record timing, dataset size, and environment. |
| NFR-07 | Reliability | Failure of the optional AI provider must not disable place browsing, tour browsing, or bookings. | Simulate AI timeout/error. |
| NFR-08 | Maintainability | Code is separated into sensible modules; naming and error handling are consistent. | Code review and repository structure check. |
| NFR-09 | Reproducibility | Another team member can run the application using the README and sample configuration. | Fresh setup by a teammate. |
| NFR-10 | API clarity | Implemented API endpoints are available through Swagger/OpenAPI or equivalent endpoint documentation. | Endpoint documentation review. |
| NFR-11 | Compatibility | The website should work in current versions of major browsers used for the demonstration. | Manual smoke testing. |
| NFR-12 | Data quality | Place, fee, facility, and tour facts should include reliable source notes during seed-data preparation; unknown values remain unknown. | Seed-data review. |
| NFR-13 | Accessibility | Forms have labels, meaningful image alt text, visible focus states, and sufficient text/background contrast where practical. | UI checklist. |
| NFR-14 | Backup/demo readiness | Seed data and setup instructions are available so the main demo can run without manual re-entry of all data. | Re-run the demo from a clean database. |

## 6. Use Cases

### UC-01 — Discover a place
- **Primary actor:** Visitor or Traveler
- **Preconditions:** Published destination records exist.
- **Main flow:** Open Explore Places → apply search/filter → select a result → view place details, photos, category, location, fees/facilities where available, reviews, and related tours.
- **Alternative flows:** No matching places → show an empty state and allow filters to be cleared; missing fee/facility data → show “Information not available” instead of a fabricated value.
- **Postcondition:** The user has viewed a destination; no login is required.

### UC-02 — Submit a tour booking request
- **Primary actor:** Registered Traveler
- **Supporting actor:** Company representative
- **Preconditions:** Traveler is logged in; a published tour has a valid departure.
- **Main flow:** Open tour details → choose departure → select participant count → review total price → submit request → system saves it as Pending → company reviews it → company confirms or rejects it → traveler sees updated status.
- **Alternative flows:** Not logged in → prompt login then return to the selected tour; invalid participant count or no capacity → prevent submission/confirmation and explain the issue; company no longer approved → prevent new public bookings according to the policy.
- **Postcondition:** A booking record exists with a status and price snapshot; no payment is processed by Explore Egypt.

### UC-03 — Company manages a tour
- **Primary actor:** Approved Company representative
- **Preconditions:** User has access to an approved company account.
- **Main flow:** Open company dashboard → create/edit tour → select place stops and their order → create departure schedule → validate details → publish tour.
- **Alternative flows:** Company is pending/rejected → allow profile review but prevent public publishing; invalid date/price/capacity → show validation errors.
- **Postcondition:** A valid tour/schedule is stored and is visible to visitors only if publication and approval rules permit.

### UC-04 — Admin approves a company
- **Primary actor:** Admin
- **Preconditions:** A company application exists.
- **Main flow:** Open company management → review submitted information → approve or reject → save status → company permissions/public visibility update accordingly.
- **Postcondition:** The company status is recorded and controls whether it may publish tours.

### UC-05 — Generate an itinerary
- **Primary actor:** Visitor or Traveler
- **Preconditions:** Published destinations are available; planner is enabled.
- **Main flow:** Enter area, duration, interests, and optional budget → backend selects suitable stored places → planner orders and describes suggestions → display day-by-day draft → signed-in user may save it.
- **Alternative flows:** No matching places → suggest adjusting preferences; LLM/API unavailable → use database-based fallback or display a clear retry message.
- **Postcondition:** A draft itinerary is shown; it is stored only when the user chooses to save it and authentication is required by that save policy.

## 7. User Stories and Acceptance Criteria

| ID | User story | Priority | Acceptance criteria |
|---|---|---|---|
| US-01 | As a visitor, I want to browse published destinations so I can discover places in Egypt. | Must | Published places are listed; each card has core identifying information; unpublished items are hidden. |
| US-02 | As a visitor, I want to filter destinations by governorate and category so I can narrow results. | Must | Filters work individually and together and can be cleared. |
| US-03 | As a visitor, I want a complete place details page so I can decide whether to visit. | Must | Page shows available description, images, location, category, and any verified listed fees/facilities. |
| US-04 | As a visitor, I want to see tours related to a place so I can find organized ways to visit it. | Must | Place page links to published tours whose tour stops include that place. |
| US-05 | As a company representative, I want to create a tour with ordered stops and schedules so travelers can understand the offer. | Must | Approved company can manage its own tour; stops and schedules validate correctly. |
| US-06 | As a traveler, I want to submit a booking request for a specific departure so the company can respond. | Must | Login is required; participant count and total price are stored; initial status is Pending. |
| US-07 | As a company representative, I want to accept or reject incoming requests so I can manage my tours. | Must | Company sees only its own requests and can use only allowed status transitions. |
| US-08 | As a traveler, I want to track my booking status so I know whether the company accepted it. | Must | The traveler sees only their own bookings and current statuses. |
| US-09 | As an admin, I want to approve companies so only reviewed companies can publish tours. | Must | Unapproved companies cannot publish public tours. |
| US-10 | As an admin, I want to maintain places and categories so destination information remains useful. | Must | Admin can create/update/publish records and invalid relationships are rejected. |
| US-11 | As a traveler, I want to save places so I can return to them later. | Should | Favorite toggle works and duplicate user/place pairs are prevented. |
| US-12 | As a traveler, I want to rate a place so I can share feedback. | Should | Rating is validated and the review is associated with the correct user and place. |
| US-13 | As a traveler, I want itinerary suggestions based on my interests and duration so planning takes less effort. | Should | Output is based on actual stored places and clearly presented as a draft. |
| US-14 | As a traveler, I want to save an itinerary so I can reopen it later. | Should | Saved itinerary and ordered items are tied to the correct user. |
| US-15 | As a visitor, I want to see nearby facilities so I can plan practical details. | Should | Facility name/type/location are displayed only when available and checked. |

## 8. Business Rules and Data Rules

1. Only published place records are visible on public pages.
2. Categories may have a parent category; religious tourism is a supported category type, not a separate hard-coded module.
3. A place belongs to one governorate and one primary category in the MVP; if multi-category tagging is later required, replace the single category foreign key with a many-to-many join table.
4. A place may have multiple images, fee records, and nearby-facility records.
5. An unapproved company cannot publish tours.
6. Each tour belongs to one company and may include multiple places through ordered tour stops.
7. A tour may have multiple scheduled departures. A booking must reference a specific departure, not only the general tour.
8. Booking total price is stored as a snapshot at submission. Any change to price later must not silently alter historical booking totals.
9. Capacity must be checked when confirming a booking; confirmation should not exceed available seats. The implementation should handle the capacity check and update safely on the server.
10. Booking status changes must follow an explicit set of allowed transitions.
11. The MVP does not collect payments or represent a Pending/Confirmed booking as a completed payment.
12. Users can access only their own private account data; company users can manage only records belonging to their company; admin functions require explicit authorization.
13. The AI planner uses published place records as its candidate set. It must not invent price, opening times, live availability, or nearby facilities.
14. Fees, facilities, photos, and destination details are data fields, not promises that the information is complete or current. Where data is missing, the UI should communicate that clearly.
15. Images must be suitable for use in the project and attribution/source notes should be retained when required.

## 9. Prioritization and Release Boundaries

### Must-have release path
1. Public destination list and destination details.
2. Search/filter and categories/governorates.
3. Company approval and tour/schedule data.
4. Tour details connected to the places included in the tour.
5. Registration/login and role protection.
6. Booking request, company decision, and traveler status page.
7. Basic admin content management.
8. Setup documentation and tests for core flows.

### Should-have if the core path is stable
- Fees and nearby facilities on destination details.
- AI itinerary planner with a database-based fallback.
- Favorites and place reviews.
- Saved itineraries.

### Could-have / defer first if behind schedule
- Advanced analytics dashboard.
- Review moderation beyond simple hide/unhide.
- Email notifications.
- Map directions/route optimization.
- External payment gateway and commissions — explicitly out of scope for MVP.

## 10. Dependencies and Open Decisions

| Decision / dependency | Needed by | Proposed resolution |
|---|---|---|
| Frontend framework | 21 Oct 2026 | Choose the option the team can deliver and maintain; record the decision in README. |
| Local .NET/SQL Server setup for all members | 22–23 Oct | Test setup on at least two team devices; document exact steps and versions. |
| ERD and database naming | 30 Oct; design package due 6 Nov | Agree on core entities and booking/schedule relationships before APIs expand. |
| Company demo data and destinations | 29 Oct | Start with a manageable set of verified destination records and clearly labeled demo tours. |
| AI provider/API key | Before planner implementation | Choose only after the core place API is stable; never commit secrets. |
| Facility/fee data sources | Before publishing such data | Keep source notes; omit unsupported information rather than guessing. |
| Cancellation policy and seat reservation timing | Before booking acceptance tests | For MVP, company acceptance confirms a booking only if capacity remains; cancellations follow explicit allowed states. |

## 11. Traceability: Requirements to Test Evidence

At minimum, create test cases for the following critical flows:

| Test ID | Requirement coverage | Expected result |
|---|---|---|
| TC-01 | FR-05, FR-09 | Visitor can list places and open the correct details page. |
| TC-02 | FR-06, FR-07, FR-08 | Search and category/governorate filters return expected records. |
| TC-03 | FR-14, FR-15, FR-16 | Unapproved company cannot publish; approved company can manage its own tour. |
| TC-04 | FR-18, FR-19, FR-20 | Tour stops and departure schedules display in correct order/details. |
| TC-05 | FR-21, FR-22, FR-24 | Signed-in traveler submits a request; total is snapshotted; user can see their own status. |
| TC-06 | FR-23, FR-25, FR-26 | Company can decide only its own bookings; invalid status/capacity changes are blocked. |
| TC-07 | FR-03, NFR-02, NFR-04 | Unauthorized user cannot access another user's booking or company/admin actions. |
| TC-08 | FR-28, FR-29, FR-30, FR-32 | Planner suggestions reference actual places; AI failure does not break core pages. |
| TC-09 | FR-11, FR-12, FR-38 | Available fee/facility details show correctly; missing data is not invented. |
| TC-10 | NFR-09, NFR-14 | Teammate can initialize sample database and run the project using README. |

## 12. Approval and Change Control

- The team should review this document before treating requirements as frozen.
- Each Must-have requirement should map to one or more GitHub Issues.
- Any new feature should identify the value it adds, implementation cost, and what existing work it might delay.
- After the feature-freeze date, 27 November 2026, add only critical fixes or changes required for a stable final demonstration.
- Requirements changed after review should be recorded in GitHub Issues/PRs so the reason and implementation history remain visible.

---

**Document owner:** Explore Egypt team  
**Review status:** Confirm frontend selection, team assignment, and requirement priorities with all six members and the instructor.
