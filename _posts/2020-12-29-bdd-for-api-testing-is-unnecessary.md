---
layout: post
title:  BDD for API testing is unnecessary
date:   2020-12-29 12:00:00 +0000
categories: [Testing]
tags: [testing, automation]
---

When it comes to testing the APIs, do the business really care about the details of the CRUD (Create, Read, Update, Delete) operations an endpoint can perform?
<br />
<br />
It is now, unfortunately become a common practice for testers/developers to use Cucumber (or other BDD framework) to test API endpoints. I personally feel testing using the Gherkin language does not really apply in the context of integration testing. It's just adding an additional layer on top of tests.

<!--more-->

Behavior Driven Development (BDD) is a practice that encourages collaboration between technical and business stakeholders and results in scenarios/examples of requirements that are formalised using the natural `Give/When/Then` language. And there are plenty of tools to support this practice. E.g. [Cucumber](https://cucumber.io/), [Specflow](https://specflow.org/), [Behave](https://behave.readthedocs.io/en/stable/), etc.

The details of an API (e.g. type of CRUD operation, error codes. structure, etc.) is rarely of much interest by the business and there's no real need to present this information in Gherkin to them. The only thing they would be interested is the titles of scenarios that are covering some rules or requirements.

Using BDD for API testing is just going to add an unnecessary layer of complexity. There are tools such as [RestAssured](https://rest-assured.io/) that takes out this additional layer and no longer have to worry about the independence of Given/When/Then steps.

A lot of the teams are now adopting to use a tool such as [Karate](https://github.com/intuit/karate) that already has implemented Gherkin steps. But one thing I should highlight that it should NOT be mistaken for BDD but just a tool to easily implement automated api checks.

Always favour in keeping your tests simple and maintainable without adding any unnecessary layers.
