---
layout: post
title: Behavior Driven Canary Deployments
comments: true
description: Using BDD techniques to drive canary release validation and automation
---

The concept of [canary deployments is not new](https://martinfowler.com/bliki/CanaryRelease.html). At a high level, this method of deployment involves rolling out a new version to a small subset of users in order to minimize the potential impact of a bad build. When the canary is determined to be healthy, it is promoted and released to all users. If the canary is bad, it is removed from service and users are reverted to the previous stable version. Observability and automatability are two important factors involved in building a system that can be canary-ed. 

## How does Netflix do it?

In web application deployments, canaries can often be running in production without users ever really knowing that they're using a new version of an application. Netflix is famous for testing many different versions of their video streaming service against real customer traffic at any given time, [where canary releases are a core component of their infrastructure](https://www.infoq.com/presentations/canary-analysis-deployment-pattern). In general, they determine canary health by observing trends in user traffic. New builds are then scored based on these trends, and promoted when they are above a sufficient score. For Netflix, this works well: they have a very large user-base to make analysis on small slices of traffic plausible.

## What about smaller traffic loads?

In smaller application deployments, canary releases are often omitted. Traffic analysis is difficult as user numbers are too low to find meaningful trends, and therefore, automation falls by the wayside. Hopefully monitoring tools are in place to alert on a problem, but generally an engineer still has to handle the rollback and the bad build has already been exposed to all users.

One idea that I've been experimenting with is using behavioral specifications as found in [Behavior Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development) as a basis for observing and automating promotion of canaries in small traffic load applications. 

The premise of using "desired behavior" to test an application is already being used for many teams to drive the creation of other types of tests. In many applications, people actually **do** end up testing behaviors against new production builds, but it's generally a manual process and might involve a product owner or QA engineer creating some fake test data in a production system that then has to be ignored later. In some cases, gut check testing a system in production isn't even an option. Instead, behaviors can be defined against events that are generated out of an application by real customer traffic. For example, here's a simple behavior flow which we'll assume represents the core functionality of an eCommerce application:

1. User logs in
2. User searches for a product
3. User adds at least one item to their cart
4. User purchases the items in their cart

If you roll out a new canary build to a handful of users, you can watch for those events to have occurred in and automatically promote it. This makes relatively small numbers much more meaningful, as the success of this canary can be directly tied to these actions being carried out.

## Test Pilot

In hopes of leveraging this pattern with my team at Liberty Mutual, I've started building [Test-Pilot](https://github.com/rdbatch02/test-pilot) as a way to watch our application event logs when a new build is deployed as a canary, and handle automatically promoting for us. This way, we don't need to manually generate reports from our events to make decisions, and promotion can happen well after we've left the office for the day. 

Test Pilot is very much in the early stages, but many of the basic components are in place. It runs as a Spring Boot application somewhere in your infrastructure. Then it is fed an API to poll for new events, and it's provided with a test plan (JSON for now) which specifies which events and which application version it should match against. When all of the rules in the plan have been matched, promotion is triggered. Currently promotions are handled by triggering a Bamboo deployment, but I'm hoping to add email notification and basic web hooks at the very least, and let the community add other triggers as they wish.

There's a number of things that I want to add to make Test Pilot more appealing such as a web interface for developers and product owners, negative rules, other event sources, and so on, but I think this is a good starting place to demonstrate the idea and get support behind it.

At the end of the day, the way that I've chosen to use behaviors is probably only one way to solve the issue of small traffic loads in canary releases. Hopefully this will help drive the conversation to finding new ways to release code to production with less risk, and therefore, more frequency.