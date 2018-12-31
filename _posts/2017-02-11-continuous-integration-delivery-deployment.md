---
layout: post
title: "Continuous Integration, Delivery, and Deployment"
---

![Continuous](/images/2017/02-11/continuous.svg)

A team need to build and deliver software more rapidly. By doing it, it is easier to understand what users want and to build a successful software. It sounds easy but reality is not. Especially, the more programmers are working, the more difficult it is. Several software development practices have been introduced to ease the process, and those can be grouped into followings.

## Continuous Integration
This requires developers to integrate code into a shared repository as soon as possible. This doesn't get rid of bugs, but it makes them dramatically easier to be found. In addition, integration problems are reduced so it allows to deliver software more rapidly. When a developer commits a code change, an integration machine, such as [Jenkins](https://jenkins.io) and [TeamCity](https://www.jetbrains.com/teamcity/), checks whether the change can be integrated or not.

Many teams develop their integration policy differently  based on their needs. For example, one policy could be only about a build test. Another policy could include all following tests.

* Build test
* Unit test
* Integration test
* Acceptance test

{% include google-adsense-in-article.html %}

## Continuous Delivery
This is closely related to the Continuous Integration and refers readiness of software release to production. The readiness requires to pass every test, including acceptance test. Those tests are automated and mostly done in Continuous Integration. Then, you should be able to click one button, called "Release", to release readied software.

![Release button](/images/2017/02-11/release.svg)

## Continuous Deployment
This is really similar to Continuous Delivery except that you don't have to click the "Release" button. Every change that passes the automated tests will be released to production automatically.
