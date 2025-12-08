---
title: "AWS Cloud Mastery Series #2"
date: 2025-11-17
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “DevOps on AWS”

### Event Objectives

* Introduce core DevOps principles and the cultural mindset driving modern software delivery
* Demonstrate how to build automated CI/CD pipelines using AWS services
* Provide guidance on implementing Infrastructure as Code (IaC)
* Compare AWS container platforms for deploying cloud-native applications
* Share best practices for monitoring, observability, and operational visibility

### Speakers

* **Bao Huynh** – AWS Community Builder
* **Thinh Nguyen** – AWS Community Builder
* **Vi Tran** – AWS Community Builder

### Key Highlights

#### Understanding the DevOps Mindset

* Strong collaboration between development and operations enables faster releases
* Automation reduces repetitive work and improves consistency
* Continuous feedback loops lead to more stable and resilient systems

#### CI/CD Pipeline on AWS

A complete automated pipeline was demonstrated across four stages:

* **Source Control**: CodeCommit for storing and versioning code
* **Build & Test**: CodeBuild for compiling, testing, and packaging
* **Deployment**: CodeDeploy supporting rolling, canary, and blue/green strategies
* **Orchestration**: CodePipeline connecting and automating each stage

Live demos showcased how code commits triggered builds, tests, deployments, and even automated rollbacks.

#### Infrastructure as Code (IaC)

Transitioning from manual configuration to consistent, versioned infrastructure.

* **AWS CloudFormation**

  * Declarative YAML/JSON templates
  * Supports parameters, conditions, outputs, and resource definitions
  * Drift detection ensures real-world infrastructure matches the template

* **AWS CDK (Cloud Development Kit)**

  * Create infrastructure using TypeScript, Python, Java, and more
  * L1/L2/L3 constructs provide reusable and opinionated patterns
  * CLI supports synth, diff, and deploy

Examples showed how identical architectures can be reproduced with IaC instead of traditional “ClickOps.”

#### Containers on AWS

Introduction to Docker fundamentals and AWS container compute options:

* **Amazon ECR**: Secure container registry with vulnerability scanning
* **Amazon ECS**: AWS-native container orchestration with EC2 or Fargate
* **Amazon EKS**: Managed Kubernetes for standardized workloads
* **AWS App Runner**: Simplified container hosting with minimal operations

The comparison outlined which service fits best depending on skill level, scalability needs, and application patterns.

#### Observability and Monitoring

Essential practices for ensuring application health:

* **Amazon CloudWatch**

  * Metrics, logs, dashboards, and alarms
* **AWS X-Ray**

  * Distributed tracing across microservices to detect latency and bottlenecks

Emphasis was placed on building useful dashboards, actionable alerts, and proactive monitoring strategies.

### Key Takeaways

#### DevOps Practices

* Automation increases speed and reliability
* Alignment between dev and ops teams is crucial
* Use DORA metrics for continuous improvement
* Incorporate feedback loops throughout development

#### Infrastructure as Code

* Reduce manual configuration in production environments
* CloudFormation offers strong declarative control
* CDK enables flexible, programmatic definitions
* Treat infrastructure like software: test, version, automate

#### Application Delivery

* CI/CD minimizes human error and accelerates releases
* Choose deployment strategies based on risk tolerance
* Automated testing should be integrated into every pipeline stage

#### Container Strategy

* Containers improve portability, consistency, and modularity
* ECS → straightforward operations model
* EKS → Kubernetes ecosystem and flexibility
* App Runner → low-operations approach
* ECR provides the central repository for container images

#### Observability

* Combine logs, metrics, and traces for full visibility
* CloudWatch dashboards + X-Ray service maps simplify troubleshooting
* Build proactive alerting instead of reacting to failures

### Applying to Work

* **Automate CI/CD** using CodePipeline, CodeBuild, and CodeDeploy
* **Implement IaC** with CloudFormation or CDK for repeatable environments
* **Containerize applications** and choose ECS, EKS, or App Runner based on project needs
* **Enhance observability** with CloudWatch metrics, logs, dashboards, and alarms
* **Use AWS X-Ray** for tracing and debugging distributed systems
* **Adopt DORA metrics** to measure delivery performance and guide DevOps improvements

### Event Experience

Attending **“Cloud Mastery Series #2 – DevOps on AWS”** provided both strategic insights and practical knowledge for applying DevOps effectively in cloud environments.

#### Learning from highly skilled speakers

* Clear and detailed explanations of CI/CD, containers, IaC, and monitoring
* Real-world examples showing how DevOps operates in live production systems

#### Hands-on technical exposure

* Observed end-to-end CI/CD from commit → build → deployment
* Learned how CloudFormation and CDK ensure consistent infrastructure
* Understood trade-offs across ECS, EKS, and App Runner

#### Leveraging modern tools

* IaC ensures consistency and eliminates configuration drift
* CloudWatch and X-Ray form the foundation of operational excellence

#### Networking and discussions

* Opportunities to engage with experts and peers
* Discussions underscored the importance of culture, automation, and continuous improvement

#### Lessons learned

* Automation is crucial for safe scaling
* Observability is essential for reliability
* Selecting the right container platform reduces operational load

#### Some event photos

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo1.jpg" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo2.jpg" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo3.jpg" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3</strong>
</p>

> Overall, the workshop provided a clear and practical understanding of DevOps culture, CI/CD workflows, IaC, container deployment, and observability — all critical components of modern cloud-native development.