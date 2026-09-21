TailorTech

Tailoring as easy as ordering food. Customers book a home visit, an executive measures them and collects fabric, and the stitched garment is delivered with live order tracking.

This is a customer-facing MVP built for the TailorTech internship assignment.

Live site: https://YOUR-USERNAME.github.io/tailortech/
Demo video: ADD-LINK-HERE
What it does
Feature	Where
Landing page, how it works (7 steps), rate card, FAQ	#/
Service booking: name, phone, address, garment, pickup date and slot, fabric option, notes	#/book
Order confirmation: order ID, booking details, ETA	#/confirmed/:id
Dashboard: active and past orders with progress	#/dashboard
Order details: full info and a 7-stage tracking timeline	#/order/:id
Profile: personal details and saved addresses	#/profile
AI style assistant: outfit suggestions by occasion and budget	#/stylist
Run it locally

There is no build step. The whole app is one file, index.html.

bash
git clone (https://abhishek9517482591-alt.github.io/tailortech/)
python3 -m http.server 8000

Open http://localhost:8000. You can also double-click index.html to open it directly.

Deploy

The site is hosted on GitHub Pages: Settings, Pages, Deploy from a branch, main, / (root). Every push to main redeploys it.

Tech
Plain HTML, CSS and JavaScript in a single file, with hash-based routing (no framework, no build)
Data stored in the browser with localStorage (orders and profile), with an in-memory fallback if storage is blocked
Fonts: Bricolage Grotesque and Instrument Sans from Google Fonts
Light and dark theme through CSS variables, responsive down to phone width, with a bottom tab bar on mobile
Product decisions
Book in one screen. Booking is a single page with a live summary (garment, visit, expected delivery, estimated price) instead of a multi-step wizard, so people can see everything they are committing to.
No advance payment. Booking and the home visit are free, and the stitching charge is paid on delivery. This removes the biggest barrier to a first order.
Tracking modelled on food delivery. The seven stages from the brief appear as a timeline with timestamps, a highlighted current step, and the expected delivery date, so customers do not need to call anyone.
Repeat orders are easy. Addresses and details are saved to the profile and prefill the booking form. "Book this again" copies an earlier order.
Fabric choice. Customers either hand over their own fabric or ask the executive to bring swatches.
Rate card instead of product cards. Prices are shown as a tailor's rate card with starting prices and turnaround days. Tapping a row starts a booking for that garment.
Mobile first. Most customers will book from a phone, so navigation moves to a bottom tab bar on small screens.
AI feature: style assistant

Customers describe an occasion, budget and fit preference and get a garment, fabric, colour, fit and a detail to tell the tailor. Each answer includes a button to book the suggested garment, which drops the suggestion into the booking notes.

The model is given the real service facts (garment list, prices, turnaround) and told not to invent others.
Fallback: if the live model is unavailable, the assistant shows built-in rule-based suggestions and labels them. On GitHub Pages the live model is not available, so this fallback is what runs there.
Production plan: call an LLM API from a backend route so the API key stays secret, add rate limiting, and cache common questions.
Demo limitations
Orders and profile live in the visitor's browser only. They are not shared between devices.
Order stages are advanced by hand with a clearly labelled "Move to next step" demo button. In production, staff and tailors would update them.
Executive names, prices and turnaround times are placeholders.
There is no login, payment or real notification.
Production architecture (planned)

Stack: Next.js frontend, FastAPI backend, PostgreSQL (for example on Supabase).

users          id, name, phone, email, created_at
addresses      id, user_id, label, line, city, pincode, is_default
measurements   id, user_id, garment_type, values (json), taken_at
orders         id (TT-xxxxxx), user_id, address_id, garment_type, pickup_at, slot,
               fabric_option, notes, est_price, eta_date, current_status, created_at
order_events   id, order_id, status, created_at, actor
order_events gets one row per stage change. It drives the tracking timeline, keeps a full history, and lets us measure how long each stage takes.
Measurements are stored separately so repeat orders do not need another visit.
Each order references an address at booking time so later profile edits do not change old orders.

API sketch

POST /orders                 create a booking
GET  /orders                 list my orders
GET  /orders/{id}            order details with events
POST /orders/{id}/cancel     cancel before fabric collection
PATCH /orders/{id}/status    staff only, appends an order event
GET/PUT /profile             profile
CRUD /addresses              saved addresses
POST /assistant              style assistant (server-side LLM call)
Scalability notes
Index orders(user_id, created_at) and order_events(order_id) to keep the dashboard fast.
Keep API servers stateless behind a load balancer, with Postgres as the source of truth.
Push status changes to customers through notifications (WhatsApp, SMS, push) or websockets instead of polling.
Add role-based access for executives, tailors and delivery staff, each with their own view.
Assign executives by area and slot capacity, which is the main operational challenge.
Cache and rate-limit the AI endpoint, and keep a fallback response.
What I would build next
A real backend and database with login by phone OTP
Staff tools to update order stages, with customer notifications
Saved measurements per garment, reused on repeat orders
Executive assignment by area and slot
Payments on delivery, with ratings and reviews
