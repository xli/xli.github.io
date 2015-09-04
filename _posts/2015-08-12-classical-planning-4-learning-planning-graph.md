---
layout: post
title:  "Classical Planning: #4 Learning Planning Graph"
categories: AI
tags: classical-planning, PDDL, planning-graph
---

Planning Graph is a data structure. More specifically, it is a directed graph.
A Planning Graph has three parts:

1. State levels
2. Action levels
3. Mutual exclusion (or mutex) links

The relationship between state and action:

> A planning graph is a directed graph organized into levels: first a level S0 for the initial state, consisting of nodes representing each fluent that holds in S0; then a level A0 consisting of nodes for each ground action that might be applicable in S0; then alternating levels Si followed by Ai; until we reach a termination condition.
>
> ------ [Artificial Intelligence: A Modern Approach (3rd Edition)] page 379

I thought it is well defined and easy to build the state and action levels. However, an example of planning graph for the Cake problem in the book page 380 confused me how we build the initial state level (S0) (Figure 10.7 and 10.8):

![Planning graph for cake problem](/images/aima-figure-10-7-and-10-8.jpg)

In this example, a literal `-Eaten(Cake)` shows up in the graph S0 level. As the example only have 2 facts: Have(Cake) and Eaten(Cake); it seems we should initialize the S0 with all undescribed facts as negative facts in S0.

This problem blocked me implement planning graph algorithm, as there is no where else in the book talking about how to build S0.
One clue found in the book is on page 367:

> Database semantics is used: the closed-world assumption means that any fluents that are not mentioned are false,

If we applied [closed-world assumption] to the cake example, then even the inital condition only mentions `Have(Cake)`, it also means `Eaten(Cake)` should be negative, which maybe the reason that `-Eaten(Cake)` is showed up in S0 level.

However, it is still confusing after I read paper _Fast Planning Through Planning Graph Analysis_ (Avrim L Blum & Merrick L. Furst, 1997).
In the paper, it defines:

> The first level of a Planning Graph is a proposition level and consists of one node for each proposition in the Initial Conditions.

There is also an example shows how planning graph works in the paper:

![Planning graph for the rocket problem](/images/graphplan-page-5-figure-2.png)

It is clear that S0 does not contain any negative literals that are not included in the Initial Condition.
The discussion in the paper is based on [STRIPS] which is slightly more restricted than PDDL, but it uses closed-world assumption too.

To understand more of how planning graph algorithm got implemented in real world, I continued to read more related papers.

As [Fast Forward] uses relaxed planning graph to implement ignoring delete list heuristic, I spent couple weeks to learn the algorithm and implement relaxed plan graph heuristic function. Because the relaxed planning graph ignores delete list, it does not need to handle mutual exclusion links (Jorg Hoffmann & Bernhard Nebel, 2001). I thought it will keep me more focused on understanding how to construct state and action levels in planning graph. But I underestimated the overall complexity of the [Fast Forward] planner. It turns out, just having a heuristic function working is not good enough for solving a relative complex problem (e.g. barman, free-cell) (Why do I need to try to solve any of complex problem? It is because I need prove my implementation is correct, and one acceptance criteria should be solving a complex problem that original [Fast Forward] algorithm can solve with similar performance).

Anyway, I did implement the relaxed planning graph used in [Fast Forward], and the S0 does not contain negative literals that are not included in the initial condition.

Conclusion
==============

When closed-world assumption applied, we probably can construct S0 with all negative literals that are not mentioned in initial condition like the example showed in the book. But for an efficient implementation or open-world assumption, we can ignore literals that are not mentioned in initial condition when building S0 level.

References
=================

1. Avrim L. Blum & Merrick L. Furst (1997): Fast Planning Through Planning Graph Analysis. Artificial Intelligence, 90(1-2), 279-298
2. Artificial Intelligence: A Modern Approach (3rd Edition)
3. Jorg Hoffmann & Bernhard Nebel (2001): The FF Planning System: Fast Plan Generation Through Heuristic Search. Journal of Artificial Intelligence Research 14 (2001) 253-302
4. https://en.wikipedia.org/wiki/STRIPS
5. https://en.wikipedia.org/wiki/Action_description_language
6. http://fai.cs.uni-saarland.de/hoffmann/ff.html

[Artificial Intelligence: A Modern Approach (3rd Edition)]:          http://www.amazon.com/Artificial-Intelligence-Modern-Approach-Edition/dp/0136042597
[Closed-world assumption]:                                           https://en.wikipedia.org/wiki/Closed-world_assumption
[Open World Assumption]:                                             https://en.wikipedia.org/wiki/Open-world_assumption
[Fast Forward]:                                                      http://fai.cs.uni-saarland.de/hoffmann/ff.html
[STRIPS]:                                                            https://en.wikipedia.org/wiki/STRIPS
[ADL]:                                                               https://en.wikipedia.org/wiki/Action_description_language