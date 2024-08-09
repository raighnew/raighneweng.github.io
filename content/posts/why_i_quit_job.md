+++
title = 'Why I Quit My Job In Two Weeks'
date = '2024-07-09T11:27:00+08:00'
+++

I joined a company as a Senior Backend Developer last month but decided to leave two weeks later. Here’s what I found during those two weeks.

Note: This article is an example of mistakes we may encounter in development. It doesn't contain sensitive information, and I don’t intend to criticize the company or any specific person.

## Why I Made This Decision

### The Awful Boilerplate

The IT department has around 20 people, most of whom are Backend Developers. On my first day, I received read access to the Backend Boilerplate. After opening and reading the `index.ts` file, my first thought was, “This is messed up.”

1. Global Instances and Numerous Environment Codes

    The codebase initializes all class instances in the `index.ts` file and passes those instances into routes/handlers via parameters. Consequently, there are numerous environment initialization codes in the main file. I have concerns about global instances for the following reasons:
    
    - We should avoid having thousands of lines of code in one function. It overloads the mental model and ideally, we should keep each function under 20 lines.

    - Global instances or environments lead to tight coupling. In large projects, you may not know if other team members modify them in the future.

    - It’s better to move all environment checks to the class constructor to improve cohesion, rather than placing them before the creation.

2. No Unit Tests

    I know not all developers enjoy writing unit tests, but if our codebase involves a lot of calculations, unit tests are essential to ensure the code doesn’t break during refactoring.

    - TypeScript provides static type checking, but we still need unit tests to test runtime verification.

    - Ensure at least 90% branch coverage.

3. Too Much Duplicate Code

    I believe in the principle of "Don't Repeat Yourself" (DRY). All repeated code should be consolidated. In this service, communication with internal services happens over the public Internet, resulting in a lot of duplicated authentication code and other redundant elements. I’ll emphasize this part further in the following sections.

### Misunderstanding of Microservices

1. No Abstraction

    To illustrate this, I’ll use a mail service as an example. If we need to integrate with multiple providers to offer a mail-sending feature to our public-facing service, what are the best practices? I believe it’s better to abstract the feature and provide a unified interface to other services, implementing third-party logic through polymorphism. However, this company created a separate microservice for each provider, leading to inconsistencies in DevOps and fragmentation of microservice management.

2. Reliance on One Centralized Service

    Ideally, each microservice should have its own database and be able to operate independently. However, I found that all our microservices rely on a centralized service called `configure-registry`. For example, Server A calls Server B to create an account, then Server A saves account information in `configure-registry`. The next time Server B needs to retrieve Server A’s accounts, it must call `configure-registry`. This setup results in inefficiency and a single point of failure in the system.
    
    - Do not communicate by sharing memory; instead, share memory by communicating.
    

### Outdated Infrastructure

I also discovered several outdated elements within the system. The absence of these key facilities can lead to slow and error-prone deployments, hindering the overall development process:

1. No Staging Environment

    A staging environment is crucial for testing new features and changes in an environment that closely mirrors production. Without it, there’s a higher risk of deploying untested code directly to production, which can lead to unexpected issues and downtime.

2. No CI/CD

    Continuous Integration and Continuous Deployment (CI/CD) pipelines are essential for automating the testing and deployment process. Without CI/CD, the deployment process becomes manual, time-consuming, and prone to errors. This not only slows down the release cycle but also increases the likelihood of bugs making it into production.

3. All Services Communicated Through the Public Internet

    Having services communicate over the public Internet poses security risks and can lead to performance bottlenecks. Ideally, services should communicate over a secure, private network to ensure data integrity and reduce latency. This setup also minimizes the risk of data breaches and other security vulnerabilities.


### When I Made This Decision

As a Senior Backend Developer, I was willing to tackle these issues step by step. I didn’t expect them to be resolved overnight, but I wanted a clear and structured path toward improvement. That’s why I scheduled a 1:1 meeting with my leader, the head of the department, to discuss the concerns and propose a way forward.

However, during the discussion, it became evident that the organization was either unable or unwilling to make the necessary changes. I realized that despite my willingness to help, I would not be able to contribute effectively under these circumstances. Moreover, I wasn’t comfortable writing code that went against my principles and best practices in software development.

Given these factors, I made the decision to leave the company. While it wasn’t an easy choice, I knew it was the right one for my professional integrity and growth.

    - Hope is not strategy.

### How to Avoid This Situation in the Future

1. Know their products before the interview

    When you receive an interview invitation, you can take the following steps to learn more about the company:

    - Check their website, LinkedIn, and Glassdoor reviews.

2. Ask Questions During the Technical Interview

    You can ask direct questions at the end of the interview. Consider the following:

    - what's the current technology stack now.

    - How many Devops and Developers in our team.

    - Who will I be working closely with on this job?

    - How do you monitor the program?

    - How do you test code?

    - What do you do to integrate and deploy code changes? Do you use CI/CD?

    - How do you prepare for failure recovery?

    - How long does it take you to set up a local test environment for your product? (minutes/hours/days)


### References

I recommend the following books and newsletters for further reading; they are excellent resources for a developer's career path:

[The Manager's Path](https://www.amazon.com/Managers-Path-Leaders-Navigating-Growth/dp/1491973897)

[Designing Data-Intensive Applications](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.amazon.sg/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321&ved=2ahUKEwjQ0K_OlueHAxXS2TgGHVtuAckQFnoECEoQAQ&usg=AOvVaw2Nlv8AuOaaLLgtnXpWAFow)

[Gregor Ojstersek from Engineering Leadership](https://newsletter.eng-leadership.com)

[The code reviews in Tencent](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247660387&idx=1&sn=2b507fa0375b8f4fc5c015ac4e4f3fdd)