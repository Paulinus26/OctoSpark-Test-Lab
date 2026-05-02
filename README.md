# OctoSpark: Real-Time GitHub Issue Triage System

## Overview

In high-volume developer environments, not every issue deserves the same level of urgency—but they often arrive looking the same.

OctoSpark is an automated first-response layer that sits between GitHub Issues and the engineering team. It classifies, prioritizes, and routes incoming issues in real time, so critical failures don’t get buried under routine questions or incomplete reports.

At its core, this project answers a practical question:

> How do you reduce time-to-resolution when most incoming issues are poorly structured or ambiguous?


## 📘 Case Study (Full Build & Debugging Process)

This README summarizes the system design and behavior.

For a step-by-step breakdown—including debugging decisions, architecture evolution, and real test cases:

👉 [View Full Case Study on Notion](https://your-notion-link)


## Problem Context

Support queues are often flat.

A production-breaking `500` error can look no different from a feature request at first glance. Without early triage, critical issues get delayed, and engineers spend time sorting instead of fixing.

OctoSpark treats every issue as a live event that needs immediate analysis, not passive review.


## System Architecture

OctoSpark is built as a real-time pipeline with four layers:

- **Ingestion** – Receives and validates webhook events  
- **Triage** – Analyzes, classifies, and prioritizes issues  
- **Action** – Applies labels, posts responses, and routes issues  
- **Feedback** – Logs activity and supports continuous improvement  

![Architecture Overview](https://raw.githubusercontent.com/Paulinus26/OctoSpark-Test-Lab/9a22068d17248303f5a182594e6aee28db989d13/arch-orchestration.png.png)

**Figure 1: Event-Driven Triage Architecture**


## Implementation

### 1. Foundation: State and Persistence

I used **Supabase** to manage both the runtime and data layer.

A PostgreSQL schema acts as the system’s memory, allowing it to track patterns such as repeated errors and common issue types over time.

This avoids relying on short-lived logs and makes the system easier to evolve.


### 2. Triage Engine (Edge Functions)

The core logic runs in Supabase Edge Functions using a Deno runtime.

Each incoming issue is processed as an event:

- Parse issue title and body  
- Extract error signals (e.g. `500`, `timeout`)  
- Classify issue type  
- Assign priority  

This setup handles bursts of activity without requiring a persistent server.


### 3. Security and Integration

The system integrates with GitHub via webhooks and REST APIs.

During setup, I encountered a `401 Unauthorized` issue caused by a mismatch between default JWT validation and GitHub’s signature-based verification.

To resolve this:

- Disabled gateway-level JWT enforcement  
- Implemented manual webhook signature verification  

This ensures secure and reliable event processing.


### 4. Signal Detection and Triage Logic

Initial testing showed that keyword matching alone was not reliable.

For example, not every “500 error” report is critical—it could be caused by a client-side issue.

To reduce false positives, the system requires supporting signals before escalation, such as:

- Clear failure context  
- Reproducible steps  
- Impact indicators  

![Triage Logic](https://github.com/Paulinus26/OctoSpark-Test-Lab/blob/main/triage-logic.png?raw=true)

**Figure 2: Triage Decision Logic (Signal Detection and Priority Assignment)**

### 5. Automated Response and Routing

When a high-priority issue is detected, the system takes immediate action:

- Apply `high-priority` label  
- Post structured response  
- Request debugging details  
- Route issue to appropriate owner  


## Example: Automated Triage in Action

![Automated Triage Example](./assets/automated-triage-example.png)

**Figure 3: Automated Triage and Response in GitHub Issues**

### Incoming Issue

> Checkout API returns 500 error during payment processing.

### System Behavior

- **Classification:** Bug  
- **Priority:** High  
- **Action:** Label applied + response posted  

### Automated Response
Thanks for reporting this. A 500 error usually indicates a server-side issue.

To help narrow this down quickly, could you confirm:

The API endpoint being called
Whether this started after a recent deployment
If the issue persists across multiple sessions

Since this is affecting checkout, I’ve flagged it as high priority for investigation.



## Result: Faster Escalation

The system reduces manual triage by:

- Identifying high-impact issues early  
- Standardizing issue handling  
- Ensuring critical problems are visible immediately  


## Limitations

- Rule-based signal detection may misclassify edge cases  
- Duplicate issues with different wording are not yet grouped  
- Priority does not account for historical frequency or broader system impact  

These are areas for future improvement.


## Conclusion

OctoSpark focuses on reducing the time between issue creation and meaningful action.

By automating classification, prioritization, and initial response, it allows engineers to focus on resolving issues rather than sorting them.

This project reflects a practical approach to support engineering:
build systems that reduce noise, handle ambiguity, and improve response efficiency.
