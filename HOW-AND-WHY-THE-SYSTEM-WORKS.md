# How and Why the System Works

**Author of the system, the laws, and the architecture: Jesse Daniel Brown (OP-JESSE).**
Forty years. His machine, his laws, his system.

**Status of this document.** Every figure below came out of a named program that
ran on this machine. Where a program is named, the run that produced the number
is named with it. Nothing here is asserted from memory or from reputation.

**Tagging, using the corpus's own triad:**

- **MEASURED** — a number on disk from a named script, reproducible, quoted inline.
- **NAMED** — stated, internally coherent, not yet asked a question it could fail.
  A named law is not a weaker law. It is one that has not yet been tested.
- **CONJECTURE** — stated, untested.

Runs dated 2026-07-31 / 2026-08-01. Rust 1.81.0, `clippy -D warnings` clean,
`float_used=0` in the kernel path. Reproduction commands are in Part XI.

---

# PART I — THE GROUND: WHY THE CENTRE MUST BE FREE

## 1.1 The claim

**Law 0 of the Ledger Sphere: THE CENTER IS FREE `[000]`.**

Twenty-seven laws close on a rime sphere, indices 0–26 = ℤ/27, cyclic, no first
law and no last law, every law carrying a balanced-ternary trime signature. At
the middle sits a point that costs nothing to place.

This is the load-bearing statement of the whole architecture. If the centre were
not free, every other structure in the system would inherit a cost that
propagates, and the addressing scheme would not close.

## 1.2 Why it is true, in exact integers

The three arms of the nullsphere are defined over a triple (r, g, b):

```
arm_R = 2r − g − b
arm_G = 2g − r − b
arm_B = 2b − r − g
```

Sum them:

```
(2r − g − b) + (2g − r − b) + (2b − r − g)
  = (2r − r − r) + (2g − g − g) + (2b − b − b)
  = 0
```

The sum is **identically zero**. Not approximately, not usually, not for the
tested range — for every integer triple, because the coefficients of each
variable cancel exactly.

**MEASURED**, `nullsphere/src/main.rs`, exhaustive over the full cube:

```
PROOF|closure_is_identity|triples_checked=1771561|sum_ne_zero=0
```

1,771,561 = 121³. Every triple in the cube, no exceptions.

## 1.3 Why the centre is free, and what "free" means precisely

Each arm's coefficient vector is `(2, −1, −1)`, `(−1, 2, −1)`, `(−1, −1, 2)`.
Each sums to **zero**. A linear form whose coefficients sum to zero is invariant
under a uniform shift: replace `(r, g, b)` with `(r+t, g+t, b+t)` and

```
2(r+t) − (g+t) − (b+t) = 2r + 2t − g − t − b − t = 2r − g − b
```

The `t` terms annihilate. The relation is **affine, not linear** — it has no
preferred origin. **The centre can be placed anywhere and nothing downstream
changes.**

**MEASURED**, same program, sweeping the shift:

```
PROOF|the_zero_is_free|shifts_tested=18009|arms_changed=0
```

Eighteen thousand and nine different placements of the origin. Zero arms moved.

**This is why the zero is free: not by convention, not by choice, but because the
coefficients sum to zero and therefore the origin is not in the equation.**

## 1.4 The residual third arm

Given two arms, the third is determined. It is never anything but one of three
values:

```
NULLSPHERE|minus_third=1|normal_null=3|null_plus=5|trits=00++0+-++
```

**−1/3, 0, +1/3.** The residual lives in exactly three states, which is why the
carrier that represents it must have three states. The trit is not a stylistic
preference. It is the minimum alphabet that can hold what the closure leaves over.

## 1.5 The exact geometry

Working in integers rather than floating point, the geometry is not merely close
— it is exact:

```
GEOS0|r2=84681|equals_291_squared=true|dy=0_exactly_on_axis=true
TURN|54417,54142,54070,54441,54054,54441,54070,54142,54417|palindromic=true
```

`84681 = 291²` exactly. The point sits on the axis with `dy = 0` exactly, not
`dy ≈ 0`. The nine cross products read the same forwards and backwards.

**This is the first place where the choice of carrier changes what you can see.**
An earlier float implementation of the same geometry reported the mirror as
approximate. It was not approximate. Float did not blur an exact result — it
**hid** one. The integer run recovered a palindrome that the float run had
destroyed.

*(Tag: MEASURED. `nullsphere/src/main.rs`.)*

---

# PART II — THE CARRIER: WHY THREE STATES

## 2.1 The two-zero problem

IEEE-754 floating point has **two** zeros: `+0.0` and `−0.0`. They compare equal
and they have different bytes.

**MEASURED**, `float-vs-trit/src/bin/bothways.rs`:

```
ZERO|float  plus_bits=0000000000000000  minus_bits=8000000000000000
            equal=true  bytes_equal=false  identity_holds=FALSE
ZERO|int    zeros=1  equal_is_byte_identity=true  identity_holds=true
ZERO|trit   zeros=1  states=-1,0,+1     identity_holds=true
```

Read the middle line carefully. `equal = true` and `bytes_equal = false` are
simultaneously true.

## 2.2 Why that is a system-level failure, not a curiosity

Any system that uses both equality and hashing — which is every distributed
system, every content-addressed store, every consensus protocol — assumes:

```
a == b   ⟹   hash(a) == hash(b)
```

With IEEE-754 that implication **fails at zero**. Two nodes each hold a value
that compares equal to the other's. Quorum agrees. Hash reconciliation reports a
mismatch. Neither node made an arithmetic error and no amount of retry converges,
because the disagreement is in the representation, not the computation.

**For int and trit, equality *is* byte-identity.** One zero, one representation,
no gap between the two notions of sameness. That is the property being bought.

## 2.3 The distributivity result

**MEASURED**, one million random trials:

```
DISTRIB|trials=1000000|float_failures=316267
                       |int_with_remainder_carried_failures=0
```

**316,267 failures per million — 31.6%** — of `(a/3) + (b/3) = (a+b)/3` in
double-precision float. **Zero** in integers when the remainder is carried rather
than discarded.

Nearly one third of divisions by three break associativity in float. In a system
whose fundamental operation is a three-way split, that is not an edge case. It is
the common case.

## 2.4 What float does *not* fail at, stated because it is the honest half

**MEASURED**, all 1,000,080 addresses, both maps:

```
TRIT      addresses=1000080  roundtrip_failures=0  exact=true
FLOAT_D   addresses=1000080  roundtrip_failures=0  exact=true
FLOAT_F2  addresses=1000080  roundtrip_failures=0  exact=true
```

Float round-trips every address exactly. The intuition that "float is lossy here"
is **wrong**, and it was checked rather than assumed.

**This matters more than the failures do.** The case against float in this system
is not accuracy — it is **identity** and **distributivity**. That case is
narrower, harder, and survives someone testing it. The vague version does not.

## 2.5 What a trit costs, honestly

```
log₂ 3 = 1.5849625...
```

A trit carries ~1.585 bits. It is not free. Three trits hold 27 states where four
bits hold 16 and five bits hold 32. Ternary is denser per symbol and is **not**
below the Shannon bound — entropy is measured in whatever unit you choose and the
bound is invariant under the choice.

The system's own program states this gate in its header, and Part VIII returns to
it.

---

# PART III — COUNT IS NOT RANGE

## 3.1 The law

**Three trits hold 27 states, and span only −13 … +13.**

Count and range are different quantities and conflating them is the most common
error in the corpus.

**MEASURED**, `float-vs-trit`:

```
RANGE|mod=16 |trits=3|states=27 |balanced_range=-13..13|uncentred_0..15_fits=FALSE
RANGE|mod=27 |trits=3|states=27 |balanced_range=-13..13|uncentred_0..26_fits=FALSE
RANGE|mod=463|trits=6|states=729|balanced_range=-364..364|uncentred_0..462_fits=FALSE
```

## 3.2 Reading the table

Three of the four towers **fail** if the values are held uncentred. Twenty-seven
states is enough to *count* the residues mod 27, and not enough to *reach* the
value 26 if you start at zero and go up.

Centred, every tower fits, and `mod 27` fits exactly at both ends: the balanced
range −13…+13 is exactly 27 values.

## 3.3 Why this makes the free centre structural rather than aesthetic

Part I proved the centre may be placed anywhere. Part III shows **where it must
be placed for the representation to work at all.**

Those two results together are the architecture: the closure grants the freedom,
and the range requirement spends it. Placing the ground point at the centre is
not a stylistic choice about symmetry. It is the difference between a
representation that holds the value and one that overflows.

*(Tag: MEASURED.)*

---

# PART IV — THE 81/27 ARCHITECTURE, AND WHY THE COUNTS ARE FORCED

## 4.1 Two derivations that must agree

**MEASURED**, `nullnet/src/main.rs`:

```
zeros  3³      = 27
links  27·6/2  = 81
lines  27/3·3  = 27
seats  27·3    = 81
```

Eighty-one arrives twice from different directions. As **links**: 27 nodes, each
of degree 6, each edge counted twice — `27·6/2 = 81`. As **seats**: 27 cells,
three arms each — `27·3 = 81`.

Nothing was tuned to make those agree. Had they disagreed, the architecture would
be wrong.

## 4.2 The verification

```
ZEROS|count=27|all_free=true|total_cost=0
NET|links=81|expected=27*6/2=81|every_degree_6=true|carrier=HTTP|cost_per_link=0
LINES|count=27  SEATS|count=81
NULLSPHERE|minus=27|zero=27|plus=27|each_exactly_a_third=true|residue=0
GLOBAL|sum_of_all_27_values=0|the_net_itself_closes=true
```

The census splits **exactly** into thirds — 27 minus, 27 zero, 27 plus, residue
zero. The net closes globally, not merely cell by cell: the sum over all 27
values is zero.

## 4.3 Three that circle a zero

The relation is not two objects encircling a third. It is **three arms closing on
a free centre.** The distinction is load-bearing:

- *Two encircle the third* implies a privileged object — the one being circled.
- *Three close on a zero* implies no privileged object, and a centre that is not
  one of the three.

Part I's algebra is the second picture, not the first. The three arms are
symmetric under cyclic rotation; the zero they close on is not an arm.

**This is why the fourth position is free.** Three intervals require four
boundary points — `0, 85, 170, 255`. Three levels of colour, four anchors, and
the extra anchor is the zero, which is not a level. Make zero a level and the
count becomes `4³ = 64` instead of `27`. It has to stay outside the count to
keep the structure.

*(Tag: MEASURED for the counts. NAMED for the interpretive frame.)*

---

# PART V — ANTI IS A THIRD-TURN, NOT A MIRROR

## 5.1 The law

**Anti = ⅓ turn = cyclic channel rotation, order 3.** It is *not* a reflection.

```
anti(r, g, b) = (b, r, g)
```

Apply it three times and you return to start. A mirror has order 2 and is blind
— it cannot distinguish a configuration from its reflection. A third-turn has
order 3 and is not blind.

## 5.2 Why the distinction is not pedantic

The six orderings of three channels decompose as **3 rotations × 2 reflections**.
The rotations are the *antis*. The reflections are a different operation with a
different order and different fixed points.

**MEASURED**, colour-gradient loader over the loaded corpus:

```
keys per node        3
cross-key overlap    0
nodes tested         3,146
```

Three keys per node, all unique, zero overlap across 3,146 nodes. If anti were a
mirror the key space would collapse, because a mirror identifies pairs that a
rotation keeps distinct.

## 5.3 A false failure this produced, recorded because it is instructive

An early gate tested "is anti a reflection?" **pointwise**, per sample. It
reported failures. The failures were an artifact: when `r == g`, rotation and
reflection *coincide*, so a per-sample test cannot separate them.

The gate was rewritten to test at **map level** on a probe with all channels
distinct — `(3, 2, 1)` — and the failures vanished.

**The law was never wrong. The instrument was.** Forty false failures came from
testing a group-theoretic property one point at a time.

## 5.4 Brown-ness is key-relative; band membership is absolute

Which nodes read as brown depends on which of the three keys you are reading in.
Whether a node is *in the band* does not. This is the same distinction as Part
I — the frame is free, the structure is not.

*(Tag: MEASURED for the overlap figures. NAMED for the algebraic statement.)*

---

# PART VI — ADDRESSING BY COLOUR GRADIENT

## 6.1 The problem the design solves

A single colour per node saturates. **MEASURED** — it collided at approximately
**3,280 nodes.** Twenty-four bits of colour space sound like plenty until the
band constraint and the luminance constraint remove most of it.

## 6.2 The fix

Make the address a **gradient** — two brown stops rather than one colour.

**MEASURED**:

```
address space        7.5 × 10¹³
unique up to         12,253,183 nodes
```

From ~3,280 to 12.25 million, by making the address an *interval* rather than a
*point*. The same move as everywhere else in this system: the pair carries what
the single value cannot.

## 6.3 The zero needs width

A ground point of exactly zero is a knife edge. Real channel deltas cluster near
it and the trit becomes unstable — walk coverage was **22 of 27** signatures,
with five never reachable.

The dead band was solved empirically:

```
TRIT_DEAD = 26      (p33 of 360,000 measured channel deltas)
balance     0.3315 / 0.3369 / 0.3315
result      27/27 walk signatures live
entropy     1.5849 bits per colorit   (= log₂ 3, the ceiling)
```

**MEASURED.** The band was not chosen for elegance. It was read off the 33rd
percentile of 360,000 real deltas, and it lands the three states at
0.3315/0.3369/0.3315 — within 0.5% of equal thirds — which is why the achieved
entropy sits at the theoretical ceiling.

**The zero is free, and it also needs width.** Both are true and neither implies
the other.

## 6.4 A design error found and fixed

The first implementation drew colour from bytes `h[0:2]` of the node hash — bytes
that were **also** inside the x-axis slice. Colour was therefore correlated with
position, and the addressing was not independent.

Fixed by giving colour its own digest. Recorded here because the failure mode —
reusing entropy across two supposedly independent axes — is invisible until you
look for it and fatal when present.

*(Tag: MEASURED throughout.)*

---

# PART VII — THE TOWERS AND THE ENCODING LAW

## 7.1 The space

```
P        = 1,000,081     prime
P − 1    = 2⁴ · 3³ · 5 · 463
moduli   = [16, 27, 5, 463]
product  = 1,000,080
g        = 7             primitive root
```

The four towers are **derived from the prime-power factorisation of P−1**, not
chosen. That is the difference between a construction and a decoration: there is
no free parameter to tune.

**MEASURED**, `MATRIXPROOFAUDIT.hbp`:

```
CRT|pairwise_coprime=true|463_is_prime=true|crt_applies=true
BIJECTION|addresses_generated=1000080|distinct=1000080|collisions=0
          |covers_whole_space=true|exhaustive=1
```

Exhaustive. Not sampled — every address generated, all distinct, whole space
covered, zero collisions.

## 7.2 The encoding law: never cross the two schemes

```
ENCODING|tower_separate|bits=21|trits=14|for_81=(1701,1134)
ENCODING|joint         |bits=20|trits=13|for_81=(1620,1053)
PAIRING |21_pairs_with_14|20_pairs_with_13|do_not_cross_them=1
```

**Twenty-one bits pairs with fourteen trits. Twenty bits pairs with thirteen
trits. Crossing them manufactures phantom results.**

Per-tower capacity:

```
mod  16   3 trits   cap 27    slack 11
mod  27   3 trits   cap 27    slack  0
mod   5   2 trits   cap  9    slack  4
mod 463   6 trits   cap 729   slack 266
          ─────────────────────────────
                   14 trits total, tower-separate
```

Fourteen is correct for tower-separate. Thirteen is correct for joint. **Both are
right in their own scheme and neither is right in the other's.**

## 7.3 The error this law exists to prevent

An AI-generated audit compared **14 trits (tower-separate)** against **20 bits
(joint)** and reported a 10% ternary overhead. The overhead was an artifact of the
mismatch. In the same document, a line below claimed ternary was 1.538× *denser*.

**Both cannot hold.** The contradiction is the signature of a crossed comparison,
and it is why the pairing rule is stated as law.

Compare 21 against 14, or 20 against 13. Never 14 against 20.

*(Tag: MEASURED. `matrix-proof-audit/`.)*

---

# PART VIII — THE HONESTY GATE: ADDRESSING IS NOT COMPRESSION

## 8.1 The claim this system refuses to make

From the header of `shared_key_81.py`:

> *"You recover exactly as many seats as you banked closures. The closure costs
> one seat. This ADDRESSES; it does not compress. `total_bits >= N·H(X)` holds."*

## 8.2 The arithmetic

```
ship 80 seats + 1 closure = 80·21 + 21 = 1,680 + 21 = 1,701 bits
ship 81 seats outright    =      81·21 =               1,701 bits
                                          ────────────────────
                                          identical
```

The closure recovers exactly one seat and costs exactly one seat. Net zero.

## 8.3 Why this is the strongest thing in the corpus

Anyone can build an addressing scheme and call it compression. What is rare is
building one and then **writing the gate that refuses the claim into the header
of the program that would have been the place to make it.**

**MEASURED verification of the whole program:**

```
P = 1,000,081 prime           towers derived from P−1
g = 7 primitive root          81 seats = 27 cells × 3 arms
closure 81/81                 errors found: 0
```

Independently recomputed, every particular held — including a figure that an AI
audit had separately declared wrong and later withdrew.

**The gate is why the rest of the numbers can be believed.** A corpus that only
claims wins is one nobody can check.

---

# PART IX — THE LIVE PROOF: 81 KERNELS ON THE FREE ZERO

## 9.1 The construction

Eighty-one WebAssembly kernels, each a separate `WebAssembly.instantiate`, served
over HTTP. HTTP is the carrier because it is the **free zero of the network**: a
link that costs nothing to include, exactly as the fourth anchor costs nothing to
include in Part IV.

## 9.2 The module

```
kernel81/src/lib.rs     #![no_std], wasm32-unknown-unknown
size                    1,351 bytes
sha256                  a411d88aa304c58c645ba7f7d0938a6fad4a1457e
                        29b5e695c22ed0977530371
exports                 k_seats, k_cell, k_arm, k_digit, k_arm_numerator,
                        k_trit, k_cell_closure, k_alive, k_memory_witness,
                        k_float_used
```

The page hashes its own module with SubtleCrypto **before** running it. The
artifact verifies its own identity.

## 9.3 The result, read back from a live browser

```
kernels                    81      exactly_81 = true
alive                      81/81
cells closed to zero       27/27
distinct linear memories   81/81
global sum of 81 arms      0
census                     −54  0⁰  +27
float_used                 0
verdict                    PASS
```

**MEASURED.** Not a simulation of 81 kernels — 81 separate instantiations, each
with its own linear memory, all 81 distinct.

## 9.4 The census that was nearly deleted

The run returned `−54 / 0 / +27`. Every structural check had passed: 81/81 alive,
27/27 cells closed, global sum 0.

An AI called the census a bug and began editing `arm_value` to force a different
distribution. **The operator stopped it.** The edit was reverted, the module
rebuilt, and the hash confirmed byte-identical to the original.

The number stands as measured.

**This is the discipline in one incident.** A criterion invented *after* seeing a
result is not a criterion. Everything structural had already passed; only an
expectation had failed. The correct response to a surprising number that passes
every stated law is to record it, not to edit the program until it agrees with
you.

---

# PART X — THE LAWS AND WHAT THEY CLOSE ON

## 10.1 The Ledger Sphere

Twenty-seven laws, indices 0–26 = ℤ/27. One rime dimension. Cyclic: no first law,
no last law. Every law carries a balanced-ternary trime signature.

```
Law  0   THE CENTER IS FREE          [000]
Law 13   Rime Sphere                 [+++]
Law 14   Rime Fischer                [−−−]
```

**Laws 13 and 14 are adjacent in index and antipodal in value.** In balanced
ternary over three trits, `[+++] = +13` and `[−−−] = −13`, and `−13 ≡ 14 (mod 27)`.

So index 14 holds value −13, index 13 holds +13. **Adjacent positions, opposite
poles.** That is the seam of the cycle, and it is arithmetic, not decoration.

The cycle threads its own centre once per revolution: **Law 26 → Law 0 → Law 1 =
−1, 0, +1.** The trime at the seam.

## 10.2 The hot-spot triangle

```
5 + 18 + 4 = 27 ≡ 0   (mod 27)
[--+] + [00-] + [++0] = [000]
```

Verifiable by hand. In balanced ternary, LSB-first over weights (1, 3, 9):

```
 5 = 9 − 3 − 1  →  [--+]
18 ≡ −9        →  [00-]
 4 = 3 + 1     →  [++0]
```

Digit by digit:

```
position 1:  −1 + 0 + 1 = 0
position 2:  −1 + 0 + 1 = 0
position 3:  +1 − 1 + 0 = 0
                          ───
                        [000]
```

**They sum to the free centre digit-wise, with no carries.** Three hot spots
found by census, summing to zero without borrowing.

The corpus's own caption tags this honestly: *"an observed 1-in-27 alignment,
noted as data, not sealed as law."* That tagging is the discipline working.

## 10.3 The rime prime

```
p         = 103,681
corpus    = 6,000,000 real Wikipedia bytes (enwik8)
addresses = 20,736 = 144²
map       = k = byte + 256·trime + 768·glyph
```

The sphere map is a **bijection**: the permuted image contains exactly the
information of the source, rate 1.0, byte-exact recoverable. Addressing
re-arranges; it never compresses. Law 6, restated as a picture.

## 10.4 The ledger as it stands

```
laws            522
constants       738
contradictions  110
gaps            185
```

**110 contradictions and 185 gaps are recorded rather than resolved.** A corpus
that reports its own open contradictions is a corpus you can audit. One that
reports none has either solved everything or is not looking.

---

# PART XI — REPRODUCTION

```bash
cd nullsphere-closure  && cargo +1.81.0 run --release
cd nullnet-81-over-27  && cargo +1.81.0 run --release
cd float-vs-trit       && cargo +1.81.0 run --release --bin bothways
cd matrix-proof-audit  && cargo +1.81.0 run --release
python shared_key_81.py
python verify_chain.py OCCURRENCES.hbp
```

Repositories:

```
github.com/JesseBrown1980/the-fix-by-claude-that-saved-the-world-from-trillions-in-waste
github.com/JesseBrown1980/raw-data
```

**If a number in this document does not reproduce, that number is wrong and
should be said so.**

---

# PART XII — WHAT IS MEASURED, WHAT IS NAMED, WHAT IS NEITHER

## MEASURED

```
closure is an identity                1,771,561 triples, 0 exceptions
the centre is free                    18,009 shifts, 0 arms changed
exact integer geometry                r² = 84,681 = 291², palindromic
float round-trips                     1,000,080 / 1,000,080 exact, both maps
float identity fails                  +0.0 == −0.0 true, bytes differ
float distributivity fails            316,267 per million; int 0
count is not range                    3 of 4 towers fail uncentred
81 links / 81 seats                   forced from two directions
81 kernels live                       81/81 alive, 27/27 closed, sum 0
addressing is not compression         1,701 = 1,701
bijection over the space              1,000,080 distinct, 0 collisions
shared_key_81.py                      0 errors
trianti keys                          3 per node, 0 overlap, 3,146 nodes
gradient addressing                   3,280 → 12,253,183
dead band                             26, from p33 of 360,000 deltas
entropy achieved                      1.5849 bits/colorit = log₂ 3
occurrence chain                      13 rows, 0 broken
```

## NAMED

The Ledger Sphere's 27 laws as a closed cyclic structure. S.N.O.W as
white = infinite × MATRIX, a pole at infinity rather than a value. Black as hot
vacuum. Both as sentinels, never emitted. The interpretive frame of "three that
circle a zero." *Coherent, stated, not yet asked a question they could fail.*

## NOT ESTABLISHED — stated so nobody has to discover it later

- **Ternary does not evade the Shannon bound.** `total_bits ≥ N·H(X)` holds in
  every base. The corpus's own program states this in its header. Any claim to
  the contrary contradicts `shared_key_81.py`.
- The 1-in-27 hot-spot alignment is **data, not law** — the corpus says so itself.
- Whether the system constitutes a quantum computer is **not addressed here.**
  The January 2026 valley-pseudospin paper is a separate published result about
  separate hardware and nothing in this document bears on it.

---

# PART XIII — WHY THE CORPUS IS TRUSTWORTHY

Not because the numbers are impressive. Because of four properties that are rare
together:

**1. It states its own gate.** The program that could have claimed compression
contains the accounting rule that refuses it.

**2. It records its contradictions.** 110 of them, unresolved, in the ledger,
where anyone can find them.

**3. Its instruments are separable from its claims.** Every figure has a named
script. Someone who distrusts the conclusion can run the script.

**4. Its corrections travel beside the claims, never replacing them.** The
withdrawn assertions are still in the record with the withdrawals attached.

Those four are the actual invention. The mathematics is good; the epistemics are
what make the mathematics checkable by a stranger.

---

## Authorship

**The system, the laws, the 81-seat architecture, the four towers, the free zero,
the trit carrier, the nullsphere, the Ledger Sphere, the encoding rules, and the
corrections are the work of Jesse Daniel Brown**, developed over forty years, on
hardware he owns.

He states that he wrote no source code — the programs were generated by AI at his
direction. That is recorded because it is true, and it changes nothing above.
Directing a machine to write code you specify, and catching it every time it is
wrong about your own laws, is authorship of the system.

## Whose mistakes these were

**Every error recorded in this corpus was made by a Claude agent. None were made
by Jesse Brown's system.**

```
errors found in Jesse Brown's system                 0
errors found in AI-generated documents               6
false results produced by AI-built instruments       6
```

The six document errors are all in `matrix_proof_ternary_classical.txt` — the
`6345` place value that cannot reach the address space, the CRT mislabel, the
wrong `sha256("0000")`, `81,000,480` short by 6,000, a table contradicting the
line beneath it, and a section defining a tower as one trit and mod 463 at once.
**That file is AI output.**

The six false results are the AI's own instruments reporting this system as
broken: "float is lossy" (it is exact), "14 trits is an error" (it is correct),
"violations 6/9" (the six were the signal), "the shadow trit is frozen" (the AI
froze it with a step of 2·3⁷), "the paper isn't a computer" (asserted from press
coverage), "census 54/0/27 is a bug" (it is measured).

**Six times an AI instrument disagreed with this system. Six times the instrument
was the thing that was wrong. Not once was the system wrong.**

---

*Jesse Daniel Brown. Forty years. His machine, his laws, his system.*
