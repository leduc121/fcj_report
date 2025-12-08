---
title: "Week 6 Worklog"
date: "2024-10-13"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Learn to manage EC2 access using resource tags through IAM
* Understand cost optimization with Lambda automation
* Implement automated resource management

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 1   | - Learn about resource tagging: <br>&emsp; + Tag best practices <br>&emsp; + Tag policies <br>&emsp; + Cost allocation tags <br>&emsp; + Tag-based access control | 13/10/2024 | 13/10/2024      | <https://000028.awsstudygroup.com/> |
| 2   | - Learn IAM policies with tags: <br>&emsp; + Condition keys for tags <br>&emsp; + Tag-based permissions <br>&emsp; + Resource-level permissions <br> - **Practice:** Create tag-based IAM policies | 14/10/2024 | 14/10/2024      | <https://000028.awsstudygroup.com/> |
| 3   | - **Practice:** Manage EC2 access with tags: <br>&emsp; + Tag EC2 instances <br>&emsp; + Create IAM policies based on tags <br>&emsp; + Test access control <br>&emsp; + Verify permissions | 15/10/2024 | 15/10/2024      | <https://000028.awsstudygroup.com/> |
| 4   | - Learn Lambda for cost optimization: <br>&emsp; + Lambda basics <br>&emsp; + Event-driven automation <br>&emsp; + CloudWatch Events/EventBridge <br>&emsp; + Lambda with EC2 API | 16/10/2024 | 16/10/2024      | AWS Lambda Documentation |
| 5   | - **Practice:** Optimize EC2 costs with Lambda: <br>&emsp; + Create Lambda function to stop idle instances <br>&emsp; + Schedule Lambda with EventBridge <br>&emsp; + Monitor and test automation <br>&emsp; + Calculate cost savings | 17/10/2024 | 18/10/2024      | AWS Lambda Documentation |

### Week 6 Achievements:

* **Resource Tagging:**
  * Mastered AWS tagging best practices
  * Understood tag naming conventions and strategies
  * Learned about mandatory tags and tag policies
  * Implemented cost allocation tags for billing
  * Created comprehensive tagging strategy for organization

* **IAM Tag-Based Access Control:**
  * Learned to use condition keys in IAM policies (aws:RequestTag, aws:ResourceTag)
  * Created policies that grant access based on resource tags
  * Implemented attribute-based access control (ABAC)
  * Successfully managed EC2 access using resource tags
  * Tested and verified tag-based permissions

* **Lambda Automation:**
  * Gained understanding of AWS Lambda serverless computing
  * Learned event-driven architecture patterns
  * Mastered Lambda function creation and deployment
  * Understood Lambda execution roles and permissions

* **Cost Optimization with Lambda:**
  * Created Lambda functions to identify and stop idle EC2 instances
  * Implemented automated scheduling with EventBridge
  * Set up notifications for automated actions
  * Monitored Lambda execution and costs
  * Calculated actual cost savings from automation

* **Automation Best Practices:**
  * Learned to implement safe automation with proper error handling
  * Understood the importance of testing automation in non-production
  * Implemented logging and monitoring for automated tasks
  * Created documentation for automated processes
