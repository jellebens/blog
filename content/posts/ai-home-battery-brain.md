*My electricity bill kept changing by the hour, so I taught a battery to outsmart it.*

## I Built an AI Brain for My Home Battery

Electricity hasn't been one flat price for me in a while. It moves every hour. Some hours are nearly free, others are painful, and on a really volatile day the worst hour can cost eight or nine times the cheapest one.

A home battery is the obvious lever. Charge it when power is cheap, run the house off it when power is dear. Simple idea. The hard part is the timing — get it wrong and you've happily paid to store expensive electricity, which is the opposite of the point. So I handed the timing to a bit of software. A small "brain" that watches the prices, decides when to charge and when to discharge, and just gets on with it.

**How it started**

The first version was almost embarrassingly simple. One rule: if the price drops below a threshold, charge. If it climbs above one, discharge. A basic automation, no intelligence at all.

And honestly, it was fine. For a while. But a fixed rule is short-sighted by design. It doesn't know there's a cheaper hour coming at 3am, or that tomorrow evening's peak is the one really worth saving for, or that a hot afternoon means the air conditioning is about to push demand up. It only ever reacts to right now. Closing that gap — making it plan instead of react — is the whole reason the rest of this exists.

*[IMAGE 1 — the live status panel: battery at 63% (8.2 kWh stored), currently DISCHARGING, the current hour flagged as the most expensive of the day (100%), and today's running tally — €0.17 saved so far, 25.8 kWh charged, 17.8 kWh discharged, 0 failures]*

**What it does now**

Roughly once an hour it looks ahead and makes a plan for the next day and a half.

The "looking ahead" part is where the AI sits. It predicts how much electricity the house will use, and it factors in the weather, because a warm day means more cooling and more demand. Then it lines that forecast up against the coming hourly prices and figures out the cheapest way through: when to fill the battery, when to lean on it, and — importantly — always leaving some charge in reserve in case the power goes out. Then it carries the plan out itself. It'll even shut off the office air conditioner while charging, because the two were fighting over the same limited supply.

*[IMAGE 2 — the dashboard: what the battery's doing, the hourly prices, and the cost with the battery versus without]*

**I don't guess, I check**

The part I'm actually proud of isn't the AI. It's that I don't change anything on the real battery on a hunch.

That caution starts before anything reaches the battery at all. The code that runs the show has its own suite of automated tests, and they run on every single change. A careless edit gets caught on my screen, not on the hardware keeping my lights on. It sounds boring. It's the boring that lets me trust the rest.

Before I touch a setting, I replay it against months of past prices and see what it *would* have done. That's how I settled an annoying question — was it worth upgrading my power connection? I ran it both ways. The bigger connection roughly quadrupled the savings in the simulation, so I went ahead. The numbers decided, not me.

I run these experiments for the small decisions too. Take a simple one: how hard should the battery work? Chase every last cent and it cycles more often, and every cycle wears it out a little. So I replayed a whole range of settings against the same historical prices and watched for the point where the extra savings stopped being worth it. The sweet spot kept almost all of the money while cycling the battery about a third less — far gentler on an expensive piece of hardware, for a difference I'd never have noticed on the bill.

One result genuinely surprised me. Roughly a third of the savings I *could* be making slips away purely because no forecast is ever perfect. That's a slightly humbling number, but a useful one — it tells me the next thing to improve is the predictions, not anything else.

**Seeing what it's doing**

A system that quietly spends my money while I'm not looking has to be glanceable. So honestly, most of what I built *around* the AI is about visibility — turning raw numbers into something I can read in a second.

The panel at the top of this post is the live view: how full the battery is, what it's doing right now, how cheap the current hour is, and today's running tally. In that screenshot it was discharging straight into the single most expensive hour of the day — which is exactly the point.

Then there's the forecast view: the AI showing its homework. How much it expects to save over the coming day and a half, split into what the bill *would* have been with no battery versus the optimized plan, with a running total that climbs slot by slot.

*[IMAGE 3 — the forecast view: expected savings over the next ~36 hours, the optimized plan versus a no-battery baseline]*

But a forecast is only worth trusting if you check it against reality. So a separate view keeps score on **how good the predictions actually were** — predicted household usage laid over what really happened, and an error trend over the month. That scoreboard is where the "a third of the savings lost to imperfect forecasts" figure comes from: I can literally watch the gap between what was possible and what the forecast managed to capture. It's also what tells me forecasting — not anything else — is the next thing worth improving.

*[IMAGE 4 — prediction accuracy: forecast versus actual household usage, with the error trend over the month]*

And then the boring-but-essential one: an operational view that answers a single question — *is everything actually working?* When did it last finish a planning cycle? Is the battery's control responding, or has it gone quiet? Does what the system *thinks* it commanded match what the battery is *actually* doing? Has anything errored out? If any of those drifts, an alert fires long before I'd spot it on a bill.

*[IMAGE 5 — the operational view: time since the last successful cycle, control health, commanded-versus-measured mode, and the error count]*

That last one matters more than it sounds. The line between a fun weekend experiment and something you let run unattended is whether it can tell you — loudly — the moment it stops behaving.

**Asking the AI what's going on**

Dashboards are great when you're actually looking at them. The part that still makes me grin, though, is that I can just *ask*.

I run a personal AI assistant at home, built on the open-source Hermes agent, and I gave it a dedicated little helper whose only job is to watch the optimizer and explain it like a human would. I can ask "what's the battery doing right now?" or "how much did we save today?" and it reads the very same live numbers the dashboards use and answers in a sentence.

The crucial word there is **read-only**. The assistant can see everything and change nothing. The optimizer stays the one and only thing allowed to touch the battery — because the quickest way to wreck a careful system is to let two things give orders to the same hardware. So the AI observes, summarizes, and flags anything odd — a stale cycle, a control surface gone quiet, savings slipping negative — but it never reaches for the controls itself.

It's a small thing, but it changed how the whole project feels. The battery stopped being a box on the wall with a dashboard, and became something I can have a conversation with.

**Where it is today**

It's live. It runs by itself, quietly shaving money off the bill, and it gets a little sharper every week as it learns from more data. Not bad for something that started as a single if-statement.
