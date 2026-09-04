# Smart Traffic RL Dashboard

> **Building AI course project** — Smart Traffic RL Dashboard is a real-time web interface for observing how a reinforcement-learning agent controls urban intersections. It translates simulated traffic telemetry into live signal states, actions, rewards, speed indicators, and vehicle events that people can inspect.

## Background

Urban traffic congestion wastes time, fuel, and public money every day. Many intersections still use fixed-time signal plans: they repeat the same schedule even when one road is empty and another has a growing queue. This can increase delays, idling emissions, and driver frustration during rush hours, incidents, or special events.

I am interested in combining reinforcement learning (RL) with practical web systems. RL can learn traffic-signal decisions from feedback, but its behaviour is difficult to understand from raw simulation output alone. This project makes the agent's observations and decisions visible in a human-readable dashboard, helping developers and traffic specialists evaluate whether the control strategy is behaving sensibly.

## How is it used?

The dashboard is intended for a traffic-control centre, a municipal simulation lab, or a research environment before any real-world rollout. A Flask server receives traffic telemetry and RL decisions, then serves them to an HTML, CSS, and JavaScript interface.

Traffic engineers can use it to monitor congestion and signal changes; AI researchers and developers can use it to inspect rewards, tune the agent, and identify undesirable policies. The interface presents:

- Live vehicle-speed information and a scrolling vehicle-event log.
- Colour-coded intersection states: green, orange, and red.
- The RL action selected for the current control step.
- Reward and cumulative-reward metrics for assessing the policy.

In a deployment, an operator would keep the dashboard open while a simulator or approved traffic-management system publishes updates. The dashboard is an observability and decision-support tool; it does not replace qualified human oversight.

## Data sources and AI methods

### Data

The initial version uses simulated traffic telemetry, for example from a traffic simulator or a custom generator. Each update can include vehicle counts, lane speeds, queue lengths, waiting times, signal phase, and vehicle arrival/departure events. Simulation data makes it possible to test rare or unsafe scenarios repeatedly, although it may not fully match real streets.

Future versions could ingest anonymized, aggregated data from induction loops, connected infrastructure, or computer-vision systems. Such sources require calibration, missing-data handling, privacy safeguards, and validation against reliable ground truth before they influence control decisions.

### AI method

The control component is a reinforcement-learning agent. At each decision step it receives a **state** containing traffic features such as queue lengths, vehicle counts, average speeds, and the active signal phase. It chooses an **action** from an action space such as keeping the current phase, extending green, or switching to the next safe phase.

The agent receives a **reward** designed to encourage efficient and fair movement, for example by reducing total waiting time and queues while penalising unsafe or overly frequent signal changes. Training can use a tabular method for a small prototype or a deep-RL method such as DQN or PPO for richer state spaces. The dashboard displays the selected action and reward rather than hiding the policy behind the server.

An illustrative Flask route connecting the web layer to an RL controller could look like this:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/api/traffic-state")
def traffic_state():
    telemetry = simulator.current_telemetry()
    state = rl_agent.encode_state(telemetry)
    action = rl_agent.select_action(state)
    reward = rl_agent.latest_reward()

    return jsonify({
        "speeds": telemetry["speeds"],
        "signal_state": telemetry["signal_state"],
        "action": action,
        "reward": reward,
        "cumulative_reward": rl_agent.cumulative_reward,
        "events": telemetry["vehicle_events"],
    })
```

The frontend can poll this endpoint or receive equivalent updates through WebSockets, then update its colour indicators, metrics, and event table.

## Challenges

This project does not guarantee that an RL policy is safe for real traffic. A reward function can produce unexpected shortcuts, and performance in a simulator may not transfer to real roads because of imperfect models, sensor noise, weather, driver behaviour, incidents, or network latency. Edge cases such as emergency vehicles, pedestrians, cyclists, equipment failures, and conflicting signals require deterministic safety rules and human-approved fallback plans.

The dashboard also cannot solve city-wide congestion by itself: road design, public transport, demand patterns, and policy decisions matter. Any real telemetry or camera integration must minimise personal data collection, protect security-sensitive infrastructure, test for unequal effects across neighbourhoods and modes of transport, and keep a trained human operator accountable for final decisions.

## What next?

The next step is to connect the dashboard to a reproducible traffic simulator and train/evaluate the agent against fixed-time and adaptive baselines. I would add scenario playback, reward breakdowns, alerts for abnormal behaviour, and a clear audit trail of decisions.

Longer term, the project could coordinate multiple intersections, use safe multi-agent RL, and integrate carefully validated camera or sensor feeds. Moving toward deployment would require collaboration with traffic engineers, municipal authorities, safety specialists, ML researchers, and privacy/security experts, along with extensive field testing and regulatory approval.

## Acknowledgments

- Inspired by the **Building AI** course from Reaktor Innovations and the University of Helsinki.
- Reinforcement-learning concepts are informed by the wider open-source Python and machine-learning communities.
- Replace this section with the licences and citations for any simulator, datasets, icons, images, or third-party code used in the final implementation (for example, Creative Commons or MIT-licensed assets).

