https://www.darkreading.com/application-security/fifa-bug-world-cup-streams-remote-takeover

Hey everyone, welcome back to the show. Today we're talking about the World Cup. Specifically, about how the entire global broadcast of the World Cup was, for a little while, sitting behind a lock made of wet cardboard.

Let me set the scene. It's June 2026. FIFA — the organization that runs the sport where grown men fall over if you breathe near their ankle — has just been handed one of the biggest broadcasting events on the planet. Billions of viewers. Every camera angle. Every commentator. Every single feed, worldwide, funneled through FIFA's systems.

And a hacker going by "BobDaHacker" looked at all of that... and got in. Not through some elite zero-day. Not through a USB drive dropped in a parking lot. He signed up to be a football agent.

Yeah. You know, the guy who negotiates transfers and takes 10% when some 19-year-old signs for a club in Portugal? Anyone can apply for that. You upload an ID, you verify an email, and congratulations — FIFA spins you up an account in its Microsoft Entra system. Very normal. Very chill.

Except — and this is the part that should make every IT director reading this spill their coffee — that agent account lives in the *exact same tenant* as FIFA's core internal systems. The stuff running the actual tournament.

So Bob pokes around, tries to access FIFA's core data platform, and gets hit with an "access denied" message. Great, he thinks, security's working. Except it isn't. Because that "access denied" message was just... a website being polite. Underneath, the actual server — the backend, the part that's supposed to be the bouncer — just didn't care. It would hand over the keys to anyone who politely asked.

Bob describes this as one of the most common patterns he sees: companies build a beautiful front door with a "you shall not pass" sign taped to it, and then leave the back door not just unlocked, but slightly ajar with a breeze blowing through it.

So what did that get him? Oh, just — the live production hub for the entire World Cup broadcast. Every camera feed. Every stream. And not just *viewing* access — *control*. He could've blacked out a match mid-game. He could've swapped the feed for literally anything else.

Including — and I want you to imagine this actually happening — Rickrolling the entire planet during a live World Cup match. Billions of people, tuning in for Cote d'Ivoire vs. Ecuador, and instead getting Rick Astley telling them he's never gonna give them up. Or, as Bob himself suggested, just live Subway Surfers gameplay. On every TV network. During an active match. That's not a hack, at that point, that's performance art.

And it doesn't stop at video. Same account also opened the door to the match management system — where someone could've changed scores in real time, or rescheduled matches. The commentary system, where you could've fed commentators whatever nonsense you wanted, in any language. And the analytics and developer environments, which had financial data, transfer records, all of it.

So naturally, being a responsible human being and not a menace to society, Bob tries to report this to FIFA. And this is where it gets almost funnier than the hack itself. FIFA has no security contact page. No vulnerability disclosure policy. No bug bounty. Nothing. It is, digitally speaking, an organization with no doorbell.

So what does he do? He calls CISA. He calls the FBI. Because apparently that was genuinely the most functional path to get a message to FIFA — going through the actual United States federal government, because FIFA's own front desk didn't pick up the phone.

And to FIFA's credit — the bug did get fixed, seemingly the very next day, once federal cybersecurity agencies got involved. Which, sure, great outcome. But there's something almost poetic about the fact that a bug this severe, at an event this massive, could only be resolved by essentially skipping the company entirely and going straight to the government.

CISA, for their part, put out a statement about all the great cybersecurity work they'd done around World Cup stadiums and host cities — physical security, critical infrastructure, the whole deal. Not one word about the digital broadcast infrastructure that a guy signing up as a soccer agent had just casually strolled into.

So here's the moral of the story: the next time someone tells you their system is secure because "there's an access denied page," ask them — is that a real lock, or is that just a sign on an unlocked door? Because for a few days this June, the entire World Cup was one Angular component away from becoming the Rick Astley Invitational.

That's the show. Go check your backend access controls. Seriously. Right now. I'll wait.
