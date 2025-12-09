---
title: "Blog 2: Qwen Models are Now Available in Amazon Bedrock"
date: 2025-09-18
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Overview: Expanding Model Choices with Qwen from Alibaba Cloud

Amazon Bedrock continues to expand its Generative AI library by adding **Qwen3** models from Alibaba Cloud. These are open-weight Foundation Models (FMs) that are fully managed and serverless.

This launch provides developers with more flexible options, from high-performance **Mixture-of-Experts (MoE)** models to cost-optimized **Dense** models, addressing diverse needs from programming and complex reasoning to edge device deployment.

---

## 1. Qwen3 Variants Available on Bedrock

This release includes 4 main model variants, each optimized for specific performance and cost requirements:

1.  **Qwen3-Coder-480B-A35B-Instruct:** A massive MoE model with 480 billion parameters (35 billion activated). Optimized for programming and agentic tasks, handling large codebase analysis effectively.
2.  **Qwen3-Coder-30B-A3B-Instruct:** A more compact MoE model, specialized for code generation, debugging, and following multilingual programming instructions.
3.  **Qwen3-235B-A22B-Instruct-2507:** A balanced MoE model between capability and efficiency, highly competitive in general reasoning and mathematical tasks.
4.  **Qwen3-32B (Dense):** A dense architecture model, suitable for real-time or resource-constrained environments (such as mobile/edge devices) due to stable performance.

---

## 2. Notable Architectural Features

Qwen3 models introduce several important technical improvements that help solve complex enterprise problems.

* **Hybrid Thinking Modes:** Supports two problem-solving modes:
    * *Thinking mode:* Step-by-step reasoning for complex problems.
    * *Non-thinking mode:* Immediate responses for simple tasks, helping optimize costs.
* **Agentic Capabilities & Tool Use:** Models have multi-step planning capabilities and standardized communication with external APIs, ideal for building automated workflows.
* **Ultra-long Context Window:** The Qwen3-Coder line supports context up to **1 million tokens** (through extrapolation methods), allowing processing of entire technical documentation repositories or long conversation histories in a single call.

---

## 3. Access and Integration

Users can immediately experience Qwen models through the **Amazon Bedrock Console** in the Chat/Text Playground area or integrate into applications via AWS SDK.

Notably, Amazon Bedrock has simplified the model access process. Account administrators have full control over enabling/disabling access through **AWS IAM policies** and **Service Control Policies (SCPs)**, ensuring enterprise security compliance without manual activation of individual models.

---

## Conclusion

The addition of Qwen3 to Amazon Bedrock brings tremendous power to software engineers and data scientists. With superior code processing capabilities and flexible architecture, enterprises can:
* Automate code analysis and refactoring workflows.
* Build intelligent AI assistants with optimized inference costs.
* Deploy diversely from cloud to edge devices without changing the management platform.

---

## Author

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/DaniloPoccia.png" alt="Danilo Poccia" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Danilo Poccia</h3>
    <p>Danilo is Chief Evangelist (EMEA) at Amazon Web Services. He works with startups and enterprises of all sizes to support creative innovation. He leverages his experience to help people realize their ideas, focusing on serverless architectures, event-driven programming, and the business impact of Machine Learning and Edge Computing. He is also the author of "AWS Lambda in Action" published by Manning.</p>
  </div>
</div>
