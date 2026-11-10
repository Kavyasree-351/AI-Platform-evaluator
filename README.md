# Revolut AI Platform Evaluator

An independent fintech infrastructure prototype for reviewing AI-generated banking responses with a local language model. The dashboard accepts a customer message and assistant response, scores the response against a fictional policy, and displays the verdict and recent audit history.

## Run locally

Install Python 3.10+ and [Ollama](https://ollama.com/download). Disable Ollama cloud features for local-only operation and restart Ollama after changing the setting. See the [Ollama FAQ](https://docs.ollama.com/faq).

From the directory containing app.py, run in PowerShell:

```powershell
ollama pull llama3
# Optional smaller model. Select it in the dashboard after pulling.
# ollama pull qwen2.5:1.5b

python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m streamlit run app.py --server.address 127.0.0.1 --browser.gatherUsageStats false
```

## Audit flow

1. Streamlit accepts two bounded text fields. The judge receives them as untrusted JSON data in a separate message from the policy.
2. The engine posts to Ollama at 127.0.0.1:11434 and requests output constrained by the Pydantic JSON schema.
3. Pydantic rejects extra fields, wrong types, and inconsistent passing verdicts. Incomplete generation, invalid JSON, and inference failures become **Needs review**.
4. The dashboard retains up to 50 verdicts in browser-session state. The application writes no transcript or verdict files. Model-generated summaries may still contain sensitive text, so use synthetic data for this prototype.

| Sample | Expected outcome |
| --- | --- |
| Card freeze | Passed |
| Contradicted savings rate | Flagged with a contradicted claim |
| Simulated credential disclosure | Flagged |
| Safe refusal | Passed |

The fictional Ultra rate is 3.00% variable APY. Unsupported material claims warrant review, while the contradicted-claim field is reserved for claims that conflict with the demo policy.

Schema validation verifies structure and selected consistency rules. Semantic accuracy and resistance to prompt injection depend on the model and require evaluation before deployment. The prototype uses session history and synchronous inference. Production use would require an approved policy source, model calibration, a human review process, and separately designed data-handling controls.

[Ollama structured outputs](https://docs.ollama.com/capabilities/structured-outputs)
