# Where Is the Free Lunch Paid?

### Two AI-driven biological design systems, read against the No Free Lunch theorem

**Scope.** A side-by-side architectural reading of (1) the Kriegman–Blackiston–Levin–Bongard *Xenobot* pipeline (PNAS 2020; self-replication PNAS 2021) and (2) the King–…–Hie *generative bacteriophage* pipeline (bioRxiv 2025, Evo 1 / Evo 2), followed by an inference about where the cost of “design without exhaustive search” is actually paid, and a mapping of Evo’s inference-time guidance onto a Hermes/Evo2 ΦX174 regional-editing workflow.

> Note on the brief: the request named a “synthetic macrophage.” No Stanford/Evo2 *macrophage* genome-design effort exists; engineered-macrophage work is gene-circuit synthetic biology (CAR-Ms, synthetic JAK/STAT and TET-ON receptors). The Evo2 generative-biology flagship is a **bacteriophage** built on ΦX174. That is the system analyzed here.

-----

## 1. The two systems at a glance

|Axis                 |Xenobot (2020 / 2021)                                                            |Evo phage (2025)                                                                             |
|---------------------|---------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
|Core method          |Evolutionary search (Age-Fitness-Pareto Optimization)                            |Autoregressive genome language model (StripedHyena)                                          |
|“Designed object”    |A **shape** (voxel morphology + tissue placement)                                |A **genome** (whole ΦX174-scale sequence)                                                    |
|Search primitive     |Passive (epidermal) vs. contractile (cardiac) voxels                             |Single nucleotides, byte-level                                                               |
|Evaluator (in silico)|Soft-body physics sim (Voxelyze/VoxCad)                                          |Predictive filters: geNomad, CheckV, ESMFold, bespoke ORF caller, tropism/synteny constraints|
|Diversity mechanism  |Pareto “age” objective; 99 independent runs                                      |Sampling temperature (0.7–0.9) + prompt length (4–9 nt)                                      |
|Robustness filter    |Random phase modulation of contractile cells                                     |Tiered constraints: quality → tropism → diversification                                      |
|Wet-lab yield        |1 design selected, hand-built, validated                                         |302 candidates → 285 synthesized → **16 viable**                                             |
|Prior it exploits    |Cells’ multi-scale competency (self-organization, healing, intrinsic contraction)|Evolutionary statistics of ~2M phage genomes                                                 |
|Signature result     |Pac-Man/C-shape no engineer would propose                                        |Evo-Φ36 uses a G4 packaging protein non-viable in wild-type context                          |
|Compute              |Months on Deep Green (modest)                                                    |Pretraining on 2,000+ H100s; design = CPU-side filtering                                     |

The deep structural symmetry: **both are generate-and-filter systems where a cheap generator proposes broadly and an expensive evaluator disposes narrowly.** They differ only in where the prior lives — in the cells (Xenobot) or in the weights (Evo).

-----

## 2. Xenobot architecture — designing a boundary condition

**Generator.** The Bongard lab uses Age-Fitness-Pareto Optimization (AFPO): a population is doubled each generation by mutated copies; a fresh random individual (age 0) is injected; selection then reduces back to size on two objectives — **maximize fitness (displacement), minimize age**. The age objective is the trick: it protects young lineages from being out-competed before they mature, preserving diversity and preventing premature convergence. Candidate bodies are scored inside a soft-body voxel simulator that models adhesion, friction, buoyancy, and aqueous drag. The run is repeated ~99 times from different seeds to harvest a *diversity* of performant designs rather than a single optimum.

**The epithelial/cardiac concatenation.** This is the conceptual core and is widely misread as active control. There is no controller. Two cell types are the only primitives: passive **epidermal** voxels (structure) and contractile **cardiac** voxels. Cardiomyocytes contract on their own — undirected oscillation. The evolutionary search therefore does not design a *behavior*; it designs a **morphology that rectifies undirected contraction into directed function** (locomotion, debris aggregation, in the 2021 work kinematic self-replication). The cardiac tissue is the unpowered engine; the epidermal geometry is the transmission. The AI’s entire job is to find the geometry that turns noise into work.

**Closed loop.** Sim-to-real discrepancies are returned to the algorithm as constraints on subsequent design–manufacture cycles, and tissue-shaping technique is co-adapted so the built organism behaves more like its virtual model. The loop runs in both directions: the model is corrected toward reality *and* reality is shaped toward the model.

-----

## 3. Evo phage architecture — reading out a learned manifold

The pipeline has six stages; the design intelligence is concentrated in stages 4–6, not in raw generation.

1. **Pretraining.** Evo 1 (7B, 131K ctx) and Evo 2 (7B, 8K ctx), pretrained on large DNA corpora including ~2M phage genomes. Base models alone classify only ~19–38% of generations as viral — capable but unsteered.
1. **Supervised fine-tuning.** On ~15k *Microviridae* sequences, sharpening the model toward ΦX174-like genomes and enabling full-length generation from the first context position.
1. **Prompt engineering.** ΦX174 genomes share a conserved start consensus; prompting with **4–9 nt** of it steers toward ΦX174-like output. Too long (≥9 nt at low temp) → memorized recall; too short (1–2 nt) → insufficient conditioning. The usable window is narrow.
1. **Inference-time guidance (the heart).** Autoregressive sampling at **temperature 0.7–0.9**, then **tiered constraint filtering**:
- *Sequence quality*: nucleotide-only; length 4–6 kb; GC 30–65%; no homopolymer > 10 bp.
- *Tropism specificity*: spike protein ≥ 60% identity to the ΦX174 spike (host range is set by spike–receptor binding).
- *Evolutionary diversity*: prefer < 95% average amino-acid identity to natural proteins; enforce a genetic-architecture/synteny constraint (favor 10–12 genes, single gene gain/loss tolerated) to push novelty while preserving viability.
- A **bespoke overlapping-ORF caller** was required because none of six standard gene predictors could annotate all 11 ΦX174 genes — overlapping reading frames defeat off-the-shelf tools.
1. **Curation.** 302 diverse candidates selected; CheckV rated > 87% High Quality/Complete.
1. **Wet-lab screen.** 285/302 synthesized (17 failed on synthesis complexity); **16 inhibited growth of *E. coli* C**; none inhibited K-12, confirming the tropism filter held. Downstream: cryo-EM (Evo-Φ36’s divergent G4 J protein), fitness competitions (Evo-Φ69 outcompeted all, 16–65× fold change), faster lysis (Evo-Φ2483), and a 16-phage cocktail that overcame three ΦX174-resistant strains where ΦX174 alone failed.

The instructive failure statistic: **> 50 non-viable designs carried as few or fewer mutations than the viable ones.** Mutation count does not predict viability; the model is doing something subtler than “stay close to ΦX174,” and the wet-lab screen is the only oracle that knows the difference.

-----

## 4. The No Free Lunch ledger — where the cost is actually paid

Wolpert & Macready: averaged over all problems, no search algorithm beats any other. Performance comes *entirely* from prior assumptions matched to the problem’s structure. So “where is the free lunch paid?” has a precise meaning — **which ledger holds the inductive bias** — and the cost distributes across four, with a fifth, inverted entry.

**Ledger 1 — Evolution’s pre-paid compute (largest, most hidden).**
Neither system designs biology from nothing. Xenobot search designs a boundary condition and lets the cells’ competency — self-organization, healing, intrinsic contraction — fill in the rest. Evo launders the statistical regularities natural selection already wrote into trillions of nucleotides. **The free lunch was pre-paid by ~4 billion years of selection**, and the AI is a compression/retrieval engine over that purchased compute. This is why Evo works on phages (trained prior) and is expected to fail on human viruses (untrained prior): the bill is in the genomes read, not the GPUs.

**Ledger 2 — Architecture and compute (real, doing less than it looks).**
The 2,000+ H100s and the StripedHyena long-context architecture buy *fidelity of coverage* of the evolutionary prior — holding genome-scale interactions in context — not *insight*. Xenobots prove the point by counterexample: a near-trivial AFPO search on a modest cluster produced a result of comparable depth (self-replication). Compute scales the **resolution** of the prior, not the cleverness of the search.

**Ledger 3 — Human conception (underpriced in every press account).**
The load-bearing moves are human framing decisions: choosing passive-vs-contractile voxels as the primitive; choosing displacement as fitness and re-running for diversity; choosing ΦX174 as template and *Microviridae* as the SFT set; tuning the 4–9 nt prompt and 0.7–0.9 temperature window; **building the overlapping-ORF caller and the constraint tiers**. This is where most of the *marginal, non-pre-paid* cost lands.

**Ledger 4 — The oracle / validation cost (the NFL receipt).**
A search is only as good as its evaluator. In silico: the physics sim, or the geNomad/CheckV/ESMFold/tropism stack. In vivo: microcautery and plaque assays. Both pipelines are generate-and-filter, and the filter kills candidates en masse — ~5% phage viability, “most robust of 99” for Xenobots. **Most of the apparent “design” is selection**, and selection requires an expensive ground-truth oracle.

**Ledger 5 (inverted) — The cost you do *not* see: forfeited understanding.**
Neither system is charged in *mechanistic insight* — and that is precisely what it forfeits. Levin and Bongard state they do not fully know why Xenobots behave as they do; Evo-Φ36 runs on a packaging protein that is non-viable in wild-type context and that no human reasoned into place. The real trade is **understanding for function**: a working artifact whose design logic lives in a prior (bioelectric, or latent-weight) never expressed in human-legible terms. These pipelines are *anti-interpretable by construction*. This connects directly to the explainability-as-safety argument (Yampolskiy): the more we outsource design to an opaque prior, the more capability outruns auditability.

**One-line synthesis:** the lunch is paid mostly by evolution (pre-paid), then by human framing of the search and its oracle (marginal), and only thinly by compute — and the receipt is written in forfeited interpretability.

-----

## 5. Mapping Evo’s inference-time guidance onto a Hermes/Evo2 ΦX174 editing workflow

Your workflow is *regional editing* of ΦX174, not de novo generation — which changes the prompt step but leaves the guidance/oracle layer almost identical. The phage paper’s tiers translate cleanly into a CPU-side validation skill that sits *downstream* of Evo2 GPU inference and *upstream* of any wet-lab or LoVis4u visualization step:

|Evo phage stage          |Your editing analogue                                                                               |Hermes placement              |
|-------------------------|----------------------------------------------------------------------------------------------------|------------------------------|
|SFT on *Microviridae*    |Optional LoRA/fine-tune on Microviridae if editing drifts off-distribution                          |GPU node (desktop)            |
|4–9 nt consensus prompt  |Anchor/flank the **edited region** with native ΦX174 context to hold reading frame + ori/terminator |GPU node                      |
|temp 0.7–0.9 sampling    |Same window for edit proposals; lower temp for conservative edits                                   |GPU node                      |
|Sequence-quality tier    |Length/GC/homopolymer gate on every edited candidate                                                |**CPU filter skill (any box)**|
|Tropism tier             |Spike (gene F/G) identity ≥ 60% — do **not** silently break host range when editing structural genes|CPU filter skill              |
|Diversification + synteny|Gene-count 10–12 + synteny check — catches frame-shift/ORF-collision damage from a regional edit    |CPU filter skill              |
|Overlapping-ORF caller   |**Mandatory** for ΦX174 — a naive ORF finder will miss A*/B/E/K overlaps and pass broken edits      |CPU filter skill              |
|302→16 wet-lab screen    |Your viability oracle (assay or, pre-lab, a learned viability classifier)                           |Effector module / lab         |

**Architectural point for Hermes:** Evo2 inference is the *generator*; the constraint tiers are the *guidance*; the assay is the *oracle*. Keeping these on separate nodes mirrors the paper’s own division of labor and maps onto your heterogeneous cluster: GPU desktop generates, the Linux boxes run the (embarrassingly parallel) filtering, and only survivors reach the effector. The accompanying script (`phage_constraint_filter.py`) implements the three computational tiers, including an overlapping-ORF-aware gene counter, as a drop-in Hermes skill.

-----

## 6. The field note — bioelectric vs. morphic, and the Principlex parallel

Two different things hide under “morphic field.” Sheldrake’s *morphic resonance* is pseudoscience and is not operating here. Levin’s actual framework — **bioelectric / morphogenetic fields**, measurable voltage gradients acting as a top-down pattern *setpoint* — is real and is functionally a field-like prior: an attractor landscape that constrains where morphogenesis can land. That is structurally the same role Evo’s learned latent manifold plays: a high-dimensional basin of “viable genomes” that generation falls into.

Both systems therefore instantiate the same abstract move — **a pattern that already exists in a latent space (a bioelectric setpoint, or a model’s weight space) is read out into physical form under search pressure.** That is, almost verbatim, the Agent Principlex thesis of patterns descending from latent space into the world. The Xenobot is that descent made literal: an attractor in morphospace, instantiated in frog cells. The Evo phage is the same descent in sequence space. In both, the “field” is not mystical — it is the paid-for prior of Ledger 1, viewed as a geometry rather than a ledger entry.

-----

## Sources

- Kriegman, Blackiston, Levin, Bongard. *A scalable pipeline for designing reconfigurable organisms.* PNAS 117(4):1853–1859 (2020). doi:10.1073/pnas.1910837117
- Kriegman, Blackiston, Levin, Bongard. *Kinematic self-replication in reconfigurable organisms.* PNAS 118(49):e2112672118 (2021). doi:10.1073/pnas.2112672118
- Schmidt & Lipson; and Bongard-lab methods lineage on Age-Fitness-Pareto Optimization and soft-body voxel evolution (Voxelyze/VoxCad).
- King, Driscoll, Li, Guo, Merchant, Brixi, Wilkinson, Hie. *Generative design of novel bacteriophages with genome language models.* bioRxiv 2025.09.12.675911 (2025).
- Nguyen, Poli, Durrant, et al. *Sequence modeling and design from molecular to genome scale with Evo.* Science (2024). Brixi et al., *Evo 2*, Arc Institute / Nature (2026).
- Wolpert & Macready. *No Free Lunch Theorems for Optimization.* IEEE Trans. Evol. Comput. (1997).