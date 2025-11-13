# AI Agent for Financial Analysis Challenge

## Overview

This AI Agent is designed to answer high-level financial reasoning questions. It leverages the power of the large language model **Qwen/Qwen3-30B-A3B**, and is divided into two specialized modules:

1. **Multiple-Choice Question Module**
2. **Stock Rise/Fall Prediction Module**

The system is optimized for performance, accuracy, and interpretability, with support for reasoning, and auto-evaluation.

---

## System Architecture

* **Model**: Qwen3-30B-A3B
* **Deployment**: vLLM with tensor parallelism (4-way)
* **Tokenizer**: typhoon21-gemma3-12b
* **Outputs**: Final answers in standardized CSV format

---

## 1. Multiple-Choice Question Module

### Purpose

Handles only multiple-choice questions with answer options:

* A, B, C, or D

### Design

* **Prompt-based Querying**: Uses a structured system prompt that defines the task and constraints.
* **Optional Reasoning Mode**: Enables chain-of-thought generation when enabled.
* **Judging System**: A second LLM pass is used to validate answers:

  * Tags reasoning in `<WHY>...</WHY>`
  * Declares correctness via `<FINAL>TRUE/FALSE</FINAL>`
* **Retry Logic**: If the answer is marked incorrect, feedback is appended and the model retries (up to `MAX_RETRY`).

---

## 2. Stock Rise/Fall Prediction Module

### Purpose

Predicts stock price movement over the next 5 days using feature-engineered inputs and EDA summaries. Answers are limited to:

* Rise
* Fall

### Design

* **Prompt Instruction**: Sets role and format expectations:

  * Requires concise reasoning
  * Demands answer to be wrapped in `<answer>Rise</answer>` or `<answer>Fall</answer>`
* **Statistical Summary Generation**: Each input is derived from a computed summary using the `cal_stat()` function from `helper.py`.
* **Step-by-Step Reasoning**: Prompts include explicit thinking instructions (e.g., "/think Let's think step by step").
* **Token Check**: Token length is monitored to ensure efficiency and compliance.
* **Postprocessing**: Only predictions with valid tag format are accepted; others default to `NaN`.


---

## Key Features

| Feature                     | Description                                                             |
| --------------------------- | ----------------------------------------------------------------------- |
| Modular Design              | Separate flows for multiple-choice and time-series based questions      |
| LLM Reasoning & Judging     | Self-evaluation loop with retry-on-failure                              |
| Token Optimization          | Controls inference length for cost and performance balance              |

---

## `helper.py` Module Overview

This module supports the stock prediction system by preprocessing and analyzing raw text-formatted stock data. Key functions:

* `is_rise_fall_question()` — Detects whether a question pertains to stock prediction.
* `get_stock_df()` — Filters only Rise/Fall stock questions.
* `get_csv()` — Converts textual stock data into structured DataFrame.
* `cal_stat()` — Computes EDA summary, engineered features (e.g., momentum, volatility), target labels, and correlation stats, and returns a prompt-ready formatted string.
