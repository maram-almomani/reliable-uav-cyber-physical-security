# Model Card

## Model

Stage-1 UAV Cyber Attack Detector

Model family:

**XGBoost**

Task:

**Benign vs Attack**

where:

`Attack = DoS or Replay`

## Input

Fixed windows of 20 cyber observations.

Final source families:

- `time_since_last_packet`
- `wlan.fc.type`

Aggregated features:

**14**

## Probability Output

The detector produces:

`P(Attack)`

Temperature scaling:

**T = 1.71111364**

## Frozen Operating Point

Attack if:

`P(Attack) >= 0.5`

Locked-Test performance:

| Metric | Value |
|---|---:|
| Accuracy | 0.6958 |
| Balanced Accuracy | 0.7915 |
| Attack Precision | 1.0000 |
| Attack Recall | 0.5830 |
| Attack F1 | 0.7366 |
| ROC-AUC | 0.9369 |
| PR-AUC | 0.9777 |

## Risk-Aware Routing

Uncertainty:

`1 - max(P(Benign), P(Attack))`

Frozen threshold:

**0.114250**

Equivalent confidence:

**0.885750**

Actions:

| Condition | Action |
|---|---|
| High-confidence Benign | Continue monitoring |
| High-confidence Attack | Generic protective response |
| Low confidence | Verification required |

Locked-Test automated accuracy:

**92.82%**

Locked-Test error capture:

**87.04%**

## Intended Use

Research on:

- UAV cybersecurity
- temporal distribution shift
- intrusion-detection reliability
- probability calibration
- uncertainty-aware routing

## Out-of-Scope Use

Do not treat this model as:

- a certified flight-safety system,
- a universal UAV IDS,
- an autonomous attack-specific controller,
- or a reliable DoS-vs-Replay attribution engine.

## Major Risk

The Locked Test showed substantial Attack Recall degradation.

A fixed 0.5 decision threshold can miss attacks under temporal shift.

## Evaluation Integrity

The Locked Test was evaluated once after freezing:

- selected features,
- model,
- temperature,
- routing threshold,
- response policy.

No post-test tuning is permitted.
