# Part 1 — The ACAS Xu Benchmark and the Vehicle Language

This part is based on the *Getting Started with the Vehicle Specification Language*
chapter of the Vehicle tutorial:
<https://vehicle-lang.github.io/tutorial/chapters/language/>

The goal is to understand a realistic neural-network verification benchmark
(ACAS Xu), to see how a *specification* is written in Vehicle, and then to
practise writing and verifying specifications yourself.

---

## 1. The ACAS Xu benchmark

**ACAS Xu** ("Airborne Collision Avoidance System for unmanned aircraft, X
variant") is a collision-avoidance system that advises an autonomous aircraft
(the *ownship*) how to manoeuvre to avoid an approaching aircraft (the
*intruder*). It is one of the most widely used case studies in neural-network
verification, originally introduced as a benchmark by Katz et al. in the
*Reluplex* paper.

### What the system does

ACAS Xu reads the current geometry between the ownship and the intruder and
issues one of five **advisories** telling the pilot/autopilot how to steer.

### Inputs (5 measurements)

| Index | Input | Meaning | Unit |
|-------|-------|---------|------|
| 0 | `distanceToIntruder` | distance to the intruder | feet |
| 1 | `angleToIntruder`    | angle to the intruder (relative to ownship heading) | radians |
| 2 | `intruderHeading`    | heading direction of the intruder | radians |
| 3 | `speed`              | speed of the ownship | feet/second |
| 4 | `intruderSpeed`      | speed of the intruder | feet/second |

(The original system also uses *time to loss of vertical separation* and the
*previous advisory* to select which network to use — see below.)

### Outputs (5 scores → 1 advisory)

The network outputs a **score** for each of five possible advisories. The
advisory with the **lowest** (minimal) score is the one that is issued.

| Index | Advisory | Meaning |
|-------|----------|---------|
| 0 | `clearOfConflict` | no evasive action needed |
| 1 | `weakLeft`        | turn left, small rate |
| 2 | `weakRight`       | turn right, small rate |
| 3 | `strongLeft`      | turn left, large rate |
| 4 | `strongRight`     | turn right, large rate |

### Why neural networks — and why verification?

The original ACAS Xu policy is a huge lookup table. To make it deployable, it
was compressed into **45 small neural networks**, each trained on a distinct
sub-region of the input space (indexed by the previous advisory and the
time-to-vertical-separation). Each network takes the 5 normalised inputs and
produces the 5 scores.

Because these networks fly safety-critical hardware, we want **formal
guarantees** that they behave sensibly — not just that they score well on test
data. This is exactly what Vehicle lets us express and check.

---

## 2. Writing the specification in Vehicle

The tutorial focuses on **Property 3** from the Reluplex paper:

> *If the intruder is directly ahead and is moving towards the ownship, the
> score for "Clear of Conflict" will not be minimal.*

(In other words: the system must **not** advise "do nothing" when a collision is
imminent.)

### Key language ideas introduced

- **Tensor types with fixed dimensions** — the input and output are vectors of 5 reals:
  ```vehicle
  type Input  = Tensor Real [5]
  type Output = Tensor Real [5]
  ```
- **Network declarations** — a network is an *uninterpreted function* annotated with `@network`:
  ```vehicle
  @network
  acasXu : Input -> Output
  ```
- **Named indices** — instead of magic numbers, we name the input/output positions
  (`distanceToIntruder = 0`, `clearOfConflict = 0`, …) so the spec reads like English.
- **The problem space vs. the input space.** The network expects *normalised*
  inputs, but humans reason in real-world units (feet, radians). Vehicle lets us
  write the property over *unnormalised* (problem-space) values and bridge the gap
  with a `normalise` function:
  ```vehicle
  normalise : UnnormalisedInput -> Input
  normalise x = foreach i .
    (x ! i - meanScalingValues ! i)
      / (maximumInputValues ! i - minimumInputValues ! i)

  normAcasXu : UnnormalisedInput -> Output
  normAcasXu x = acasXu (normalise x)
  ```
- **Vector operations** — indexing with `!` and constructing vectors with `foreach`.
- **Quantifiers and logic** — `forall`, `exists`, `and`, `or`, `=>`, `not` are used to
  state the property over the (infinite) set of valid inputs:
  ```vehicle
  @property
  property3 : Bool
  property3 = forall x .
    validInput x and directlyAhead x and movingTowards x =>
    not (minimalScore clearOfConflict x)
  ```

The takeaway: a Vehicle specification is a **readable, machine-checkable
mathematical statement** about the network's behaviour, written in problem-space
terms.

---

## 3. Exercises

Work through these from the tutorial chapter. Difficulty is marked with stars.

### Exercise 1 — ⭑ (Easy): Verify Property 3
Download the ACAS Xu Property 3 specification and trained network from the
tutorial repository and run Vehicle's verification command to confirm the
property holds.

### Exercise 2 — ⭑⭑ (Moderate): Specify Property 1
Write **Property 1** in Vehicle yourself:

> *If the intruder is distant and is significantly slower than the ownship, the
> score of a "Clear of Conflict" advisory will always be below a certain fixed
> threshold.*

Note this property talks about an **absolute output score** (not just which score
is minimal), so you will have to handle the embedding/normalisation of the output
space too.

### Exercise 3 — ⭑⭑ (Moderate): All ten properties
State **all ten** ACAS Xu properties from the original Reluplex paper in a single
`.vcl` file and run verification on them.

### Exercise 4 — ⭑⭑⭑ (Hard): The Iris dataset
Using the Iris flower dataset and a trained classifier:
1. examine the dataset,
2. decide on some "obvious" properties that *should* hold,
3. express them as Vehicle specifications,
4. type-check them, and
5. verify them with the `vehicle` command.

---

## References

- Vehicle tutorial — Getting Started: <https://vehicle-lang.github.io/tutorial/chapters/language/>
- G. Katz et al., *Reluplex: An Efficient SMT Solver for Verifying Deep Neural
  Networks* (CAV 2017) — origin of the ACAS Xu properties.


