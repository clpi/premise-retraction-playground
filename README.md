# Premise Retraction Playground

A teaching miniature of one idea: **learning is premise-tracked**. Every training
example is a named premise that contributes one exact, removable term to the model.
Retract a premise and the weights become *exactly* the weights of a model trained
only on the remaining premises — by construction, verified automatically, not
approximately.

Single self-contained `index.html`. No build step, no dependencies, no network
calls, no storage.

## The model

One input `x`, one output `y`, two weights: bias `b` and slope `w`.

    ŷ(x) = b + w·x
    L(θ) = Σᵢ cᵢ (ŷ(xᵢ) − yᵢ)²        (least squares, cᵢ = premise weight)

Zeroing the gradient gives the normal equations `A θ = q` with

    A = Σᵢ cᵢ x̃ᵢx̃ᵢᵀ ,  q = Σᵢ cᵢ yᵢ x̃ᵢ ,  x̃ᵢ = (1, xᵢ)

So the weights are a function of a **sum of per-premise terms**, and nothing else.

## The retraction mechanism

Each premise stores exactly its own 2×2 block `Aᵢ` and 2-vector `qᵢ` (open a ledger
row to see them). The app keeps running totals:

- **add**: `A += Aᵢ`, `q += qᵢ`
- **retract**: `A −= Aᵢ`, `q −= qᵢ`, then re-solve `A θ = q`
- **restore**: add the same block back

There is no replay, no re-shuffle, no retraining job. The retracted premise's block
is the only thing that changes.

## Why the check cannot fail

All arithmetic is exact rationals (BigInt numerator/denominator), so no subtraction
leaves a residue. The check compares two genuinely different paths:

1. the **incrementally maintained totals** with one premise's block subtracted, versus
2. totals **rebuilt by summing only the remaining premises** from zero.

It then compares `b`, `w` and every prediction as exact rationals, confirms the
removed block re-adds to the previous totals, and confirms every surviving
contribution is byte-identical. Five checks, all exact. Floats appear only as pixel
coordinates on the chart.

Sanity property: tampering with the running totals by 1e-6 flips the banner to FAIL,
which is how you know the check is live rather than decorative.

## What is real and what is illustrative

Everything in the panels is computed live in your browser: real rational arithmetic,
a real closed-form solve, a real check. Nothing is replayed from a recording. The
demo premises (readings A, B, C) are arbitrary teaching data, not measurements.

The collapsible **float contrast** panel is explicitly illustrative: an
order-dependent gradient-descent learner, ordinary doubles, one pass, η = 0.1. It
exists to show why a naive "subtract the recorded update" undo does *not* reproduce
a model trained without that example. It is not the app's model.

Scope is two weights on purpose. The claim being demonstrated is the accounting, not
the capacity: with a sequential optimizer the same guarantee requires freezing each
premise's contribution rather than recomputing it against moving weights.

## Using it

Open `index.html`. Three ways to see the point:

1. **Run the demo** — adds three premises, retracts the middle one, checks exactness.
2. **By hand** — add premises, press *Retract* on any row; the check runs automatically.
3. **Retract each** — retracts every active premise in turn and checks every case.

Inputs accept decimals and fractions (`0.1` becomes 1/10, `1/3` stays 1/3); all are
read exactly. *Download receipt* exports premises, per-premise contributions,
weights and check results as exact fractions.
