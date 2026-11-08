# HCN — Handoff Commons for Neighbourhoods

A small, open-source errand board for community food pickup and delivery.

[中文说明](#中文说明)

---

## What it is

HCN is a task board. It is not a delivery company.

1. A customer orders and pays at a restaurant, outside this app.
2. The customer posts a pickup task: restaurant, order number, delivery area, and fee.
3. All riders see the same board. A rider chooses a task.
4. The rider collects the order and delivers it.
5. The customer gives the rider a delivery code. The rider enters the code.
6. The customer pays the fee directly to the rider (for example, by PayID).
7. Both sides rate each other.

The app does not hold money. The app does not set prices. The app does not assign tasks.

## Why

Riders must never carry the cost of food.

On many platforms, a rider can lose pay when an order is lost or stolen. This creates pressure to chase or confront people. That is dangerous.

In HCN, the customer pays for the food before pickup. The rider pays nothing. If an order is stolen, the rider has no financial loss. The rider has no reason to chase anyone.

## Core rules

These rules are part of the design. A deployment that breaks them is not HCN.

1. **Riders never pay for food.** The food is paid before pickup.
2. **No money in the app.** Fees go directly from customer to rider.
3. **No dispatch.** Every rider sees the same board, in the same order.
4. **Public rules.** Board order, limits, and rating rules are in the code and in [PROTOCOL.md](PROTOCOL.md). Changes are public.
5. **Zero-cost exit.** Customers, riders, and merchants keep their own data in their own Solid Pod. They can move to another platform at any time and lose nothing. See below.
6. **No hidden penalties.** A rider can skip any task. Skipping has no effect on rating.

## Zero-cost exit

This is the most important protection in HCN. If a platform becomes bad, people must be able to leave at once, with no loss.

### The data lives with its owner, not with the platform

Each customer, rider, and merchant owns a [Solid Pod](https://solidproject.org/). The Pod holds their profile, history, and ratings. A platform reads and writes this data only with permission. The owner can remove that permission at any time.

- **A merchant** keeps their profile, menu, opening hours, and reputation in their own Pod. If a better platform starts, the merchant connects the same Pod and starts work there on the same day.
- **A rider** keeps their delivery history and ratings. A good record follows the rider to any platform.
- **A customer** keeps their addresses and order history. They can change apps without starting again.

### Many platforms, one open data layer

Because the data is not inside any platform, many platforms can run at the same time on the same data. A platform is only an interface and an index. Each platform must earn its users every day.

If a platform raises fees or treats riders badly, people leave. They take everything with them. This makes a competitive market of many platforms. The best platform wins because people choose it, not because it holds their data.

This matters more in the AI era. New and better apps can now appear in weeks. With Pods, anyone can build a better app on the same data, and people can move to it at no cost. No platform can lock people in.

### Rules that make exit real

**For customers, riders, and merchants:**

1. **Portable reputation.** Each rating is signed by the person who gives it and stored in the Pod of the person who gets it. The owner cannot change it. Any platform can check the signature. A rider or merchant never starts from zero.
2. **No export needed.** The data is already in the owner's Pod. To leave a platform, the owner removes its access. For users without a Pod, the platform gives one-click export and delete, with no waiting period.
3. **Nothing to lose.** The app holds no money, no credits, no points, and no balances. Nothing is locked in.
4. **No exclusivity.** Riders and customers can use any other app at the same time. The app never penalises this.
5. **Open clients.** The API is public. Anyone can build another client for the same deployment.

**For the community:**

1. **Fork right.** The code is AGPL. Anyone can run a new deployment.
2. **Rebuild from public data.** The public event log and user exports are enough to start a new deployment with real history.
3. **Shared format.** All platforms use the same data format and the same protocol. Moving between them is like moving between email providers.
4. **Shared board.** Open tasks are published in a public feed. A rider on one platform can see and accept tasks posted on another platform.

## For merchants: keep your customer relationships

On big platforms, a merchant's regular customers belong to the platform. If the merchant leaves, they lose them.

HCN separates two things: **the merchant owns the relationship; the customer owns the data.**

1. **Follows.** A customer can follow a merchant. The follow is stored in both Pods, not in the platform. It stays when either side changes platform.
2. **A different private ID for each merchant.** A merchant can see that "this customer ordered 12 times". The merchant cannot see the customer's name, phone, or address. Two merchants cannot join their data.
3. **Messages with limits.** A merchant can send messages to followers' Pod inboxes. Maximum 2 messages per week. A customer can unfollow at any time, and access stops at once.
4. **Loyalty stamps.** For example, "buy 10, get 1 free". The merchant signs each stamp. The stamps stay in the customer's Pod and work on any platform.
5. **Sales records.** A merchant keeps its own sales data: dishes, times, and order counts per private ID.

These rules protect the rating system:

- A merchant must not give rewards for ratings.
- A merchant sees only the rating summary. It cannot see which customer gave which score.
- Ratings from followers are marked, and repeat ratings count once per month.

## MVP scope

In scope:

- Login with a Solid Pod (phone sign-up as a fallback)
- Profiles, history, and ratings stored in the owner's Pod
- Merchant profile in the merchant's Pod
- Post a pickup task
- Public task board
- Accept, pick up, deliver (with delivery code)
- Two-way signed ratings
- Limits for new accounts
- Data export for users without a Pod
- Public feed of open tasks
- Public, anonymous event log

Out of scope (for now):

- Menus and in-app ordering
- In-app payments
- Dispatch algorithms
- Insurance
- Blockchain or tokens
- Merchant follows, messages, and loyalty stamps (planned for v0.2)

## Technology

| Part | Choice | Reason |
|---|---|---|
| Client | PWA (web app) | No app store. Works on any phone. |
| Backend | PocketBase | Open source. One file. Easy to self-host. |
| Data | Solid Pods | People own their data. Zero-cost exit. |
| Maps | OpenStreetMap + Leaflet | Free and open. |
| License | AGPL-3.0 | All forks must stay open source. |

## Quick start

Code is not yet available. See [PROTOCOL.md](PROTOCOL.md) for the design.

## Governance

HCN must not become the thing it replaces. These principles apply to any official project body:

1. Low exit cost comes first. Users and riders can leave and take their data.
2. No investors. Funding comes from donations and membership fees only.
3. Fee caps, if any, are in the rules. A change needs a vote.
4. Domains, accounts, and trademarks belong to a group, not to one person.
5. Critical access needs two or more people to approve.
6. Riders hold the majority of seats in any governing body.

## Legal notice

**This is a reference implementation. It is not a running service.**

If you deploy or operate this software, you are responsible for local law. This includes labour law, consumer law, privacy law, food safety, tax, and insurance.

In Australia, read the Fair Work Act 2009 rules on digital labour platforms (sections 15L to 15P). Also read the current Fair Work Commission minimum standards orders. Get legal advice before you operate a service.

Riders should have personal accident insurance. A community group can buy group cover.

The authors give no warranty. See [LICENSE](LICENSE).

## Contributing

Issues and pull requests are welcome. Better ideas are welcome too. If you can design a better model, please do. That is the goal of this project.

---

## 中文说明

HCN 是一个开源的社区跑腿任务板，不是外卖公司。

**流程**

1. 用户在 App 外向餐厅下单并付款。
2. 用户发布取餐任务：餐厅、取餐号、送达区域、配送费。
3. 所有骑手看到同一个任务板，自己选择接单。
4. 骑手取餐、送达。
5. 用户把送达码告诉骑手，骑手输入送达码。
6. 用户通过 PayID 等方式直接把配送费付给骑手。
7. 双方互相评分。

App 不经手资金，不定价，不派单。

**核心原则**

骑手永远不承担餐费。餐费在取餐前已经付清。餐被抢时，骑手没有经济损失，也就没有理由去追赶任何人。

**零成本退出**

用户、骑手和商家的数据都存在自己的 Solid Pod 里，不存在平台里。平台只在获得授权时读写数据，所有者可以随时收回授权。

- 商家带着自己的 Pod（资料、菜单、营业时间、口碑），可以在任何新的、更好的平台上当天开业。
- 骑手的配送记录和评分跟着骑手走，换平台不用从零开始。
- 评分由评价人签名，被评价人无法修改，任何平台都可以验证。

因此，多个平台可以同时运行在同一层开放数据上。平台只是界面和索引。哪个平台更好，大家就选哪个。在 AI 时代，新的应用可以很快出现。数据在人手里，就没有平台能把人锁住。

**商家：留住老客，但不拿走顾客数据**

商家拥有"关系"，顾客拥有"数据"。

1. 顾客可以关注商家，关注记录存在双方的 Pod 里，换平台也不会丢失。
2. 顾客对每家商家使用不同的匿名 ID。商家能认出回头客，但看不到姓名、电话和地址。不同商家之间无法拼接数据。
3. 商家可以给关注者发消息，每周最多 2 条。顾客取消关注后，商家立即失去联系权限。
4. 会员积分由商家签名，存在顾客的 Pod 里，任何平台都可以验证。
5. 商家保留自己的经营数据：菜品销量、时段、各匿名 ID 的下单次数。

为保护评分机制：商家不得以好评换优惠；商家只能看到汇总评分，看不到具体是谁打的分；关注者的评分会被标注，回头客评分每月只计一次。

**法律声明**

这是开源参考实现，不是运营中的服务。任何人部署和运营本软件，需要自行遵守当地法律，包括劳动法、消费者法、隐私法、食品安全、税务和保险。在澳大利亚，请阅读《公平工作法》中关于数字劳动平台的条款（第 15L 至 15P 条），并在运营前咨询律师。

**欢迎贡献**

欢迎提交 Issue 和 PR，也欢迎提出更好的方案。如果你能设计出更好的模式，请去做。这正是本项目的目的。
