# Limitations

## Single Dataset

The study uses one UAV cyber-physical dataset.

The findings should not automatically be generalized to other UAV
platforms, environments, radios, or attack implementations.

## Heterogeneous Schemas

The raw dataset contains multiple incompatible cyber and physical
schemas.

A fair global five-class experiment was therefore not possible without
substantial schema-leakage risk.

## Temporal Generalization Gap

Stage-1 Attack Recall changed from:

**95.61% on validation**

to:

**58.30% on the Locked Test**

This is the most important limitation of the final detector.

The frozen 0.5 operating point did not generalize reliably across
later temporal segments.

## Limited Independent Temporal Groups

The number of independent temporal segments is substantially smaller
than the number of windows.

Window-level sample counts should therefore not be interpreted as fully
independent experimental repetitions.

## Physical Confounding

The perfect physical validation result depended strongly on absolute
barometer level.

Removing the barometer caused performance to collapse.

This prevents interpretation of the physical model as a general UAV
attack detector.

## No Causal Barometer Claim

The analysis identifies a strong session/scenario confound.

It does not prove that barometer behavior can never be affected by
cyberattacks.

Independent controlled flight experiments would be required.

## Weak Attack Attribution

DoS-vs-Replay validation Macro F1:

**0.5262**

ROC-AUC:

**0.5537**

This is not sufficient for reliable autonomous attack-specific action.

## Confidence Is Not Full Epistemic Uncertainty

The routing policy uses calibrated predictive confidence.

It is not a complete Bayesian or epistemic uncertainty estimate.

## Review Burden

On the Locked Test, the frozen policy routed:

**45.07%**

of windows for additional verification.

This may be operationally expensive.

## No Human Study

The verification workflow is simulated.

No analyst workload, response-time, or human accuracy experiment was
performed.

## No Live UAV Deployment

No autonomous intervention was executed on a real UAV.

The protective-response state is conceptual.

## No Post-Test Tuning

The Locked Test has already been consumed.

Any future methodological improvement must use a new external dataset
or newly collected test set.
