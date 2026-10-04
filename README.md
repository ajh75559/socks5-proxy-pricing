# socks5 proxies: what the protocol actually does, how to wire one into your browser or scraper, and what per-IP vs per-GB pricing really costs

Most people typing "socks5 proxies" into a search bar are not trying to read RFC 1928. They have a tool open in front of them — BitBrowser, AdsPower, Proxifier, a scraper, sometimes a phone — and that tool wants a host, a port, a username, a password, and a dropdown set to SOCKS5. What they actually need is IPs that don't die on the first request and a price they can predict.

SOCKS5 is the easy part. It's a protocol your tool already supports. The hard part is everything behind it: which network those IPs come from, how you're billed, and whether the vendor survives long enough for your balance to be useful. That's what this covers. 9Proxy is the provider used as the working example, because its pricing model (per-IP with unlimited bandwidth, or per-GB) lines up neatly with the two ways people actually use SOCKS5.

## SOCKS5 is a protocol, not a type of proxy

This distinction matters more than it sounds, because half the confusion around "SOCKS5 proxy lists" comes from treating the label as a quality grade.

SOCKS5 sits at the session layer. It forwards raw TCP and, in the spec, UDP packets between your client and the destination without reading or rewriting them. It doesn't know or care whether those bytes are HTTP, a game protocol, an SMTP session, or a torrent handshake. It just moves them.

An HTTP proxy works one layer up. It understands web requests, can add or modify headers, can cache responses, and is built specifically for browsing and scraping websites.

Compared with the older SOCKS4, version 5 added UDP support and three authentication methods: no authentication, username/password, and GSS-API. Most commercial providers give you username/password. UDP is hit-or-miss — some providers support it, some are TCP only — so check per vendor if your workload depends on it.

One detail that trips people up in practice: `socks5://` usually means DNS is resolved on your machine, while `socks5h://` pushes DNS resolution to the proxy. If your tool offers both and local DNS lookups leak your real location or break geo-targeting, use the `h` variant.

And the thing worth internalizing before you buy anything: SOCKS5 doesn't make you anonymous. Your anonymity is a property of the IP the proxy hands you. A SOCKS5 connection to a flagged datacenter IP is still a flagged datacenter IP.

## Picking between SOCKS5 and HTTP proxies

The honest rule is: pick whatever your tool accepts, then pick the strongest network that supports it.

|  | SOCKS5 | HTTP/HTTPS |
| --- | --- | --- |
| Layer | Session (forwards raw packets) | Application (parses web requests) |
| Works with | Any TCP traffic, plus UDP where supported | Web traffic |
| Typical use | Antidetect browsers, automation tools, non-browser clients, P2P | Pure web scraping, header control, caching |
| Watch out for | UDP support varies by provider | Won't handle non-HTTP protocols |

Plenty of stacks end up using both — HTTP for the scraping tier, SOCKS5 for the browser profiles. There's no prize for standardizing on one.

## The choice that actually changes your block rate: residential vs datacenter

If your SOCKS5 endpoints keep getting hit with CAPTCHAs, the protocol isn't the problem. The IP is.

Datacenter IPs live in hosting ASNs. Sites behind serious bot protection can see that in a single lookup, which is why they get challenged fast. Residential IPs come from consumer ISPs, so a request looks like it came from someone's living room.

There's a useful public number here. In Geekflare's 2026 review, a 300-request test ran through 9Proxy's rotating residential pool against a major e-commerce site sitting behind Cloudflare. The results: 293 passes (97.7%), 5 CAPTCHA challenges (1.7%), 2 hard blocks (0.6%), average response time 0.63 seconds [1]. The same test pattern through a datacenter pool produced a 34% block rate on the first pass.

The 0.63-second figure is worth reading honestly. Residential routing adds hops, so it's slower than datacenter by nature. What you're buying isn't speed, it's the success rate.

## How SOCKS5 pricing actually works

You're never paying for SOCKS5. It's free. You're paying for the network, and providers bill it in two ways.

**Per GB.** Traffic-based. Cheap for high-rotation jobs where each request pulls little data: SERP checks, price lookups, API polling, ad verification. Prices across the market range widely. A 2026 roundup published by DataImpulse puts entry-level residential traffic at $1/GB on its own service, several providers in the $3 to $4 range, and IPRoyal at $7.35/GB [2]. So "cheap SOCKS5 proxies" is a phrase that spans roughly a 7x cost difference for the same protocol label.

**Per IP.** A fixed number of IPs, usually with unlimited bandwidth for as long as the IP is alive. This is the model that fits long sessions: logged-in accounts, carts, multi-day automation, anything where a mid-flow IP change breaks the job.

Both models have a failure mode. Per-GB plans punish you when your block rate is high, because blocked requests still burn traffic. Per-IP plans punish you when the IP dies faster than your workflow finishes with it.

## Where 9Proxy fits into a SOCKS5 workflow

9Proxy is a residential proxy platform offering 20M+ verified residential IPs across 90+ countries, with targeting down to country, city, ZIP code, and ISP level, and support for HTTP/HTTPS and SOCKS5 [3][4]. It's been running since 2023, and it sells balance rather than subscriptions — you top up and spend against it [5].

The two models behave quite differently, and the difference shows up in how you connect, not just how you pay.

**Residential by IP.** Pay per IP, bandwidth unlimited while the IP is active. Unused IPs never expire. Natural IP lifespan runs from a few hours up to about 24 hours, and there's no natural rotation — if you need rotation you use the Auto Rotation Proxy feature, which rotates on custom intervals on selected ports. Access requires the 9Proxy desktop app, which does local port forwarding [6].

**Residential by GB.** Pay per GB, and you can generate unlimited endpoints from that traffic. Balance is valid 180 days (unlimited on the enterprise GB tiers). Sessions run in rotating mode or sticky mode, and you authenticate with username/password or an IP whitelist. Setup happens directly in the dashboard, no desktop app required [6].

That last point decides a lot for people. The per-GB route is the one that works on a headless server or a Linux box. The per-IP route effectively means Windows or macOS.

Both models accept SOCKS5, and the official docs walk through the common integrations: Proxifier, BitBrowser, ixBrowser, AdsPower, Dolphin Anty, and mobile setups [7][8][9]. There's also a "Today List" feature and an auto-refresh that detects and replaces dead IPs, both of which reduce how much of your balance goes to IPs that were already gone when you grabbed them [10].

If you want to see the current package ladder before reading further: 👉 [check 9Proxy's live pricing and plans](https://bit.ly/9-Proxy)

## Setting up SOCKS5 with 9Proxy, both paths

### Path A: IP-based, desktop app

1. Install and launch the 9Proxy app, then log in.
2. Fill in Country, State, and the other targeting fields, then search.
3. Right-click the IP you want in the proxy list and choose **Port Forwarding To Proxy**, then pick a port.
4. Open the Forwarding List and copy the IP and port.
5. In your browser profile or tool, set the type to **SOCKS5** and paste the address.
6. Run a proxy check before you start working.

On iOS there's a catch that catches people out: iOS only speaks HTTP proxies natively. To use SOCKS5 on an iPhone or iPad you need a client like Shadowrocket — add a config, set Type to SOCKS5, and enter the IP and port from the forwarding list [7].

### Path B: GB-based, dashboard only

1. Create a sub-account in your dashboard.
2. Generate a proxy session.
3. Build the username string, which is how targeting is passed:


<subaccount>-country-<code>-st-<state>-city-<city>-isp-<isp>-ssid-<session_id>-sst-<session_time>


4. Use your sub-account password as the password.
5. Paste host, port, username, and password into BitBrowser, AdsPower, Dolphin Anty, or whatever else you're running, select SOCKS5, and hit the proxy check button before creating the profile [8][9].

The username-as-configuration approach is unusual if you're used to picking a country from a dropdown, but it's why the same credential works across every tool that accepts a SOCKS5 endpoint.

## Every 9Proxy package and what it costs

9Proxy adjusted pricing on June 1, 2026 for IP-based and bundle packages. GB-based pricing was left unchanged [5]. These are all the currently published tiers.

**Residential by IP (one-off purchase, IPs never expire)**

| Package | Effective rate | Price | Purchase |
| --- | --- | --- | --- |
| 100 IPs | $0.24/IP | $24 | [Get 100 IPs](https://bit.ly/9-Proxy) |
| 500 IPs | $0.144/IP | $72 | [Get 500 IPs](https://bit.ly/9-Proxy) |
| 1,000 IPs + 500 bonus | $0.084/IP | $126 | [Get 1,500 IPs](https://bit.ly/9-Proxy) |
| 2,500 IPs | $0.084/IP | $210 | [Get 2,500 IPs](https://bit.ly/9-Proxy) |
| 5,000 IPs | $0.072/IP | $360 | [Get 5,000 IPs](https://bit.ly/9-Proxy) |
| 15,000 IPs | $0.048/IP | $720 | [Get 15,000 IPs](https://bit.ly/9-Proxy) |
| 25,000 IPs | $0.035/IP | $863 | [Get 25,000 IPs](https://bit.ly/9-Proxy) |
| 50,000 IPs | $0.029/IP | $1,438 | [Get 50,000 IPs](https://bit.ly/9-Proxy) |
| 100,000 IPs | $0.023/IP | $2,300 | [Get 100,000 IPs](https://bit.ly/9-Proxy) |
| 200,000 IPs | $0.021/IP | $4,140 | [Get 200,000 IPs](https://bit.ly/9-Proxy) |
| 500,000 IPs | $0.018/IP | $8,625 | [Get 500,000 IPs](https://bit.ly/9-Proxy) |

**Residential by GB**

| Package | Rate | Price | Validity | Purchase |
| --- | --- | --- | --- | --- |
| 5 GB | $3.00/GB | $15 | 180 days | [Get 5 GB](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | $2.10/GB | $105 | 180 days | [Get 55 GB](https://bit.ly/9-Proxy) |
| 100 GB | $1.50/GB | $150 | 180 days | [Get 100 GB](https://bit.ly/9-Proxy) |
| 200 GB | $1.00/GB | $200 | 180 days | [Get 200 GB](https://bit.ly/9-Proxy) |
| 1,000 GB | $0.80/GB | $800 | 180 days | [Get 1,000 GB](https://bit.ly/9-Proxy) |
| 2,000 GB | $0.75/GB | $1,500 | 180 days | [Get 2,000 GB](https://bit.ly/9-Proxy) |
| 3,000 GB | $0.72/GB | $2,160 | Unlimited | [Get 3,000 GB](https://bit.ly/9-Proxy) |
| 6,000 GB | $0.70/GB | $4,200 | Unlimited | [Get 6,000 GB](https://bit.ly/9-Proxy) |
| 10,000 GB | $0.68/GB | $6,800 | Unlimited | [Get 10,000 GB](https://bit.ly/9-Proxy) |

**Bundle packages (IPs plus traffic)**

| Bundle | Contents | Price | Purchase |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 | [Get the Pro bundle](https://bit.ly/9-Proxy) |

Those headline "from $0.015/IP" numbers you'll see in marketing copy refer to the largest tiers, not the entry ones. Worth remembering before you assume any of this is priced at fifteen thousandths of a dollar.

## Which model is worth it for which job

The bundles are where the arithmetic is easiest to check, so let's do it.

Starter is $30 for 100 IPs plus 5 GB. Bought separately, that's $24 + $15 = **$39**, so the bundle saves $9 over buying the same two packages à la carte.

Popular is $180 for 1,500 IPs plus 50 GB. The closest IP package is the 1,000 + 500 bonus tier at $126, and the matching GB tier is the 50 + 5 package at $105. That's **$231** separately, against $180 in the bundle. $51 saved, and you get the extra 5 GB either way.

Pro is $720 for 5,000 IPs plus 500 GB. The 5,000-IP package alone is $360. Standalone traffic at that volume sits between the 200 GB tier at $1.00/GB and the 1,000 GB tier at $0.80/GB, so 500 GB purchased separately would land somewhere between $400 and $500. The bundle beats that.

Now the qualitative side, which matters more than the discounts:

- **Long-lived logged-in sessions, e-commerce accounts, anything where one IP needs to persist through a whole workflow.** Per-IP. Unlimited bandwidth means you don't pay extra for a page-heavy session, and there's no traffic meter to watch.
- **Lightweight scraping with high rotation, geo-checks, SERP monitoring where each request pulls a few hundred KB.** Per-GB. You burn maybe a gigabyte a day and pay accordingly.
- **Mixed workloads, or you haven't measured your traffic yet.** A bundle, specifically the Starter one. It's the cheapest way to learn your own numbers.
- **Headless Linux setup.** Per-GB, because the IP model expects the desktop app.

One caveat for anyone planning to run 4G-style mobile workflows: none of these are mobile or datacenter proxies at the moment. The published lineup is residential. Datacenter access has been listed as "coming soon" on third-party coupon pages, and mobile isn't advertised at all, so if your project needs carrier IPs, that's a different vendor.

## What to check before you commit your budget

**Buy small first.** Because 9Proxy runs on a balance rather than a subscription, there's no annual contract to get trapped in, and no auto-renewal to cancel. Test with the actual target site you care about, not a generic "what is my IP" page. A 5 GB package costs $15. That's a cheap way to find out whether a network handles your specific targets.

**Weigh the reliability chatter carefully.** Some third-party pages claim 9Proxy went dark for stretches during 2026, and others say it's fully operational; those pages also contradict each other and several are run by resellers selling rival networks. There's no official statement I could verify either way. The practical takeaway doesn't depend on resolving it: keep working balances modest rather than parking months of budget in any single provider's wallet.

**Check whether you need SOCKS5 at all.** If you're scraping plain web pages with a normal HTTP client, SOCKS5 buys you nothing except a slightly more awkward setup. Use it where your tool requires it, or where you're routing non-HTTP traffic.

**Confirm who answers support.** 9Proxy advertises 24/7 human support through Telegram, email, and tickets, and Geekflare's review specifically flags the absence of scripted chatbot responses [1][10]. That's the kind of claim you only verify the hard way, when a pipeline breaks at an inconvenient hour.

## FAQ

**Is SOCKS5 better than an HTTP proxy?**
Not better, different. SOCKS5 forwards any TCP traffic and doesn't touch your packets, which makes it the right pick for antidetect browsers, P2P, and non-web protocols. HTTP proxies understand web requests and can handle headers and caching. Choose based on what your tool needs, then choose the best IP network that supports it.

**Do free SOCKS5 proxy lists work?**
They work in the sense that traffic sometimes moves. They're recycled or blacklisted IPs with short lifespans, unpredictable uptime, and no accountability when something logs your traffic. For a script you're debugging once, fine. For anything with an account or a budget attached, no.

**Do 9Proxy IPs expire?**
Unused IPs from per-IP packages never expire. GB-based balances are valid 180 days, except on the 3,000 GB, 6,000 GB, and 10,000 GB tiers, which don't expire at all.

**Does SOCKS5 work with antidetect browsers?**
Yes, and it's the standard setup. In BitBrowser, AdsPower, Dolphin Anty, and similar tools you select SOCKS5, paste host and port, add your username and password, and run the built-in proxy check before creating the profile.

**Can I use SOCKS5 on iPhone?**
Not natively. iOS only supports HTTP proxies in its Wi-Fi settings. For SOCKS5 you need an app like Shadowrocket, where you add a config, set the type to SOCKS5, and enter the IP and port.

**Per-IP or per-GB for a first purchase?**
Per-GB if you can't predict your traffic, per-IP if you know your sessions need to stay on the same address. The Starter bundle at $30 exists precisely so you don't have to guess correctly on the first try.

## Where this lands

If you're buying SOCKS5 access, you're buying a network wearing a protocol label. The things that decide whether it works are IP quality, targeting precision, session behaviour, and whether the billing model matches how your workflow consumes resources, not the fact that SOCKS5 appears in a dropdown.

9Proxy's case rests on a clean-ish residential pool at mid-market prices, a genuine choice between per-IP and per-GB billing, and integration docs that cover the tools most people actually run. The per-IP tiers got more expensive in June 2026, so if you were comparing against a snapshot from earlier in the year, re-check the numbers. Start with one package sized to a real project, measure your block rate and traffic consumption on your own targets, and scale from there.

👉 [Set up a 9Proxy account and pick the package that matches your workload](https://bit.ly/9-Proxy)
