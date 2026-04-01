<h1 align="center">λ-RLM — The Y-Combinator for LLMs</h1>

<p align="center">
  <a href="https://arxiv.org/abs/2603.20105">
    <img src="https://img.shields.io/badge/arXiv-2603.20105-b31b1b.svg" alt="arXiv">
  </a>
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT">
  </a>
  <a href="#quickstart">
    <img src="https://img.shields.io/badge/Python-3.11%2B-blue.svg" alt="Python 3.11+">
  </a>
  <a href="https://github.com/lambda-calculus-LLM/lambda-RLM/stargazers">
    <img src="https://img.shields.io/github/stars/lambda-calculus-LLM/lambda-RLM?style=social" alt="GitHub stars">
  </a>
</p>

<p align="center">
  <b>Typed recursive long-context reasoning for LLMs.</b>
</p>

<p align="center">
  λ-RLM replaces free-form recursive code generation with a typed functional runtime grounded in λ-calculus.
</p>

<p align="center">
  <a href="https://github.com/lambda-calculus-LLM/lambda-RLM">
    <img src="https://img.shields.io/badge/Star%20this%20repo-yellow?style=for-the-badge" alt="Star this repo">
  </a>
  <a href="https://arxiv.org/abs/2603.20105">
    <img src="https://img.shields.io/badge/Read%20the%20paper-b31b1b?style=for-the-badge" alt="Read the paper">
  </a>
</p>

<br>
<p align="center">
  <img src="banner.png" alt="Lambda-RLM: Typed Recursive Reasoning for Long-Context LLMs" width="1000">
</p>

## Why use λ-RLMs?

λ-RLM is a framework for long-context reasoning that replaces **free-form recursive code generation** with a **typed functional runtime** grounded in **λ-calculus**.  
Instead of letting the model write arbitrary recursive control logic during execution, λ-RLM executes a compact library of **pre-verified combinators** and uses neural inference only on **bounded leaf subproblems**.

> More reliable recursive reasoning. More predictable compute. Stronger formal structure.

Across weak, medium, and strong model families, λ-RLM improves accuracy over standard RLM while substantially reducing latency.

<p align="center">
  <img src="intro.png" alt="Lambda-RLM overview and results" width="900"/>
</p>

## Why this repo matters?

Standard direct LLM inference is limited by the context window.  
Standard Recursive Language Models (RLMs) go further, but often rely on **open-ended REPL-based recursive code generation**, which is powerful yet difficult to verify, predict, and analyse.

λ-RLM takes a different route:

- **deterministic recursive decomposition**
- **typed symbolic control flow**
- **bounded model calls at leaf subproblems**
- **structured composition through functional operators**
- **formal guarantees absent from standard RLMs**

## Highlights

- **29 / 36** wins over standard RLM across model-task comparisons
- **Up to +21.9** average accuracy points across model tiers
- **Up to 4.1×** lower latency
- Formal guarantees including:
  - **termination**
  - **closed-form cost bounds**
  - **controlled accuracy scaling with recursion depth**
  - an **optimal partition rule** under a simple cost model

## Intuition

The key idea is simple:

- break a long reasoning problem into smaller pieces
- solve only the bounded leaf subproblems with the LLM
- combine intermediate results using a fixed library of symbolic operators

This turns recursive reasoning from an unconstrained agentic loop into a **structured functional program with explicit control flow**.

Instead of relying on arbitrary generated recursion, λ-RLM uses operators such as:

- `SPLIT`
- `MAP`
- `FILTER`
- `REDUCE`
- `CONCAT`
- `CROSS`

These operators let the system process long inputs compositionally while keeping individual model calls local and manageable.


## What makes λ-RLM different?

### Standard RLM
Standard RLM-style systems often:
- generate recursive code on the fly
- use a REPL-style execution loop
- make decomposition strategies harder to predict
- offer less formal structure for analysis

### λ-RLM
λ-RLM instead:
- plans decomposition ahead of execution
- uses a typed functional runtime
- executes deterministic recursive structure
- restricts neural inference to bounded leaves
- composes results symbolically

In short:

> **standard RLMs use generated control code**  
> **λ-RLM uses typed functional control**

## Installation and Setup

From the project root (the directory containing pyproject.toml):

```bash
conda create -n lambda-rlm python=3.11 -y
conda activate lambda-rlm

pip install -e .
```
The project supports multiple API-compatible model providers. For example, you can request a [NVIDIA NIM API key](https://build.nvidia.com) or a [TOGETHER AI API key](https://api.together.ai/) to access the available model backends. Set your API key as an environment variable:

```bash
export NVIDIA_API_KEY="nvapi-..."
```

```bash
export TOGETHER_API_KEY="tgp_..."
```

### Supported datasets:
- `sniah` — Sequential-NIAH examples loaded from the public GitHub JSONL source
- `oolong` — single-document QA examples loaded from `THUDM/LongBench-v2`
- `browsecomp` — multi-document QA examples loaded from `THUDM/LongBench-v2`
- `codeqa` — code repository understanding examples loaded from a local JSONL file or from `THUDM/LongBench-v2`

## Usage

### Quick Start

```python
import os
from rlm import LambdaRLM

document = """
This report discusses the development of a new battery technology.
It covers technical design choices, manufacturing trade-offs, safety concerns,
cost reduction strategies, and future commercialization plans.

One section focuses on performance improvements in energy density.
Another section discusses supply-chain risks and regulatory constraints.
The final section outlines expected market impact and open research questions.
"""

prompt = f"""Context:
{document}

Question: Summarize the main ideas discussed in this document.

Answer:"""

rlm = LambdaRLM(
    backend_kwargs={
        "model_name": "meta/llama-3.3-70b-instruct",
        "api_key": os.environ["NVIDIA_API_KEY"],
        "base_url": "https://integrate.api.nvidia.com/v1",
    }
)

result = rlm.completion(prompt)
print(result.response)
```

## Repository structure
### Normal RLM

This repository uses upstream Normal RLM components for comparison: `https://github.com/alexzhang13/rlm`

The upstream code is licensed under the MIT License. See `THIRD_PARTY_NOTICES.md` for attribution and licensing details.

Key files:
- `rlm/core/rlm.py` — main REPL-based RLM loop
- `rlm/environments/local_repl.py` — sandboxed Python REPL execution, context storage, and helper functions
- `rlm/utils/parsing.py` — parsing of ```repl``` code blocks and FINAL markers; formatting of execution output back into the model history
- `rlm/clients/openai.py` — OpenAI-compatible client used with NVIDIA NIM

### $\lambda$-RLM

- `rlm/lambda_rlm.py` — LambdaRLM implementation, including task detection, planning, and deterministic execution through $\Phi$

## Benchmarking
The benchmark entry point is used to run the supported datasets under the same setup and compare behavior, latency and output quality across Normal RLM (`rlm`) and Lambda-RLM (`lambda_rlm`).


### Compare both methods on the same dataset

```bash
python benchmarks/benchmark.py --datasets sniah --model meta/llama-3.3-70b-instruct --methods rlm lambda_rlm --n-samples-per-bucket 2 --max-iter 8 --max-depth 2 --context-window 100000 --output-dir ./results/llama-3.3-70b-instruct
```

Outputs are written to the specified output directory, typically including:
- `results.json`
- `stats.json`
- `averages.json`

### Run only Normal RLM

```bash
python benchmarks/benchmark.py --datasets sniah --model meta/llama-3.3-70b-instruct --methods rlm --n-samples-per-bucket 2 --max-iter 8 --max-depth 2 --context-window 100000 --output-dir ./results/llama-3.3-70b-instruct_rlm
```

### Run only $\lambda$-RLM

```bash
python benchmarks/benchmark.py --datasets sniah --model meta/llama-3.3-70b-instruct --methods lambda_rlm --n-samples-per-bucket 2 --max-iter 8 --max-depth 2 --context-window 100000 --output-dir ./results/llama-3.3-70b-instruct_lambda_rlm
```
