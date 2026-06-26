My electricity price now changes every hour — on a bad day the worst hour costs 8–9x the cheapest.

So I built an AI to outsmart it. It runs my home battery: charge when power's cheap, run the house off it when it's dear.

It started as one if-statement — cheap, charge; expensive, discharge. But that's short-sighted: it can't see the cheaper hour coming at 3am, or tomorrow's peak. So now, once an hour, it forecasts the home's usage, lines it up against the coming prices, plans the cheapest day-and-a-half ahead — then runs it itself.

But the part I'm proud of isn't the AI. It's everything around it that makes it trustworthy enough to act on its own:

→ a test suite that runs on every change, so a bad edit gets caught on my screen — not on the hardware powering my house
→ backtests against months of real prices, so settings change on evidence, not hunches
→ dashboards tracking forecast accuracy and system health, with alerts the moment anything drifts

And the fun part: I can just ask. My home AI assistant (built on the open-source Hermes agent) has a read-only helper that reads the live data and tells me what the battery's doing and how much I saved today. It can see everything and change nothing — because two things fighting over one battery is exactly how you break it.

The lesson I keep relearning: the model is the easy part. Tests, version control, monitoring, and the discipline to prove a change before you ship it — that's what earns an AI the right to run on its own.

The AI makes the decision. The engineering earns the trust.

#AI #MachineLearning #SoftwareEngineering #Automation #CleanEnergy
