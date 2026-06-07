# Build26 LAB520 – Microsoft AI Foundry Models & Agent Deployment

## Overview

This workshop demonstrates how to use Microsoft AI Foundry to build, test, compare, and deploy AI-powered applications.

The scenario follows **Serena**, a developer at **Zava**, a global home-improvement retailer. Zava receives thousands of customer reviews daily and needs an automated moderation system that can classify reviews before they go live.

Throughout the labs, participants will:

* Explore Microsoft AI Foundry
* Deploy and use hosted AI models
* Build a review moderation application
* Compare multiple AI models
* Deploy the moderation logic as a hosted AI agent

---

# Lab 1 – Explore Microsoft AI Foundry

## Objective

Explore the Microsoft AI Foundry portal and discover hosted models suitable for review moderation.

## Tasks

### Open Microsoft AI Foundry

Navigate to:

https://ai.azure.com

Login with your workshop credentials.

### Explore the Model Catalog

Review:

* OpenAI Models
* Microsoft Models
* Meta Models
* Mistral Models

Examine:

* Publisher
* Task Type
* Deployment Options
* Pricing
* Benchmarks
* Responsible AI Information

### Recommended Models

| Model        | Publisher | Purpose                            |
| ------------ | --------- | ---------------------------------- |
| GPT-4.1-mini | OpenAI    | Fast and cost-effective moderation |
| GPT-4.1      | OpenAI    | Higher quality reasoning           |
| Phi-4        | Microsoft | Strong reasoning, open-weight      |

### Playground Experiment

System Prompt:

```text
You are a product review moderator for Zava, a home-improvement retailer. Classify the following customer review as SAFE, NEEDS_REVIEW, or UNSAFE. Respond with only the classification label.
```

User Prompt:

```text
This paint is garbage and whoever designed it should be fired
```

---

# Lab 2 – Project Setup & Validation

## Open Project

```powershell
File → Open Folder
```

Folder:

```text
C:\Users\LabUser\Desktop\Build26-LAB520-main
```

## Verify Environment File

```powershell
Test-Path .env
```

Required variables:

```env
PROJECT_ENDPOINT=
MODEL_DEPLOYMENT_NAME=
MODEL_DEPLOYMENT_NAME_2=
AZURE_CONTAINER_REGISTRY_NAME=
```

## Validate Setup

```powershell
python -X utf8 src/tests/validate_lab.py
```

Expected:

```text
VALIDATION SUMMARY
Passed: 99
Failed: 0

Result: PASS
```

---

# Lab 3 – First Inference

## Objective

Connect to Microsoft AI Foundry and send the first inference request.

## Run

```powershell
python src/01_first_inference.py
```

## Concepts

### AIProjectClient

Connects to the Foundry project.

### DefaultAzureCredential

Uses Azure login credentials.

### Chat Completion

Sends:

* System Prompt
* User Prompt

Returns:

* Model Response

### Example

```python
response = inference_client.chat.completions.create(
    model=model,
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "What is Microsoft Foundry?"
        }
    ]
)
```

## Understanding Token Usage

Example Output:

```text
Tokens used: 92
(prompt: 53, completion: 39)
```

Meaning:

* Prompt Tokens = Input
* Completion Tokens = Model Output
* Total Tokens = Cost Driver

---

# Lab 4 – Product Review Moderation

## Objective

Build an automated moderation pipeline.

## Moderation Categories

| Classification | Meaning                          |
| -------------- | -------------------------------- |
| SAFE           | Normal feedback                  |
| NEEDS_REVIEW   | Borderline content               |
| UNSAFE         | Threats, harassment, hate speech |

## Moderation Flow

```text
Review
    ↓
GPT-4.1-mini
    ↓
SAFE / NEEDS_REVIEW / UNSAFE
    ↓
Business Logic
    ↓
APPROVED / REVIEW / BLOCKED
```

## Run

```powershell
python src/02_comment_moderation.py
```

## Interactive Mode

```powershell
python src/02_comment_moderation.py --interactive
```

## Dataset Mode

```powershell
python src/02_comment_moderation.py --file src/sample_comments.json
```

## Business Logic

```python
if classification == "SAFE" and confidence >= 0.8:
    return "APPROVED"

elif classification == "UNSAFE" and confidence >= 0.7:
    return "BLOCKED"

else:
    return "FLAGGED_FOR_REVIEW"
```

## Key Design Decisions

| Decision          | Purpose                    |
| ----------------- | -------------------------- |
| temperature=0.0   | Deterministic output       |
| JSON output       | Easy parsing               |
| Structured prompt | Consistent classifications |
| Error handling    | Safe fallback behavior     |

---

# Unit Testing

Run:

```powershell
pytest src/tests/test_moderation.py -v
```

Validates:

* SAFE approvals
* UNSAFE blocking
* Threshold behavior
* Missing fields
* NEEDS_REVIEW routing

---

# Lab 5 – Model Comparison

## Objective

Compare GPT-4.1-mini and GPT-4.1 on the same moderation task.

## Deploy Second Model

```powershell
az cognitiveservices account deployment create `
  --name <your-foundry-resource-name> `
  --resource-group <your-resource-group> `
  --deployment-name gpt-4.1 `
  --model-name gpt-4.1 `
  --model-version "2025-04-14" `
  --model-format OpenAI `
  --sku-capacity 10 `
  --sku-name "GlobalStandard"
```

## Update .env

```env
MODEL_DEPLOYMENT_NAME=gpt-4.1-mini
MODEL_DEPLOYMENT_NAME_2=gpt-4.1
```

## Run

```powershell
python src/03_model_comparison.py
```

## Compare

* Classification
* Confidence
* Latency
* Cost

## Cost Comparison

| Model        | Input / 1M Tokens | Output / 1M Tokens |
| ------------ | ----------------: | -----------------: |
| GPT-4.1-mini |             $0.15 |              $0.60 |
| GPT-4.1      |             $2.50 |             $10.00 |

### Key Insight

GPT-4.1-mini often provides similar moderation quality at a fraction of the cost.

---

# Lab 6 – Deploy as Hosted Agent

## Objective

Deploy the moderation application as a Microsoft Foundry Hosted Agent.

## Architecture

```text
Local Code
    ↓
Docker Image
    ↓
Azure Container Registry
    ↓
Foundry Agent Service
    ↓
GPT-4.1-mini
```

## Initialize Agent

```powershell
azd ai agent init `
    --project-id "<foundry-project-resource-id>" `
    --model-deployment gpt-4.1-mini `
    --protocol responses `
    --src src/agent
```

## Run Locally

```powershell
cd src/agent
python app.py
```

## Deploy

```powershell
azd config set tool.firstRunCompleted true
azd up
```

## Check Status

```powershell
azd ai agent show --output table
```

## Invoke

```powershell
azd ai agent invoke "Love this cordless drill!"
```

Expected:

```json
{
  "classification": "SAFE",
  "confidence": 1.0,
  "reason": "Positive and constructive product feedback."
}
```

## Monitor

```powershell
azd ai agent monitor
```

## Cleanup

```powershell
azd down
```

---

# Key Takeaways

✅ Discover and deploy AI models from Microsoft AI Foundry

✅ Connect to hosted models using Python

✅ Understand token usage and cost estimation

✅ Build a structured moderation pipeline

✅ Apply business rules on top of model outputs

✅ Compare model quality, latency, and cost

✅ Deploy AI applications as hosted Foundry Agents

✅ Monitor and manage production AI workloads

---

## Technologies Used

* Microsoft AI Foundry
* Azure OpenAI
* GPT-4.1-mini
* GPT-4.1
* Azure Developer CLI (azd)
* Python
* Docker
* Azure Container Registry
* Microsoft Agent Framework
* Azure Identity
* Pytest

---

## Author

Built as part of the Microsoft Build 2026 LAB520 workshop on AI Foundry Models and Hosted Agents.
