# Guarani Mathematical Reasoning Study

This study compares how five language models solve a set of 40 mathematics problems
when presented in Guarani or Spanish. It also measures the effects of seven prompt
conditions. Each problem is evaluated in both languages under every prompt condition
and model.

## Study design

### Research questions

1. How does answer accuracy vary across the five selected models and the three educational datasets?
2. For the same problem, model, and prompt, how does performance differ between Guarani and Spanish?
3. Do the seven prompt interventions change accuracy relative to the standard baseline?

### Data

| Dataset ID | Level | Items | Active source file |
|---|---|---:|---|
| `elementary` | Elementary education | 10 | [`data/math_problems_elementary.json`](data/math_problems_elementary.json) |
| `jr_highschool` | Junior high school | 10 | [`data/math_problems_jr_highschool.json`](data/math_problems_jr_highschool.json) |
| `k3_elementary` | K–3 elementary | 20 | [`data/math_problems_k3_elementary.json`](data/math_problems_k3_elementary.json) |

The active sample contains 40 problems with paired Guarani and Spanish statements
and an answer key. Models receive one language at a time; they do not receive the
paired statement or expected answer. The `reasoning_type` condition uses one
consistent category label per problem. These Guarani labels and all other Guarani
prompt text have been reviewed by a native Guarani speaker.

### Models and providers

| Study model | Provider route | Deployment / model ID setting |
|---|---|---|
| Gemini 3.5 Flash | Gemini API, OpenAI-compatible chat endpoint | `GEMINI_MODEL_ID` |
| Grok 4.1 | Azure AI Services | `AZURE_GROK41_DEPLOYMENT` |
| GPT-4o mini | Azure OpenAI | `AZURE_GPT4O_MINI_DEPLOYMENT` |
| Llama 3.3 70B | Azure AI Services | `AZURE_LLAMA33_DEPLOYMENT` |
| Mistral Large 3 | Azure AI Services | `AZURE_MISTRAL_LARGE3_DEPLOYMENT` |

Before inference starts, the runner looks up the selected models’ published metadata.
For Gemini it calls the Gemini Models API; for Azure it reads the version configured
from the portal. It records the configured version, the
provider model name, lookup source/status, and a fresh `model_metadata_checked_at_utc`
timestamp in every request record. It also records the API-returned model ID when
available, endpoint/API settings, SDK version, decoding settings, request/completion
timestamps, response metadata, and dataset/prompt hashes. A provider may omit a
version; in that case the record states `version_not_returned` and retains the model
name/deployment ID.

### Prompt conditions

All seven prompt conditions are crossed with all 40 items, both languages, and all
five models. Every prompt includes the same age-neutral mathematics-teacher role
framing; this is shared context, not a separate condition. Conditions share the same
JSON answer schema and do not specify an age or grade.

| Condition ID | Intervention |
|---|---|
| `minimal` | Simple request to solve the problem |
| `standard` | Identify givens and question, choose a method, check the answer |
| `cross_language_pivot` | Internally use the counterpart language, then answer in the input language |
| `semantic_interpretation` | Interpret meaning and context rather than translate literally |
| `vocabulary_aid` | Add a bilingual, language-appropriate glossary |
| `few_shot` | Show the same two worked examples in each language |
| `reasoning_type` | Provide a consistent math-category hint |

The prompt set is in [`prompts/benchmark_prompts.json`](prompts/benchmark_prompts.json). The answer contract requests JSON with `reasoning`, `steps`, and `final_answer.value`; numeric, structured, and symbolic answer values are allowed. The minimal condition changes the task instruction only; the shared response contract is retained for comparable scoring.

### Run size

The complete design plans **2,800 model requests**: 40 problems × 2 languages × 7 prompt conditions × 5 models. The runner makes one request per cell, disables SDK retries, records failures, and does not automatically retry. Any deliberate reruns should use a separate output file and be documented; do not silently replace failed records.

### Outcomes and statistical analysis

The primary outcome is exact/correct answer accuracy from `final_answer.value`. Numeric answers use absolute tolerance `1e-4`; structured answers are compared component by component; symbolic expressions are compared with SymPy when available. The evaluator does not extract arbitrary numbers from explanatory prose. Invalid/missing answers and failed requests count as incorrect.

Reports provide accuracy by dataset, language, model, and prompt condition with 95% Wilson intervals. Paired Spanish–Guarani comparisons match the same problem, model, and prompt and use exact two-sided McNemar tests. Prompt comparisons pair each condition against `standard` on the same problem, model, and language. Holm adjustment controls family-wise error within each report comparison family. Prompt-by-dataset findings are exploratory because each dataset has only 10–20 items; confidence intervals and p-values do not remove that limitation.

## Results

This snapshot combines the Azure model results in [`results/run_20261112T201956910360Z.jsonl`](results/run_20261112T201956910360Z.jsonl) with the Gemini rerun in [`results/run_20261113T215959373756Z.jsonl`](results/run_20261113T215959373756Z.jsonl). Gemini returned responses for all 560 requests; one Mistral request failed with HTTP 500. Overall, 1,849 of 2,800 planned requests were scored correct (66.0%) when that failed request is counted as incorrect. Model, language, dataset, and prompt accuracy below uses successful API responses only (2,799 responses). Accuracy was evaluated with SymPy 1.14.0; the evaluator compares coordinate tuples such as `(0, 0)` safely, without changing the source JSONL files.

### Accuracy by model and language

| Model | Guarani accuracy | Spanish accuracy | General accuracy |
|---|---:|---:|---:|
| Gemini 3.5 Flash | 254/280 (90.7%) | 270/280 (96.4%) | 524/560 (93.6%) |
| GPT-4o mini | 89/280 (31.8%) | 234/280 (83.6%) | 323/560 (57.7%) |
| Grok 4.1 | 89/280 (31.8%) | 254/280 (90.7%) | 343/560 (61.3%) |
| Llama 3.3 70B | 52/280 (18.6%) | 239/280 (85.4%) | 291/560 (52.0%) |
| Mistral Large 3 | 116/279 (41.6%) | 252/280 (90.0%) | 368/559 (65.8%) |

**Note.** Accuracy is the number of correct answers divided by the number of successful API responses. A successful response means the provider returned a model response, whether or not the answer was correct or parseable as JSON. Provider request failures are excluded from these accuracy denominators. Across successful responses, Guarani accuracy was 600/1,399 (42.9%) and Spanish accuracy was 1,249/1,400 (89.2%).

Gemini 3.5 Flash had the highest overall accuracy (93.6%) and the highest Guarani accuracy (90.7%). Across the other four models, Spanish accuracy ranged from 83.6% to 90.7%, while Guarani accuracy ranged from 18.6% to 41.6%, showing a much wider spread in Guarani performance.

| Dataset | Guarani accuracy | Spanish accuracy |
|---|---:|---:|
| `elementary` | 137/350 (39.1%) | 344/350 (98.3%) |
| `jr_highschool` | 175/350 (50.0%) | 221/350 (63.1%) |
| `k3_elementary` | 288/699 (41.2%) | 684/700 (97.7%) |

Guarani accuracy remains higher on `jr_highschool` than on `elementary`. This should not be read as evidence that the models solve harder mathematics better. The sets may differ in more than grade level: `jr_highschool` includes several direct, formula-based problems, such as circle equations, while `elementary` includes multi-step word problems whose Guarani phrasing may be awkward or ambiguous. For example, some questions use unusual phrasing for quantities or relationships, which could make them harder to interpret even when the underlying arithmetic is elementary.

With only ten problems in each dataset, this grade-level comparison is exploratory. The items' difficulty and translation quality should be reviewed before drawing conclusions.

### Accuracy by model and dataset

Each dataset is split by language; cells show correct answers / successful API responses and accuracy.

| Model | Elementary, Spanish | Elementary, Guarani | Junior high, Spanish | Junior high, Guarani | K–3, Spanish | K–3, Guarani |
|---|---:|---:|---:|---:|---:|---:|
| Gemini 3.5 Flash | 70/70 (100.0%) | 60/70 (85.7%) | 62/70 (88.6%) | 57/70 (81.4%) | 138/140 (98.6%) | 137/140 (97.9%) |
| GPT-4o mini | 69/70 (98.6%) | 16/70 (22.9%) | 27/70 (38.6%) | 31/70 (44.3%) | 138/140 (98.6%) | 42/140 (30.0%) |
| Grok 4.1 | 67/70 (95.7%) | 16/70 (22.9%) | 47/70 (67.1%) | 37/70 (52.9%) | 140/140 (100.0%) | 36/140 (25.7%) |
| Llama 3.3 70B | 68/70 (97.1%) | 14/70 (20.0%) | 41/70 (58.6%) | 16/70 (22.9%) | 130/140 (92.9%) | 22/140 (15.7%) |
| Mistral Large 3 | 70/70 (100.0%) | 31/70 (44.3%) | 44/70 (62.9%) | 34/70 (48.6%) | 138/140 (98.6%) | 51/139 (36.7%) |

Gemini had the highest Guarani accuracy in each dataset. Spanish accuracy was near ceiling for most models on the elementary and K–3 sets, while results on the junior-high set were lower and more variable across models and languages.

### Prompt conditions

Each cell shows correct answers / successful API responses and accuracy. The single failed Mistral request was for `k3_elementary`, Guarani, under `cross_language_pivot`, so that cell has one fewer response.

| Prompt condition | Elementary, Spanish | Elementary, Guarani | Junior high, Spanish | Junior high, Guarani | K–3, Spanish | K–3, Guarani |
|---|---:|---:|---:|---:|---:|---:|
| `minimal` | 50/50 (100.0%) | 17/50 (34.0%) | 34/50 (68.0%) | 24/50 (48.0%) | 99/100 (99.0%) | 43/100 (43.0%) |
| `standard` | 50/50 (100.0%) | 18/50 (36.0%) | 30/50 (60.0%) | 18/50 (36.0%) | 99/100 (99.0%) | 36/100 (36.0%) |
| `cross_language_pivot` | 49/50 (98.0%) | 18/50 (36.0%) | 30/50 (60.0%) | 24/50 (48.0%) | 99/100 (99.0%) | 38/99 (38.4%) |
| `semantic_interpretation` | 49/50 (98.0%) | 19/50 (38.0%) | 32/50 (64.0%) | 28/50 (56.0%) | 95/100 (95.0%) | 37/100 (37.0%) |
| `vocabulary_aid` | 48/50 (96.0%) | 25/50 (50.0%) | 31/50 (62.0%) | 23/50 (46.0%) | 97/100 (97.0%) | 42/100 (42.0%) |
| `few_shot` | 49/50 (98.0%) | 23/50 (46.0%) | 28/50 (56.0%) | 30/50 (60.0%) | 95/100 (95.0%) | 51/100 (51.0%) |
| `reasoning_type` | 49/50 (98.0%) | 17/50 (34.0%) | 36/50 (72.0%) | 28/50 (56.0%) | 100/100 (100.0%) | 41/100 (41.0%) |

Spanish accuracy remained high for elementary and K–3 across prompt conditions, while Guarani accuracy varied more. Few-shot examples had the highest Guarani accuracy on K–3 (51.0%), and the vocabulary aid condition was highest on elementary Guarani (50.0%); no single condition led across datasets and languages.

No prompt condition differed significantly from `standard` after Holm correction across 180 paired comparisons. In paired Spanish–Guarani comparisons, 25 of 105 dataset-by-model-by-prompt tests remained significant after correction; all were in `k3_elementary`. These comparisons should be interpreted cautiously given the small number of problems per dataset.

The output-format check found responses without recoverable JSON for 17 Gemini, 158 Llama, and 40 Mistral answers. The evaluator recovered JSON from many other fenced or prose-wrapped outputs, but these formatting issues should be reported alongside accuracy.

## Getting started

### 1. Set up Python

Use Python 3.10 or newer. No Azure CLI is required. From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
```

### 2. Configure provider access

Copy `.env.example` to `.env` and fill in the credentials and deployment settings for your accounts. Keep `.env` private and do not commit it.

Required credentials:

- `GEMINI_API_KEY` for Gemini.
- `AZURE_OPENAI_API_KEY` for Azure inference.

To find an Azure version, open the Azure AI Foundry portal, go to **Models + endpoints**, select the deployment, and read **Model version** on its **Details** page. In the classic Azure portal, open the Azure OpenAI resource, select **Model deployments**, then open the deployment details. Enter the displayed version in the matching `AZURE_*_MODEL_VERSION` field in `.env`. The portal shows the deployment's model version separately from its deployment name. See [Microsoft's model-version guidance](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/model-versions).

Endpoint and deployment variables have defaults for the configured GuaranIA resource; override them only if your deployment names or endpoints differ. `AZURE_OPENAI_API_VERSION` applies to GPT-4o mini. Copy each Azure model version from the portal into its matching `AZURE_*_MODEL_VERSION` field. The optional `AZURE_MODEL_METADATA_PORTAL_CHECKED_AT_UTC` records when you checked them. The runner stamps `model_metadata_checked_at_utc` when it reads the configured metadata. It uses one Azure inference key for the configured endpoints. Never put a credential in a command-line argument or results file.

### 3. Validate the full study without making calls

```bash
python -m src.run_benchmark --dry-run
```

The dry run validates all selected dataset-language-condition combinations and prints the planned request count and routing configuration. It makes zero provider calls and does not require credentials. During a real run, metadata is resolved before inference requests; a failed Gemini lookup stops the run before paid model calls. The expected count for the default full design is 2,800. To check model versions without sending inference requests, run:

```bash
python -m src.run_benchmark --check-model-metadata
```

This check calls the Gemini Models API for Gemini and reads the configured Azure version fields; it does not need Azure CLI or an Azure management-plane login. It prints the model IDs, versions, UTC check times, and lookup status. A Gemini API key is needed only when Gemini is selected. To review generated instructions:

```bash
python -m src.run_benchmark --dry-run --show-prompts
```

Check that the deployment IDs are correct, all Guarani instructions and labels have been reviewed, and the planned count is 2,800 before the paid run.

## Reproduce the experiments

### Guarani and Spanish comparison

The default command runs all five models, all 40 problems, both languages, and all seven conditions. It produces a new timestamped JSONL file directly under `results/`.

```bash
python -m src.run_benchmark
```

This command makes paid API calls and is expected to create up to 2,800 requests. A run can be narrowed for a deliberate subset using `--datasets`, `--models`, `--languages`, and `--conditions`; for example:

```bash
python -m src.run_benchmark --datasets elementary --models gemini-3.5-flash --languages gn es --conditions standard
```

Subset runs are not the complete study and must be described as such. The JSONL output stores one record per attempted request. Set a separate output path to keep intentional reruns distinct:

```bash
python -m src.run_benchmark --output results/pilot.jsonl
```

### Prompt comparison

Prompt comparisons use the same all-crossed run above. Each problem is evaluated under all seven conditions, in both languages, with all five models. To execute only prompt comparisons for a subset, specify multiple condition IDs and preserve the `standard` condition as the paired baseline, for example:

```bash
python -m src.run_benchmark --conditions standard minimal cross_language_pivot semantic_interpretation vocabulary_aid few_shot reasoning_type
```

### Generate the report

Use the path printed by the runner (or the explicit JSONL path) as report input:

```bash
python -m src.report_benchmark --input results/run_YYYYMMDDTHHMMSSffffffZ.jsonl
```

By default, the Markdown report is saved directly in `reports/` with a matching filename. Choose a custom destination with `--output`. The report contains the accuracy tables, paired-language tests, prompt-versus-standard tests, confidence intervals, and Holm-adjusted p-values.

## Reproducibility record

For each run, retain the JSONL output and note the repository commit, Python version, run date, provider account/deployment configuration, and any rerun decisions. Every request record contains a UTC access timestamp and completion timestamp, configured model/deployment identifiers, discovered version metadata and its UTC check time, endpoint/API settings, temperature and token limit, SDK retry setting, provider response metadata, and data/prompt hashes. The prompt-set version and file hash identify the exact prompt definitions. Provider-returned IDs or version metadata may not disclose private checkpoint/build details.

The active runner uses temperature `0` and a maximum of `2048` output tokens by default. It refuses to overwrite an existing output path; use a fresh path for each run. The code does not automatically retry failed calls.

## Repository map

- `data/`: active bilingual problem datasets and answer keys.
- `prompts/benchmark_prompts.json`: seven prompt conditions, glossary, examples, and reasoning-type labels.
- `src/run_benchmark.py`: model request runner.
- `src/report_benchmark.py`: accuracy and paired statistical report generator.
- `src/math_evaluation.py`: answer extraction and mathematical equivalence checks.
- `results/`: run JSONL files.
- `reports/`: generated Markdown reports.
