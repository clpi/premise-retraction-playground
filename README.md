# Premise Retraction Playground

A teaching miniature of one idea: **learning is premise-tracked**. Every training
example is a named premise that contributes one exact, removable term to the model.
Retract a premise and the weights become *exactly* the weights of a model trained
only on the remaining premises — by construction, verified automatically, not
approximately.

Single self-contained `index.html`. No build step, no dependencies, no network
calls, no storage.

## The model

Two inputs `x₁, x₂`, one output `y`, three weights: bias `b` and slopes `w₁, w₂`.

    ŷ = b + w₁x₁ + w₂x₂
    L(θ) = Σᵢ cᵢ (ŷ(xᵢ) − yᵢ)²        (least squares, cᵢ = premise weight)

Zeroing the gradient gives the normal equations `A θ = q` with

    A = Σᵢ cᵢ x̃ᵢx̃ᵢᵀ ,  q = Σᵢ cᵢ yᵢ x̃ᵢ ,  x̃ᵢ = (1, x₁ᵢ, x₂ᵢ)

So the weights are a function of a **sum of per-premise terms**, and nothing else.
The dimension is a constant (`NF = 2` in `core.js`); the solve, the statistics, the
check and the receipt are all generic in it.

## The retraction mechanism

Each premise stores exactly its own 3×3 block `Aᵢ` and 3-vector `qᵢ` (open a ledger
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
order-dependent gradient-descent learner over `(b, w₁, w₂)`, ordinary doubles, one
pass, η = 0.1. It exists to show why a naive "subtract the recorded update" undo does
*not* reproduce a model trained without that example. It is not the app's model.

Scope is three weights on purpose. The claim being demonstrated is the accounting,
not the capacity: with `n` inputs the same code path solves an `(n+1)`-dimensional
system and every per-premise block stays additive. With a sequential optimizer the
same guarantee requires freezing each premise's contribution rather than recomputing
it against moving weights.

## Using it

Open `index.html`. Three ways to see the point:

1. **Run the demo** — adds five premises in two input dimensions, retracts the middle
   one, checks exactness.
2. **By hand** — add premises, press *Retract* on any row; the check runs automatically.
3. **Retract each** — retracts every active premise in turn and checks every case.

Inputs accept decimals and fractions (`0.1` becomes 1/10, `1/3` stays 1/3); all are
read exactly. The chart draws the fitted plane as iso-lines of constant `ŷ` over the
`(x₁, x₂)` plane, with the pre-retraction plane dashed, and each premise labelled with
its residual. *Download receipt* exports premises, per-premise contributions, all
three weights and check results as exact fractions.

Demo data: (1, 0.5)→3, (2, 1.5)→5, (3, 2)→4.5, (1.5, 2.5)→4, (3.5, 0.8)→6. Full fit
`b = 23263/9398, w₁ = 4371/4699, w₂ = −70/4699`; after retracting the middle premise
`b = 3829/2129, w₁ = 2518/2129, w₂ = 540/2129`, exactly equal to the fresh fit, 5/5
checks.
