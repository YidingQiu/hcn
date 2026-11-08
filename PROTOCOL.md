# HCN Protocol v0.1.1 (draft)

This document defines the data, task states, and rules of HCN. Any client or server that follows this document can work with HCN data.

HCN data lives in the Solid Pods of the people and businesses who own it. A platform is an interface and an index. Many platforms can work on the same data at the same time.

---

## 1. Roles

| Role | Description |
|---|---|
| Customer | Posts a pickup task. Pays the restaurant and the rider directly. |
| Rider | Chooses a task from the board. Picks up and delivers the order. |
| Merchant | Publishes a profile in their own Pod. Receives orders outside the app in v0.1. Owns follows, loyalty stamps, and its own sales records. |
| Platform | Runs a client and an index. Reads and writes Pod data only with permission. |
| Moderator | Reviews reports and disputes. Cannot change ratings or task history. |

One account can be both customer and rider. A user cannot accept their own task.

## 2. Data objects

### 2.0 Where data lives

| Data | Stored in | Platform role |
|---|---|---|
| User profile, PayID | Owner's Pod | Read with permission |
| Tasks a customer posted | Customer's Pod | Index copy for the board |
| Deliveries a rider made | Rider's Pod | Read with permission |
| Ratings a person received | Receiver's Pod (signed by the giver) | Verify signature, show summary |
| Merchant profile and menu | Merchant's Pod | Read, show to customers |
| Follows | Customer's Pod and merchant's Pod | Read with permission |
| Loyalty stamps | Customer's Pod (signed by the merchant) | Verify signature |
| Merchant sales records | Merchant's Pod | Read with permission |
| Board index, rate limits, sessions | Platform | Platform data |
| Public event log | Platform, published | Public |

Rules:

1. The Pod copy is the true copy. A platform copy is a cache.
2. When the owner removes access, the platform deletes its cache within 24 hours.
3. Data uses JSON-LD with public vocabularies (for example schema.org `Restaurant`, `Order`, `Rating`). Any platform can read it.
4. Users without a Pod can use platform storage. The platform must give one-click export in the same format.

### 2.1 User

| Field | Type | Visibility | Notes |
|---|---|---|---|
| `id` | string | public | Random ID |
| `display_name` | string | public | |
| `phone` | string | private | Verified by SMS code |
| `payid` | string | counterpart only | Shown to the customer at `DELIVERED` or `DELIVERED_UNVERIFIED` |
| `abn` | string | private | Riders only. Required from trust level 1. Checked with ABN Lookup. |
| `abn_verified` | boolean | public | |
| `devices` | list | private | Bound device IDs. See section 5. |
| `trust_level` | integer 0–3 | public | See section 5 |
| `rating_as_customer` | object | public | Count and average |
| `rating_as_rider` | object | public | Count and average |
| `stats_as_rider` | object | public | `completed`, `incidents`, `unverified_deliveries`, `late_releases` |
| `stats_as_customer` | object | public | `closed`, `unpaid_disputes`, `late_cancels`, `wasted_trips` |
| `created_at` | datetime | public | |

The server calculates `stats_*` from task events. Users cannot edit them. Clients show them as rates, for example "2 incidents in 150 deliveries". These objective numbers are harder to manipulate than ratings.

### 2.2 Task

| Field | Type | Visibility | Notes |
|---|---|---|---|
| `id` | string | public | |
| `customer_id` | string | public | |
| `restaurant_name` | string | public | |
| `restaurant_address` | string | public | |
| `order_ref` | string | rider only | Restaurant order number. Shown after accept. |
| `dropoff_area` | string | public | Suburb or area only |
| `dropoff_address` | string | rider only | Shown after accept. Deleted 24 hours after close. |
| `fee_aud` | decimal | public | Set by the customer |
| `food_prepaid` | boolean | public | Must be `true` to post |
| `notes` | string | rider only | |
| `delivery_code` | string | customer only | 4 digits. Made by the server. |
| `status` | enum | public | See section 3 |
| `rider_id` | string | public | Empty until accept |
| `created_at` | datetime | public | |
| `expires_at` | datetime | public | Default: 60 minutes after `created_at` |

### 2.3 Rating

| Field | Type | Notes |
|---|---|---|
| `task_id` | string | |
| `from_id` | string | |
| `to_id` | string | |
| `score` | integer 1–5 | |
| `tags` | list | Fixed list, for example `on_time`, `polite`, `late` |
| `created_at` | datetime | |
| `signature` | string | Signed by the giver's WebID key |
| `witness` | string | Optional. Platform counter-signature that the task took place. |

A rating is allowed only after `DELIVERED`, `DELIVERED_UNVERIFIED`, or `INCIDENT`. One rating per user per task. A rating cannot be edited. A user can appeal a rating (section 7).

General rating rules:

1. **Blind ratings.** Neither side sees the other's rating until both have rated, or until 7 days pass. This stops revenge ratings.
2. **One pair, one count per 30 days.** Ratings between the same customer and the same rider count once per 30 days. This applies to riders and customers, the same as for merchants.
3. **Fast tasks have low weight.** If the time from `ACCEPTED` to `DELIVERED` is under 5 minutes, the rating has low weight. This reduces fake tasks between friends.
4. **The witness signature is not proof of a real order.** In v0.1 the platform cannot check `food_prepaid` or `order_ref`. The witness signature only shows that the task went through the platform. Real order checks come when merchants join (merchant pickup scan).

Rules for ratings of merchants:

1. A merchant sees only the summary of its ratings. It cannot see which private ID gave which score.
2. A merchant must not give rewards, stamps, or discounts for ratings. A merchant that breaks this rule has its ratings frozen.
3. Ratings from followers have the tag `follower`. The rating summary shows the follower share.
4. Ratings from the same customer to the same merchant count once per month.

### 2.4 Event

Every state change makes one event. The event log is append-only.

| Field | Type | Notes |
|---|---|---|
| `id` | string | |
| `task_id` | string | |
| `actor_role` | enum | `customer`, `rider`, `system`, `moderator` |
| `from_status` | enum | |
| `to_status` | enum | |
| `created_at` | datetime | |

The public log does not include user IDs or addresses.

### 2.5 Merchant

| Field | Type | Visibility | Notes |
|---|---|---|---|
| `webid` | URL | public | Merchant's Solid identity |
| `name` | string | public | |
| `address` | string | public | |
| `abn` | string | public | Checked with ABN Lookup |
| `hours` | object | public | |
| `menu` | list | public | Optional in v0.1 |
| `order_channels` | list | public | Phone, website, or in person |

The merchant controls this data. A platform cannot edit it.

### 2.6 Follow

| Field | Type | Notes |
|---|---|---|
| `merchant_webid` | URL | |
| `customer_pid` | string | Pairwise private ID (section 6) |
| `inbox` | URL | Customer's Pod inbox for this merchant |
| `created_at` | datetime | |

The customer writes the follow to their own Pod and grants the merchant write access to the inbox. To unfollow, the customer deletes the follow and removes the access.

### 2.7 Loyalty stamp

| Field | Type | Notes |
|---|---|---|
| `merchant_webid` | URL | Issuer |
| `customer_pid` | string | Pairwise private ID |
| `program` | string | For example `buy10get1` |
| `count` | integer | |
| `issued_at` | datetime | |
| `signature` | string | Signed by the merchant's WebID key |

Stamps are stored in the customer's Pod. A platform does not hold stamps or balances.

## 3. Task states

```
OPEN ──accept──▶ ACCEPTED ──pickup──▶ PICKED_UP ──deliver──▶ DELIVERED ──confirm_paid──▶ CLOSED
 │                 │                     │                      ▲    └──dispute──▶ DISPUTED
 │                 │                     │                      │ (customer confirms or 24 h pass)
 │                 │                     ├──deliver_no_code──▶ DELIVERED_UNVERIFIED ──customer_dispute──▶ DISPUTED
 │                 │                     └──report_incident──▶ INCIDENT
 │                 └──release──▶ OPEN
 ├──cancel──▶ CANCELLED
 └──timeout──▶ EXPIRED
```

### 3.1 Transitions

| From | To | Who | Condition |
|---|---|---|---|
| — | `OPEN` | Customer | `food_prepaid = true`; limits in section 5 |
| `OPEN` | `ACCEPTED` | Rider | Rider has no other active task (trust levels 0–1) |
| `OPEN` | `CANCELLED` | Customer | No penalty |
| `OPEN` | `EXPIRED` | System | Time passes `expires_at` |
| `ACCEPTED` | `OPEN` | Rider | Release. No penalty within 3 minutes of accept. After 3 minutes, see release limits in section 5. |
| `ACCEPTED` | `CANCELLED` | Customer | Counts as a late cancel |
| `ACCEPTED` | `PICKED_UP` | Rider | |
| `PICKED_UP` | `DELIVERED` | Rider | Rider enters the correct `delivery_code` |
| `PICKED_UP` | `DELIVERED_UNVERIFIED` | Rider | No code. Rider is within 100 m of the drop-off, uploads a photo of the order at the door, and waits 10 minutes after notifying the customer. |
| `DELIVERED_UNVERIFIED` | `DELIVERED` | Customer or system | Customer confirms, or 24 hours pass with no dispute |
| `DELIVERED_UNVERIFIED` | `DISPUTED` | Customer | Customer says the order did not arrive. Moderator review. |
| `PICKED_UP` | `INCIDENT` | Rider | Order lost, stolen, or damaged. Photo or description required. |
| `DELIVERED` | `CLOSED` | Rider | Rider confirms fee received |
| `DELIVERED` | `DISPUTED` | Rider | Fee not received after 24 hours |

Each `DELIVERED_UNVERIFIED` adds 1 to the rider's `unverified_deliveries`. A high rate is visible to customers and triggers review.

### 3.2 Incident rule

If the order is lost, stolen, or damaged:

1. The rider reports `INCIDENT`.
2. The rider and the platform have **no** liability for the food cost. This rule cannot be changed by any deployment.
3. The customer bears the food cost, or claims from an optional mutual-aid pool if one exists. A mutual-aid pool is run by a separate body (for example a rider or community association), not by the platform.
4. The incident adds 1 to the rider's `incidents`. A high incident rate is visible and triggers review. This limits riders who report false incidents.
5. The app shows this message to the rider: **"Do not chase. Do not confront. You owe nothing."**

## 4. Board order

1. The board shows all `OPEN` tasks.
2. The server sorts tasks by `created_at`, oldest first.
3. The client can filter by distance or fee. The filter runs on the device.
4. The server does not rank tasks or riders in any other way.
5. All riders get the same list.

## 5. Trust levels and limits

| Level | How to reach | Customer limit | Rider limit |
|---|---|---|---|
| 0 | Phone verified | 2 open tasks per day | 1 active task |
| 1 | 5 closed tasks, average rating ≥ 4.0 | 5 per day | 1 active task |
| 2 | 20 closed tasks, average ≥ 4.3 | 10 per day | 2 active tasks |
| 3 | Set by moderator vote | No daily limit | 3 active tasks |

Other limits:

Payment limits (customers):

- **Levels 0–1:** a customer cannot post a new task while any of their tasks is in `DELIVERED`, `DELIVERED_UNVERIFIED`, or `DISPUTED`. The rider must confirm payment first. Maximum loss to riders: one fee.
- **Level 0:** one unpaid `DISPUTED` task pauses the account for review.
- **Levels 2–3:** three unpaid `DISPUTED` tasks pause the account for review.
- A rider sees the customer's level and `stats_as_customer` before accept. A rider can ask a level 0 customer to pay the fee before pickup.

Release limits (riders):

- Release within 3 minutes of accept: free.
- Release after 3 minutes: 2 free per day. After that, each release starts a 15-minute block on new accepts.
- Each late release adds 1 to `late_releases`.

Account and device limits:

- One account per phone number.
- One account per ABN (riders).
- Each account binds up to 2 devices. A new device must wait 24 hours before it can accept tasks.
- At random accepts, the client can ask the rider for a selfie check against the profile photo.
- One active session per device.
- Board refresh: maximum once per 5 seconds per account.

All numbers in this section are defaults. A deployment can change them but must publish them.

## 6. Privacy

1. A rider sees `order_ref`, `dropoff_address`, and `notes` only after accept.
2. A customer sees the rider's `payid` only at `DELIVERED` or `DELIVERED_UNVERIFIED`.
2a. A `DELIVERED_UNVERIFIED` photo is kept for 7 days, then deleted.
3. The server deletes `dropoff_address` 24 hours after `CLOSED`, `INCIDENT`, or `CANCELLED`.
4. The public event log contains no personal data.
5. The server does not sell or share user data.
6. **Pairwise private IDs.** A customer has a different private ID for each merchant. The ID is made from the customer's key and the merchant's WebID. A merchant can recognise a repeat customer. It cannot see the customer's name, phone, or address. Two merchants cannot match their IDs.
7. **Merchant messages.** A merchant can write to a follower's inbox. Maximum 2 messages per week per merchant (default). A message must not ask for a rating.

## 7. Reports, disputes, and appeals

1. Any user can report a user or a task.
2. A moderator reviews each report.
3. A moderator can pause an account. A moderator cannot delete ratings or events.
4. Each moderator decision makes a public record, without personal data.
5. A user can appeal a decision once. A different moderator reviews the appeal.

## 8. Portability and many platforms

### 8.1 Login and access

1. Users log in with Solid-OIDC, using their WebID.
2. The user grants a platform access to named parts of their Pod.
3. The user can remove access at any time. No reason is needed.

### 8.2 Signed reputation

1. When a person gives a rating, their client signs it with their WebID key.
2. The rating is written to the receiver's Pod. The receiver cannot change it.
3. The platform can add a `witness` signature to show the task took place on that platform. This stops fake ratings between friends.
4. Any platform can check all signatures and compute the same rating summary.

### 8.3 Shared task board

1. A customer writes the task to their own Pod.
2. The customer's client sends the task to one or more platforms.
3. Each platform publishes open tasks in a public feed (`/tasks/feed`).
4. A platform can show tasks from other platforms' feeds.
5. To accept a task, a rider sends the accept to the task's home platform. The first valid accept wins.

### 8.4 Leaving a platform

1. The user removes the platform's access in their Pod.
2. The platform deletes its cache within 24 hours.
3. The user connects the same Pod to another platform. Their profile, history, and reputation are there at once.

## 9. API (draft)

| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/otp` | Send SMS code |
| `POST` | `/auth/verify` | Verify code, get session |
| `GET` | `/tasks?status=OPEN` | Get the board |
| `POST` | `/tasks` | Post a task |
| `POST` | `/tasks/{id}/accept` | Accept |
| `POST` | `/tasks/{id}/release` | Release |
| `POST` | `/tasks/{id}/cancel` | Cancel |
| `POST` | `/tasks/{id}/pickup` | Mark picked up |
| `POST` | `/tasks/{id}/deliver` | Body: `{ "delivery_code": "1234" }` |
| `POST` | `/tasks/{id}/deliver-no-code` | Body: photo and location. Starts the 10-minute wait. |
| `POST` | `/tasks/{id}/confirm-received` | Customer confirms an unverified delivery |
| `POST` | `/tasks/{id}/incident` | Body: `{ "type": "stolen" }` |
| `POST` | `/tasks/{id}/confirm-paid` | Rider confirms fee received |
| `POST` | `/tasks/{id}/dispute` | Rider reports unpaid fee |
| `POST` | `/ratings` | Rate the other side |
| `POST` | `/reports` | Report a user or task |
| `GET` | `/events/public` | Public anonymous event log |
| `GET` | `/tasks/feed` | Public feed of open tasks, for other platforms |
| `POST` | `/auth/solid` | Log in with Solid-OIDC |
| `GET` | `/me/export` | Export my data (users without a Pod) |
| `POST` | `/merchants/{id}/follow` | Follow a merchant (v0.2) |
| `DELETE` | `/merchants/{id}/follow` | Unfollow (v0.2) |
| `GET` | `/merchants/{id}/ratings/summary` | Rating summary with follower share |

Errors use HTTP status codes and this body: `{ "error": "code", "message": "text" }`.

## 10. Rules that a deployment cannot change

1. Riders never pay for food.
2. The app does not hold user money.
3. The server does not rank tasks except by time.
4. People own their data. A platform must work with Pod storage and must delete its cache when access is removed.
5. A merchant cannot see who gave which rating, and cannot reward ratings.
6. A merchant gets a pairwise private ID, never a customer's identity, unless the customer shares it.
7. A platform must not lock people in. No exclusivity rules, no balances, no penalties for using other platforms.
8. A platform must publish its open tasks in the public feed format.
9. Section 5 limits and fees must be public.

## 11. Versioning

This document uses semantic versions. A change to section 10 needs a new major version and a public discussion.
