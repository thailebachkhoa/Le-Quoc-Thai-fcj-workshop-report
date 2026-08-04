---
title: "Week 3 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Quickly complete the FE, at minimum the store/product page
* Implement the BE based on the UML diagrams designed earlier
* Practice deploying sample code following the documentation researched earlier

### Tasks to be carried out this week:

| No. | Task | Start Date | Estimated Time |
| :-: | :--- | :-: | :-: |
| **1** | - Complete the rough website (especially the Views in MVC) and demo it on XAMPP <br> - Suggestion: base it on the UI of sample websites combined with AI tools to develop as fast as possible | `22/06/2026` | 3 days |
| **1.1** | - Expand static pages such as FAQ or News | `22/06/2026` | Daily |
| **2** | - Leverage the MVC template from previous assignments, develop the Controller and Core in MVC (BE task) | `25/06/2026` | 3 days |
| **3** | - Learn additional supporting services: AWS IAM, Cognito (for the Google login feature), create an additional Elastic IP, and explore setting up HTTPS via the third-party service DuckDNS | `28/06/2026` | 1 day |

### Week 3 Achievements:

* Got the project's core feature running: product page - shopping cart on localhost.
* Successfully reused MVC source code to speed up project development.

### Limitations:

This is a trade-off resulting from the chosen architecture:

* Scalability is limited (suited for a small-startup customer base — pure sales operations rather than the many business logic layers of big tech).
* System load handling will be weaker (because peak-hour purchase/sale operations that should ideally be handled by Lambda are instead processed by the multi-layered MVC architecture on EC2, causing CPU/RAM overload) — an issue already mentioned in Week 1.