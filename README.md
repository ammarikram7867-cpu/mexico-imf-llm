# Mexico IMF staff report: LLM extraction, judge, RAG and dashboard

This Colab notebook turns IMF Country Report No. 25/301 into checked summaries, structured JSON, a RAG question-answering step and a six-panel dashboard. The report is *Mexico: Request for an Arrangement Under the Flexible Credit Line and Cancellation of the Current Arrangement - Press Release; and Staff Report*, published 14 November 2025. Coursework for Data Analysis with AI (LLM Coding Case Study), allocation Mexico 2025-2026.

## Run it

1. Open `Mexico_IMF_LLM_Pipeline.ipynb` in Google Colab.
2. Put this repository's URL in `CFG.repo_url` in section 0.2, or upload `data/` and `cache/` to the Colab working folder.
3. Runtime > Run all.

Every model reply is stored in `cache/llm_cache.json`. With the shipped cache, a rerun gives the same numbers and makes no live model calls; section 6.2 prints the count of live calls. To call the models again, delete the cache. The backend is chosen automatically:
- `colab`: Colab's built-in Gemini, no key needed
- `gemini_api`: used when a `GEMINI_API_KEY` Colab secret exists
- `replay`: cache only, set with `MEXICO_LLM=replay`

The recorded run: most replies were made through Colab's built-in Gemini on 3 November 2026. The last seven judge calls hit the Colab AI quota and were made on 4 November through the Gemini API with the same model ids. The shipped notebook is a Restart and run all from that cache: 166 calls, 0 live, 0 errors.

## What it does

| Section | Task | Output in `outputs/json/` |
|---|---|---|
| 1.1-1.4 | pypdf extraction (OCR for the image-only Table 5), cleaning, section detection, 360-token chunks with 50-token overlap | `pages.json`, `clean_text.json`, `sections.json`, `chunks.json` |
| 1.5-1.6 | section summaries under 100 words, judged and revised once if weak; executive summary with a number check | `section_summaries.json`, `executive_summary.json` |
| 2.1 | counts over the 7 financial-stability themes (lexicon), cross-checked by LLM chunk labels | `theme_counts.json`, `theme_chunk_labels.json`, `themes.json` |
| 2.2 | 5-7 strengths and 5-7 vulnerabilities, each with a quote located in the PDF by code | `strengths_vulnerabilities.json` |
| 2.3-2.4 | the six core indicators in the brief's schema, evidence, judge audit and a hand check | `core_indicators.json`, `core_indicators_evidence.json`, `judge_audit.json` |
| 2.5 | time series from Tables 1 and 5, each value verified against the table row | `time_series.json` |
| 3 | all-mpnet-base-v2 embeddings, FAISS `IndexFlatL2`, nine grounded questions, judge scores | `rag_results.json` |
| 4 | extension: system instruction vs generic prompt (generic, persona, strict, pipeline) | `extension_runs.json`, `extension_comparison.json` |
| 5 | `pd.melt` to long format and the dashboard (300 dpi) | `dashboard_data.json`, `outputs/figures/*.png` |
| 6 | every prompt and the call log | `prompts.json`, `call_log.json` |

## Models

| Role | Model |
|---|---|
| Writer | `google/gemini-3.8-flash` |
| Judge | `google/gemini-3.1-pro-preview` |
| Embeddings | `sentence-transformers/all-mpnet-base-v2` |

The writer and the judge are different models, so the judge never marks its own work.

## Files

- `data/mexico_cr25301.pdf`: the report, from the IMF eLibrary (DOI 10.5089/9798229031585.002)
- `cache/`: model replies, OCR text and chunk vectors from the recorded run
- `outputs/`: JSON results and figures
- `requirements.txt`: package versions of the Colab run

Licence: MIT (see `LICENSE`). The IMF report itself is © International Monetary Fund and is included only so the notebook can run.
