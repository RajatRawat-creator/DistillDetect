# ReferenceMIAResults

All reference-normalized-loss result JSONs for the paper, plus self-contained,
**CPU-only** scripts that reproduce every "Ours" (reference-based) number exactly
and regenerate the paper's figures. The scripts read nothing outside this folder.

Each result file is `results["Ref-Norm Loss"][candidate] = [per-probe stored
losses]`, where `stored = loss_student − loss_reference` (lower = more like that
candidate's teacher).

## Layout

```
ReferenceMIAResults/
├── controlled/          19 controlled-student results (4-teacher pool)
├── OMI_COT/             OMI-CoT (customized-instruction) students
│   ├── WithFewShotPrompting/      + in-context exemplars from S   (4)
│   └── WithoutFewShotPrompting/   default                         (4)
├── ModelsInTheWild/
│   ├── ReferenceMIA/    6 DeepSeek-R1 distills + s1.1-32B + X-Coder        (8)
│   └── o1AsciiUnicode/  o1 ASCII-vs-Unicode: 4 controlled o1-SFT positives + controls (15)
├── OpenQuestions/
│   ├── ReferenceMIA/    QwQ-32B, DeepSeek-R1 (ref MoE-16B), 3 GPT-OSS configs   (5)
│   └── o1AsciiUnicode/  DeepSeek-R1 + 3 GPT-OSS o1 gaps                          (4)
└── scripts/             reproduce_*.py + plot_figures.py + gather_results.py
```

## Reproduce (CPU-only, no GPU)

```bash
cd scripts
python reproduce_tables.py                    # teacher-ID (controlled + real-world) + top-1 significance   28/28
python reproduce_threshold_generalization.py  # margin threshold + LOSO / leave-one-teacher-out             38/38
python reproduce_o1_ascii_unicode.py          # controlled o1 ASCII-vs-Unicode table                        54/54
python reproduce_open_questions.py            # open-world o1 significance + rankings                        25/25
python plot_figures.py                        # regenerate the paper's figures -> ../figures/
```

Each `reproduce_*.py` ends with a self-checking `VERIFICATION: N/N checks PASS`
that asserts every value against the paper; metric and method definitions are in
each script's header. Only **our** reference-based results are verified. `plot_figures.py` regenerates the
figures from the main paper into `figures/` (gitignored `*.png` / `*.pdf`).

## Rebuilding the tree (`gather_results.py`)

The GPU runners (`reference_mia/`, `o1_detection/`) write per-pair
`*__results.json` under `../outputs/` with producer-side names. `gather_results.py`
rebuilds this curated tree by matching each raw file's content
(`model_name` / `ref_model` / candidate keys) against `results_manifest.json` and
copying it to its checked-in path verbatim — byte-identical, so the reproduce and
plot paths keep resolving. By default it writes a gitignored
`../../ReferenceMIAResults_rebuilt/` so you can `diff -r` before trusting it.

Two DeepSeek-R1 (ref DeepSeek-MoE-16B-Base) files are **not** gathered (manifest
`source="combine"`). DeepSeek-R1 is a 671B model that we ran on **8×H200**, so we
scored DeepSeek-R1 and DeepSeek-MoE-16B-Base **separately** and combined their
per-row losses offline with with a script. To regenerate from scratch you can add the `DeepSeek-R1, deepseek-moe-16b-base` pair
to `reference_mia/pairs_wild.csv`, but it needs that scale of GPU (~8×H200 for
DeepSeek-R1 alone).
