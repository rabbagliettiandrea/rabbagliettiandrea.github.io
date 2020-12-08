---
layout: post
title: "Strategy to detect performance issues and regressions during the development/release cycle of a software component"
date: 2020-12-08
description: "Strategy to detect performance issues and regressions during the development/release cycle of a software component"
img: regression.jpg
---
Instead of relying on reactive monitoring to detect and identify issues and regressions ex-post, it is way more efficient for the team to be mindful about what they deploy, identifying and quantifying the impact of changes of the infrastructure and applications.

It is necessary to measure and monitor the entire (and probably distributed) full stack through the development/release cycle.

To do that I would create a "no-touch" deploy-on-commit code pipeline: when someone commits a file, something checks it out, runs the tests, deploys the changes, and reports back.

In order to do efficient monitoring, each step of the pipeline has to report to an external "agent". We could call that agent "Application Performance Monitor" or putting it simply, APM.

When the team push new code to the VCS, a hook (or similar) will report to the APM that new code has been pushed (commit metadata) ready to be deployed and passes the control to the build phase. When the build is done, the system will report to the APM the status of the build and the test results, then the APM will "tag" and link the commit metadata with the currently running deployed code.

At the application level there will be a middleware that will be responsible to trace each request and track down average performance metrics.

Done that, since we have tagged with commit metadata (who has commited that code, when, why - commit message) the currently deployed build, we can track down performance losses and regressions.

**Clearly, the black box of this scenario is the APM - we can develop it internally and host on-prem on a server outside the infrastructure to monitor or we could use a commercial SaaS that implements this actor and its application-level agent (e.g. New Relic, Datadog).**