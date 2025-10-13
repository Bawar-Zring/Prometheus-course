# Alerts & Notifications (Prometheus)

This section explains how alerts and notifications work in Prometheus and Alertmanager. It covers alerting rules, the flow from Prometheus to notification receivers, Alertmanager responsibilities and best practices.

---

## Overview

Alerts help you detect and respond to problems in your systems. Prometheus evaluates alerting rules against collected metrics. When a rule's condition is met, Prometheus sends an alert to Alertmanager. Alertmanager forwards alerts to configured contact points such as email, Slack, Microsoft Teams, or custom webhooks.

High-level flow:
1. Prometheus evaluates alerting rules.
2. When a condition is true, Prometheus fires an alert.
3. Prometheus sends the alert to Alertmanager.
4. Alertmanager sends notifications to configured receivers.
5. Receivers deliver notifications (email, Slack, Microsoft Teams, webhook, etc.).

---

## Why Alertmanager is important

Alertmanager is the central component that takes care of:
- Deduplication: Prevents multiple identical alerts from causing repeated notifications.
- Routing: Sends alerts to different receivers based on labels and routing rules.
- Inhibition: Suppresses lower-priority alerts when a higher-priority alert is firing (for example, suppress disk-space warnings when a node is down).
- Silences: Temporarily mute alerts (for maintenance windows or known incidents).
- Templates: Customize notification messages.

Without Alertmanager, Prometheus itself would only generate alerts but not reliably deliver them in a controlled, grouped, or deduplicated fashion.

