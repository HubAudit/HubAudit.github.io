# HUBAUDIT

- [GitHub Repository](https://github.com/Hubaudit/HubAudit)
- [Paper](./assets/HubAudit-Paper.pdf)
- [Supplementary Material](./assets/HubAudit-Paper-Suplementary.pdf)
- [Website](https://hubaudit.github.io/)

# HUBAUDIT: A Large-Scale Empirical Study of the Security Gap in Artifact Scanning on ML Model Hubs

This is the official repository for HUBAUDIT.

## Overview

Machine learning (ML) model artifacts often contain malicious/vulnerable payloads, code snippets and packages that may jeopardize end-users and system security. Model hubs (e.g., Hugging Face (HF)) rigorously scan ML models (e.g., Pickles) using several mature scanners, but provide little or no support for scanning model artifacts (e.g., loading/inference scripts, configuration files, packages). To address this, we conduct an empirical study to examine the security gap in ML model artifacts scanning between HF artifact scanners and SOTA scanners.

Our empirical study (called HUBAUDIT) combines scanner result clustering, manual analysis, LLM review and dynamic execution. It involves eight security scanners and over half a million scanned (525K) repositories from the top-five most downloaded task categories on HF. HUBAUDIT first collects HF's security report for each repository/artifact from the two HF artifact scanners (VirusTotal and ClamAV). Then, we scan each repository and its artifacts using six SOTA static scanners (Bandit, OpenGrep, Trivy, PySentry, Pip-Audit, ModelAudit). We then score and cluster all security verdicts based on unique scanner outputs. Next, we sample and prioritize the top insecure clusters, and feed the remaining clusters to an LLM (GPT-4o) for further inspection. Finally, we execute and inspect the resulting malicious or vulnerable artifacts to confirm they are real.

Results show that there is a security gap in model artifact scanning on HF and its scanners do not provide sufficient security for model artifacts. We found and confirmed several previously unknown security issues on HF, including **16 previously unknown malicious Python scripts, 30 live leaked secrets and 318 vulnerable packages**. This study motivates the need for model hubs to deploy better artifact scanners such as Bandit and OpenGrep.

## Workflow Diagram
<img alt="Workflow" src="./assets/Workflow-Diagram.png" />

## Datasets
We use Hugging Face as the primary platform. HUBAUDIT crawled 540K model repositories and scanned 525K of them after artifact filtering. The repositories come from the five popular task categories on Hugging Face (Text generation, Text classification, Feature extraction, Image classification, Image segmentation)

Only repository files with relevant artifact extensions (25 file formats, see Table II and Appendix Table XI in the [paper](./assets/HubAudit-Paper.pdf)) are collected for scanning. Across all scanned repositories, the scanners flagged 105,447 files. We also record whether Hugging Face itself flagged each repository as "unsafe" (VirusTotal and ClamAV).

## Artifact location
Other artifacts, including the scanner outputs, flagged-file lists and other results, can be found on [Zenodo](https://zenodo.org/records/23714003?token=eyJhbGciOiJIUzUxMiJ9.eyJpZCI6ImIxNDMwZDU0LWYyMWYtNDc3Ny1iMTRmLTg5MTM0NTNhODVmZCIsImRhdGEiOnt9LCJyYW5kb20iOiIyYjQ5ODc2ZDZmYTM1MmJhNTZjZWRlZDIxMGFiMGU3MSJ9.22dy5nH7UHybMj_wUQu2WR-rebYDO-Pu7x7RlH76-7pT2KjUpjF-TihtL1P0ETXd0p6JV5-PlUvtBzu8UDEGHg).

## Reproducibility
To reproduce HUBAUDIT, follow the repositories in this order (Steps):

1. **Preliminary experiment** - counts the file formats in the repositories of the five task categories on HF, which determines the artifact extensions collected for scanning: [Preliminary-Experiment](https://github.com/HubAudit/Preliminary-Experiment).
2. **HubAudit SOTA Scanner ** - crawls the repositories, and scans each one with the six SOTA scanners. We  collects HF's security report (VirusTotal and ClamAV) for each repository: [HubAudit](https://github.com/HubAudit/HubAudit).
3. **Analysis** - It runs `experiments_and_results/get_results.ipynb` on the scan databases, then you also find the script for the LLM review of the suppressed clusters (`LLM/`), both in the same repository.

The Preliminary experiment scanner outputs are on Zenodo (see above), so step 1 can be run without re-scanning.

Check analysis in `get_results.ipynb` to confirm the results in the paper. It follows the sections of the paper. "3.1 Corpus and detection analysis" gives the total scanned and malicious repositories (Section 3.1), and "Table 1" gives the scanned, malicious, local, HF and overlap counts per task category. "3.3 Overlap" compares the repositories flagged by HF and by the SOTA scanners (Section 3.3), and "3.5 Risk-category classification" gives the risk taxonomy (Section 3.5). The suppressed clusters are reviewed with GPT-4o by running `submit.py`, `batch_fetch.py` and `batch_parse.py` in `experiments_and_results/LLM/`.

## Scanners
We compare the security verdicts of Hugging Face's artifact scanners ([VirusTotal](https://huggingface.co/blog/virustotal), [ClamAV](https://huggingface.co/docs/hub/en/security-malware)) against six state-of-the-art static scanners ([Bandit](https://github.com/PyCQA/bandit), [OpenGrep](https://github.com/opengrep/opengrep), [Trivy](https://github.com/aquasecurity/trivy), [Pip-Audit](https://github.com/pypa/pip-audit), [PySentry](https://github.com/nyudenkov/pysentry), [ModelAudit](https://github.com/promptfoo/modelaudit)); flagged clusters deprioritised during manual inspection were reviewed with an LLM (GPT-4o).
