# Query–Item Relevance: An Evidence-Centric Research Program for ESCI

## Research Motivation

The real-analysis analogy is a useful way to attack this problem.

The central question is not merely:

> “My cross-encoder is bad on long inputs.”

There is a more fundamental modeling question:

> **Given a very short query \(q\) and a much longer product document \(d\), what mathematical evidence should be sufficient to conclude that \(d\) is relevant to \(q\)?**

For ESCI, this becomes especially interesting because the labels are not merely binary. The dataset defines four query–product relationships:

- **E — Exact**
- **S — Substitute**
- **C — Complement**
- **I — Irrelevant**

The goal is therefore to formulate the problem progressively, much like real analysis:

1. Define primitive objects.
2. Define evidence and relevance.
3. State desirable properties.
4. Turn those properties into empirical hypotheses.
5. Design controlled experiments to test them.
6. Identify violations.
7. Develop models that address the violations.
8. Discover and validate new properties from data.
9. Ultimately improve ESCI prediction while also improving scientific understanding.

---

# 1. Primitive Objects

Let

\[
q=(q_1,\ldots,q_m)
\]

be a query, where typically

\[
m\ll n,
\]

and let

\[
d=(d_1,\ldots,d_n)
\]

be the product document.

For the ESCI setting, a representative regime is:

\[
m=1\ldots5,\qquad n\approx64.
\]

Define the relevance relationship:

\[
R(q,d)\in\{E,S,C,I\}.
\]

The fundamental prediction problem is:

\[
\boxed{f(q,d)\rightarrow\{E,S,C,I\}}
\]

But rather than starting with an architecture, first ask:

> **What properties should a well-behaved relevance function satisfy?**

---

# 2. Definition 1 — Query Evidence

For every query token \(q_i\), define its evidence in the product as

\[
e(q_i,d).
\]

Evidence is not simply whether the token occurs in the document.

For example:

> Query: `iphone 15 case`

Product:

> `Spigen Ultra Hybrid MagFit Case for iPhone 15 Pro Max...`

There are lexical matches, but relevance also depends on semantic and attribute compatibility.

Define token-level evidence as:

\[
e(q_i,d)
=
\max_{j\in d}
s(q_i,d_j),
\]

where \(s(q_i,d_j)\) measures compatibility between query token \(q_i\) and document token \(d_j\).

This naturally motivates token-level query–document interaction.

---

# 3. Theorem-Like Property 1 — Short-Query Relevance Should Be Query-Centric

Suppose

\[
|q|\ll|d|.
\]

A reasonable relevance model should not treat every product token symmetrically. The query should determine which parts of the product constitute evidence.

Conceptually:

\[
R(q,d)
=
F(e(q_1,d),e(q_2,d),\ldots,e(q_m,d)).
\]

This differs from a pure representation-bottleneck view:

\[
(q,d)\rightarrow\text{one CLS representation}
\]

and instead emphasizes:

\[
q_i\rightarrow\text{best evidence in }d.
\]

This is closely related to the intuition behind late-interaction retrieval models such as ColBERT, where contextualized query and document token representations are retained and fine-grained interaction occurs later.

The important research question is:

> **Does query-conditioned, token-level evidence provide a better basis for ESCI classification than a single global query–product representation?**

---

# 4. Limitation of Simple MaxSim

Suppose:

\[
q=\text{``apple watch''}
\]

and

\[
d=\text{``Apple iPhone case ... Apple Watch band ...''}.
\]

A token-level MaxSim mechanism can find:

\[
\text{apple}\rightarrow\text{Apple}
\]

and

\[
\text{watch}\rightarrow\text{Watch}.
\]

But that does not necessarily imply that the product is the desired product.

This motivates the distinction between individual evidence and **joint evidence**.

---

# 5. Definition 2 — Joint Evidence

Define a subset of the document

\[
D_q\subseteq d
\]

containing portions that jointly provide evidence for the query.

Define:

\[
J(q,d)
\]

as the joint evidence relation.

The important distinction is:

\[
\boxed{\text{individual evidence}\neq\text{joint evidence}}
\]

For example:

> Query: `nike running shoes women`

A product might contain:

- Nike
- running
- shoes
- women's

but these terms may occur in unrelated portions of the product representation.

The model must establish that the attributes describe the **same product**.

Thus, the problem is not merely token matching; it involves composition and compatibility.

---

# 6. Definition 3 — Evidence Concentration

Define an evidence distribution over document positions:

\[
P(j\mid q,d).
\]

Define an evidence concentration measure:

\[
C(q,d).
\]

Informally:

- **High \(C\):** most useful evidence is concentrated in a few positions.
- **Low \(C\):** useful evidence is scattered throughout the document.

This captures the observed single-token-query failure:

> Can the model reliably locate the small amount of information relevant to the query inside a much larger product representation?

This is fundamentally a **query-document evidence localization problem**, not necessarily a long-context problem.

---

# 7. Formalizing the Single-Token Failure

Suppose:

\[
q=(\text{``charger''})
\]

and

\[
d=(d_1,\ldots,d_{64}).
\]

Suppose the relevant occurrence appears at:

\[
d_{58}.
\]

A cross-encoder receives something like:

\[
[\text{CLS},q,\text{SEP},d_1,\ldots,d_{64}]
\]

and produces a representation:

\[
h_{\mathrm{CLS}}.
\]

The classifier computes:

\[
P(y\mid h_{\mathrm{CLS}}).
\]

The precise question is:

\[
\boxed{
\text{How much information about }d_{58}
\text{ survives in }h_{\mathrm{CLS}}?
}
\]

This turns an intuitive model failure into a measurable scientific question.

---

# 8. Definition 4 — Query-Conditioned Representation

Instead of representing the product independently:

\[
h_d=f(d),
\]

define:

\[
h_{d\mid q}=f(d;q).
\]

The idea is:

> Do not represent the entire product equally. Represent the product **with respect to the query**.

A possible decomposition is:

\[
h_{d\mid q}
=
g(h_q,h_{D_q}).
\]

This suggests the architecture:

\[
\boxed{
d
\rightarrow
\text{candidate evidence}
\rightarrow
\text{query-conditioned evidence}
\rightarrow
\text{relevance}
}
\]

rather than:

\[
q+d
\rightarrow
\text{single CLS}
\rightarrow
\text{classification}.
\]

---

# 9. Desired Property — Query Evidence Locality

If two documents \(d\) and \(d'\) differ only in portions irrelevant to \(q\), then their relevance should be approximately equal:

\[
d_{\setminus D_q}
\approx
d'_{\setminus D_q}
\]

should imply:

\[
R(q,d)\approx R(q,d').
\]

In plain English:

> **Changing irrelevant parts of the product should not substantially change its relevance to the query.**

This is a desirable property rather than a mathematical theorem.

It becomes scientifically useful because it gives us a controlled intervention.

---

# 10. Position-Invariance Experiment

Take:

> Query: `sony headphones`

Product:

> `Sony WH-1000XM5 Wireless Noise Cancelling Headphones ...`

Calculate:

\[
R(q,d).
\]

Then add irrelevant content:

> `Sony WH-1000XM5 Wireless Noise Cancelling Headphones ... [100 tokens of unrelated text]`

Calculate:

\[
R(q,d').
\]

A robust model should approximately satisfy:

\[
|R(q,d)-R(q,d')|\approx0.
\]

Now move the relevant evidence.

### Version A

> `Sony ... headphones ...`

### Version B

> `[60 irrelevant tokens] Sony ... headphones`

If the prediction changes substantially merely because the evidence moved, this demonstrates **position sensitivity**.

This is scientifically stronger than simply reporting:

> “The model performs worse on long products.”

---

# 11. Definition 5 — Minimal Sufficient Evidence

Define a subset

\[
S\subseteq d
\]

to be sufficient evidence for \(q\) if:

\[
R(q,S)\approx R(q,d).
\]

Define the minimal sufficient evidence:

\[
S^*(q,d)
=
\arg\min_{S\subseteq d}|S|
\]

subject to:

\[
R(q,S)\approx R(q,d).
\]

The research question becomes:

> **Can we find the smallest portion of a product document that is sufficient to determine its relevance to the query?**

This is a particularly useful formulation because it converts evidence selection into an explicit optimization problem.

---

# 12. Evidence-Conditioned Architecture

Instead of:

\[
(q,d)\rightarrow\text{classifier},
\]

use:

\[
q,d
\rightarrow
\boxed{\text{evidence selector}}
\rightarrow
D_q
\rightarrow
\boxed{\text{relevance classifier}}.
\]

For example:

\[
a_j
=
\operatorname{Attention}(q,d_j)
\]

followed by:

\[
D_q
=
\operatorname{TopK}(d,a).
\]

Then:

\[
R(q,d)=f(q,D_q).
\]

This produces a **query-conditioned document compression / evidence selection architecture**.

---

# 13. Definition 6 — ESCI Relationship

ESCI is richer than binary relevance.

Define:

\[
E=\text{Exact}
\]

\[
S=\text{Substitute}
\]

\[
C=\text{Complement}
\]

\[
I=\text{Irrelevant}.
\]

The prediction problem is therefore:

\[
q,d
\rightarrow
\underbrace{\text{relationship}}_{\text{E/S/C/I}}.
\]

These classes have different evidence requirements.

---

# 14. Example of E/S/C/I

Query:

> `iphone 15 case`

Product:

> `Spigen iPhone 15 Case`

Expected relationship:

\[
E.
\]

Product:

> `iPhone 15 Pro Case`

Potentially:

\[
S
\]

depending on annotation semantics.

Product:

> `iPhone 15 screen protector`

Potentially:

\[
C.
\]

Product:

> `Samsung Galaxy S24 case`

Expected relationship:

\[
I.
\]

Token overlap alone cannot solve this.

The model must reason about:

\[
\text{product type}
+
\text{attributes}
+
\text{compatibility}
+
\text{intent}.
\]

---

# 15. Definition 7 — Attribute Satisfaction

For shopping queries, introduce an intermediate latent representation.

Let:

\[
q=(a_1,\ldots,a_k)
\]

where each \(a_i\) is a query constraint or attribute.

For example:

> `black nike running shoes men`

can be decomposed approximately into:

\[
\{
\text{product type=shoes},
\text{brand=Nike},
\text{activity=running},
\text{color=black},
\text{gender=men}
\}.
\]

Define:

\[
A_i(q,d)\in[0,1]
\]

as the degree to which product \(d\) satisfies query attribute \(a_i\).

Then:

\[
R(q,d)
=
F(A_1,\ldots,A_k,\text{relationships}).
\]

For a single-token query such as:

\[
q=\text{``charger''},
\]

there may be few explicit attributes. The model must infer product intent/type from limited query information.

---

# 16. Hierarchy of Definitions

The conceptual theory can therefore be built progressively:

### Definition 1
Query and product:

\[
q,d
\]

### Definition 2
Token-level evidence:

\[
e(q_i,d)
\]

### Definition 3
Joint evidence:

\[
J(q,d)
\]

### Definition 4
Query-conditioned evidence:

\[
D_q
\]

### Definition 5
Sufficient evidence:

\[
S^*(q,d)
\]

### Definition 6
Query attributes:

\[
A(q)=\{a_1,\ldots,a_k\}
\]

### Definition 7
Attribute satisfaction:

\[
A_i(q,d)
\]

### Definition 8
Relationship:

\[
R(q,d)\in\{E,S,C,I\}.
\]

Only after these definitions should we formulate empirical hypotheses.

---

# 17. Empirical Hypothesis 1 — Evidence Locality

Irrelevant document tokens should have limited influence on relevance:

\[
\frac{\partial R(q,d)}
{\partial d_j}
\approx0
\]

when \(d_j\) is irrelevant to \(q\).

This is not a literal differentiable theorem for a discrete text model. It is a conceptual sensitivity hypothesis that can be operationalized through controlled perturbations.

---

# 18. Empirical Hypothesis 2 — Evidence Completeness

If all necessary query evidence exists somewhere in \(d\), the model should be able to recover it regardless of its position.

If:

\[
q_i\leftrightarrow d_j
\]

is the necessary evidence relation, then moving \(d_j\) from position \(j\) to position \(k\) should preserve relevance:

\[
R(q,d)\approx R(q,d').
\]

This directly attacks the observed failure mode.

---

# 19. Empirical Hypothesis 3 — Irrelevant-Token Invariance

If:

\[
d'=d\cup u
\]

where \(u\) contains no information relevant to \(q\), then:

\[
R(q,d')\approx R(q,d).
\]

This tests whether increasing product length itself causes degradation.

---

# 20. Empirical Hypothesis 4 — Evidence Monotonicity

If valid supportive evidence is added:

\[
d'=d\cup e,
\]

then relevance should generally not decrease:

\[
R(q,d')\ge R(q,d).
\]

However, this must be formulated carefully for multiclass ESCI.

Adding evidence that appears relevant in isolation may actually reveal incompatibility. Therefore, a stronger and more defensible formulation is:

> **Adding explicitly compatible evidence should not decrease the model's support for the corresponding relationship.**

---

# 21. Research Literature

There is substantial literature around pieces of this problem.

## 21.1 Late Interaction — ColBERT

ColBERT independently encodes query and document tokens and retains fine-grained interactions until a later scoring stage.

Its central idea is closely related to:

\[
e(q_i,d)
=
\max_j s(q_i,d_j).
\]

This is particularly relevant to the hypothesis:

> Each query token should be able to find its strongest evidence somewhere in the product.

The important distinction is that ColBERT was developed primarily for retrieval/ranking rather than specifically for four-way ESCI classification.

---

## 21.2 SPLADE

SPLADE learns sparse lexical-semantic representations and can capture lexical matches and semantic expansions.

It is relevant when the problem involves:

- rare product terms,
- exact terminology,
- synonyms,
- morphological variants,
- semantic expansion.

---

## 21.3 Long-Document Transformers

Longformer and related architectures address efficient processing of genuinely long sequences.

However, a product representation of approximately 64 tokens is not particularly long for a modern transformer.

Therefore:

\[
\boxed{\text{short-query/evidence localization}}
\]

is likely a more fundamental framing than:

\[
\boxed{\text{long-context processing}}.
\]

---

## 21.4 Amazon Query–Product Relevance

Amazon research has explicitly studied high-precision query-product semantic similarity using BERT cross-encoders.

This confirms that query-product cross-encoding is an established and practical paradigm.

The research opportunity is therefore not simply:

> “Use a cross-encoder.”

It is:

> **Understand the evidence properties under which cross-encoders fail and develop methods that address those failures.**

---

## 21.5 Rationale-Guided Distillation

Recent e-commerce relevance work uses LLM-generated rationales to guide smaller cross-encoders.

This is particularly relevant because a rationale provides information about:

> **which product evidence caused the relevance decision.**

The connection to this research program is therefore:

\[
\text{relevance prediction}
+
\text{evidence supervision}.
\]

---

# 22. The Core Research Question

The research question should not be:

> “How can I make my cross-encoder better?”

That is too implementation-specific.

A stronger formulation is:

> **How can a query-product relevance model reliably identify and aggregate query-specific evidence when the query is substantially shorter than the product document?**

This can be decomposed into:

\[
\boxed{\text{Evidence Localization}}
\]

\[
\boxed{\text{Evidence Aggregation}}
\]

\[
\boxed{\text{Attribute Satisfaction}}
\]

\[
\boxed{\text{Relationship Classification}}.
\]

---

# 23. Progressive Research Program

The research program should proceed similarly to a real-analysis development.

## Stage 1 — Establish the Baseline

Start with:

\[
(q,d)\rightarrow\text{CrossEncoder}\rightarrow E/S/C/I.
\]

Measure performance by:

- query length,
- product length,
- evidence position,
- lexical overlap,
- semantic overlap,
- product field,
- attribute type.

---

## Stage 2 — Establish the Failure Properties

Construct controlled examples.

### Position

\[
d_1=[e,\text{noise}]
\]

\[
d_2=[\text{noise},e].
\]

If:

\[
R(q,d_1)\neq R(q,d_2),
\]

the model demonstrates position sensitivity.

### Noise

\[
d_3=[e,\text{noise}]
\]

\[
d_4=[e,\text{more noise}].
\]

If:

\[
R(q,d_3)\neq R(q,d_4),
\]

the model demonstrates irrelevant-context sensitivity.

This provides a rigorous empirical characterization of the model's failure.

---

# 24. Stage 3 — Query-Conditioned Evidence

Introduce:

\[
q,d
\rightarrow
\text{evidence selector}
\rightarrow
d_q
\rightarrow
\text{classifier}.
\]

The selector can be learned, heuristic, rationale-based, or based on late interaction.

---

# 25. Stage 4 — Compare Interaction Mechanisms

Compare:

\[
\text{Cross Encoder}
\]

vs.

\[
\text{Cross Encoder + Evidence Selection}
\]

vs.

\[
\text{ColBERT / Late Interaction}
\]

vs.

\[
\text{Sparse / SPLADE}
\]

vs.

\[
\text{Lexical + Semantic Hybrid}.
\]

The key is not merely which model has the highest overall score.

The key is:

> **Which model satisfies which evidence properties?**

---

# 26. Preserve Product Structure

A product is not necessarily an undifferentiated 64-token document.

ESCI contains fields such as:

- product title,
- product description,
- product bullet points,
- product brand,
- product color.

Represent:

\[
d=
(d_{\text{title}},
d_{\text{description}},
d_{\text{bullet}},
d_{\text{brand}},
d_{\text{color}}).
\]

Then define field-specific evidence:

\[
e_f(q,d_f).
\]

And:

\[
R(q,d)
=
F(
e_{\text{title}},
e_{\text{description}},
e_{\text{bullet}},
e_{\text{brand}},
e_{\text{color}}
).
\]

This may be more informative than simply concatenating:

```text
query [SEP] title [SEP] description [SEP] bullets [SEP] brand [SEP] color
```

because fields have different semantic roles.

---

# 27. Deeper Mathematical Formulation

Ultimately:

\[
\boxed{
R(q,d)
=
F\left(
\underbrace{\phi(q,d_{S_1})}_{\text{local evidence}},
\underbrace{\phi(q,d_{S_2})}_{\text{local evidence}},
\ldots,
\underbrace{\phi(q,d_{S_k})}_{\text{local evidence}},
\underbrace{A(q,d)}_{\text{attribute compatibility}}
\right)
}
\]

where:

\[
S_1,\ldots,S_k
\]

are query-relevant regions of the product.

The deeper research question becomes:

> **How should query-relevant regions \(S_1,\ldots,S_k\) be selected, represented, and aggregated so that the relevance function is robust to irrelevant context and to the location of relevant evidence?**

This is more fundamental than simply using a larger language model.

---

# 28. Axiomatic Test Suite for ESCI

The first major empirical artifact should be a controlled test suite.

| Property | Experiment |
|---|---|
| **Position invariance** | Move relevant evidence from beginning → middle → end |
| **Noise invariance** | Add irrelevant tokens |
| **Evidence sufficiency** | Keep only query-relevant sentences/spans |
| **Evidence deletion** | Remove one relevant attribute |
| **Evidence addition** | Add a compatible missing attribute |
| **Field sensitivity** | Move evidence between title/description/bullets |
| **Query-length sensitivity** | 1, 2, 3, 4, 5+ tokens |
| **Attribute sensitivity** | brand/model/color/type/etc. |
| **E/S/C/I separation** | Construct controlled E/S/C/I pairs |

For each intervention, compare:

\[
\Delta R
=
|R(q,d)-R(q,d')|.
\]

This creates a quantitative characterization of model behavior.

---

# 29. Experimental Properties and Their Scientific Status

The proposed “theorems” should be treated carefully.

They are generally **not mathematical theorems** in the real-analysis sense.

Instead, use:

### Definition

Precisely define a property.

### Desideratum / Axiom

State what a well-behaved model ideally should satisfy.

### Hypothesis

State a testable empirical prediction.

### Controlled intervention

Modify only the variable of interest.

### Statistical test

Determine whether the observed effect is significant.

### Model intervention

Design a model or training procedure to address the failure.

### Validation

Test whether the intervention improves both the property and actual ESCI prediction.

This distinction is essential for publication-quality work.

---

# 30. Controlled Experiments

## 30.1 Position Invariance

Select examples where the query-relevant evidence can be moved through the product representation.

Positions:

\[
1\rightarrow16\rightarrow32\rightarrow48\rightarrow64.
\]

Measure:

- accuracy,
- macro-F1,
- probability difference,
- label-flip rate,
- position sensitivity.

A useful measure is:

\[
\Delta_{\pi}(q,d)
=
P_R(y\mid q,d)
-
P_R(y\mid q,\pi(d)).
\]

---

## 30.2 Noise Invariance

Append controlled irrelevant text:

\[
N\in\{0,10,20,50,\ldots\}.
\]

Measure:

\[
\Delta_N
=
|P_R(y\mid q,d)-P_R(y\mid q,d+N)|.
\]

Plot performance as a function of noise level.

---

## 30.3 Evidence Sufficiency

Construct reduced products:

\[
d_k
\]

containing only the top \(k\) evidence tokens/spans.

Measure:

\[
\text{Accuracy}(k)
\]

and:

\[
\text{MacroF1}(k).
\]

This yields an **Evidence Sufficiency Curve**.

---

## 30.4 Evidence Deletion

Remove one relevant attribute:

\[
d^- = d\setminus a_i.
\]

Measure:

\[
\Delta_{\text{delete}}
=
P_R(y\mid q,d)
-
P_R(y\mid q,d^-).
\]

Pay particular attention to meaningful transitions such as:

\[
E\rightarrow S
\]

or:

\[
E\rightarrow I.
\]

---

## 30.5 Evidence Addition

Add compatible evidence:

\[
d^+=d\cup a_i.
\]

Measure the fraction of cases where the model's support for the intended relationship increases.

Define a monotonicity-violation rate:

\[
V_{\text{mono}}
=
\frac{
\#\{(q,d):P_R(y\mid q,d^+)<P_R(y\mid q,d)\}
}{N}.
\]

---

## 30.6 Field Sensitivity

Move semantically equivalent evidence across:

- title,
- bullets,
- description,
- brand,
- color.

Measure:

\[
|P_R(y\mid q,d)-P_R(y\mid q,d^{f\rightarrow f'})|.
\]

This determines whether the model is overly dependent on field placement.

---

## 30.7 Query-Length Sensitivity

Stratify by:

\[
|q|\in\{1,2,3,4,5+\}.
\]

Report:

- per-bucket accuracy,
- macro-F1,
- class confusion,
- confidence,
- calibration.

The central question is not merely whether short queries are harder, but:

> **What mechanism causes performance to degrade as query information decreases?**

---

## 30.8 Attribute Sensitivity

Construct controlled interventions involving:

- brand,
- model,
- color,
- size,
- product type,
- compatibility,
- activity,
- gender,
- material.

Measure how strongly each attribute influences:

\[
E/S/C/I.
\]

---

## 30.9 E/S/C/I Separation

Evaluate the complete confusion matrix.

Pay special attention to:

\[
E\leftrightarrow S
\]

\[
S\leftrightarrow C
\]

\[
C\leftrightarrow I
\]

rather than treating all classification errors as equivalent.

---

# 31. Metrics

## Classification

Report:

- overall accuracy,
- macro-F1,
- per-class precision,
- per-class recall,
- per-class F1,
- confusion matrix.

Because ESCI is multiclass and potentially imbalanced, macro-F1 is particularly important.

## Calibration

Measure:

- Expected Calibration Error (ECE),
- reliability diagrams,
- class-wise confidence.

## Invariance

Define an invariance score such as:

\[
\text{InvScore}
=
1-
\frac{1}{N}
\sum_i
\left\|
P_R(y\mid q_i,d_i)
-
P_R(y\mid q_i,d'_i)
\right\|_1.
\]

Higher values indicate greater invariance.

## Evidence Sufficiency

Plot:

\[
\text{Accuracy vs. evidence tokens retained}.
\]

The goal is to determine how much evidence is actually required.

## Statistical Testing

Use paired tests appropriate to the metric and experimental design, such as:

- paired bootstrap confidence intervals,
- Wilcoxon signed-rank tests,
- McNemar's test for paired label outcomes,
- permutation tests.

Avoid treating a single \(p<0.05\) as sufficient evidence. Report effect sizes and confidence intervals as well.

---

# 32. Candidate Models

| Model | Key Idea |
|---|---|
| **Cross-Encoder** | Jointly encode query and product; strong baseline |
| **Cross-Encoder + Evidence Selector** | Select query-relevant evidence before classification |
| **ColBERT / Late Interaction** | Token-level query-document interaction |
| **SPLADE / Sparse Retrieval** | Sparse lexical-semantic representation |
| **COIL / uniCOIL** | Contextualized token-level matching |
| **Rationale-Distilled Encoder** | Use rationale supervision to focus on relevant evidence |
| **Field-Aware Encoder** | Model title, bullets, description, brand, etc. separately |
| **Lexical + Semantic Hybrid** | Combine exact-match and semantic signals |
| **BM25 / lexical baselines** | Simple lower-bound reference |

The objective is not merely to find the strongest model.

It is to understand:

\[
\boxed{\text{model architecture}\rightarrow\text{property satisfaction}\rightarrow\text{ESCI performance}}.
\]

---

# 33. Potentially Novel Research Directions

## 33.1 Permutation-Invariant Training

If controlled experiments demonstrate position sensitivity, train on semantic-preserving permutations:

\[
R(q,d)\approx R(q,\pi(d)).
\]

Hypothesis:

> Evidence-position augmentation improves robustness without reducing discrimination.

---

## 33.2 Evidence-Contrastive Learning

Train the model not only to predict E/S/C/I, but also to identify evidence.

A multi-task objective could combine:

\[
L
=
L_{\text{ESCI}}
+
\lambda L_{\text{evidence}}.
\]

The hypothesis is that explicit evidence supervision produces better relevance representations.

---

## 33.3 Minimal-Evidence Retrieval

Learn an evidence selector approximating:

\[
S^*(q,d).
\]

Then:

\[
(q,d)
\rightarrow
S^*(q,d)
\rightarrow
\text{cross-encoder}.
\]

This could improve both:

- prediction,
- efficiency.

---

## 33.4 Field-Aware Evidence Aggregation

Learn:

\[
e_f(q,d_f)
\]

for each field and dynamically weight fields according to query intent.

---

## 33.5 Query Expansion

For ambiguous short queries, infer implicit attributes.

For example:

\[
\text{charger}
\rightarrow
\{\text{device type},\text{connector},\text{power},\ldots\}.
\]

Then test whether explicit query expansion reduces short-query errors.

---

## 33.6 Product-Relation Modeling

Use relationships among products to help distinguish:

\[
E,S,C,I.
\]

This may be particularly valuable for distinguishing:

- Exact vs Substitute,
- Substitute vs Complement.

---

## 33.7 New Evaluation Metrics

Potential metrics include:

### Position Sensitivity Score

\[
PSS
=
\mathbb E_{\pi}
\left[
\|P_R(y\mid q,d)-P_R(y\mid q,\pi(d))\|
\right].
\]

Lower is better.

### Evidence Sufficiency Curve

\[
ESC(k)=\text{performance using top-}k\text{ evidence tokens}.
\]

### Noise Robustness Curve

\[
NRC(n)=\text{performance after adding }n\text{ irrelevant tokens}.
\]

### Monotonicity Violation Rate

\[
MVR
=
P[
P_R(y\mid q,d^+)
<
P_R(y\mid q,d)
].
\]

These metrics allow models to be evaluated on **behavioral properties**, not just final classification accuracy.

---

# 34. Research Program Structure

The full program can be organized as:

\[
\boxed{\text{Definitions}}
\]

↓

\[
\boxed{\text{Desiderata / Axioms}}
\]

↓

\[
\boxed{\text{Empirical Hypotheses}}
\]

↓

\[
\boxed{\text{Controlled Perturbation Dataset}}
\]

↓

\[
\boxed{\text{Baseline Diagnosis}}
\]

↓

\[
\boxed{\text{Model Intervention}}
\]

↓

\[
\boxed{\text{Property Validation}}
\]

↓

\[
\boxed{\text{ESCI Improvement}}
\]

↓

\[
\boxed{\text{New Property Discovery}}
\]

This is much closer to a **research program** than a single model experiment.

---

# 35. Publication Strategy

The strongest paper would not claim:

> “We created another better cross-encoder.”

That is unlikely to be sufficiently novel by itself.

A stronger contribution would combine:

### Contribution 1 — A formal behavioral framework

Define query-conditioned evidence properties for query-product relevance.

### Contribution 2 — A controlled ESCI evaluation suite

Create systematic interventions for:

- position,
- noise,
- sufficiency,
- deletion,
- addition,
- field,
- query length,
- attributes,
- E/S/C/I relationships.

### Contribution 3 — Empirical discovery

Show which properties current models systematically violate.

### Contribution 4 — Mechanistic explanation

Connect violations to observable model behavior.

### Contribution 5 — A model or training method

Design a method specifically to correct the discovered failure.

### Contribution 6 — Downstream improvement

Show that improving the behavioral properties improves actual ESCI prediction.

The strongest causal chain would therefore be:

\[
\boxed{
\text{Property violation}
\rightarrow
\text{controlled evidence}
\rightarrow
\text{model intervention}
\rightarrow
\text{property improvement}
\rightarrow
\text{ESCI improvement}
}
\]

That is much stronger than:

\[
\text{new architecture}
\rightarrow
\text{+1\% F1}.
\]

---

# 36. Important Novelty Caveat

The individual ideas are not all novel.

Existing research already studies:

- query-product matching,
- ESCI classification,
- cross-encoders,
- late interaction,
- rationales,
- query understanding,
- attributes,
- robustness,
- evidence extraction,
- long-context positional effects.

Therefore the novelty should **not** be claimed as:

> “Nobody has studied evidence.”

Instead, the potential novelty is the **unified formulation and empirical methodology**:

\[
\boxed{
\text{What properties should a query-product relevance function satisfy,}
}
\]

\[
\boxed{
\text{which existing models violate them,}
}
\]

\[
\boxed{
\text{why they violate them, and}
}
\]

\[
\boxed{
\text{whether enforcing those properties improves ESCI prediction.}
}
\]

The exact novelty must ultimately be established by a comprehensive literature review and experiments.

---

# 37. Important Terminology Correction

The properties should generally **not** be called mathematical “theorems” unless they can actually be derived from explicit assumptions.

Use terminology such as:

- **Definition**
- **Axiom / desideratum**
- **Property**
- **Proposition**
- **Hypothesis**
- **Empirical theorem** only when explicitly defined as an empirical claim supported by reproducible evidence.

For example:

> **Position Invariance Property.** A relevance model is position invariant if semantic-preserving permutations of irrelevant/evidence positions do not change its prediction.

Then:

> **Hypothesis.** Existing cross-encoders exhibit measurable position sensitivity on short-query ESCI examples.

Then test it.

This preserves the real-analysis style while remaining scientifically rigorous.

---

# 38. Initial Research Hypotheses

The initial hypotheses should be registered before looking at the final results.

### H1 — Position Sensitivity

Cross-encoder ESCI predictions degrade when necessary evidence is moved toward less favorable positions.

### H2 — Noise Sensitivity

Cross-encoder predictions become less stable as irrelevant product context increases.

### H3 — Evidence Sufficiency

A small subset of query-conditioned product evidence can preserve most of the original classification performance.

### H4 — Evidence Deletion

Removing necessary attributes disproportionately increases E→S, E→C, or E→I errors.

### H5 — Compatible Evidence Addition

Adding valid missing evidence improves support for the corresponding ESCI relationship.

### H6 — Field Dependence

The same semantic evidence has different effects depending on the product field in which it appears.

### H7 — Query-Length Dependence

Shorter queries produce larger uncertainty and greater dependence on product-side evidence.

### H8 — Attribute Dependence

Certain attributes have disproportionately high influence on E/S/C/I classification.

### H9 — Property–Performance Relationship

Models satisfying evidence properties more strongly achieve better ESCI generalization.

### H10 — Intervention Effect

Training specifically to correct property violations improves both robustness metrics and standard ESCI performance.

---

# 39. Discovery of New Properties

Do not stop with the initial hypotheses.

After running the perturbation suite, search for systematic patterns.

For example, suppose you discover:

> A brand token has a very different effect when it occurs alongside a product-type token than when it occurs alone.

That could motivate a new property:

### Attribute Interaction Property

\[
\Delta R(a,b)
\neq
\Delta R(a)+\Delta R(b).
\]

This would reveal that relevance is not simply additive over attributes.

Similarly, you may discover:

### Evidence Coherence

Evidence from the same semantic product entity should have stronger joint influence than evidence scattered across incompatible entities.

Or:

### Compatibility Dominance

A single contradictory compatibility attribute may outweigh several matching generic attributes.

For example:

\[
\text{“iPhone 15 case”}
\]

may match many words, but:

\[
\text{“for Samsung S24 only”}
\]

can dominate the relevance decision.

Such discoveries could lead to genuinely new theory.

---

# 40. The Ultimate Goal

The final objective is not simply:

\[
\boxed{\text{maximize ESCI accuracy}}
\]

but:

\[
\boxed{
\text{understand the structure of query-product relevance}
}
\]

and use that understanding to improve prediction.

The ideal final theory would explain:

\[
\boxed{
R(q,d)
=
\text{evidence localization}
+
\text{evidence composition}
+
\text{attribute compatibility}
+
\text{relationship inference}.
}
\]

The ideal model would then implement this structure.

---

# 41. One-Sentence Research Thesis

A concise formulation of the overall program is:

> **We hypothesize that query-product relevance can be understood as an evidence-conditioned decision process, and that systematic violations of evidence locality, invariance, sufficiency, and attribute compatibility explain a significant fraction of ESCI classification errors; we therefore develop a formal behavioral framework, a controlled perturbation benchmark, and evidence-aware models to test and exploit these properties.**

---

# 42. Practical First Step

Do **not** begin by implementing a new architecture.

First:

1. Freeze the current cross-encoder.
2. Establish its exact ESCI baseline.
3. Build the controlled perturbation generator.
4. Run the position, noise, evidence-deletion, evidence-sufficiency, field, query-length, and attribute experiments.
5. Quantify the violations.
6. Determine which violations correlate with actual E/S/C/I errors.
7. Only then design the intervention.
8. Compare the intervention against strong existing baselines.
9. Validate on untouched ESCI test data.
10. Repeat on additional datasets/models if possible.

The central scientific question should always remain:

\[
\boxed{
\text{Does understanding the evidence properties of relevance lead to better relevance prediction?}
}
\]

That is the research program.
