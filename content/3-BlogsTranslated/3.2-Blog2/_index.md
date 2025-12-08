---
title: "Blog 2"
date: "2025-10-08"
weight: 2 # Sửa từ 1 sang 2
chapter: false
pre: " <b> 3.2. </b> " # Sửa từ 3.3. sang 3.2.
---
# **CrazyGames upgrades platform with real-time friends system using AWS AppSync**

**by Emily McKinzie | on 08 OCT 2024 | in [Amazon EC2](#), [Amazon MemoryDB](#), [AWS AppSync](#), [Compute](#), [Customer Solutions](#), [Database](#), [Front-End Web & Mobile](#), [Game Development](#), [Industries](#), [Top Posts](#) | [Permalink](#) | [Comments](#) | [Share](#)**

---

![CrazyGames Gamer](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2024/10/01/CrazyGames_Gamer.jpg)

Multiplayer games platform **CrazyGames** engages more than 35 million players around the world with browser-based titles such as *Ludo King* and *Paper Delivery Boy* in 24 different languages. Whether playing through a desktop or mobile device, all gamers can now enjoy an enhanced social experience with a new real-time friends system built using **Amazon Web Services (AWS) AppSync**.

CrazyGames has been all-in on AWS since day one, running on a single **Amazon Elastic Compute Cloud (Amazon EC2)** instance when it was initially built in 2014. Since then, the platform has grown organically and now hosts more than 3,000 games spanning different genres. Reflecting on the role AWS has played in the company’s continued evolution, CrazyGames Founder and CEO Raf Mertens noted:

> “AWS offers a wide range of highly available, scalable services that we can try, test and use effortlessly within a very short period of time. By using AWS AppSync, we reduced our development time from eight months to eight weeks.”

### Boosting the social experience

Playing games with friends is infinitely more fun than playing solo and can help players level up their skills. Thankfully, social games provide opportunities to forge new friendships with people who share similar interests while also reinforcing existing bonds. Multiplayer games with strong social components also provide benefits. They tend to better engage players, increase long-term retention, and draw in new users as players invite their friends.

Until recently, CrazyGames visitors couldn’t always easily play with friends. They faced a disjointed experience, which inspired the company to design and implement its new friends system. Now, users can befriend one another, see when their friends are online, view the games they are playing, and send game invites in real time.

To develop the friends feature, CrazyGames used the ‘shape up’ methodology, which entailed establishing a small, dedicated team solely focused on creation of a production-ready friends system. The developers built the new functionality using **AWS AppSync**, which coordinates the status of multiple users in near real time. Players can see right away when their friends log on or off.

At the start of development, the team also investigated an open-source framework and a custom solution built using the WebSocket API before selecting AWS AppSync.

> “Users today expect immediate gratification, so real-time operations are essential. AWS AppSync emerged as the ideal solution for our friend system. It required less development effort due to its abstraction over WebSocket API, was more cost-effective compared to other options, and seamlessly integrated with our existing GraphQL client,” explained Mertens. “AWS AppSync covered all we needed, so the setup was effortless, and it’s continued to perform well as it handles tens of thousands of simultaneous users with real-time status updates.”

In addition to AWS AppSync, CrazyGames uses **Amazon Aurora** for managing long-lived data, such as notifications, in the new friend system. For more transient data that requires rapid updates, like invites and user statuses, the company uses **Redis Cloud on AWS**, with **Amazon MemoryDB**.

![CrazyGames Devices](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2024/10/01/CrazyGames_Devices.jpg)

### Building with AWS

With a decade of history working with AWS, CrazyGames has evolved its technology ecosystem to include a wide range of AWS solutions and services. This helps its team focus on software development instead of building, configuring, and maintaining servers. Last year, the company implemented a new analytics system to gain additional insight into user behaviors and needs. The analytics system efficiently tracks millions of daily events across its platform’s more than two million daily users as they interact with complex features.

> “Most of the AWS services we’ve chosen handle operational responsibilities like availability and maintenance. This means our team can spend more time on software development,” said Mertens. “Moreover, our platform is subject to traffic spikes, and since our compute power is scalable, these surges are automatically absorbed by the backend without downtime or latency.”

Currently, CrazyGames has integrated the new friends system within 30 games on their platform, ranging from first-person shooters such as *Kour.io* to classics such as *8 Ball Pool Billiards*. As they plan to roll the friend system out more widely in the next 12 months, the company is looking to streamline the onboarding experience. Mertens shared, “Once users know the new friend feature exists, they really love it, but we still have to guide them through specific steps to get started. We want to make that process easier for them.”

### Planning for the next new feature

By leveraging AWS, CrazyGames developers can quickly and cost-efficiently experience new features, helping the company to maintain its reputation as a top destination for engaging, web-based social games. Mertens concluded:

> “With AWS managed services, we can build more features with our time, instead of dealing with infrastructure management. Plus, there’s a huge risk reduction. You can experiment and get new features live faster to figure out whether they work, instead of wasting engineering resources.”

Learn more about building entertaining, scalable experiences with speed and flexibility.
Contact an [AWS Representative](#) to know how we can help accelerate your business.

---



<img src="https://d2908q01vomqb2.cloudfront.net/fb644351560d8296fe6da332236b1f8d61b2828a/2023/05/18/EmilyPhoto.jpg" width="150" alt="Emily McKinzie">

**Emily McKinzie**
*Emily McKinzie is an Industry Marketing Manager at Amazon Web Services.*