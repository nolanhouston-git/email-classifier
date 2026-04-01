# AI Email Classification System

A production email monitoring and triage system built on AWS Bedrock that autonomously classifies incoming messages by urgency, generates daily HTML summaries, and delivers prioritized action lists to a team of analysts. Built to eliminate a recurring manual workflow that was consuming 2+ hours per analyst per day.

This repository contains architectural documentation only. The original implementation was built during employment and cannot be shared publicly.

---

## The Problem

A sales operations team of 12 analysts was managing high-volume inboxes manually. Each analyst spent 2 or more hours per day scanning email, deciding what required action, and determining what could be deferred or ignored. The process was entirely manual, inconsistent across analysts, and consumed time that should have been spent on analytical work.

There was no system. There was no prioritization logic. There was no way to ensure important messages were caught before end of day.

---

## What the System Does

The system connects to Outlook via the Microsoft Graph API, pulls unread messages on a schedule, classifies each message using AWS Bedrock, and generates a structured daily HTML summary delivered to each analyst. The summary organizes messages into three tiers: action required, informational, and low priority.

It runs automatically. Analysts receive their prioritized inbox summary without doing anything.

### Classification Architecture

Each email is passed to a classification prompt via AWS Bedrock with the following context:

- Sender, subject, and message body
- Historical classification patterns for that sender (learning component)
- Defined classification criteria for the three tiers

Bedrock returns a classification and confidence score. The system uses the classification to route the message into the appropriate summary tier. The learning component refines classifications over time based on analyst behavior — messages that analysts consistently override are used to improve future classifications for that sender.

### Daily Summary

The HTML summary is generated once per day and uploaded to S3. Each summary includes:

- Action-required messages with direct links to the original email
- Informational messages grouped by sender or thread
- Low-priority messages collapsed by default
- A count of total messages processed and classification breakdown

### Infrastructure

| Component | Technology |
|---|---|
| Language | Python |
| AI Classification | AWS Bedrock |
| Email Access | Microsoft Graph API (Outlook) |
| Credential Management | AWS Secrets Manager |
| Output Storage | Amazon S3 |
| Scheduling | Task Scheduler / cron |
| Execution | Fully automated — no manual steps |

---

## Outcomes

- **Reduced daily email triage from 2+ hours to 30 minutes** per analyst across a team of 12
- Consistent prioritization logic applied across all analysts — no more missed messages or inconsistent handling
- Analysts shifted time from inbox management to analytical work
- Learning component improved classification accuracy over time without manual retraining

---

## Why Bedrock

AWS Bedrock was the right choice for this deployment context for two reasons. First, the team was operating entirely within the AWS ecosystem — Redshift, S3, Secrets Manager, and internal tooling were all AWS-native. Adding an external API dependency for a production workflow would have introduced unnecessary complexity and credential management overhead. Second, Bedrock's managed inference meant no model hosting, no scaling concerns, and no infrastructure to maintain. The classification logic lives in the prompt and the Python wrapper. Everything else is managed.

---

## What I Would Do Differently

The learning component was implemented as a simple override tracker — if an analyst reclassified a message, that signal was logged and used to adjust future classifications for that sender. A more robust approach would treat this as a proper feedback loop: collecting override signals, periodically fine-tuning the classification prompt with high-confidence examples, and surfacing drift when classification accuracy drops below a threshold. The foundation was there; the feedback loop was not fully closed.

---

## Notes

This system was built and deployed during employment. No proprietary code, data, or confidential business information is included in this repository. Architecture and outcomes are documented from memory and public-facing deliverables only.
