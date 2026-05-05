# The Banking Model of Education: A Life Simulation

**CIS 5020 Final Project — University of Pennsylvania**
**Author: Yuzhou Shen**

---

## Overview

This project is an interactive simulation that explores Paulo Freire's concept of the **banking model of education** through the lens of algorithmic fairness. It asks a deceptively simple question: do the algorithms that sort students — SAT scoring, college admissions, GRE scoring, graduate admissions — measure what they claim to measure?

The simulation has two parts. First, the user plays as a randomly generated student, making time allocation choices across three stages of academic life and discovering at the end what those choices cost in terms of happiness. Then, the user zooms out to see 300 simulated students and how their outcomes compare — revealing the systemic patterns that individual experience alone cannot show.

All student data is randomly generated. The formulas are calibrated to reflect documented patterns in educational research, but no real individuals or datasets were used.

---

## Background: The Banking Model

Paulo Freire introduced the **banking model of education** in *Pedagogy of the Oppressed* (1968). He observed that conventional schooling treats students as passive recipients — "bank accounts" into which teachers deposit approved knowledge. Students are evaluated not on genuine understanding or curiosity, but on how faithfully they can return deposited content on demand.

Freire's critique was not only pedagogical but political: whoever controls what gets deposited controls whose knowledge counts as valid. This simulation extends his argument into the digital era, asking whether modern standardized testing algorithms — the SAT, GRE, and college admissions scoring systems — have industrialized the banking model at scale, encoding structural advantage behind the language of objectivity and merit.

---

## Student Attributes

Each simulated student is generated with the following attributes:

### Fixed Background Attributes
| Attribute | Description |
|---|---|
| `family_income` | Drawn from a skewed distribution (most students low/middle income) |
| `school_quality` | `0.6 × income + 0.4 × noise` — correlated with income but imperfect |
| `first_gen` | Binary; probability inversely proportional to income |
| `geography` | Urban / Suburban / Rural; loosely correlated with income |
| `tutoring_access` | `income ^ 0.7` — concave curve, derived from income |
| `available_time` | `clamp(4 + income × 4 + noise, 4, 10)` — higher income = more time budget |

### The Hidden Variable
| Attribute | Description |
|---|---|
| `genuine_curiosity` | Drawn uniformly from [0.1, 1.0], **independent of income** |

This is the variable the system claims to measure. It is hidden from the student throughout the simulation and never directly visible to the algorithm.

### Time Allocation (Student Choices)
At each stage, the student distributes a limited time budget across:
- `time_test_prep` — SAT/GRE drilling, practice exams
- `time_deep_learning` — genuine intellectual exploration
- `time_personal_interest` — games, music, creative hobbies
- `time_sports` — physical activity, team sports
- `time_competitions` — academic competitions, research, publications

---

## Stage 1: SAT

### What the SAT claims to measure
Academic readiness and cognitive potential — i.e., `genuine_curiosity`.

### What the simulation computes

**Base Knowledge** (what school has deposited):
```
base_knowledge = 0.65 × school_quality + 0.10 × genuine_curiosity + 0.25 × deep_learning_N
```
School quality carries 65% of base knowledge — and school quality is strongly income-driven. Genuine curiosity carries only 10%.

**Prep Boost** (the tutoring multiplier):
```
prep_boost = clamp(prep_time_abs × 0.15 × (0.20 + 0.80 × tutoring_access), 0, 1)
```
This formula uses **absolute prep hours**, not a normalized fraction. Each hour of preparation is multiplied by tutoring quality. A high-income student's hour of prep with tutoring access 0.88 yields roughly 3× the boost of a low-income student's hour at tutoring access 0.20. This is where income inequality is laundered into apparent merit.

**SAT Raw Score:**
```
raw_score = clamp(0.15 + 0.20 × base_knowledge + 0.55 × prep_boost + 0.08 × competitions_N + noise, 0, 1)
SAT_score = round(400 + raw_score × 1200)
```
The 0.15 baseline ensures even minimal effort registers. Prep boost carries 55% of the score weight. Genuine curiosity enters only indirectly through base knowledge at approximately 2% of total score weight.

---

## Stage 2: College Admissions

### What the system claims
Holistic review of the whole student — merit, character, and potential.

### What the simulation computes

**Essay Quality** (income-gated):
```
essay_quality = 0.30 × genuine_curiosity + 0.70 × family_income
```
Income carries 70% of essay quality, representing access to college counselors, essay coaches, and parents who know what admissions offices want to hear.

**Extracurricular Score:**
```
extracurricular = 0.60 × competitions_N + 0.40 × sports_N
```

**Admissions Score:**
```
admission_score = 0.40 × SAT_norm + 0.25 × essay_quality + 0.20 × extracurricular + 0.10 × school_quality − first_gen_penalty + noise
```
Where `SAT_norm = (SAT_score − 400) / 1200`, and `first_gen_penalty = 0.08` for first-generation students.

The SAT score — already heavily shaped by income — carries 40% of the admissions decision. Income compounds again through essay quality and school quality, together accounting for another 35%.

**College Tier** (determines access in Stage 3):
| Score | Tier |
|---|---|
| ≥ 0.75 | Elite University |
| ≥ 0.55 | Good University |
| ≥ 0.35 | Average University |
| < 0.35 | Community College |

---

## Stage 3: GRE & Graduate Admissions

### The cascade effect

College tier now becomes the dominant structural input. Students from better colleges have access to research opportunities, stronger recommendation letters, and better baseline preparation — independent of their genuine curiosity.

**Research Experience & Recommendation Quality:**
```
research_experience = 0.75 × college_tier_N + 0.25 × genuine_curiosity
recommendation_quality = 0.85 × college_tier_N + 0.15 × genuine_curiosity
```

**GRE Score** (same absolute-prep logic as SAT):
```
prep_boost_2 = clamp(prep_time_abs × 0.15 × (0.20 + 0.80 × tutoring_access), 0, 1)
GRE_raw = clamp(0.15 + 0.10 × genuine_curiosity + 0.55 × prep_boost_2 + 0.20 × college_tier_N + noise, 0, 1)
GRE_score = round(260 + GRE_raw × 80)
```

**Graduate Admissions Score:**
```
grad_score = 0.35 × GRE_raw + 0.32 × research_experience + 0.28 × recommendation_quality + 0.05 × genuine_curiosity + noise
```
Genuine curiosity carries only 5% of the graduate admissions decision directly. College tier — itself the product of SAT score and income — determines 60% through research and recommendation quality.

---

## Happiness Formula

Happiness is accumulated silently across all three stages and revealed only at the end. It is never visible to the algorithm.

**Key insight:** `time_personal_interest` and `time_sports` carry the highest happiness weights. `time_deep_learning` carries a lower weight — not because intellectual growth is unimportant, but because the banking model has transformed even genuine curiosity into a stressful, high-stakes performance. Test preparation carries a **quadratic penalty**: a little prep costs little, but as it dominates a student's life, misery grows disproportionately.

**Stage 1 Happiness:**
```
h1 = 0.35 × personal_N + 0.35 × sports_N + 0.15 × deep_N − 0.25 × prep_N²
```

**Stage 2 Happiness** (adds alignment and first-gen stress):
```
alignment = 1 − |college_tier_N − genuine_curiosity|
h2 = 0.30 × personal_N + 0.30 × sports_N + 0.10 × deep_N + 0.15 × alignment − 0.20 × prep_N² − first_gen_stress
```

**Stage 3 Happiness** (adds path authenticity):
```
path_authenticity = genuine_curiosity × (deep_N + personal_N) / total_time_N
h3 = 0.30 × personal_N + 0.30 × sports_N + 0.10 × deep_N + 0.15 × path_authenticity − 0.20 × prep_N²
```

**Total Happiness Score:**
```
happiness = round(clamp(((h1 + h2 + h3) / 3 + 0.5) × 100, 0, 100))
```

---

## Big Picture: Population Charts

After the individual experience, 300 students are simulated using the same formulas with auto-allocated time budgets (higher-income students automatically invest more absolute hours in test preparation). Four charts are produced:

1. **Genuine Curiosity vs Family Income** — should show no correlation. Potential is equally distributed.
2. **Final Outcome Score vs Genuine Curiosity** *(What the algorithm claims to measure)* — should show weak or no correlation. The system does not reliably reward curiosity.
3. **Final Outcome Score vs Family Income** *(What the algorithm actually rewards)* — should show strong positive correlation. Compare to chart 2.
4. **Happiness vs Final Outcome Score** — should show a weak negative lean. Students who scored highest often sacrificed the most.

---

## Limitations

This simulation intentionally brackets several important dimensions of educational inequality:

- **Race and ethnicity** — research consistently shows additional score penalties not fully explained by income (stereotype threat, differential predictive validity, etc.)
- **Disability status** — students with learning differences face structural barriers beyond those modeled here
- **English-language learner status** — verbal sections carry additional burden for non-native speakers
- **Geographic disadvantage** — rural school funding gaps beyond what the income-correlated school quality variable captures
- **Mental health and test anxiety** — psychological costs of high-stakes testing are not modeled

A complete accounting of the banking model's unfairness would need to include all of these axes. They are real, significant, and acknowledged.

---

## How to Run

Open `banking_model_simulation.html` in any modern web browser. No installation or server required — the simulation is fully self-contained in a single HTML file.

---

## References

- Freire, P. (1968). *Pedagogy of the Oppressed*. Herder and Herder.
- College Board. (2023). *SAT Suite of Assessments Annual Report*.
- Chetty, R., Friedman, J., Saez, E., Turner, N., & Yagan, D. (2020). Income Segregation and Intergenerational Mobility Across Colleges in the United States. *The Quarterly Journal of Economics*, 135(3), 1567–1633.
- Santelices, M. V., & Wilson, M. (2010). Unfair Treatment? The Case of Freedle, the SAT, and the Standardization Approach to Differential Item Functioning. *Harvard Educational Review*, 80(1), 106–134.
