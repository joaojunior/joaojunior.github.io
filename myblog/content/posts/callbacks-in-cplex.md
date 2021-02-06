---
title: "Callbacks in Cplex"
date: 2017-01-08T16:48:25-05:00
draft: false
tags: ["CPLEX", "Branch And Cut", "Optimization", "Operational Research"]
---

Callbacks in Cplex
==================

I used [LazyConstraintCallback](https://www.ibm.com/support/knowledgecenter/SSSA5P_12.6.0/ilog.odms.cplex.help/refpythoncplex/html/cplex.callbacks.LazyConstraintCallback-class.html)
to implement a [Branch and Cut](https://en.wikipedia.org/wiki/Branch_and_cut) algorithm with
[CPLEX](https://www-01.ibm.com/software/commerce/optimization/cplex-optimizer/), but when I implemented and run this algorithm, it was very slow if I compare it with its version that doesn't use callbacks in Cplex. The reason is because ControlCallback(LazyConstraintCallback is a ControlCallback) switch Cplex to sequential mode by default and turn off dynamic search.
I wrote this post, because I didn't find this information in Cplex’s documentation, I only found it in this [technical forum](https://www.ibm.com/developerworks/community/forums/html/topic?id=55bc851b-cba5-446f-a6b9-696ad8ab0481).
