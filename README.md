# cloud computing security: from misconfigurations to DDoS, what actually breaks and how to protect your workloads

Most people who search for cloud computing security aren't looking for a textbook definition. They have servers, databases, or customer data sitting in someone else's data center, and they want to know two things: what am I actually responsible for, and what's the most likely way this goes wrong?

Those are the right questions, because cloud security doesn't fail the way most people imagine. It's rarely a sophisticated zero-day burning through your defenses. It's almost always a misconfigured storage bucket, a leaked API key, an over-privileged account, or a network segment that was never supposed to be reachable from the internet. Meanwhile, on the infrastructure side of the fence, things like DDoS absorption and network isolation are largely your hosting provider's job — and that's where your choice of provider quietly matters.

This guide walks through the parts of cloud security that cause real incidents, the practices that prevent them, and what to look for at the infrastructure layer when you're choosing where your workloads live. Along the way, we'll use Sharktech — a DDoS-focused hosting provider operating since 2003 — as a concrete example of what provider-side security controls look like, with their current cloud plans and real pricing.

## What cloud computing security actually covers

Cloud computing security is the set of controls, policies, and practices that protect data, applications, identities, and infrastructure running in a cloud environment. In practice it splits into a handful of domains:

- **Data security** — encryption at rest and in transit, backups, data classification
- **Identity and access management (IAM)** — who (or what) can touch which resource, with how much privilege
- **Network security** — firewalls, segmentation, private networking, DDoS mitigation
- **Workload security** — OS patching, container and Kubernetes hardening, vulnerability scanning
- **Visibility and response** — logging, monitoring, incident response planning
- **Compliance** — mapping all of the above to whatever GDPR, SOC 2, HIPAA, or PCI obligations apply to you

None of these are optional in a real deployment, and — this is the part that surprises people — your cloud provider only handles some of them. Which ones depend on the service model you're using.

## The shared responsibility model: the most misunderstood concept in cloud security

Every major cloud provider operates on a shared responsibility model. The provider secures the underlying infrastructure — the physical data centers, the hypervisor, the hardware. You secure everything you build on top of it. CrowdStrike, Sysdig, and CISA/NSA guidance all lead with this point for a reason: organizations that don't understand where the line sits routinely leave entire layers of their stack unsecured.

Where that line sits depends on what you're running:

| Layer | IaaS (VPS, cloud VMs) | PaaS | SaaS |
| --- | --- | --- | --- |
| Physical infrastructure & data center | Provider | Provider | Provider |
| Operating system & network config | **You** | Provider | Provider |
| Applications & code | **You** | **You** | Provider |
| Data & access policies | **You** | **You** | **You** |
| Identity, accounts, credentials | **You** | **You** | **You** |

The blunter version: the moment you rent a cloud VM or a VPS, the OS, firewall, patching, application code, and every account that logs in are your problem. The provider's job is to keep the platform underneath you standing — redundancy, network capacity, and increasingly, absorbing volumetric attacks before they reach your door.

That last part is worth dwelling on, because it's the one infrastructure-layer control that's impossible to self-provide at small scale. A determined DDoS attack measured in tens of gigabits will saturate anything you can buy on a small budget. Either your provider scrubs it upstream, or your service is offline. We'll come back to this.

## The threats that actually cause cloud incidents

Security vendors' threat lists mostly converge on the same culprits, and they're noticeably less exotic than the news suggests.

**Misconfiguration is the front door.** CrowdStrike lists unmanaged attack surface, human error, and misconfiguration as the top cloud security risks. An S3-equivalent storage bucket left world-readable, a database bound to 0.0.0.0/0, a firewall rule copied from a dev environment into production — these are the incidents that fill postmortems. Misconfigured services are a consistently popular attack vector precisely because no exploit is required, just an open door.

**Identity and credential attacks.** Account hijacking appears on virtually every cloud threat list. Attackers don't break encryption; they log in, using phished credentials, leaked API keys, or tokens committed to a public repository. Sysdig's guidance puts zero-trust and least-privilege IAM near the top of its practice list for exactly this reason.

**Insecure APIs.** Public cloud is API-driven by design, which means every over-permissive service account or unauthenticated endpoint is a doorway. Darktrace and several other vendors flag insecure APIs as a top-tier threat.

**DDoS.** Distributed denial-of-service remains the blunt instrument of the internet. It's cheap for attackers and, as one multi-year Sharktech customer running game servers put it in a review, attacks in the 3–8 Gbit range are just a fact of life for anything public-facing. The question isn't whether the attack comes — it's whether your upstream provider filters it before your users notice.

**Unpatched software and insider risk.** Old vulnerabilities on unmanaged systems and overly broad internal access round out most lists. Neither is glamorous, and both are preventable with basic hygiene.

Notice what's absent: exotic cryptography-breaking attacks. Encrypt your data properly and patch your systems, and you've removed most of the script-driven attack surface.

## Cloud security best practices that actually matter

The practices below are the consensus across CrowdStrike's 20-point list, Sysdig's guidance, and CISA/NSA recommendations. In order of return on effort:

1. **Map your responsibility first.** You can't secure what you think the provider is handling. Before anything else, write down which layers of the table above apply to your deployment.
2. **Least privilege IAM with MFA everywhere.** Every human and service account gets the minimum permissions it needs, multi-factor authentication is mandatory, and admin accounts are audited periodically. This single practice blocks the majority of account-hijacking paths.
3. **Encrypt data at rest and in transit.** Storage volumes, database disks, backups, and every connection between services. Modern guidance also suggests planning for post-quantum-safe encryption so "harvest now, decrypt later" attacks don't pay off later.
4. **Segment your network.** Split workloads into separate virtual networks, only allow communication that's actually required, and keep backend traffic off the public internet entirely. Microsegmentation limits how far an attacker can move laterally once they're inside.
5. **Use firewall rules and security groups as policy, not decoration.** Default-deny inbound, explicit allow rules, and regular review. On platforms that expose OpenStack-style security groups, this is the fastest meaningful hardening available.
6. **Monitor for misconfiguration continuously.** Automated posture checks beat annual audits. A security score that degrades when you make a mistake is worth more than a quarterly review that finds the mistake three months late.
7. **Patch on a schedule, scan continuously.** Test patches in isolation, then apply them consistently. Prioritize vulnerabilities that touch production over ones on a dev box.
8. **Keep logs and write an incident response plan.** Know who does what when something breaks. Rehearse it once. After an incident, fix whatever the plan got wrong.
9. **Train the humans.** Phishing remains the most common initial access vector. Security awareness isn't a checkbox; it's the control that covers the attacks no firewall can stop.

If you only do three things from this list: least-privilege IAM with MFA, encryption everywhere, and default-deny firewalling. That trifecta eliminates most real-world attack paths.

## The infrastructure layer: what your hosting provider is responsible for

Here's where the abstract best-practice lists meet an actual purchasing decision. Once your workloads are deployed, several security-critical controls live below the OS — and you want a provider whose defaults match the checklist above rather than selling security back to you as an add-on.

Sharktech is a useful example because their positioning is explicitly security-shaped: DDoS-protected hosting since 2003, with mitigation points of presence in Los Angeles, Denver, Chicago, and Amsterdam. Looking at how their cloud platform is built, you can see the provider-side half of the shared responsibility model in concrete terms:

- **DDoS protection is included, not upsold.** Every VPS, bare-metal, and cloud plan ships with up to 60 Gbps of DDoS mitigation at no extra cost. You don't have to forecast attack risk before buying a protection tier — the floor is built in. For context, a third-party comparison review noted this is standard across their plans rather than a trial or teaser, and a long-running user review on LowEndTalk reported their mitigation successfully absorbing repeated attacks on live services.
- **Firewall and security groups are native.** The OpenStack-based cloud platform exposes granular traffic rules — the exact control the best-practice lists tell you to use for default-deny inbound filtering.
- **Private networking isolates backend traffic.** VMs can communicate over private networks with no public exposure, which is the practical implementation of segmentation advice: databases and internal services simply never get a public interface.
- **Free integrated VPN for hybrid setups.** Bridging cloud VMs to on-premises infrastructure without sending that traffic across the open internet is included at no charge.
- **Weekly-refreshed OS images.** Official Linux cloud images are updated weekly, so a freshly deployed VM starts with current patches — not a snapshot from eight months ago that you have to patch before it's safe to expose.
- **Billing and infrastructure are isolated by design.** Keeping the billing system separated from the cloud management plane reduces cross-system attack risk — a small architectural detail that reflects actual threat modeling rather than marketing.
- **No vendor lock-in, including on your data.** You can download your disk images at any time — for offsite backup, disaster recovery, or leaving entirely. From a security governance standpoint, this matters more than it sounds: an exit strategy is a control against provider-side risk.

The honest caveat from third-party testing: Sharktech's standard SSD storage tier delivers roughly 350 MB/s sequential throughput (their own published estimate), which is fine for general workloads but not stellar for I/O-heavy databases. Their NVMe tier — quoted at up to 1.2 GB/s and 18,000 IOPS per volume — is the right choice if you're running anything storage-sensitive. HostAdvice's independent review measured the network at roughly 10 Gbps down and 22 Gbps up internally with 0.17 ms idle latency, which puts the network layer firmly in "not your bottleneck" territory.

## Sharktech's current cloud plans and pricing

Sharktech's public cloud runs on the OpenStack-based platform described above, deployed in Los Angeles, Las Vegas, Denver, Chicago, or Amsterdam. Plans are resource tiers rather than fixed boxes — each tier includes a committed baseline with the ability to scale within a capped maximum, so an unplanned spike can't produce an unplanned invoice. The current lineup:

| Plan | vCPU | RAM | SSD Storage | Bandwidth | Price (from, monthly) | Get started |
| --- | --- | --- | --- | --- | --- | --- |
| Small | 4–16 cores | 8–32 GB | 300–2400 GB | 20 TB+ | $39.00/mo | [Deploy Small](https://bit.ly/SharKTech) |
| Medium | 8–32 cores | 16–64 GB | 800–6400 GB | 20 TB+ | $79.00/mo | [Deploy Medium](https://bit.ly/SharKTech) |
| Large | 32–128 cores | 64–256 GB | 1500–12000 GB | 20 TB+ | $249.00/mo | [Deploy Large](https://bit.ly/SharKTech) |
| Enterprise | 64+ cores | 128 GB+ | 5000 GB+ (no cap) | 20 TB+ | $499.00/mo | [Deploy Enterprise](https://bit.ly/SharKTech) |
| Custom | Fully configurable | — | NVMe/SSD/HDD tiers | — | Quoted by sales | [Request a custom plan](https://bit.ly/SharKTech) |

A few pricing mechanics worth knowing before you commit:

- **Billing model.** Public Cloud is pay-as-you-go: each plan includes a fixed resource commit, and usage above the base is billed hourly. If you'd rather have fixed monthly costs, Sharktech's Dedicated Cloud runs on the same infrastructure with prepaid resource pools — you pay for exactly what you ordered, no more, no less.
- **Hourly rates for overage** (per the official pricing page): $0.0025/hr per CPU core, $0.0035/hr per GB of RAM, $0.00006/hr per GB of SSD, $0.00009/hr per GB of NVMe, $0.00002/hr per GB of HDD. The Enterprise tier's HostAdvice listing works out to roughly $0.74/hr.
- **Bandwidth:** incoming is unlimited and free, 5,000 GB outgoing is included, and extra egress is $0.002/GB. One public IPv4 address is free at activation; additional ones are $1.50/month each.
- **Uptime:** the cloud and VPS platform pages carry a 99.999% uptime figure.
- **Payments:** credit card, PayPal, wire transfer, Western Union, and Alipay.

For smaller single-server workloads, Sharktech also sells Smart VPS plans starting at $7.95/month on monthly billing — dropping to $3.98/month when paid annually (quarterly and semi-annual cycles get 25% and 35% off respectively). Those include the same 60 Gbps DDoS protection, Xeon Gold CPUs, NVMe storage, and a 1 Gbps port, with 4–300 TB of data transfer depending on size. If a public-facing service doesn't need multi-VM infrastructure, that's the cheaper route to the same network-layer protection.

> **One caveat before you order:** per HostAdvice's review, Sharktech has no general money-back guarantee — payments, including setup fees and monthly charges, are non-refundable. The only exception is a billing dispute raised within 30 days that Sharktech upholds, and even then you receive a credit rather than a refund. There's no free trial either. The upside of hourly billing is that testing a configuration costs cents; the flip side is that you should be reasonably sure before you commit to a large annual plan.

Third-party testing has been positive where it matters for a security-conscious buyer: HostAdvice's expert review scored the public cloud 9.4/10 overall, with support tickets answered in under 40 minutes during a deliberately 1 AM test, and a stress test running clean with no failures. The same review's honest con was the limited number of regions — five data centers against the dozens you'd get from a hyperscaler. That's the real trade-off with a smaller provider: more focused service and transparent pricing, less geographic redundancy.

## Putting it together: a practical security checklist when you deploy

If you're evaluating cloud security — whether on Sharktech or anyone else — here's the order of operations that matches how real incidents happen:

1. **Confirm what the provider covers.** DDoS mitigation included or upsold? Native firewall and private networking? How fresh are the deploy images? Can you get your data out?
2. **Lock down identity on day one.** MFA on the portal, least-privilege SSH keys rather than shared passwords, and audit accounts regularly.
3. **Default-deny your inbound firewall** before the first service goes public. Use the platform's security groups so the rules live with the infrastructure, not in a wiki page.
4. **Segment as you build.** Public frontend in one network, database and internal services in a private network with no public interface at all.
5. **Encrypt volumes and connections**, enable backups, and test a restore once — an untested backup is a hope, not a control.
6. **Patch on a schedule** and lean on freshly-updated deploy images rather than long-lived snowflake servers.
7. **Decide your exit plan before you need it.** The ability to download your own disk images is the difference between a migration and a hostage situation.

Cloud computing security isn't a product you buy once — it's the working arrangement between what your provider filters at the edge and what you configure above the OS. Get both halves right, and the most common attack paths in the cloud — open doors, leaked credentials, saturated pipes — simply stop being available. If you're comparing providers on the infrastructure half of that equation, 👉 [take a look at Sharktech's cloud plans and current pricing](https://bit.ly/SharKTech) — the included 60 Gbps DDoS protection and native firewall tooling give you a reasonable baseline to measure other quotes against.
