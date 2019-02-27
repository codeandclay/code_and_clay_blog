---
layout: post
title: Dependency injection is not a virtue
---

via [DHH](https://dhh.dk/2012/dependency-injection-is-not-a-virtue.html)

>In languages less open than Ruby, hard-coded class references can make testing tough. If your Java code has `Date date = new Date();`` buried in its guts, how do you set it to a known value you can then compare against in your tests? Well, you don't. So what you do instead is pass in the date as part of the parameters to your method. You inject the dependency on Date. Yay, testable code!

>As has unfortunately happened with a variety of patterns that originate from rigid languages like Java, Dependency Injection has spread and been advocated as a cross-language best practice on trumped up benefits of flexibility and malleability. If your code never knows exactly who it's talking to, it can talk to anyone! Testing stubs, mocks, and future collaborators. Hogwash.
