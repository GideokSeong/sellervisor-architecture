# Automation Engine

## Overview

Automation is rule-based and deterministic.

No machine learning is used for decision making.

---

## Core Signals

- ACOS
- Clicks
- Impressions
- Conversion rate
- Sales (1d / 7d fallback)

---

## Example Logic

Case 1: No conversion

if clicks > threshold AND conversions == 0
→ decrease bid

---

Case 2: High ACOS

if ACOS > target
→ reduce bid

---

Case 3: Good performance but low visibility

if ACOS < target AND impressions low
→ increase bid

---

## Why Rule-Based?

- Predictable behavior
- Easier debugging
- Easier customization per user

---

## Execution

- Runs as background job
- Processes keywords in batches
- Applies updates via API

---

## Challenges

- Avoid over-adjustment
- Handle delayed attribution (1d vs 7d)
