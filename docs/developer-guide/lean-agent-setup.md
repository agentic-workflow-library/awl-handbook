# AWL Agent Repository Quick‑Start

A concise guide for **setting up a lean, self‑contained AWL agent repo** that includes

1. a human‑friendly **Repo Charter** (`CHARTER.md`) and
2. a machine‑readable **MARC YAML** spec (`meta.awl.yaml`).

---

## 1  Create the Repo Charter → `CHARTER.md`

The charter is the repo's *constitution*: it defines mission, scope, governance, and success criteria.

### 1.1  YAML front‑matter schema

```yaml
---
name: sv-agent                  # repo slug
spec_version: 0.1.0             # charter schema version
license: MIT
maturity: incubating            # incubating | stable | deprecated
owners:                         # GitHub handles with merge rights
  - github: davidroberson
review_policy: 2_approvals      # PR merge rule enforced by branch protection
ci:
  lint: meta-schema@v1          # charter/CWL validator
  test: pytest                  # unit‑test entrypoint
dependencies:
  - awlkit>=0.3.0               # shared helper library
  - gatk-sv==2.2.1              # pinned external dep
---
```

### 1.2  Narrative sections (Markdown)

```markdown
# Structural‑Variant Agent — **Repo Charter**

## Mission
Detect, filter, and QC structural variants from short‑read WGS data with an auditable, cost‑aware pipeline.

## Success Criteria
| Metric | Target |
|--------|--------|
| F1 score (HG002 benchmark) | ≥ 0.92 |
| Runtime on c6a.4xlarge | ≤ 48 h |
| Cost per sample (AWS spot) | ≤ $4 |

## Scope
- **In scope**  short‑read WGS ≥ 30×, GRCh38, VCF + BEDPE outputs, QC dashboard
- **Out of scope**  long‑read callers, somatic SVs, streaming analysis

## Interfaces
- **Inputs**  `sample.bam`, `reference.fa`
- **Outputs**  `sv.vcf.gz`, `qc.json`, `multiqc_report.html`
- **Invoke**  `awl run sv-agent --input sample.bam`

## Governance
1. Code style `black`, `ruff`
2. PR workflow feature branch → draft PR → CI → ≥2 owner approvals → squash merge
3. Release cadence monthly (semver)
4. Deprecation set `maturity: deprecated` + archive repo

## Community Guidelines
We follow the [AWL Code of Conduct](https://github.com/agentic-workflow-language/.github/blob/main/CODE_OF_CONDUCT.md).

## Provenance & Citation
If you use this agent, cite **Agentic Workflow Library – sv‑agent v0.1.0**  
DOI `10.5281/zenodo.1234567`
```

> 🔖 *Tip — Keep `CHARTER.md` ≤ 300 lines. Anything longer belongs in the shared docs site (`awl‑handbook`).*

---

## 2  Define the MARC YAML → `meta.awl.yaml`

Machine‑parsable spec consumed by the AWL runtime and orchestrators.

```yaml
name: sv-agent
spec_version: 0.1.0

purpose: Detect and QC structural variants from short‑read WGS data.

goals:
  - high‑confidence SV VCF per sample
  - QC metrics JSON for downstream triage

capabilities:
  - call_structural_variants
  - filter_low_quality
  - generate_qc_dashboard

tools:
  - cwl: cwl/tools/sv-caller.cwl
  - cwl: cwl/tools/sv-qc.cwl
  - api: "https://gtexportal.org/rest"

knowledge_sources:
  - pubmed: "PMC4384249"
  - vector_db: sova_bio_embeddings

inputs:
  sample_bam: File
  reference_fasta: File

outputs:
  sv_vcf: File
  qc_report: File

constraints:
  compute_budget: "200 CPU hours"
  max_runtime: "48h"
  license: MIT

success_metrics:
  precision: ">=0.95"
  recall: ">=0.90"

collaboration:
  upstream: ['alignment-agent']
  downstream: ['annotation-agent']

version: 0.1.3
```

*Everything >100 kB (datasets, Docker images, long docs) **stays out of Git**.
Use Git LFS, cloud buckets, or reference repos as needed.*

---

## 3  Recommended repo skeleton

```
sv-agent/
├─ agent/                # Python package
├─ cwl/
│   ├─ tools/
│   └─ workflows/
├─ tests/                # fast unit tests; heavy tests live in test‑harness repo
├─ meta.awl.yaml
├─ CHARTER.md
├─ README.md             # this quick‑start doc
└─ .github/workflows/ci.yml
```

---

## 4  CI Size Guard (≤ 20 MB pack)

```yaml
- name: Size guard
  run: |
    BYTES=$(git count-objects -vH | grep "size-pack" | awk '{print $2$3}')
    [[ $BYTES -lt 20M ]] || {
      echo "Repo too large ("$BYTES") – move data to LFS or cloud bucket";
      exit 1;
    }
```

---

### Next Steps

1. Copy the **YAML front‑matter** and narrative template into `CHARTER.md`.
2. Fill in the placeholders in `meta.awl.yaml` with your agent specifics.
3. Commit, push, and let CI verify size + schema.

That's it—your AWL agent repo is lean, auditable, and ready for collaborators!