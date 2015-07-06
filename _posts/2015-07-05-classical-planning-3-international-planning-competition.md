---
layout: post
title:  "Classical Planning: #3 International Planning Competition"
categories: AI
tags: classical-planning, PDDL, IPC
---

Although performance is not my primary goal for my first attempt of implementing a PDDL planner, I'm still care about it. After implemented paramenters support in action schema, I found the performance was too bad at the stage, that it took 20+ seconds on my laptop to resolve **cargo transportation problem** described in the [AIMA book].

So I spent little time to profile my code. It turns out that the bottleneck is using `Array` operations, as I used `Array` to represent state object, precondition and effect in actions. It is obvious that it's not efficient, but I'm supprised that it is so slow in such a small problem searching.

The fix is straightforward: use `Set` instead of `Array`.

The next bottleneck is hash code generation. After used `Set`, hash code is wildly used to look up objects in `Set`. However, hash code of `Set` will be regenerate when you call `hash` method even there is nothing changed in the `Set`. So I introduced `State` class to cache hash code of state object. Since we have a customized class for state object, it is better to let `State` object hold on action sequence data that we need for final solution now. Thus we elimate the need of `Problem#describe` interface method.

While these changes improved performance for the small domain problems using breadth-first search, I was curious what's state of the art performance of PDDL planners and what's algorithms they're using.

As "International Planning Competition" is mentioned in the [AIMA book], I found [website][IPC-2014] of its latest competition event International Planning Competition 2014 (aka IPC-2014). The [website][IPC-2014] contains PDDL descriptions of problems used in competition, source code of all competing planners, and other useful resources (e.g. PDDL papers, plan validator, rules).

The competition used [PDDL version 3.1][PDDL3.1]. Comparing to the one described in [AIMA book], it has more complex structure and syntax to support more complex domains and problems. Another major difference is PDDL separates domain description and problem description:

1. The domain description includes actions/functions, types, constants, predicates and other domain specific extension descriptions.
2. The problem description includes initial, goal, objects, related domain name and other problem specific descriptions.

Separating domain and problem descriptions is necessary when you try to validate the planner in various domains and problems. One domain can be used to generate different complex level problems by adding more objects, more complex initial and goal setups, etc.. And more importantly, when you try to construct a complex problem with lots of objects, you'd like to have a problem generator to do that for you.

I probably still can hand convert several problems for developing and learning algorithms. But for further development, I will need implement standard PDDL syntax, because all existing resources, including domain descriptions, problem generators and plan validator, support standard PDDL syntax. To moving forward, I also need change my language and vocabularies to be more close to standard PDDL. The minimum support for PDDL 3.1 is **STRIPS** model, however, most PDDL descriptions of domains and problems I found in [IPC-2014] require **STRIPS** and **TYPING** models (for model definitions, see section 1.3 Requirements of the [PDDL3.1]); **STRIPS** model is premuch what is described in the [AIMA book], and I'm implementing it anyway; then supporting **TYPING** model became my next target to support for trying out some examples of PDDL domains and problems from [IPC-2014].

It's not surprise that the search component of most of planners is written in C++ for speed. However, I found one Java planner named [Freelunch] in [IPC-2014]. It is open sourced, but it seems no one is maintaining it.

For a summary of [IPC-2014], you can checkout these [slides]. Here are some interesting observations of [IPC-2014] listed in the [slides]:

1. More planners were built on top of platforms ([FD], [FF],...). 29 planners out of 67 built on top of [FD].
2. Portfolio-based systems are now a concrete reality. 29 portfolios; 3 awarded.
3. Only small number of planners are able to deal with **PREFERENCES** and **TEMPORAL** models.

(For portfolio-based system, there is recent paper discussing it: [Portfolio-based Planning: State of the Art, Common Practice and Open Challenges][PBP2015])

My primary goal is still learning classical planning basing on [AIMA book], so it won't make sense for me to use any existing platforms to speed up implementing a planner that can compete with planners in [IPC-2014]. I will continue with Ruby implemention. However, I've decided to jump to **GRAPH PLANNING** section as my next step for getting a better performanced planner that can solve harder problems from [IPC-2014].




[AIMA book]:       http://www.amazon.com/Artificial-Intelligence-Modern-Approach-Edition/dp/0136042597
[IPC-2014]:        https://helios.hud.ac.uk/scommv/IPC-14/index.html
[FF Domain Collection]: http://fai.cs.uni-saarland.de/hoffmann/ff-domains.html
[Freelunch]:       http://ktiml.mff.cuni.cz/freelunch/
[slides]:          https://helios.hud.ac.uk/scommv/IPC-14/repository/slides.pdf
[FD]:              http://www.fast-downward.org/
[PBP2015]:         http://eprints.hud.ac.uk/24291/1/VallatiPort.pdf
[PDDL3.1]:         https://helios.hud.ac.uk/scommv/IPC-14/repository/kovacs-pddl-3.1-2011.pdf
[FF]:              http://fai.cs.uni-saarland.de/hoffmann/ff.html
