---
title: "CloudWatch Alarms Aren't as Easy as They Seem: The Story of Setting the Greater/Lower Than Condition Backward"
date: 2026-08-04
draft: false
tags: ["aws", "cloudwatch", "monitoring"]
description: "A small but easy-to-make configuration error when creating a CloudWatch Alarm to monitor RDS storage, and how to spot and fix it promptly."
---

## Background

After deploying RDS for the Plantify Co project, I wanted an automated alarm when database disk storage runs low — avoiding situations where the database stops writing data without anyone knowing beforehand. CloudWatch Alarm + SNS (email notifications) was the ideal setup for this.

## Initial Setup — and the First Shock

I created an Alarm to monitor the RDS `FreeStorageSpace` metric, setting the threshold to trigger when free storage dropped below 2GB. A few minutes after enabling it, an email landed in my inbox:

```
ALARM: "Plantify-RDS-LowStorage" in Asia Pacific (Singapore)
Reason: Threshold Crossed: 1.95011616768E10 was greater than the threshold (2.0E9)
```

Translated into plain English: current free storage (~19.5GB) was **greater than** the 2GB threshold — and the Alarm triggered because the condition was set to... **"Greater than"** instead of **"Lower than"**. In other words, I accidentally configured it as: "alert when free space EXCEEDS 2GB" — and since the database almost always has more than 2GB free, the Alarm fired immediately even though nothing was wrong.

## Why This Mistake Is Easy to Make

When creating an Alarm in the CloudWatch Console, the "Conditions" section lets you choose between operators: Greater than, Greater than or equal, Lower than, Lower than or equal. For certain metrics (like high CPU usage being bad → Greater Than is intuitive), but for other metrics (like low free storage being bad → you must use Lower Than), picking the wrong operator is easy to do if you don't pause to think carefully about what the metric actually measures before selecting a condition.

## How to Spot and Fix It

Read the alert email carefully — CloudWatch always clearly details the actual value, threshold set, and operator used in the "Reason for State Change" section. This was where the root cause stood out: the actual value and threshold weren't bad in a real-world sense; the comparison operator was just pointing in the wrong direction.

Back in the Alarm settings, update:
```
Before: Whenever FreeStorageSpace is Greater than 2000000000
After:  Whenever FreeStorageSpace is Lower than 2000000000
```

After saving the fix, the Alarm automatically returned to the **OK** state within a few minutes — confirming that the Alarm functions properly in both directions: triggering when there's an issue and recovering automatically when the issue resolves.

## Key Takeaways

- **Always carefully read the alert email/log content** instead of panicking at the red "ALARM" label — the "Reason for State Change" section always explains precisely what happened.
- **Test both ways before trusting in production**: Don't just verify that the Alarm triggers; make sure it turns off (returns to OK) when the condition resolves.
- For every metric, pause for 5 seconds to ask yourself: "Is a high value good or bad?" before picking Greater/Lower Than — for `FreeStorageSpace`, high is good (plenty of free space) so you alert when **low**; for `CPUUtilization`, high is bad (overloaded) so you alert when **high**. Simple as it sounds, it's remarkably easy to mix up when rushing through the Console.

A small error, easy to fix, but if you don't read the logs closely, it can confuse beginners or — worse — lead them to mute Alarms thinking they're "false alarms," when it was just configured in the wrong direction.