# KASHISH Website — Backend & Payment Integration Plan

This covers what turns the front-end prototype into a live, bookable, paid site. Written to hand to a developer or agency.

## 1. What the prototype already does
- Full booking UI: service selection, calendar with mock availability, time slots, client details form, order summary.
- Client-side only. Nothing persists beyond the visitor's own browser, and no payment is actually processed.

## 2. What needs a real backend

### a. Database
A simple relational database (Postgres via Supabase, or Firebase/Firestore for a faster build) with two core tables:

**bookings**
| field | type | notes |
|---|---|---|
| id | uuid | primary key |
| service | text | bridal / softglam / party |
| date | date | |
| slot | text | e.g. "10:00 AM" |
| status | text | pending / confirmed / cancelled |
| client_name | text | |
| phone | text | |
| email | text | |
| venue | text | |
| notes | text | |
| advance_amount | integer | in paise/rupees |
| payment_id | text | gateway transaction reference |
| created_at | timestamp | |

**blocked_slots** (or generate availability from confirmed bookings directly)
| field | type |
|---|---|
| date | date |
| slot | text |
| reason | text (booked / artist unavailable) |

### b. Booking API
Minimal REST endpoints:
- `GET /availability?month=2026-10` → returns booked/blocked dates and slots, computed from the bookings table — replaces the hardcoded mock logic in the prototype.
- `POST /bookings` → creates a `pending` booking and returns a booking id, used to start payment.
- `POST /bookings/:id/confirm` → called by the payment webhook once payment is verified; flips status to `confirmed` and locks the slot.
- Slot locking should happen at `pending` creation with a short expiry (e.g. 10 minutes) so two people can't pay for the same slot simultaneously.

### c. Payment gateway
For India, UPI + cards + net banking in one integration: **Razorpay** is the most common choice (PayU and Cashfree are comparable alternatives).

Flow:
1. Frontend calls `POST /bookings` → gets a `booking_id` and `order_id` from Razorpay (created server-side).
2. Frontend opens Razorpay's checkout with that `order_id` (their JS SDK handles UPI/card/net banking selection — this replaces the current payment-method radio buttons).
3. Razorpay sends a **server-to-server webhook** on successful payment. The backend verifies the signature and calls `POST /bookings/:id/confirm`.
4. Never confirm a booking purely from a client-side "success" callback — always confirm from the verified webhook, since client-side callbacks can be spoofed.

Rough cost: Razorpay charges ~2% per transaction (varies by payment method); no fixed monthly fee on standard plans.

### d. Notifications
- WhatsApp confirmation: WhatsApp Business Platform (via a provider like Interakt, Gupshup, or Twilio) to send an automated confirmation message with the reference code once `confirm` fires.
- Email confirmation: any transactional email service (Resend, SendGrid, AWS SES).

### e. Admin view
A lightweight authenticated page (or just a Retool/Airtable view on top of the same database) for the artist to see upcoming bookings, mark trials, and manually block dates for personal unavailability.

## 3. Hosting
- Frontend: the static site can stay as-is on Vercel, Netlify, or Cloudflare Pages.
- Backend: a small Node/Express or Next.js API layer works well alongside a Supabase/Postgres database; Supabase specifically can also replace most of the custom API with its auto-generated REST + built-in auth if that suits the build speed better than hand-rolled endpoints.

## 4. Suggested build order
1. Database + availability API (kills the hardcoded mock calendar logic).
2. Booking creation endpoint + slot locking.
3. Razorpay order creation + checkout wiring.
4. Webhook verification + booking confirmation.
5. WhatsApp/email notifications.
6. Admin view.

## 5. Before launch — content and account items
- Replace placeholder WhatsApp number, email, Instagram handle, and domain (`kashishmakeup.com` is a placeholder throughout, including in `robots.txt` and `sitemap.xml`).
- Register a Google Business Profile for Raipur — this generally matters more for local map-pack visibility than on-site SEO alone.
- Swap portfolio gradient placeholders for real, compressed photography (WebP, lazy-loaded) and short vertical video clips.
