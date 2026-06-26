*My electricity bill kept changing by the hour, so I taught a battery to outsmart it.*

## I Built an AI Brain for My Home Battery

Electricity hasn't been one flat price for me in a while. It moves every hour. Some hours are nearly free, others are painful, and on a really volatile day the worst hour can cost eight or nine times the cheapest one.

A home battery is the obvious lever. Charge it when power is cheap, run the house off it when power is dear. Simple idea. The hard part is the timing — get it wrong and you've happily paid to store expensive electricity, which is the opposite of the point. So I handed the timing to a bit of software. A small "brain" that watches the prices, decides when to charge and when to discharge, and just gets on with it.

**How it started**

The first version was almost embarrassingly simple. One rule: if the price drops below a threshold, charge. If it climbs above one, discharge. A basic automation, no intelligence at all.

And honestly, it was fine. For a while. But a fixed rule is short-sighted by design. It doesn't know there's a cheaper hour coming at 3am, or that tomorrow evening's peak is the one really worth saving for, or that a hot afternoon means the air conditioning is about to push demand up. It only ever reacts to right now. Closing that gap — making it plan instead of react — is the whole reason the rest of this exists.

*[IMAGE 1 — the live status: charging or discharging, how full the battery is, how cheap the current hour is]*

**What it does now**

Roughly once an hour it looks ahead and makes a plan for the next day and a half.

The "looking ahead" part is where the AI sits. It predicts how much electricity the house will use, and it factors in the weather, because a warm day means more cooling and more demand. Then it lines that forecast up against the coming hourly prices and figures out the cheapest way through: when to fill the battery, when to lean on it, and — importantly — always leaving some charge in reserve in case the power goes out. Then it carries the plan out itself. It'll even shut off the office air conditioner while charging, because the two were fighting over the same limited supply.

*[IMAGE 2 — the dashboard: what the battery's doing, the hourly prices, and the cost with the battery versus without]*

**I don't guess, I check**

The part I'm actually proud of isn't the AI. It's that I don't change anything on the real battery on a hunch.

Before I touch a setting, I replay it against months of past prices and see what it *would* have done. That's how I settled an annoying question — was it worth upgrading my power connection? I ran it both ways. The bigger connection roughly quadrupled the savings in the simulation, so I went ahead. The numbers decided, not me.

One result genuinely surprised me. Roughly a third of the savings I *could* be making slips away purely because no forecast is ever perfect. That's a slightly humbling number, but a useful one — it tells me the next thing to improve is the predictions, not anything else.

*[IMAGE 3 — the forecast: how much it expects to save over the coming day and a half]*

**Where it is today**

It's live. It runs by itself, quietly shaving money off the bill, and it gets a little sharper every week as it learns from more data. Not bad for something that started as a single if-statement.
