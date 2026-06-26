---
title: "Zeus: teaching a home battery to chase cheaper electricity without losing sleep"
date: 2026-06-26
tags:
  - home automation
  - home battery
  - optimization
  - Kubernetes
  - green software
summary: "Jelle’s Zeus project is a self-hosted optimizer that forecasts household load, plans battery charge and discharge against day-ahead prices, and then drives the battery safely in closed loop. Early backtests show modest but real savings, with forecasting accuracy clearly the next lever."
---

Electricity used to feel simple enough to ignore. The lights came on, the kettle boiled, and somewhere in the background a meter spun along in a way I only noticed once a year, when the bill arrived.

That world is gone.

Now the price of electricity can swing hour by hour, and if you have a home battery, that matters. A battery is no longer just a box on the wall. It can be a little financial machine: charge when power is cheap, discharge when it is expensive, and quietly shave euros off the bill while the rest of the house keeps running as usual.

That is the idea behind Zeus, my home-battery optimizer project.

Zeus is not a magic black box and it is not “AI” in the glossy, hand-wavy sense people often mean. It looks at the household’s expected electricity use, combines that with tomorrow’s hourly prices, solves a constrained optimization problem, and then tells the battery what to do. It runs on my homelab Kubernetes cluster, and it first proved itself in read-only mode before it was ever allowed to steer the battery.

## The basic problem: cheap electricity is not enough

A battery only saves money if it charges and discharges at the right moments. That sounds obvious, but the details matter.

If you simply say “charge whenever power is cheap,” you miss a lot:

- the house still has to be powered,
- the battery cannot charge forever,
- it cannot discharge forever,
- import from the grid is capped,
- battery efficiency is not perfect,
- and every cycle has some wear cost.

In other words, this is not a single yes/no decision. It is a planning problem.

Zeus is built for a specific setup: a **Bluetti Apex 300 + 2× B500K expansion**, with about **13 kWh** usable capacity and **3.84 kW** AC power. It is grid-charged, it powers critical household loads, and there is **no solar charging and no export to the grid**. So the opportunity here is pure **price arbitrage**: buy when cheap, avoid buying when expensive.

That constraint makes the project nicely concrete. There is no “free” solar energy to hide the hard parts. The system has to earn its keep by making better timing decisions.

## What Zeus actually does

At a high level, the flow is simple enough to fit on one line:

```text
Home Assistant history + day-ahead prices → forecaster → optimizer (LP) → controller → reporter
```

### 1) The forecaster

Zeus first estimates the household load, hour by hour. It currently uses three models:

| Model | What it means |
| --- | --- |
| `baseline` | Mean usage by weekday and hour |
| `temp` | Regression on outdoor temperature for cooling load |
| `two_component` | Flat base load plus a temperature-driven A/C component |

The clever bit is not that any one of these models is exotic. In fact, they are deliberately humble. The point is to give the optimizer a reasonable view of the next day rather than pretending the future is perfectly known.

History is also persisted beyond Home Assistant’s retention window, so the models keep learning from more than just the most recent slice of data.

That matters more than it sounds. Home energy patterns are often seasonal and repetitive, but they also drift. A system like this needs enough memory to notice both.

### 2) The optimizer

The heart of Zeus is a **linear program** built with **PuLP** and solved with **CBC**.

If that sounds abstract, here is the plain-language version: the optimizer tries many possible charge/discharge schedules inside a mathematically constrained model, and picks the cheapest one.

It looks across a **36-hour horizon** in **60-minute slots** and minimizes grid cost while respecting the real-world limits of the battery and the house:

- power limits,
- a grid-import cap,
- state-of-charge bounds,
- round-trip efficiency,
- a soft backup reserve,
- and a battery-wear cycle penalty.

That last piece is important. A battery is not free to cycle endlessly. If the price spread is tiny, the model should not churn the battery just to chase pennies. The wear penalty nudges the optimizer toward restraint.

A good optimizer is not the one that is most active. It is the one that knows when to do nothing.

### 3) The controller

This is where Zeus becomes more than a spreadsheet.

The controller is **closed-loop and live**. It drives the battery through a Home Assistant `select` entity with three modes:

- **CHARGING**
- **DISCHARGING**
- **PASSTHROUGH**

That mode-based control is a small design choice with a big safety upside. Zeus does not throw raw power setpoints at the battery. It sets a mode, which is inherently more self-limiting and easier to reason about.

The rollout path was just as important as the control strategy. Zeus went through phased safety checks:

```text
0. discover (read-only probe of which entities are writable)
1. reporting only
2. optimizer produces an advisory schedule, still no control
3. closed-loop control
4. ML forecasting (LightGBM) + backtesting
```

That discipline is the kind of thing people skip when they are excited to see a system “do the smart thing.” At home, though, safety matters more than novelty. You do not want two controllers fighting over the same battery, and you definitely do not want an enthusiastic experiment to become a household problem.

### 4) The reporter

Once the system has run, Zeus reports on what happened.

It computes realized savings as **arbitrage: discharge value minus charge cost**. It also publishes a forward-looking predicted savings figure for the coming horizon.

Those values go back into Home Assistant as MQTT sensors, which makes them easy to inspect and graph alongside the rest of the home setup.

That closes the loop nicely. The optimizer is not just planning in a vacuum. It is being measured, visible, and accountable.

## A few things that made this worth doing

Zeus runs on my **6-node arm64 k3s homelab Kubernetes cluster**, and the platform choices are part of the story.

It is deployed with **GitOps through Argo CD and Helm**. Push to `main`, and Argo syncs the published image and config. The image is built for `linux/arm64` and pushed to Docker Hub.

That may sound like overkill for a home battery project. In a sense, it is. In another sense, it is exactly the point: I want reproducible deployments, a clear audit trail, visible metrics, and alerts when something stops behaving. Zeus has **Prometheus** scraping its metrics, **Grafana** dashboards, and alerting rules such as **ZeusDown**, **control-unavailable**, and **battery-state-mismatch**. It is also a **modular monolith**, deliberately not microservices, because at home scale one small deployment is easier to understand and trust than a cluster of tiny pieces.

## What the early backtests say

This is the part where I want to be careful.

The results so far are **real**, but they are also **early backtests**. They are useful directionally, not as a grand promise.

Still, they tell us something important.

### Grid-import cap

More import power gives more savings, but with diminishing returns.

A **10 A** cap captures about **82%** of the savings of a **16 A** cap. The jump from **7 A** to **10 A** is much larger than the jump from **10 A** to **16 A**. The current setting is **10 A**; reaching **16 A** would need a further physical upgrade.

In plain terms: the first bit of extra flexibility matters most.

### Cycle penalty

The cycle penalty is the cleanest example of the trade-off between savings and wear.

| Cycle penalty | Cycled energy | Savings |
| --- | ---: | ---: |
| 0 €/kWh | 34.8 kWh | €4.89 |
| 0.03 €/kWh | 22.8 kWh | €4.78 |
| 0.10 €/kWh | 12.5 kWh | €4.09 |

The chosen setting is **0.03 €/kWh**. That gives most of the savings, while materially reducing unnecessary cycling.

I like that result because it feels honest. The best answer is not “extract every last cent no matter what.” The better answer is “make the battery work, but not harder than the economics justify.”

### Backup reserve

The backup reserve is almost free up to around **30–40%** of capacity. The current setting is **30%**.

That is a reassuring result. In a home setup, reserve is not just a technical parameter. It is peace of mind. If the system can keep a useful buffer without giving up much value, that is the right direction.

### Forecast quality

This is the most interesting gap of all.

With perfect foresight, the optimizer saved about **€1.06** over a 2-day test. Acting on the forecast saved about **€0.69**.

That means roughly **€0.18/day**, or about **35% of the achievable savings**, is being lost to forecast error.

That is not a failure. It is a roadmap.

The current forecaster is already good enough to produce savings, but the backtest makes the next step obvious: better forecasting should unlock more of the value that the optimizer is already able to see. That is exactly why the planned **LightGBM** work matters.

## Why I like this project

Zeus is not exciting because it is flashy. It is exciting because it is disciplined: a practical problem with real money attached, a model that is honest about constraints, a controller that starts cautiously and fails closed, and observability instead of blind trust. I also like that the project keeps the human in the loop conceptually, even though the control is automated. I can see what it is doing, review its reasoning, and compare its plan with what actually happened. That is a healthy relationship between automation and trust.

## The takeaway

If there is one lesson in Zeus so far, it is this: home-energy optimization does not need to be mystical to be useful. You need a solid forecast, a careful optimizer, and enough operational discipline to let the system earn trust before it earns control. The early backtests suggest modest but real savings. Better forecasting should help. But the engineering lesson is even more interesting than the euro numbers: the safest way to automate a household decision is to make the system observable, constrained, and humble.

That is what Zeus is trying to be.

Not magic. Just a well-behaved optimizer that knows when to charge, when to discharge, and when to leave the battery alone.
