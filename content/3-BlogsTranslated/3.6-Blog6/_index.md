---
title: "Blog 6: Top 10 AWS Cloud Operations Announcements at re:Invent 2025"
linktitle: "Blog 6: Top 10 AWS Cloud Operations Announcements at re:Invent 2025"
date: 2025-01-01
weight: 6
chapter: false
pre: " <b> 3.6. </b> "
---

# Overview: Reshaping Cloud Operations with AI

At re:Invent 2025, AWS announced a series of significant improvements for Cloud Operations. This year's focus revolves around applying **Generative AI** for observability and troubleshooting, while simplifying the management of large-scale operational data through centralization features.

Below are detailed notes on the top 10 announcements that help optimize operational processes on AWS.

---

## 1. Generative AI Observability on CloudWatch and AgentCore

AWS launched comprehensive observability capabilities for Generative AI applications. This feature provides detailed insights into latency, token usage, and errors occurring in the AI stack. Notably, it integrates seamlessly with **Amazon Bedrock AgentCore** and open-source frameworks like LangChain or CrewAI without requiring manual instrumentation code.

![Amazon CloudWatch Generative AI dashboard](/images/3-BlogsTranslated/6.1.png)

> [Figure 1] Amazon CloudWatch Generative AI dashboard

The Agent Management View also helps track the agent's workflow from end-to-end:

![Agent Management View](/images/3-BlogsTranslated/6.2.png)

> [Figure 2] Agent Management View

---

## 2. CloudWatch Application Map with Automatic Service Discovery

Previously, creating an Application Map often required complex instrumentation setup. Now, **CloudWatch Application Signals** can automatically detect and visualize application topology, displaying service dependencies instantly. This feature helps operations teams gain a system overview more quickly.

![CloudWatch Application Map](/images/3-BlogsTranslated/6.3.png)

> [Figure 3] CloudWatch Application Map

---

## 3. Root Cause Analysis (5 "Whys") and Incident Report Generation

This is a major advancement in **AIOps**. CloudWatch Investigations leverages Generative AI to automate the root cause analysis process. Instead of manually aggregating data, the system generates an interactive incident report.

![Amazon CloudWatch Investigations Incident Report](/images/3-BlogsTranslated/6.4.png)

> [Figure 4] Amazon CloudWatch Investigations Incident Report

Most notably, the **"5 Whys"** analysis process is built-in, simulating Amazon's internal Correction of Errors (COE) methodology, helping to systematically identify the underlying causes of issues.

![5 Whys Analysis in the CloudWatch investigations Incident Report](/images/3-BlogsTranslated/6.5.png)

> [Figure 5] 5 Whys Analysis in the CloudWatch investigations Incident Report

---

## 4. Model Context Protocol (MCP) Servers Support

CloudWatch and Application Signals now support **Model Context Protocol (MCP)** servers. This acts as a bridge, allowing AI assistants to naturally interact with observability data (metrics, logs, traces). This enables us to build autonomous operational workflows and integrate CloudWatch data into AI-powered development tools.

---

## 5. GitHub Action Integration and MCP Improvements for Application Signals

To better support developers, CloudWatch Application Signals has been integrated directly into **GitHub Actions**. This feature provides observability insights right within Pull Requests and CI/CD pipelines.

It helps identify performance issues or system errors without leaving the GitHub development environment:

![Automated Root Cause Analysis in the GitHub issues](/images/3-BlogsTranslated/6.6.png)

> [Figure 6] Automated Root Cause Analysis in the GitHub issues

The system can even suggest automated bug fixes through Pull Requests:

![Automated GitHub Pull Request to Fix the Issue](/images/3-BlogsTranslated/6.7.png)

> [Figure 7] Automated GitHub Pull Request to Fix the Issue

---

## 6. Enhanced Log Analytics Experience on OpenSearch Service

Amazon OpenSearch Service introduces significant improvements for **Piped Processing Language (PPL)**. Log analysis becomes faster and more intuitive. Enhanced query capabilities help process complex analytical queries efficiently, while seamlessly integrating with CloudWatch Logs for unified log analysis.

---

## 7. Real User Monitoring (RUM) Support for Mobile (iOS/Android)

Amazon CloudWatch RUM extends real user experience monitoring to mobile platforms. We can now track performance, user journeys, and client-side errors on **iOS and Android** applications, ensuring consistent experiences across all devices and geographic locations.

---

## 8. CloudTrail: Data Event Aggregation

To address the massive volume of logs from API activity, AWS CloudTrail adds the **Event Aggregation** feature. Instead of logging each individual entry, the system summarizes high-frequency activities into aggregated reports every 5 minutes.

This helps:
* Reduce log storage costs.
* Easily detect anomalous patterns (such as unusual S3 access or DynamoDB throttling) without manually analyzing too much raw data.

---

## 9. Multi-Account & Multi-Region Log Centralization

This is a highly anticipated feature for large organizations. **CloudWatch Logs Centralization** allows collecting logs from multiple Accounts and Regions into a single destination account.
* Integrates with AWS Organizations.
* Automatically adds `@aws.account` and `@aws.region` context to logs for easy source tracking.
* Cost savings (no ingestion fees for the first copy).

---

## 10. Multi-Account & Multi-Region Centralized Database Monitoring

Similar to logs, **CloudWatch Database Insights** also supports centralized monitoring. We can track the performance of Amazon RDS, Amazon Aurora, and Amazon DynamoDB across the entire AWS organization from a single monitoring account. This makes it easier to correlate database performance with application health.

---

## Conclusion

The 2025 announcements show that AWS is focused on solving the "operational data overload" problem by using AI to filter noise and automate analysis. From monitoring GenAI, to using GenAI to fix errors, and centralization capabilities, these tools help operators shift from a reactive state to proactively controlling systems.

---

## Authors

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/NereidaWoo.png" alt="Nereida Woo" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Nereida Woo</h3>
    <p>Nereida is a WW Specialist Solutions Architect in Cloud Operations focusing on Centralized Operations Management and Application operations on AWS. When she isn't working, she enjoys traveling to attend music concerts.</p>
  </div>
</div>

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/CalvinWeng.png" alt="Calvin Weng" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Calvin Weng</h3>
    <p>Calvin Weng is a Product Marketing Manager for AWS Cloud Operations, focusing on observability and monitoring services. Outside of work, Calvin travels, practices pottery, plays ping pong competitively, and explores the Pacific Northwest with his dog Kai.</p>
  </div>
</div>

<div style="display: flex; align-items: flex-start; margin-bottom: 30px; padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
  <img src="/images/3-BlogsTranslated/RavitejaSunkavalli.png" alt="Raviteja Sunkavalli" style="width: 150px; height: 150px; object-fit: cover; margin-right: 20px; border-radius: 5px;">
  <div>
    <h3 style="margin-top: 0;">Raviteja Sunkavalli</h3>
    <p>Raviteja Sunkavalli is a Senior Worldwide Specialist Solutions Architect at Amazon Web Services, specializing in AIOps and GenAI observability. He helps global customers implement observability and incident management solutions across complex and distributed cloud environments. Outside of work, he enjoys playing cricket and exploring new cooking recipes.</p>
  </div>
</div>
