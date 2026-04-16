# Paper Vault Taxonomy

Defines the fixed classification system used for paper tagging.
**Do not add new tags without explicit user approval.**

---

## Research Area Tags

### Pregnancy Prediction
- **Definition:** Models/studies predicting pregnancy success, clinical pregnancy rate, live birth, or implantation rate
- **Match keywords:** pregnancy prediction, clinical pregnancy, live birth prediction, implantation prediction, implantation rate, pregnancy outcome
- **Example paper:** "Development and Validation of Deep Learning Based Embryo Selection..."

### Gardner Grade
- **Definition:** Embryo morphological grading, Gardner grading system, morphology-based embryo selection
- **Match keywords:** Gardner grade, Gardner score, morphological grading, blastocyst grading, morphology assessment, embryo grading, ICM grade, TE grade
- **Example paper:** "Characterization of an AI model for Ranking Static Images of Blastocyst Stage Embryos"

### PGT Result Prediction
- **Definition:** PGT-A/PGT-M result prediction, chromosomal aneuploidy/euploidy prediction, non-invasive aneuploidy testing
- **Match keywords:** PGT-A, PGT-M, PGS, ploidy prediction, euploidy, aneuploidy, chromosomal, non-invasive PGT, NIPT embryo
- **Example paper:** "Retrospectively Comparing Non-Invasive AI Models and Morphological Assessments for Embryo Euploidy Prediction"

### Oocyte
- **Definition:** Oocyte quality assessment, ovarian reserve, egg selection, maturity evaluation
- **Match keywords:** oocyte, egg quality, ovarian reserve, AMH, follicle, oocyte maturation, oocyte selection, oocyte vitrification

### Sperm
- **Definition:** Sperm quality assessment, sperm selection, sperm analysis for ICSI
- **Match keywords:** sperm, spermatozoa, motility, morphology (in sperm context), ICSI, sperm selection, DFI, sperm DNA fragmentation

### Time-lapse
- **Definition:** Time-lapse imaging of embryo development, morphokinetic analysis, time-lapse microscopy systems
- **Match keywords:** time-lapse, time lapse, timelapse, TLI, TLM, morphokinetics, morphokinetic, time-lapse microscopy, time-lapse incubator, EmbryoScope, Primo Vision

---

## Affiliation Tags

Match against the author's affiliation field.
Case-insensitive. Partial string match allowed.

| Tag | Match strings | Note |
|-----|---------------|------|
| `KaiHealth` | KaiHealth, Kai Health, Kai-Health | |
| `Alife` | Alife, ALife | |
| `Presagen` | Presagen | |
| `AiVF` | AiVF, Ai-VF, AIVF | |
| `Fairtility` | Fairtility | |
| `Vitrolife` | Vitrolife | |
| `Future Fertility` | Future Fertility, FutureFertility | |
| `IVF 2.0` | IVF 2.0, IVF2.0 | |
| `Embryonics` | Embryonics | |
| `FertilAI` | FertilAI, Fertil AI | |
| `IMVITRO` | IMVITRO, ImVitro | |
| `MIM Fertility` | MIM Fertility, MIMFertility | |
| `GENESYS AI LABS` | Genesys AI, GENESYS | |

---

## Tagging Rules

1. **Match source:** search abstract + title + affiliation text for the keywords above
2. **No forced tags:** if no match, leave empty — do not guess
3. **Multiple tags allowed:** a paper can belong to multiple research areas
4. **Affiliation priority:** check affiliation field first, then abstract body
5. **Adding new tags:** only when user explicitly requests; add to this file
