<p align="center">
  <h1 align="center">Ryde</h1>
  <p align="center">A rider/driver ride-hailing platform — built to expose the parts of an Uber-style app that are actually hard, not the parts that are just CRUD with a map on top.</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React_Native-Expo-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" alt="React Native" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Stripe-635BFF?style=for-the-badge&logo=stripe&logoColor=white" alt="Stripe" />
  <img src="https://img.shields.io/badge/NativeWind-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="NativeWind" />
</p>

<p align="center">
  <a href="https://ryde-five.vercel.app"><b>Live app</b></a>
</p>

---

## Overview

<p>
Most ride-hailing tutorials stop at "show a map and a book button." The actual product has three parties (rider, driver, platform) who each need a different, partially-overlapping view of the same trip, a payment that has to be authorized before a stranger gets in a car and captured after they get out safely, and a matching problem that's really a real-time state machine wearing a UI.
</p>

<p>
Ryde is built around that state machine, not around the map widget. The map is the least interesting part of this codebase; the ride lifecycle is the part that had to be right.
</p>

## Design Decisions

<p><b>A ride is a state machine, not a database row you update in place.</b></p>
<p>
<code>requested → matched → en_route_to_pickup → in_progress → completed | cancelled</code> is treated as an explicit sequence rather than a status string any client can overwrite. The alternative — a rider's app and a driver's app both PATCHing a <code>status</code> field whenever they feel like it — is how you end up with a ride that's simultaneously "cancelled" on one screen and "in progress" on the other. Transitions are validated server-side against the current state; an app that thinks a ride is still <code>requested</code> cannot force it into <code>completed</code>.
</p>

<p><b>Fare estimation and fare charged are two different numbers, deliberately.</b></p>
<p>
The estimate shown before booking is a quote, not a commitment — traffic, route changes, and wait time all move the final number. Collapsing "estimate" and "final fare" into one field is the fastest way to generate a support ticket and a chargeback. They're computed and stored separately, with the estimate frozen at booking time so a rider can be shown exactly what they were quoted versus what they were charged.
</p>

<p><b>Payments are authorize-then-capture, not charge-on-booking.</b></p>
<p>
Stripe's payment intent flow lets a card be authorized (funds held) when a ride is requested and captured only once the trip actually completes. Charging up front means refunding every no-show or driver-side cancellation; authorizing up front and capturing on completion means the failure path is "release the hold," which is cheaper, faster, and doesn't touch the rider's statement at all.
</p>

<p><b>Driver location updates are throttled and ephemeral, not logged as history by default.</b></p>
<p>
Live tracking needs frequent position updates while a ride is active; it does not need those updates retained forever. Location pings during an active ride drive the live map and are not treated as a permanent trail unless a ride's actual route (pickup → dropoff) is what's being stored for the trip record. Continuous fine-grained location history is a liability, not a feature, for a product whose users are, by definition, telling the app where they're going.
</p>

## Architecture

```
apps/
  rider/                 React Native + Expo — booking, live tracking, payment, ratings
  driver/                React Native + Expo — dashboard, ride requests, earnings, navigation
  admin/                 Web panel — user management, ride analytics, payment oversight

api/
  routes/
    auth/                 Rider + driver auth, role-scoped tokens
    rides/                Request → match → lifecycle transitions
    fares/                Estimate calculation, final fare computation
    payments/             Stripe payment intents, capture, refunds
    drivers/               Availability toggle, earnings, ride history
    admin/                 User + ride + payment oversight

db/                       PostgreSQL — riders, drivers, rides, fares, payments, ratings
```

<p>
Rider, driver, and admin are three separate apps against one API, not one app with three modes gated by a role flag. A driver mid-shift and a rider mid-booking need almost nothing from each other's surface area; sharing a codebase across them buys very little and makes it easy to accidentally ship rider-only UI into the driver build.
</p>

## Tech Stack

<table>
  <tr><th>Layer</th><th>Choice</th><th>Reasoning</th></tr>
  <tr><td>Mobile</td><td>React Native + Expo</td><td>One codebase for iOS/Android, OTA updates for non-native changes without a store review cycle</td></tr>
  <tr><td>Language</td><td>TypeScript</td><td>Ride state, fare breakdowns, and payment payloads are exactly the kind of data where a typo in a field name should fail at compile time, not in a driver's car</td></tr>
  <tr><td>Backend</td><td>PostgreSQL</td><td>Rides, fares, and payments are relational and need transactional integrity — a fare and its payment record cannot silently diverge</td></tr>
  <tr><td>Payments</td><td>Stripe (Payment Intents)</td><td>Authorize/capture semantics fit a service charged after the fact for a variable amount, not a fixed-price checkout</td></tr>
  <tr><td>Styling</td><td>Tailwind (NativeWind)</td><td>Fast iteration on two structurally different UIs (rider, driver) without hand-rolled stylesheets for each</td></tr>
</table>

## Trade-offs

<ul>
<li><b>No real-time driver-matching optimization.</b> Matching is closest-available rather than a proper dispatch algorithm weighing ETA, driver rating, and demand balancing across a zone. That's the right call for a demo-scale system and the wrong one at any real volume — it's the first thing that would need to change.</li>
<li><b>Fare estimation is distance/time-based, not a full pricing engine.</b> No surge pricing, no zone-based multipliers, no dynamic demand response. Real fare engines are a product decision as much as an engineering one, and this doesn't attempt to make that call.</li>
<li><b>No background location tracking.</b> Location is only tracked while the app is foregrounded and a ride is active. More privacy-respecting, and it also means a driver who backgrounds the app mid-ride can appear to "disappear" from the rider's map — a real trade-off, not a free one.</li>
<li><b>Admin panel trusts the API's role checks, not a separately audited permission system.</b> Admin actions (refunds, account suspension) go through the same auth layer as everything else. A platform actually handling money at scale would want a narrower, separately reviewed admin surface with an audit log of who did what — this doesn't have one yet.</li>
<li><b>Single-region deployment assumption.</b> Nothing here handles multi-currency, multi-region compliance (KYC for drivers varies wildly by jurisdiction), or timezone-correct earnings reporting. It's built like a single-city product because that's the problem it was built to solve.</li>
<li><b>No automated tests on the ride state machine.</b> This is the part of the system where a bug is a rider left in "matched" limbo or a driver charged twice — exactly the surface that most needs coverage and currently has none.</li>
</ul>

## Roadmap

- [ ] Automated tests around ride-state transitions and payment capture/refund paths
- [ ] Proper dispatch/matching algorithm (ETA + rating + zone balancing) in place of closest-available
- [ ] Admin action audit log, separate from general auth
- [ ] Surge/zone-based fare adjustments
- [ ] Multi-region driver onboarding (KYC varies by jurisdiction)

## Running Locally

<p>Requires Node 18+, Expo CLI, and a PostgreSQL instance.</p>

```bash
git clone https://github.com/LaeeqtheDev/Ryde.git
cd Ryde
npm install
```

Environment (`.env` — never committed):

```
DATABASE_URL=
STRIPE_SECRET_KEY=
```

```bash
npm start                         # start dev server
npx pod-install && npm run ios    # iOS
npm run android                   # Android
```

## Author

<p>
<b>Syed Laeeq Ahmed</b> — Full-Stack Lead Engineer @ North Foundry
</p>

<p>
<a href="https://laeeqthedevportfolio.vercel.app">Portfolio</a> · <a href="https://linkedin.com/in/syed-laeeq-ahmed/">LinkedIn</a> · <a href="https://github.com/LaeeqtheDev">GitHub</a> · laeeqthedev@gmail.com
</p>

## License

<p>All rights reserved. Source is public to read; not licensed for reuse or redeployment.</p>
