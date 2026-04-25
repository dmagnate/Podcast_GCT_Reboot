"The Roblox-to-Vercel Pipeline: How One AI Tool Unlocked a $2M Breach"

[INTRO]
(upbeat, slightly urgent tone)
Welcome back to the show. I'm going to tell you a story today that involves a Roblox cheat code, a productivity AI tool, and a data breach now being sold for two million dollars. It sounds like a stretch — but every single link in that chain is real, and it happened just this week. The target? Vercel — one of the most widely-used cloud deployment platforms on the internet, the company behind the Next.js framework that powers an enormous chunk of the modern web.
Let's break it down.

[SEGMENT 1: WHO IS VERCEL?]
(informative, accessible tone)
If you're not deep in the developer world, here's the quick pitch on Vercel. They're a cloud infrastructure company that lets developers deploy web applications at scale — think of them as the launch pad for a huge portion of SaaS products, fintech dashboards, DeFi interfaces, and more. Next.js, their flagship open-source framework, racked up over 520 million downloads in 2025 alone. So when Vercel has a bad day, a lot of organisations feel it downstream.
On April 19th, 2026, Vercel had a very bad day. They published a security bulletin confirming that an attacker had gained unauthorised access to certain internal systems and — critically — a limited subset of customer credentials had been compromised. The attacker is being described as "highly sophisticated" with, quote, deep operational knowledge of Vercel's product API surface. And a threat actor going by the name ShinyHunters has reportedly listed the stolen data on BreachForums for two million dollars in Bitcoin.
So how did this happen?

[SEGMENT 2: THE ATTACK CHAIN ]
(narrative, storytelling mode)
This is a supply chain attack — which means the attacker didn't break directly into Vercel. They found a door that was already open, in a tool that Vercel trusted. And to find that door, we have to rewind even further — to a gaming forum and a piece of malware.
According to cybersecurity firm Hudson Rock, the whole chain begins with an employee at a company called Context.ai — a small AI productivity startup. This employee, sometime around February 2026, was browsing for Roblox game exploit scripts. Cheat codes, essentially. One of those scripts was laced with something called Lumma Stealer — a well-known piece of infostealer malware. Once executed, Lumma quietly harvested the employee's credentials: their Google Workspace login, API keys for services like Supabase, Datadog, Authkit — and crucially, a support account at Context.ai itself.
Now here's where Context.ai becomes the pivot point. Context.ai makes AI productivity tools — think agents that help you build presentations and manage documents. One of their products is called AI Office Suite. And to use it, users connect it to their Google account. A Vercel employee had signed up for AI Office Suite — not through an enterprise agreement, just as an individual consumer — using their corporate Vercel email address. And when they connected it, they clicked "Allow All" on the OAuth permissions screen.
That "Allow All" click handed the app broad access to Vercel's Google Workspace environment. Not just the employee's personal files — the internal OAuth configuration at Vercel apparently allowed that single grant to propagate enterprise-wide.
So by March 2026, when the attacker — armed with the stolen Context.ai support credentials — accessed Context's AWS environment and harvested OAuth tokens for consumer users, one of those tokens belonged to a Vercel employee. And that token opened a door straight into Vercel's Google Workspace. From there, the attacker pivoted into Vercel's internal environments, enumerated and decrypted environment variables that weren't marked as "sensitive," and moved laterally through the systems. The good news — if there is any — is that environment variables explicitly flagged as sensitive are encrypted in a way that prevents them from being read. Vercel says there's currently no evidence those were accessed. But the non-sensitive variables? Exposed. And for some customers, that meant their Vercel credentials were compromised too.
No zero-day. No sophisticated exploit. Just a gaming cheat, an overpermissioned OAuth grant, and a supply chain that nobody was watching closely enough.

[SEGMENT 3: THE FALLOUT]
(measured, factual)
Vercel moved quickly once they identified the breach. They engaged Google Mandiant — Google's incident response arm — along with other cybersecurity firms and law enforcement. They also co-ordinated with GitHub, Microsoft, npm, and the security firm Socket to confirm that no npm packages published by Vercel had been tampered with. For the developer community, that confirmation was a relief — a compromised npm package could have cascading effects across thousands of downstream applications.
Vercel CEO Guillermo Rauch stated publicly on X that Next.js, Turbopack, and their open-source projects remain unaffected.
Affected customers have been contacted directly and urged to rotate credentials immediately. Vercel also published a list of recommended actions: review your activity logs, rotate environment variables that aren't marked sensitive, enable multi-factor authentication, and check that your Deployment Protection settings are at least at Standard level. They've also released the OAuth App ID for Context.ai — which Google Workspace admins can use to check whether anyone in their organisation granted it access.
Context.ai, for their part, published their own security bulletin, acknowledged the March incident involving their AWS environment, and confirmed they've been working with CrowdStrike to validate containment.

[SEGMENT 4: THE BIGGER LESSON ]
(reflective, authoritative)
Here's what makes this story more than just a dramatic headline.
Vercel is not an outlier. The pattern that played out here is playing out quietly across thousands of organisations right now. A JetBrains survey from late 2025 found that 85% of developers use AI tools in their daily work. Most of those tools ask for broad OAuth permissions. Most organisations have no centralised inventory of which AI tools their employees have connected to corporate accounts. And most Google Workspace and Microsoft 365 environments — by default — allow any employee to grant any third-party app access to their enterprise account.
Jaime Blasco, CTO of Nudge Security, put it plainly: move to admin-managed OAuth consent. If a new app needs to touch corporate data, it goes through a review first. That single policy change would have blocked this entire attack chain before it started.
Giuseppe Trovato, Head of Research at Geordie AI, frames it as an agent permissions problem: treat AI tool access the way you treat service account access. Audit it. Minimise it. And make sure you can revoke it fast.
And there's a third dimension here — environment variable hygiene. Vercel has a "sensitive" flag that encrypts secrets at rest and makes them unreadable after creation. But it's opt-in. It requires someone to make a deliberate choice. Vercel has now added a team-level setting to enforce sensitive-by-default. If you're on Vercel, go find that setting and turn it on.
The summary from David Lindner, CISO at Contrast Security, is blunt and worth quoting: "No exploit. No zero-day. Just an unsanctioned AI tool, an overpermissioned OAuth grant, and a gaming cheat download."


What we're watching in 2026 is the maturation of a new class of supply chain attack. Attackers aren't breaking in anymore — they're logging in, through the tools you already trust. The entry point isn't your firewall. It's your employee's "Allow All" click on a Tuesday afternoon.
The Vercel breach is a case study in what happens when AI tool adoption outpaces governance. The fix isn't to stop using AI tools — that ship has sailed. The fix is to treat them like the privileged access they are.
Rotate your keys. Audit your OAuth grants. And maybe don't download Roblox cheats on your work machine.