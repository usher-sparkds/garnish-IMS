<div align="center">

# Garnish IMS

**An operations platform for a multi-campus food business, connecting the kitchen, mobile sales agents, the till, delivery drivers and management in real time**

![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Expo](https://img.shields.io/badge/Expo_(React_Native)-000020?style=flat-square&logo=expo&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Source](https://img.shields.io/badge/source-private-lightgrey?style=flat-square)

</div>

---

## 1. Project Overview

**What it is.** Garnish Hub is the internal operating system of a food business that sells across university campuses. It runs the whole day in one place: kitchen preparation, stock handed to roaming salespeople, sales and payments, a walk-in till, deliveries and online orders, staff shifts and commission.

**The problem it solves.** A business with a central kitchen, staff selling on foot across campuses loses track of things easily. Where is the stock right now? Who sold what, and was the money actually received? Who is on shift? What commission does each person earn? Without a system, all of that lives in WhatsApp messages and hard to manage excel spreedsheets, and stock and cash go missing. Garnish IMS records every movement of stock and money, so managers can see the operation live and staff are paid accurately.

**Who it's for.** The business's own team. Each role has its own dashboard:
- **Kitchen staff**: prepare and dispatch stock, manage daily inventory
- **Salespeople**: sell from their stock bag, take payments, earn commission
- **Cashiers**: run a point-of-sale till for walk-in customers
- **Drivers**: deliver online orders and move stock between the kitchen and salespeople
- **Overseers (management)**: monitor everything, approve exceptions, manage the menu and staff

---

## 2. What It Does

**Kitchen**
- Start and end shifts, and take daily opening and closing stock counts that carry forward from one day to the next
- Receive live "notify kitchen" stock requests from salespeople, prepare them and mark them ready
- Dispatch stock to salespeople or drivers, confirmed with a **QR-code handover**
- Accept, prepare and update online orders, and chat with the salesperson or driver handling a delivery
- Record write-offs and donations (waste, staff meals, giveaways), and switch online ordering on or off

**Salespeople**
- Carry a digital **stock bag** showing exactly what they've been given
- Sell with card/online payment links, QR payments, cash, or EFT with proof of payment
- Request more stock, return unsold stock, and claim online-order deliveries
- Track earnings in a **commission wallet** with sales history

**Cashier (till)**
- Point-of-sale for walk-in customers against the shared menu, with receipts, sales history and shift controls

**Drivers**
- Claim online-order deliveries and stock-transfer jobs, with a two-step custody handover (kitchen → driver → salesperson)

**Overseers (management)**
- A live activity feed and team views for sales, kitchen, cashier and delivery staff
- Analytics and per-person performance over selectable periods
- Menu management, user and role management (one person can hold several roles), refunds, and manual sale entry with payment verification
- Approve or decline **override requests**, where staff ask for an exception instead of bypassing the rules

**Customers (online ordering)**
- The back end powers online ordering for a separate customer-facing website: a public menu, customer verification, online payment, order status updates and post-order feedback.

**Across the platform**
- Push notifications on mobile, and an in-app notification bell on web

<img width="1878" height="902" alt="image" src="https://github.com/user-attachments/assets/b8aced8d-eafc-48df-8394-78b6ae6ca95d" />
| Figure 1: Overseer Dashboard 


---

<img width="1911" height="903" alt="image" src="https://github.com/user-attachments/assets/f4017441-4562-4409-b9fd-526cc1512814" />
| Figure 2: Kitchen staff handing over products

---

<img width="1895" height="897" alt="image" src="https://github.com/user-attachments/assets/a82b173d-58f6-4c76-b505-4253e605bff2" />
| Figure 3: Salesperson making a sale.

---

## 3. Tech Stack

| Layer | Technology |
|---|---|
| **Web front end** | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui (Radix), TanStack Query, Recharts |
| **Mobile app** | Expo (React Native) with NativeWind, camera-based QR scanning, push notifications |
| **Shared logic** | An internal TypeScript package (types, roles, business rules) used by web and mobile, in an npm workspaces monorepo |
| **Back end** | Node.js on Firebase Cloud Functions (serverless), including scheduled jobs and database triggers |
| **Database** | Cloud Firestore (NoSQL, real-time) |
| **Auth** | Firebase Authentication with role-based access, including multi-role staff |
| **Payments** | South African payment gateways (PayFast) via verified server-side callbacks |
| **Email** | Transactional email service for customer verification and notifications |
| **Hosting** | Firebase Hosting (web) and EAS Build (mobile), with separate staging and production environments |

**Why this stack?**
- **Real-time by default.** Kitchen requests, handovers and order status have to show up on other people's screens instantly, and Firestore's live listeners provide that without building socket infrastructure.
- **Serverless back end.** It is cheap to run for a small business, scales with peak selling hours, and has no servers to maintain.
- **Mobile-first with Expo.** Salespeople and drivers work on their phones, so a native app with camera and push support was essential. Sharing code with the web app keeps the two consistent.

---

## 4. Architecture (High-Level)

<img width="852" height="402" alt="image" src="https://github.com/user-attachments/assets/929deb35-3972-45fc-9ce2-cb6833e0c83e" />
| Figure 4: Architecture diagram of Garnish IMS

- **Clients.** Each role gets a focused dashboard. The apps *display* live data straight from the database, but every action that moves stock or money goes through the server API.
- **Server.** The server owns the rules: pricing, stock levels, commission, shift state, payment confirmation and permissions. Background jobs handle end-of-day auto-close, stale-order follow-up and data clean-up.
- **Payments.** Gateway payments are confirmed only by the payment provider's callback to the server. The client never says "this was paid".
- **Auth and validation:**
  1. **Client.** Role-based routing and form validation, for usability.
  2. **API.** Identity and role are checked on every call. Inputs are validated, and anything extra is stripped before it's stored.
  3. **Database.** Security rules limit what each role can read or write directly.

---

## 5. Key Technical Decisions

**1. Never trust the client with money.**
Sale totals are always recalculated on the server from live menu prices. Payment methods that mean "the gateway confirmed it" can only be recorded by the gateway's verified callback, never by the app. Sales also carry a unique ID created on the device, so a retried or double-tapped "Complete Sale" can't charge twice. *Why:* this system pays commission and tracks cash, so any number the client could tamper with is a financial risk to the business.

**2. A chain of custody for every item of stock.**
Stock moves between people only through confirmed handovers (QR-scanned, with the driver as a tracked middle step), and every change is logged. Unsold stock comes back through a return flow, and exceptions go through override requests that a manager approves. *Why:* the business's biggest operational risk was stock "disappearing" between the kitchen and the street, so the system makes someone accountable for every item at every moment.

**3. Pre-computed daily statistics instead of live aggregation.**
Every sale, donation or handover updates a small daily summary as it happens. Management dashboards read a handful of summaries rather than scanning every transaction. *Why:* dashboard speed and database cost stay flat however much the business grows.


---

## 6. What I Learned / Challenges

- **Modelling a real, messy business.** Real operations don't follow a neat happy path. Salespeople run out of stock mid-shift, deliveries go unclaimed, and customers dispute orders. I learned to design explicit flows for the exceptions (restock check-ins, re-claimable deliveries, review-gated cancellations, override requests) rather than letting staff work around the system.
- **Financial integrity under bad network conditions.** Staff sell outdoors on mobile data. Making sales idempotent and confirming payments only on the server meant retries and dropped connections never created duplicate or fake sales.
- **Concurrency between people and background jobs.** Scheduled jobs (such as shift auto-close) can collide with a staff member's action. Running both inside transactions, with clear rules on which one wins, removed a whole class of "my shift closed by itself" bugs.
- **Multi-role users.** Some staff work several roles. Supporting a choice of active role per session, without sending people to the wrong dashboard on refresh, took careful auth design.
- **Shipping fast while staying safe.** The platform grew quickly from feature requests. Keeping security rules, indexes and the shared business rules in step across web, mobile and server was the discipline that made that pace sustainable.

---

> This is a public overview. The full technical report, detailed architecture, and private demo are available on request. If you would like access, please contact me and I can add you.
