# Explore Egypt - System Analysis & Design

**Document version:** 1.0  
**Prepared:** 6 November 2026  
**Project:** Explore Egypt - Tourism Discovery and Trip Planning Platform  
**Submission:** System Analysis & Design  
**Status:** Design baseline for team and instructor review

> **Design status:** This document turns the current Planning and Requirements baselines into a coherent system design. ASP.NET Core Web API, C#, SQL Server and Entity Framework Core are the proposed backend/data stack. The frontend framework and external AI provider remain decisions to be confirmed by the team. Deployment topology is proposed, not a claim that the system is already deployed.

## 1. Executive Summary

Explore Egypt is a managed tourism discovery and trip-planning marketplace for Egyptian destinations, including lesser-known places. Visitors can discover places and tours, registered travelers can submit booking requests and track their status, approved tourism companies can manage their tours and departures, and administrators manage destination content and company approval. An optional AI-assisted planner proposes itineraries from published destination records already stored in the platform.

The design uses a layered web architecture: a browser-based frontend calls an ASP.NET Core Web API; application services enforce business rules and authorization; Entity Framework Core persists data in SQL Server. The optional AI provider is called by the backend only after candidate places have been retrieved from the database.

## 2. Problem Statement and Objectives

### 2.1 Problem statement

Travelers may need to consult multiple sources to discover Egyptian destinations, understand practical visit information, compare organized tours, and plan a coherent trip. Lesser-known destinations may be harder to discover. Explore Egypt consolidates destination discovery, tour discovery and basic itinerary planning in one website.

### 2.2 Objectives

1. Provide a searchable catalogue of Egyptian tourism destinations.
2. Present place details such as governorate, category, images, location, and verified fees/facilities when data is available.
3. Allow approved tourism companies to publish tours and scheduled departures.
4. Let registered travelers submit booking requests and track company decisions.
5. Provide an AI-assisted itinerary proposal grounded in published destination records.
6. Deliver a maintainable, documented and testable academic MVP.

## 3. Scope and Actors

### 3.1 In scope

- Public home page, place browsing, place details, search and filters.
- Authentication and role-based access for travelers, company members and administrators.
- Company registration/review and approval.
- Tour records, ordered tour stops and scheduled departures.
- Booking requests with a Pending/Confirmed/Rejected/Cancelled lifecycle; no online payment in the MVP.
- Basic admin content management for places, categories, companies and publication.
- Optional AI itinerary generation using stored places; favorites, reviews and saved itineraries as planned requirements.

### 3.2 Out of scope for MVP

- Online payments, refunds, commissions and payouts.
- Integration with real third-party company booking engines.
- Mobile native application, live chat, guaranteed live availability, real-time traffic/weather prediction, and production marketplace operations.

### 3.3 Actors and permissions

| Actor | Main capabilities | Access boundary |
|---|---|---|
| Visitor | Browse published places, categories, companies and tours; use public search | Public data only |
| Registered Traveler | Visitor capabilities; submit booking requests; view own bookings; favorites, reviews and saved itineraries | Own private records only |
| Company Representative | Manage assigned company profile, its tours, stops, departures and booking requests | Records belonging to their company only; publishing depends on approval |
| Administrator | Manage places/categories, approve companies, moderate content and users | Protected administrative actions |
| External AI Provider | Return itinerary wording based on supplied candidate places | Optional; called by backend, not directly by the browser |

## 4. Functional and Non-Functional Design Baseline

The requirements IDs below refer to `02_Requirements_Gathering.md`. The system design must implement the Must requirements first. Should/Could requirements may be deferred if needed to protect the end-to-end discovery-to-booking flow.

### 4.1 Main functional capabilities

| Capability | Related requirement IDs | Design responsibility |
|---|---|---|
| Registration, login and role authorization | FR-01 to FR-04 | Identity service; server-side role and ownership checks |
| Place catalogue, search, filters and details | FR-05 to FR-13 | Place catalogue service and normalized place records |
| Company approval, tours and departures | FR-14 to FR-20 | Company/tour services; only approved companies publish |
| Booking request and company decision | FR-21 to FR-27 | Booking service; snapshot total price and capacity validation |
| Itinerary planner | FR-28 to FR-32 | Retrieve eligible published places first; optional LLM formatter and deterministic fallback |
| Reviews and favorites | FR-33 to FR-35 | User/place associations with uniqueness and moderation rules |
| Admin content management | FR-36 to FR-40 | Protected administrative service and publication workflow |

### 4.2 Non-functional constraints

- **Security:** password hashes only; no credentials or API keys committed to GitHub.
- **Authorization:** enforce roles and record ownership in the API, not only in the UI.
- **Validation:** validate all user input on the server, including dates, capacity, prices, ratings and participant counts.
- **Privacy:** users access only their own bookings, favorites and itineraries; company users access only their company's records.
- **Performance target:** core list/detail API requests should generally respond within two seconds on the team's demo environment and seeded dataset, excluding external AI response time.
- **Reliability:** AI/provider failures must not disable place browsing, tour browsing or booking.
- **Reproducibility:** a teammate should be able to initialize the sample database and run the system from the README.
- **Data quality:** unknown fees/facilities remain unavailable; never replace missing facts with invented values or zero-cost assumptions.

## 5. Use Case Model

See `01_use_case` in the diagrams file. The main use cases are:

| Use Case | Primary actor | Preconditions / outcome |
|---|---|---|
| Discover a Place | Visitor / Traveler | Published place records exist; user searches/filters and views details |
| View a Tour and Departures | Visitor / Traveler | Published tour is associated with an approved company |
| Submit a Booking Request | Registered Traveler | Authenticated traveler selects a valid departure and positive participant count; booking saved as Pending |
| Manage a Tour | Approved Company Representative | Company manages only its own tour, ordered stops and schedules |
| Decide a Booking | Company Representative | Only the company owning the tour can accept/reject; capacity is rechecked before confirmation |
| Approve a Company | Administrator | Pending company application is reviewed; approval controls publishing access |
| Manage Place Content | Administrator | Admin creates/updates/publishes places and categories |
| Generate an Itinerary | Visitor / Traveler | Planner selects candidates from published places and returns a draft; login required only to save if that policy is retained |

### 5.1 Critical use case - Submit a booking request

1. Traveler opens a published tour and selects a scheduled departure.
2. Traveler enters participant count and submits the request while authenticated.
3. API validates the input and ownership-independent booking rules.
4. Booking service loads the departure and current tour price.
5. System calculates the total and stores unit/total price snapshots with `Pending` status.
6. Company representative sees the request in the company dashboard.
7. On acceptance, the backend rechecks remaining departure capacity and updates the status in a safe transaction.
8. Traveler sees `Confirmed` or the relevant rejection/conflict result.

The MVP does not process payment. A Pending or Confirmed status is not evidence of payment.

## 6. Software Architecture

### 6.1 Architecture style

Use a **layered modular monolith** for the academic MVP rather than microservices. The modules are separated by responsibility but can be deployed as one ASP.NET Core API. This reduces integration and deployment overhead for a six-person team while preserving clear service boundaries.

### 6.2 Layers and responsibilities

| Layer | Main components | Responsibilities |
|---|---|---|
| Presentation | Public web UI, traveler dashboard, company dashboard, admin dashboard | Render data, collect input, call API; route guards improve UX but do not replace API authorization |
| API / Application | Controllers/endpoints, DTOs, validators, application services | HTTP contracts, authentication, authorization, orchestration and consistent errors |
| Domain | User, Company, Place, Tour, Departure, Booking, Itinerary and related rules | Core business entities, states, relationships and invariants |
| Persistence | EF Core DbContext, mappings, migrations, SQL Server | Data access, constraints, transactions, indexes and seed data |
| External adapter | Optional AI provider adapter | Sends only selected candidate place information and validated preferences; handles timeout/fallback |

### 6.3 Proposed technology stack

| Area | Proposed choice | Decision status |
|---|---|---|
| Backend | ASP.NET Core Web API, C# | Proposed baseline |
| ORM | Entity Framework Core | Proposed baseline |
| Database | SQL Server | Proposed baseline |
| API documentation/testing | Swagger / OpenAPI | Proposed baseline |
| Frontend | Angular or HTML/CSS/JavaScript | Team decision still to be confirmed |
| Version control | GitHub repository, branches and pull requests | In use / intended workflow |
| AI | External LLM API behind backend adapter | Provider not selected; optional feature |
| Packaging | Docker if time permits after local integration | Optional release-stage enhancement |

### 6.4 Request flow

1. Browser sends an HTTPS request to the frontend or API.
2. API authenticates the identity and checks role/ownership policies.
3. Controller validates the request DTO and calls the appropriate application service.
4. Service applies business rules and accesses SQL Server through EF Core.
5. API returns a documented response DTO and appropriate HTTP status.
6. For itinerary generation only, the planner may call an external AI API after it has retrieved candidate places from the database.

## 7. Database Design and Data Modeling

### 7.1 ERD selection

This submission uses the **second ERD (V2)** as the baseline. It models the tourism marketplace more accurately than a flat places/tours/bookings design because it separates tours from scheduled departures and bookings, models ordered tour stops, and supports optional place fees and nearby facilities. The model also supports company membership, user roles, reviews, favorites and saved itineraries.

### 7.2 Entity inventory

| Domain | Entity | Purpose |
|---|---|---|
| Identity | Users | Traveler, company and admin identities |
| Identity | Roles | Role definitions |
| Identity | UserRoles | Many-to-many user/role assignment |
| Identity | Companies | Tourism company profile and verification status |
| Identity | CompanyMembers | Maps users to companies and their access |
| Catalogue | Governorates | Egyptian governorate reference data |
| Catalogue | Categories | Hierarchical tourism categories |
| Catalogue | Places | Destination records and publication state |
| Catalogue | PlaceImages | Multiple images per destination |
| Catalogue | PlaceFees | Optional fee records with currency and validity |
| Catalogue | FacilityTypes | Types such as hospital, pharmacy, hotel, restaurant, ATM |
| Catalogue | Facilities | Nearby facility records and location |
| Catalogue | PlaceFacilities | Associates a destination with a nearby facility |
| Marketplace | Tours | A company's tour offering and base price/duration |
| Marketplace | TourStops | Ordered places included in a tour |
| Marketplace | Departures | Specific scheduled instances with dates and capacity |
| Booking | Bookings | Traveler request for a specific departure and price snapshot |
| Engagement | PlaceReviews | User feedback and moderation status |
| Engagement | Favorites | Saved user/place relationships |
| Planner | Itineraries | Saved itinerary header and preferences |
| Planner | ItineraryItems | Ordered place entries within each itinerary/day |

### 7.3 Logical schema - core attributes

The schema below is a logical design baseline. Concrete SQL types and lengths should be finalized with the team's SQL Server/EF Core implementation and migrations. `Guid` is suggested for primary identifiers of transactional/content entities; integer keys are reasonable for small reference tables.

| Table | Primary key | Important attributes and foreign keys | Key constraints |
|---|---|---|---|
| Users | UserId | FullName, Email, PasswordHash, Phone, IsActive, CreatedAt | Email unique; password hash only |
| Roles | RoleId | Name, Description | Name unique |
| UserRoles | (UserId, RoleId) | FK UserId -> Users; FK RoleId -> Roles | Composite PK |
| Companies | CompanyId | Name, LicenseNumber, VerificationStatus, ReviewedByUserId, ReviewedAt, CreatedAt | Verification status constrained to allowed values |
| CompanyMembers | CompanyMemberId | CompanyId, UserId, IsPrimaryContact, JoinedAt | Unique (CompanyId, UserId) |
| Governorates | GovernorateId | Name, Description | Name unique |
| Categories | CategoryId | ParentCategoryId nullable, Name, Description | ParentCategoryId self-FK; prevent self-parenting |
| Places | PlaceId | GovernorateId, CategoryId, Name, Description, Address, Latitude, Longitude, IsHiddenGem, IsPublished | Valid FKs; coordinate ranges; only published records public |
| PlaceImages | PlaceImageId | PlaceId, ImageUrl, AltText, IsPrimary, DisplayOrder | FK PlaceId; one primary image per place where enforced |
| PlaceFees | PlaceFeeId | PlaceId, FeeName, Amount, Currency, ValidFrom, ValidTo, IsActive | Amount >= 0; validity range consistent |
| FacilityTypes | FacilityTypeId | Name, Description | Name unique |
| Facilities | FacilityId | FacilityTypeId, Name, Address, Latitude, Longitude | FK FacilityTypeId; coordinate ranges if present |
| PlaceFacilities | PlaceFacilityId | PlaceId, FacilityId, DistanceKm nullable | Unique (PlaceId, FacilityId); distance >= 0 |
| Tours | TourId | CompanyId, Title, Description, PricePerPerson, Currency, Duration, IsPublished | Price >= 0; FK CompanyId; publish only if company approved |
| TourStops | TourStopId | TourId, PlaceId, StopOrder | Unique (TourId, StopOrder); positive order |
| Departures | DepartureId | TourId, StartAt, EndAt, Capacity, Status | EndAt > StartAt; Capacity > 0 |
| Bookings | BookingId | DepartureId, TravelerId, ParticipantCount, UnitPriceSnapshot, TotalPriceSnapshot, Currency, Status, CreatedAt | Positive count; total snapshot = unit snapshot x count; valid status |
| PlaceReviews | ReviewId | PlaceId, UserId, Rating, Comment, Status, CreatedAt | Rating within configured range; unique (PlaceId, UserId) if one review per user/place |
| Favorites | (UserId, PlaceId) | UserId, PlaceId, CreatedAt | Composite PK prevents duplicates |
| Itineraries | ItineraryId | UserId, Title, DurationDays, AreaPreference, Budget, CreatedAt | FK UserId; DurationDays > 0 |
| ItineraryItems | ItineraryItemId | ItineraryId, PlaceId, DayNumber, ItemOrder, Notes | Unique (ItineraryId, DayNumber, ItemOrder) |

### 7.4 Relationships and cardinalities

- Governorates `1:N` Places.
- Categories `1:N` Places; Categories also have a self-referencing parent/child relationship.
- Users `M:N` Roles through UserRoles.
- Users `M:N` Companies through CompanyMembers (company permissions are scoped to membership).
- Companies `1:N` Tours; Tours `1:N` TourStops and `1:N` Departures.
- Places `1:N` PlaceImages and `1:N` PlaceFees.
- Places `M:N` Facilities through PlaceFacilities.
- Tours `M:N` Places through ordered TourStops.
- Departures `1:N` Bookings; Users `1:N` Bookings.
- Users `1:N` PlaceReviews and Places `1:N` PlaceReviews.
- Users `M:N` Places through Favorites.
- Users `1:N` Itineraries; Itineraries `1:N` ItineraryItems; Places `1:N` ItineraryItems.

### 7.5 Normalization and integrity

The design targets **Third Normal Form (3NF)** for the core relational data:

- Repeating place images, fees and facilities are stored in child/association tables rather than repeated columns in Places.
- Category and facility type descriptions are reference data, avoiding repeated type text across every record.
- Many-to-many associations (users/roles, places/facilities, tours/places, users/favorites) are represented through junction tables.
- TourStops stores the ordering attribute on the relationship between Tour and Place.
- Bookings reference a specific Departure and retain price snapshots so historical totals are not silently changed when the tour price changes.
- ItineraryItems stores per-day order on the relationship between an itinerary and a place.

Recommended SQL constraints/indexes:

1. Unique index on Users.Email and Roles.Name.
2. Unique composite indexes on UserRoles(UserId, RoleId), CompanyMembers(CompanyId, UserId), PlaceFacilities(PlaceId, FacilityId), Favorites(UserId, PlaceId), and itinerary item order.
3. Index common filters: Places(IsPublished, GovernorateId, CategoryId), Tours(IsPublished, CompanyId), Departures(TourId, StartAt, Status), Bookings(DepartureId, Status), Bookings(TravelerId, CreatedAt).
4. Check constraints for non-negative price/fee, positive capacity/participant count, valid date windows, rating range, and valid coordinates.
5. Restrict deletes for historical booking/tour relationships; prefer archive/soft-delete for published content.

### 7.6 Booking capacity policy

The baseline requirements specify that capacity is checked before confirmation. A Pending request does not reserve seats automatically unless the team explicitly chooses a reservation-hold policy. The company acceptance operation must:

- Begin a database transaction.
- Read the departure capacity and already confirmed participant totals using appropriate locking/concurrency control.
- Verify that `confirmed participants + requested participants <= capacity`.
- Update booking status and commit atomically; otherwise return a conflict and keep the request unconfirmed.

This prevents two concurrent company actions from overselling the same remaining seats. Exact cancellation and seat-release rules remain a business decision to confirm.

## 8. Data Flow and System Behavior

### 8.1 DFD Context (Level 0)

The entire platform is modeled as one process exchanging information with Visitors, Travelers, Company Representatives, Administrators and an optional External AI Provider. All external interactions pass through the platform backend.

### 8.2 DFD Level 1 processes

| Process | Inputs | Data stores / outputs |
|---|---|---|
| P1 Identity & Access | Registration/login credentials, authenticated requests | D1 Users/Roles/CompanyMembers; identity result and access decision |
| P2 Place Discovery | Search terms, filters, place detail requests | D2 Place Catalogue; published place results |
| P3 Company & Tour Management | Company application, tour, stop and departure data | D1 membership/authorization; D3 companies/tours/departures |
| P4 Booking Management | Departure, participant count, company decision | D1 traveler/company identity; D3 tour/departure; D4 bookings and status |
| P5 Itinerary Planning | Area, duration, interests, budget | D2 published places; D5 saved itineraries; optional AI provider |
| P6 Admin & Content | Place/category updates, company decisions, moderation | D1 admin authorization; D2 catalogue; D3 company data; D4 moderation data |

## 9. UML Models

### 9.1 Sequence diagram - booking

See `06_sequence_booking`. The diagram covers request submission and subsequent company decision. It highlights authentication, server-side validation, price snapshotting, departure lookup and a second capacity check at confirmation.

### 9.2 Activity diagram - booking

See `07_activity_booking`. The activity flow includes authentication, input validation, pending request creation, company review, acceptance/rejection and capacity validation.

### 9.3 Booking state machine

Allowed baseline transitions:

- Initial -> Pending when a traveler submits a valid request.
- Pending -> Confirmed when the company accepts and capacity is still available.
- Pending -> Rejected when the company rejects.
- Pending -> Cancelled when a permitted cancellation occurs.
- Confirmed -> Cancelled only when the cancellation policy permits.
- Rejected and Cancelled are terminal for this MVP unless a later explicit business rule is approved.

### 9.4 Class diagram

See `09_class_diagram`. The diagram is a domain-level view. The persistence model will include additional EF Core configuration, audit fields, navigation properties and enums.

## 10. UI/UX Design and Prototyping

The low-fidelity wireframes are in `12_ui_wireframes` and define the information architecture for four high-priority screens:

1. **Home & Discovery:** primary navigation, destination search/filters, featured places and clear entry points to tours/planner.
2. **Place Details:** main image/gallery, description, category/governorate, location, verified fees/facilities, reviews and related tours.
3. **Tour Details & Booking:** company, itinerary/stops, duration, per-person price, departure selector and participant count.
4. **Role Dashboard:** role-specific navigation, summary cards, tables and management actions.

### 10.1 UI guidelines

- Use consistent navigation and terminology across all roles.
- Prioritize destination photography and readable content hierarchy.
- Use responsive card/grid layouts and mobile-friendly form controls.
- Provide clear labels, keyboard focus states, adequate contrast, and image alt text.
- Show explicit empty, loading, success and error states.
- Label demo company/tour records as sample data.
- Show fees and facilities only when sourced/verified; display unavailable information honestly.

## 11. Component and Deployment Design

### 11.1 Components

The proposed modular monolith contains Identity, Catalogue, Marketplace, Booking, Itinerary Planner and Administration components. Each module exposes its service operations through the API and uses a shared persistence boundary through EF Core. Component boundaries are logical boundaries, not separate microservices.

### 11.2 Deployment topology

See `11_deployment_diagram`. The proposed topology contains:

- Client browser on desktop/mobile.
- Frontend static hosting serving the selected web build.
- ASP.NET Core API host.
- SQL Server database host with restricted access and backups.
- Optional external AI API called from the backend over HTTPS.

Hosting vendor, domain, CI/CD provider and production scaling are not selected yet. For the academic demo, a reproducible local deployment is acceptable if no hosted environment is ready.

## 12. API Design Baseline

The API is REST-oriented and versioned under `/api`. Exact controller names and DTO fields may be refined during implementation.

| Method | Proposed route | Purpose | Access |
|---|---|---|---|
| POST | `/api/auth/register` | Create traveler account | Public |
| POST | `/api/auth/login` | Authenticate | Public |
| GET | `/api/places` | List/search/filter published places | Public |
| GET | `/api/places/{id}` | Place details | Public |
| GET | `/api/categories` | Categories tree/list | Public |
| GET | `/api/governorates` | Governorate list | Public |
| GET | `/api/tours` | Published tours and filters | Public |
| GET | `/api/tours/{id}` | Tour details, stops and departures | Public |
| POST | `/api/bookings` | Submit booking request | Traveler |
| GET | `/api/bookings/my` | Current user's bookings | Traveler |
| GET | `/api/company/bookings` | Company's booking requests | Company member |
| PATCH | `/api/company/bookings/{id}/decision` | Accept/reject request | Authorized company member |
| POST | `/api/company/tours` | Create tour | Approved company member |
| POST | `/api/company/tours/{id}/departures` | Add departure | Company member |
| POST | `/api/planner/itineraries/generate` | Generate itinerary draft | Public or authenticated per final policy |
| POST | `/api/itineraries` | Save itinerary | Traveler |
| GET/POST | `/api/admin/places` | List/create admin place records | Admin |
| PATCH | `/api/admin/companies/{id}/decision` | Approve/reject company | Admin |

Use consistent status codes (200/201/204, 400 validation, 401 unauthenticated, 403 forbidden, 404 not found, 409 capacity/state conflict). Do not expose internal exception details.

## 13. Testing and Validation Plan

| Test ID | Scenario | Expected result | Requirement mapping |
|---|---|---|---|
| TC-01 | Browse published places and open details | Correct published record and available details returned | FR-05, FR-09 |
| TC-02 | Search and combine governorate/category filters | Matching records only; unpublished records remain hidden | FR-06 to FR-08 |
| TC-03 | Attempt company publishing while unapproved | API blocks publishing; approved company can manage its own tour | FR-14 to FR-16 |
| TC-04 | Create tour stops and departure schedules | Stops retain order; invalid date/capacity rejected | FR-18, FR-19 |
| TC-05 | Traveler submits a booking | Login required; Pending record stores participant count and price snapshot | FR-21, FR-22, FR-24 |
| TC-06 | Company decides a booking | Only owning company can decide; over-capacity confirmation rejected | FR-23, FR-25, FR-26 |
| TC-07 | Unauthorized access to another user's/company's data | API returns 403/404 without leaking private data | FR-03, NFR-02, NFR-04 |
| TC-08 | Planner generation and AI outage | Recommendations reference stored places; unrelated functions remain usable | FR-28 to FR-32, NFR-07 |
| TC-09 | Fee/facility display | Known values display; missing data is not invented | FR-11, FR-12, FR-38 |
| TC-10 | Teammate runs a clean setup | README and seed data allow local execution | NFR-09, NFR-14 |

### 13.1 Validation strategy

- Unit tests: validators, booking totals, status transition rules and capacity calculations.
- Integration tests: API/database operations, authorization and transactional booking confirmation.
- API smoke tests: Swagger or an HTTP client collection for core endpoints.
- UI acceptance tests: browse place, view tour, submit request, company decision, traveler status.
- Manual security checks: role escalation, cross-user booking access and cross-company data access.

## 14. Security, Privacy and Operational Considerations

1. Use ASP.NET Core Identity or an equivalent standard password-hashing mechanism; never store plaintext passwords.
2. Protect company/admin API operations with role and resource-ownership policies.
3. Keep API keys, DB credentials and JWT signing secrets in environment variables or a secret manager.
4. Validate all inputs server-side; encode output and avoid unsafe dynamic SQL.
5. Restrict database network access to the API host; back up the database and test restore steps for the demo.
6. Avoid collecting unnecessary personal data; use sample data for demo users and companies.
7. Record sources/dates for seed destination data and use images with appropriate permissions/attribution.
8. Apply rate limits or basic abuse controls to authentication and optional AI endpoints where feasible.

## 15. Design Decisions, Assumptions and Open Items

| ID | Decision / assumption | Status / action |
|---|---|---|
| D-01 | Use ERD V2 with separate Departures and Bookings | Adopted in this design baseline |
| D-02 | Backend is ASP.NET Core Web API with C# | Proposed; team confirmation required |
| D-03 | SQL Server and EF Core | Proposed; team confirmation required |
| D-04 | Frontend: Angular or HTML/CSS/JavaScript | Open; record one final decision in repository |
| D-05 | One primary category per place in MVP | Assumption; use many-to-many if multiple tags become required |
| D-06 | CompanyMembers maps users to companies | Adopted logical design; confirm company account policy |
| D-07 | Pending booking does not reserve seats | Proposed interpretation of current requirements; confirm with instructor/team |
| D-08 | Capacity checked atomically during company confirmation | Required design control |
| D-09 | Online payment is outside MVP | Explicitly out of scope |
| D-10 | AI is optional and database-grounded | Provider not selected; fallback required |
| D-11 | Deployment vendor/domain are undecided | Proposed topology only |
| D-12 | Reviews/favorites/saved itineraries may be deferred | Follow Must/Should priorities in requirements baseline |

## 16. Conclusion

The selected V2 data model and layered modular architecture provide a practical foundation for Explore Egypt's MVP. The most important integrity rule is that a booking references a specific departure, stores a price snapshot and is confirmed only after a safe capacity check. The team should approve the open decisions, implement the core place-to-tour-to-booking path first, and keep the AI planner optional until the underlying place data and APIs are stable.

---

**Document owner:** Explore Egypt Team  
**Review status:** Team and instructor review required before treating proposed/open decisions as final.
