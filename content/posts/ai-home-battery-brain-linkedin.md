My electricity bill stopped being one price. It changes every hour — and on a volatile day the worst hour costs 8–9x the cheapest.

So I built an AI to outsmart it.

It runs my home battery: charge when power is cheap, run the house off it when power is dear. The idea is trivial. The timing is not — get it wrong and you've literally paid to store expensive electricity.

The first version was one if-statement: cheap, charge; expensive, discharge. It worked. But a fixed rule is short-sighted. It doesn't know a cheaper hour is coming at 3am, or that tomorrow's peak is the one worth saving for, or that a hot afternoon means the AC is about to spike demand.

So now, once an hour, it plans the next day and a half:

→ predicts the home's electricity use (weather included — heat means cooling means demand)
→ lines that up against the coming hourly prices
→ works out the cheapest path, always keeping a reserve in case the power goes out
→ then carries the plan out itself

But the part I'm actually proud of isn't the AI.

I don't change anything on the real battery on a hunch.

The code has an automated test suite that runs on every change, so a careless edit gets caught on my screen — not on the hardware powering my house. And before any new setting goes live, I replay it against months of past prices to see what it would have done. That's how I answered "is upgrading my power connection worth it?" — I ran it both ways, the numbers said yes, so I upgraded. The numbers decided, not me.

One result humbled me: roughly a third of the savings I could make is lost simply because no forecast is ever perfect. Which tells me exactly what to improve next — the predictions, nothing else.

The lesson I keep relearning: the model is the easy part. What makes AI trustworthy enough to act on its own is the boring engineering around it — tests, version control, and the discipline to prove a change before you ship it.

The AI makes the decision. The engineering earns the right to let it.

#AI #MachineLearning #SoftwareEngineering #Automation #CleanEnergy
