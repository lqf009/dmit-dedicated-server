# dedicated hosting: how to choose a dedicated server, with DMIT's bare metal and network tiers explained

When people search "dedicated hosting," they usually fall into one of three situations. Either a site has outgrown shared or VPS plans and needs its own machine, a workload requires predictable hardware performance with no noisy neighbors, or there's a compliance and isolation requirement that virtualized environments simply cannot satisfy. In all three cases, the underlying question is the same: what does it actually mean to rent a dedicated server, and how do you avoid overpaying for specs you don't need while still getting the network quality your users care about?

This article walks through what dedicated hosting is, when it's justified versus when a high-end VPS would do, the spec decisions that matter, and how a provider like DMIT fits into the picture — including the bare metal option DMIT actually offers, the three network tiers it sells, and the pricing reality of each.

## What dedicated hosting actually means

Dedicated hosting gives you an entire physical server that no one else shares. There's no hypervisor carving the box into virtual machines, no CPU core that another tenant can spike, and no disk I/O competing with someone else's backup job. You get root (or, on a real bare metal box, IPMI) access to the whole machine, and the hardware's performance curve is yours alone.

That's distinct from VPS hosting, where you get a virtualized slice of a larger machine. A good VPS on fast NVMe storage with dedicated vCores can come close to bare metal for many workloads, and it costs a fraction of the price. The difference shows up when you need the full memory bandwidth of the machine, sustained single-tenant disk IOPS, custom kernel modules, direct hardware access, or enough CPU headroom that no neighbor can possibly interrupt you.

In practice, dedicated hosting makes sense when:

- Your traffic has outgrown what even a large VPS can absorb, and you're paying for multiple VPS instances anyway
- You run a database that benefits from full memory channels and direct NVMe lanes
- You need strict isolation for compliance, financial data, or regulated workloads
- You want to run your own virtualization stack (KVM, Proxmox) and sell or split capacity yourself
- You need custom hardware — extra GPUs, large memory footprints, specialized network cards — that stock VPS plans don't offer

If none of those apply, a VPS is almost always the better dollar-per-performance choice. Dedicated hosting is not a status upgrade; it's a workload-driven decision.

## How to read dedicated server specs without getting lost

The spec sheet on a dedicated server page is where most buyers either overbuy or miss the thing that actually matters for their workload. Here's what each line is really telling you.

**CPU cores and threads.** More cores help with concurrent workloads (web apps with many simultaneous requests, virtualization hosts). Higher single-core clock speed helps with workloads that don't parallelize well (some game servers, single-threaded legacy apps). AMD EPYC parts, which is what DMIT uses on its bare metal line, give you high core density with solid per-core performance — useful when you're not sure which side of that trade you'll land on.

**RAM.** Memory is usually the bottleneck before CPU is. For databases, caching layers (Redis, Memcached), and virtualization hosts, err on the side of more RAM. DDR5 ECC is what you want on newer EPYC platforms; DDR4 ECC is still entirely serviceable on the previous generation.

**Storage.** NVMe SSDs for anything latency-sensitive. SATA SSDs are fine for bulk storage. HDDs only make sense for archival or backup where capacity-per-dollar matters more than IOPS. RAID matters here — a RAID 1 mirror for the OS and a RAID 10 array for data is a common pattern. Hardware RAID controllers cost more but give you battery-backed write cache; software RAID (mdadm, ZFS) is cheaper and more flexible.

**Bandwidth and port speed.** This is where dedicated hosting providers diverge wildly. Some sell "10Gbps unmetered" that's actually a shared uplink with a soft cap. Others bill per GB with a committed floor. DMIT, for example, sells bandwidth in tiers tied to routing quality — Premium, Eyeball, and Tier 1 — and the per-GB cost differs significantly between them. More on that below.

**IP and network options.** If you need multiple IPv4 addresses, IPv6 blocks, BGP to announce your own IP space, or private networking between servers, check that the provider supports it before you commit. Not all do, and adding it later is painful.

## DMIT's approach to dedicated hosting: bare metal, built to spec

DMIT is a hosting provider that operates infrastructure in three locations — Los Angeles (CoreSite and Digital Realty campuses), Hong Kong (Equinix HK2), and Tokyo (Equinix TY8). It's known primarily for VPS plans on AMD EPYC hardware with premium China-optimized routing, but it also offers a dedicated hosting product called **Bare Metal Instance**.

The Bare Metal page is explicit about what you get: a single-tenant physical server with no virtualization overhead, full root and IPMI access, consistent performance, and hardware that's built to your spec rather than picked from a fixed SKU list.

What that means in practice is that DMIT doesn't publish a one-size-fits-all dedicated server price list. You describe your workload — CPU, RAM, storage, bandwidth, location, IP needs — and their team assembles a configuration and quotes it. This is the normal model for serious bare metal; providers that sell $40/month "dedicated servers" off a SKU table are usually either older refurbed hardware or shared in ways that aren't immediately obvious from the marketing copy.

The bare metal hardware line is built on AMD EPYC platforms, and DMIT lists three configuration categories:

- **Compute Optimized** — high frequency and high core count, up to 128 cores / 256 threads, DDR4 or DDR5 ECC memory up to multi-TB. Suited for CPU-bound workloads like busy databases, application servers, and virtualization hosts.
- **Storage Optimized** — all-NVMe, SSD, or large HDD arrays with hardware or software RAID options. Tunable for IOPS or raw capacity, aimed at data-intensive workloads.
- **Enterprise & Custom** — GPU and accelerator options, custom CPU/RAM/disk combinations, IPMI and out-of-band management included. For specialized builds that don't fit the first two categories.

If you want a dedicated server from DMIT, the path is to tell them what you're running and let them spec it. 👉 [Reach out for a custom bare metal quote through DMIT](https://bit.ly/DmiT) and you'll get a configuration matched to your actual workload instead of a generic box.

## Why the network tier matters more than the box itself

Here's the part most dedicated hosting guides skip: on a modern EPYC platform, the CPU is rarely your problem. The network is.

If your users are in mainland China and your server is in Los Angeles, the question isn't "how fast is the CPU." It's "what route does the traffic take, and how much packet loss is there at peak hours." Standard Tier 1 transit into China is famously bad during evening congestion — high latency, jitter, and dropouts that make interactive services unusable.

DMIT sells three network series, and understanding the difference is the single most useful thing you can do before buying anything from them. The same applies when comparing any provider that touches China-facing traffic.

**Premium Network.** Combines Tier 1 transit with premium partners including DMIT's own backbone and China Telecom CN2 GIA. DMIT advertises ~15ms average latency to China Mainland from Hong Kong with packet loss under 0.1%. This is the tier to pick if your end users are in China and the experience has to be smooth — e-commerce, live streaming, real-time apps, gaming, cross-border finance. It's the most expensive per GB because CN2 GIA capacity is finite and costly.

**Eyeball Network.** Tier 1 transit plus "reasonable effort" China routing via CMIN2 or CMI and similar Chinese eyeball ISPs. It's a middle ground: noticeably better for Chinese residential users than plain Tier 1, but without the routing guarantees of Premium. Good fit for sites and APIs with a mixed global/China audience where you don't want to pay CN2 GIA prices for every byte.

**Tier 1 Network.** Standard multi-Tbps Tier 1 backbone, optimized for general global and intra-Asia/Americas routing, no China-specific enhancements. The cheapest tier. Right choice for backups, bulk transfers, internal tooling, CI/CD, VPN relays — anything where the user experience doesn't depend on low-latency China access.

The cost gap is real. DMIT's Hong Kong Premium plans run several times the price of equivalent Tier 1 plans in the same location, because CN2 GIA bandwidth into China is a premium resource. If you don't have China users, paying for Premium is wasted money. If you do have China users and you cheap out on Tier 1, you'll spend the next month explaining to your boss why the site is slow at 8pm.

## DMIT cloud instance pricing: what the public price list actually shows

DMIT's public pricing page lists VPS cloud instance plans, not bare metal SKUs. These are useful as a reference point for what DMIT hardware costs before you move up to a custom dedicated build, and they're also the right product for many people who think they need dedicated hosting but actually don't.

The full plan list on the Los Angeles Premium Network (AMD EPYC, NVMe SSD, 1 IPv4 + 1 IPv6 /64, free setup, basic DDoS protection) is below. Prices are monthly list price as currently shown on the DMIT pricing page.

| Plan | vCores | RAM | Storage | Transfer | Port | Price (monthly) | Order |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TINY | 1 | 2GB | 20GB SSD | 1000GB | 1Gbps | $10.90 | [View TINY plan](https://bit.ly/DmiT) |
| Pocket | 2 | 2GB | 40GB SSD | 1500GB | 4Gbps | $16.90 | [View Pocket plan](https://bit.ly/DmiT) |
| STARTER | 2 | 2GB | 80GB SSD | 3000GB | 10Gbps | $34.90 | [View STARTER plan](https://bit.ly/DmiT) |
| MINI | 4 | 4GB | 80GB SSD | 5000GB | 10Gbps | $62.90 | [View MINI plan](https://bit.ly/DmiT) |
| MICRO | 4 | 4GB | 160GB SSD | 7000GB | 10Gbps | $87.90 | [View MICRO plan](https://bit.ly/DmiT) |
| MEDIUM | 6 | 8GB | 160GB SSD | 15000GB | 10Gbps | $199.90 | [View MEDIUM plan](https://bit.ly/DmiT) |

The pricing page carries an explicit note that "products and prices in the table may not be updated in time due to adjustment, for reference only." Treat those numbers as a baseline, not a quote — confirm on the live page before you order.

The same plan names exist across the three network series and across the three locations, but the prices and bandwidth allotments change. Hong Kong Premium is the most expensive because CN2 GIA into China costs more. Tokyo Premium sits in between. Tier 1 plans in all three locations are much cheaper because they don't carry the China-routing premium.

A few examples of how the same plan name shifts by network and location, based on the live location pages:

- Hong Kong Premium **MINI** (4 vCore / 4GB / 80GB / 1500GB / 1Gbps): $149.90/mo
- Hong Kong Premium **MICRO** (4 vCore / 4GB / 160GB / 2000GB / 1Gbps): $199.90/mo
- Hong Kong Premium **LARGE** (8 vCore / 16GB / 320GB / 3000GB / 1Gbps): $359.90/mo
- Hong Kong Premium **GIANT** (12 vCore / 24GB / 640GB / 6000GB / 1Gbps): $759.90/mo
- Los Angeles Tier 1 **STARTER** (1 vCore / 2GB / 40GB / 4000GB): $12.90/mo
- Hong Kong Tier 1 **MICRO** (4 vCore / 4GB / 80GB / 16000GB): $32.90/mo

That last comparison is the clearest illustration of the tier gap: a 4-core / 4GB Hong Kong box on Tier 1 with 16TB of transfer costs less than one-fifth of what the same core/RAM count costs on Premium with a fraction of the transfer. You're paying for the route, not the silicon.

If you want to browse all current plans and configurations by location and network, 👉 [check the live DMIT pricing page here](https://bit.ly/DmiT).

## Hardware platforms: AN5, AN4, and AS3 explained

DMIT runs three hardware generations across its locations. The platform you land on affects single-core speed and price-per-core, and the platform availability also varies by network tier.

**AN5 — AMD EPYC 9005 Series (Zen 5 / Turin).** The flagship. DDR5 memory, PCIe 5.0 NVMe storage, highest single-core and multi-core performance in the lineup. Best for high-traffic sites, databases, and latency-sensitive apps. In Hong Kong, AN5 plans are currently only offered on the Premium network.

**AN4 — AMD EPYC 9004 Series (Zen 4).** The balanced workhorse. Strong per-core performance with high core density, proven in production. Suitable for virtually any general-purpose workload — web hosting, applications, dev environments.

**AS3 — AMD EPYC 7003 Series (Zen 3 / Milan).** The value tier. Mature, field-proven, most competitive price-per-core in the lineup. Aimed at budget-conscious projects, staging, and entry-level deployments. In Hong Kong, AS3 is offered on the Eyeball and Tier 1 networks. In Los Angeles, DMIT openly notes that the AS3 platform is still being built out and that during this period you may see reduced disk performance and a lower SLA than on the mature platforms — worth knowing before you pick the cheapest option.

The practical takeaway: if you're buying a dedicated or cloud instance for production and the price difference is tolerable, AN4 or AN5 is the safer call. AS3 is fine for testing, staging, or workloads where a slightly lower SLA won't hurt you.

## What DMIT's terms actually say about refunds, SLA, and support

A dedicated hosting purchase is only as good as the small print. DMIT's Terms of Service (last updated January 2026 on the version crawled) spells out a few things worth knowing before you commit.

**Refunds.** Full refund (minus payment-gateway transaction fee) is available if the service has been purchased no more than 3 days ago and you've used no more than 30GB of transfer. Partial refund is available within 30 days of purchase, calculated on whichever is lower: the value of remaining transfer or the value of remaining service time. Renewals are not refundable. Orders paid with account credit can only be refunded back to account credit. Targeted DDoS, "the network is not good enough," and IP geographic location issues are explicitly listed as non-refundable reasons — so if you buy a Hong Kong Premium box and your users in a specific region can't reach the IP, that's on you to test before the 30-day window closes.

**SLA.** DMIT currently offers 99% uptime. Below 99% gets you half a month of compensation, below 95% a full month, below 90% two months. You have to follow the SLA notification procedure within 3 days of the triggering event or you waive the credit. The AS3 platform in Los Angeles is explicitly noted as carrying a lower SLA during the buildout period.

**Support.** Most DMIT services are unmanaged. They guarantee support ticket replies within 72 hours. That's not a knock — it's the normal model for providers in this price range — but if you need a managed dedicated server with a human on call in 15 minutes, this isn't that product.

**IP replacement.** Premium and Eyeball network profiles: with the IP Care+ add-on, you can replace your IP every 7 days; without it, every 15 days; or pay $5 for an immediate replacement. Premium Secure: $15 per replacement, 30 days between replacements. Tier 1: without the IP Guarantee+ add-on, DMIT does not guarantee the IP is globally reachable, especially in China, Russia, and countries with national censorship; with the add-on, first connection in sensitive areas is guaranteed; ad-hoc replacements cost $5 with 7 days between them.

**Payment.** Billed in advance, recurring automatically. Enabling auto-renewal requires two-factor authentication on the account. DMIT has a "Glass Break" emergency function that disables all auto-pay and removes stored card credentials — useful if you suspect account compromise.

If you want the full detail, 👉 [read the current terms and order through DMIT](https://bit.ly/DmiT) so you're looking at the live version rather than a summary.

## Promo codes: what's actually verifiable

DMIT runs promo codes periodically, and several are referenced across coupon sites. The challenge is that promo codes expire, and coupon aggregators are notorious for listing codes that no longer work.

One code that appears directly on a DMIT-owned page is:

**`LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF`** — a 20% recurring discount on LAX Eyeball plans at TINY tier and above, on quarterly or longer billing cycles. The page that hosts this code also references an event window from April to July 2024, which raises the question of whether the code is still active. Treat it as "try it at checkout, don't plan around it."

Other codes referenced on third-party coupon sites (e.g. `SPRO-20OFF`, `HK-A-R49Y8YDR3P-20OFF`, various LAX T1 annual discounts) are not confirmed against an official DMIT page in this research round, and coupon sites frequently carry stale or incorrect entries. The safe move: enter any code you find at the DMIT checkout and see if it applies. If you're planning a major purchase around a discount, email DMIT sales first and ask whether a current code exists for the specific plan and billing cycle you want. Don't bake an unverified discount into your budget.

DMIT's own terms note that discount codes only apply to new customers, that they sometimes issue codes to existing customers as business compensation, and that using someone else's targeted discount code can get your service suspended until you pay full price. So don't grab a random code off a forum and assume it's fine.

## Choosing a location: Los Angeles, Hong Kong, or Tokyo

DMIT operates three data center locations, and the right one depends on where your users are and which network tier you're buying.

**Los Angeles.** The flagship North American node, across CoreSite and Digital Realty. 3.8Tbps aggregate Tier 1 capacity, direct peering with China Telecom (AS4809), China Unicom (AS9929), and China Mobile International (AS58807). Strong for serving China-facing traffic from the US, and for general APAC-to-Americas bridging. All three network series (Premium, Eyeball, Tier 1) are available, and all three hardware platforms (AN5, AN4, AS3) are present, though AS3 is still being optimized and carries a lower SLA during the buildout.

**Hong Kong.** Equinix HK2 in Kwai Chung. The premium gateway for China Mainland traffic — ~15ms average latency to Shenzhen with packet loss under 0.1% on Premium. AN5 plans here are only offered on Premium; AS3 plans are offered on Eyeball and Tier 1. If your users are mostly in China and you want the lowest latency, Hong Kong Premium is the strongest option in DMIT's lineup, and the price reflects that.

**Tokyo.** Equinix TY8 in Shinagawa, one of Tokyo's main carrier hubs. Good for intra-Asia and Europe-Asia traffic, and a reasonable China-facing alternative to Hong Kong if you want geographic diversity. All three network series are available.

A common multi-region pattern: Hong Kong or Tokyo Premium for China-facing front-end services, Los Angeles Tier 1 for bulk storage, backups, and non-China-facing traffic. That split lets you put the expensive CN2 GIA bandwidth where it matters and use cheap Tier 1 capacity for everything else.

## Dedicated hosting vs. DMIT's cloud instances: which one do you actually need?

This is the honest decision point. DMIT's bare metal is a custom-quoted dedicated server with full IPMI, single-tenant hardware, and configuration freedom. DMIT's cloud instances are KVM VPS plans on the same AMD EPYC platforms with the same network tiers, sold off a public price list from $10.90/month.

You should look at dedicated hosting (bare metal) instead of a cloud instance when:

- You need more RAM than the largest cloud instance offers (the largest published Hong Kong Premium plan is 24GB; the largest LA Premium plan is 8GB)
- You need direct hardware access, custom RAID layouts, or storage configurations that don't fit a fixed VPS SKU
- You're running a database or caching layer where sustained single-tenant memory bandwidth matters
- You want to run your own virtualization stack and carve the box up yourself
- You need GPU or accelerator hardware
- You have a compliance requirement that mandates physical isolation

You should stick with a cloud instance when:

- You need something live in minutes, not a custom-built server with a provisioning lead time
- Your workload fits comfortably within the largest published VPS plan
- You want the flexibility to upgrade or downgrade plan size on demand
- You're testing the provider before committing to a dedicated build

For a lot of people reading about dedicated hosting, the honest answer is that a Premium or Eyeball cloud instance on DMIT will handle their workload at a fraction of the cost of a bare metal box, and the upgrade path to bare metal is there when they actually need it. 👉 [You can compare current cloud instance plans and bare metal options side by side on DMIT](https://bit.ly/DmiT) before deciding which one fits.

## What to check before you commit to any dedicated hosting provider

Whether you end up with DMIT or someone else, run through this list before you pay.

1. **Confirm the network tier matches your users.** A cheap Tier 1 server in Los Angeles is a liability if your users are in Shanghai. A Hong Kong Premium box is wasted money if your users are in Berlin. Pick the tier after you know where your traffic is going.
2. **Read the refund policy in full.** Especially the non-refundable cases. If "IP not reachable in region X" is on the non-refundable list, test reachability from that region on day one.
3. **Check the SLA and the credit claim procedure.** A 99% SLA with a 3-day claim window is useless if you don't monitor uptime and file the claim in time.
4. **Verify the hardware platform.** Older EPYC generations are fine for many workloads, but you should know which one you're getting and whether it's still in a buildout or optimization period.
5. **Ask about managed vs. unmanaged.** If you need someone to fix the OS at 3am, unmanaged dedicated hosting is the wrong product for you, regardless of provider.
6. **Confirm IP and BGP options.** If you'll need additional IPv4, BGP for your own IP space, or private networking between servers, get that confirmed in writing before ordering.
7. **Try any promo code at checkout.** Don't trust coupon sites at face value. Enter the code and see if it applies to your specific plan and billing cycle.

## The short version

Dedicated hosting gives you a single-tenant physical server with predictable performance and full hardware access. It's worth the money when your workload has outgrown VPS plans, needs direct hardware control, or has compliance requirements that rule out virtualization. For everything else, a high-end VPS on the same hardware platform will usually cost less and perform nearly as well.

DMIT offers a genuine dedicated hosting product through its Bare Metal Instance line — custom-quoted AMD EPYC servers with IPMI access, configurable storage, and a choice of three network tiers (Premium, Eyeball, Tier 1) across Los Angeles, Hong Kong, and Tokyo. The differentiator isn't the hardware; it's the network. Premium CN2 GIA routing into China Mainland is the thing you're paying for, and whether it's worth it depends entirely on where your users are.

If your users are in China, Hong Kong or Tokyo Premium is the right call. If they're global or in the Americas, Tier 1 in Los Angeles gets you the same hardware at a fraction of the price. Either way, start from your workload and your users, not from the spec sheet.

👉 [Get a custom bare metal quote from DMIT](https://bit.ly/DmiT) if you know what you need and want a server built to spec, or 👉 [browse current cloud instance plans](https://bit.ly/DmiT) if you want to start smaller and scale into dedicated later.
