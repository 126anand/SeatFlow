# SeatFlow backend

Node.js + Express + PostgreSQL + Socket.IO. This is the server side for the SeatFlow app. It covers the features that cannot be real inside a single web page.

## What is here
| Feature | Where |
|---|---|
| Double-booking impossible (segment booking, seat holds) | `db/schema.sql` (`no_overlapping_segments` EXCLUDE constraint), `src/booking.js` |
| Phone OTP login, rotating refresh tokens, role checks | `src/auth.js` |
| Signed, expiring QR tickets | `src/qr.js` (tested), `POST /driver/scan` in `src/server.js` |
| Khalti and eSewa top-ups, idempotent wallet ledger | `src/payments.js` |
| Push notifications (Web Push) and inbox | `src/push.js`, `public/sw.js` |
| Live GPS and real-time events | `src/realtime.js` |
| Hold expiry and no-show release jobs | `expireHolds`, `releaseNoShows` in `src/booking.js` |
| Install as an app, offline shell | `public/manifest.webmanifest`, `public/sw.js` |
| Real street map (OpenStreetMap) | `public/osm-map.html` |

## Run it
```bash
cp .env.example .env        # fill in secrets
npm install
createdb seatflow && npm run db:init
npm test                    # 9 pure-logic tests, no database needed
npm start
```
Run behind HTTPS. Set `NODE_ENV=production` and implement `sendSms()` in `src/auth.js` for your SMS provider.

## What still needs your accounts or data
- SMS gateway credentials (OTP).
- Khalti and eSewa merchant keys. The code follows their v2 flows; check the current provider docs before going live.
- VAPID keys for push (`npx web-push generate-vapid-keys`).
- Seed data: stops, routes, route_stops (with `fare_from_origin`), buses, seats.
- The front end (`seatflow-app.html`) still uses local demo data. Replace its local state with calls to these endpoints and the Socket.IO events (`BUS_LOCATION_UPDATED`, `SEAT_RELEASED`).

## Security notes
Passwords are not used. Every booking is validated on the server. QR tokens are HMAC signed and expire. Refresh tokens rotate. Payments are confirmed with the provider's server-side lookup, and credits are idempotent. Rate limits are on all routes, with stricter limits on auth.
