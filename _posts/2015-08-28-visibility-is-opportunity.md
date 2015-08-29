---
layout: post
title:  "Visibility Is Opportunity"
categories: continuous-delivery
tags: CI, Go, build, Mingle
draft: true
---

A continous integration problem
==================

[Mingle] is a 9 years JRuby Rails project.
We do continuous integration with selenium test since Oct 2006.
We write selenium tests to test everything.
July 2007, our build time increased to 8 hours on one machine.
Then we parallelized build.
Our tests continue to grow.
And we continue to put in more hardwares.

For a long time, our **build time was 1 to 2 hours**. We thought it was a good balance of build time and resources needed.

Test code performance improvement
==================

2014, the best build time was 1 hour ran in parallel on 47 VMs.
We ran test in parallel by [Go] jobs and [TLB].
We created 46 test jobs in one [Go] stage.
Then grouped test suites into units, functionals and acceptnace (selenium) test.
Each group ran a subset of tests distributed by [TLB].
We configured [TLB] to balance test by test runtime.

For improving test performance, I did profiling on all tests.
I used [Sampling Profiler](https://rubygems.org/gems/sampling_prof) to profile our build:

1. First I ran sampling profiler with test on build.
2. Then sent all profiling data (call-graph) to a server.
3. Server merged profiling result into one big profiling result.
4. Last, I built a UI to navigate through the big profiling result.

It gave me code level visibility of where was time spending.
It opened performance improvement opportunity.
The biggest supprising I found was test slept 1 hour on build in total.
For example, the following code tries to wait for a condition:

{% highlight ruby %}
Timeout::timeout(time / 1000.0) do
  while get_eval(condition).downcase == "false"
    sleep 1
  end
end
{% endhighlight %}

The `sleep 1` code above caused build slept about 40+ minutes in total.

After fixed about 15 small bottlenecks, our **build time reduced to 45 minutes**.


Build infrastructure performance improvement
==================

Recently, our build became unstable (more random failures) and longer (1 hour).
I got time to look into build performance again.
My teammate [Jeff Norris] noticed it was too slow when running selenium test on build VM.
I realized we had not paid attention to our VM performance.
Our VMs ran with 1 vertual CPU and 3.5 GB memory.
To run a selenium test, we need launch 3 processes:

1. [Mingle] application server for testing
2. Selenium proxy server
3. Selenium test.

It is possible that 1 vertual CPU is not enough.
So we increased build VMs to 2 vCPU to try it out.
The result was not as good as we thought. Build time decreased to 50 minutes.
But I noticed some selenium test jobs ran twice faster than other selenium test jobs.
As [TLB] balanced our selenium tests by time, we expect job runtime similar.
My first hypothesis was [TLB] may not work as expected.

To verify my hypothesis, I checked out [TLB] source code. It was more complex than I thought.
And there is also no way to output more logs to verify it's correctness.
But [Go] has good support for APIs, so I wrote [Ruby script](https://github.com/ThoughtWorksStudios/goapi/blob/master/examples/compare_test_runtime.rb) to verify my hypothesis.

The following chart shows how the test runtime looks like on build. X-axis is job names; y-axis is job runtime.

![Acceptance test time](/images/acceptnace-test-build-time.png)

Blue bars are test runtime on job. Red bars are "expected time", because [TLB] distributed tests by previous build test runtime.
Red bars are test runtime calculated by previous build test runtime:

    Job expected runtime = sum of each test runtime in previous build

From this result, we can see the balance is not perfect, but OK.
Because red bars are similar high across all acceptance (selenium test) jobs.

Blue bars matches what I observed on build time.
Then I ran same script on more builds. The outputs were similar, just different jobs got longer time to run.

As [Go] random picks up build VMs to run any job, it gives me a clue that maybe its build VM performance issue.
So I wrote another [script](https://github.com/ThoughtWorksStudios/goapi/blob/master/examples/agent_stats.rb) to build the following chart:

![Go build agent runtime](/images/vms-build-time.jpg)

It's clear there are some VMs are consistent slower than others.
Then Barrow Kwan found out 2 of our VMs hosts were overloaded when we increased vCPUs on our VMs.
We have set the host NOT to overload the CPU core but there was a typo in configuration.
After sorted out VM host CPU overload issue, our **build time reduced to 30 minutes**.

Build task performance improvement
======================

[Go] introduced "timestamps in console logs" in release 15.1.
It made build task runtime visible.
I was not pay attention to the timestamps in the console logs until recently.
The following screenshot shows an example:

![timestamps in console logs](/images/timestamps-in-console-logs-example.png)

It turns out, [tlb.rb] client we used has the following code:

{% highlight ruby %}
def self.get path
  sleep 2
  Net::HTTP.get_response(host, path, port).body
end
{% endhighlight %}

It only sleeps 2 seconds for one call, but it is called lots of times at the end of tests ran.
After removed code `sleep 2`, there is almost no time spent at the end of test anymore.

Without "timestamps in console logs", it is something like following:

![no timestamps in console logs](/images/no-timestamps-in-console-logs-example.png)

No one will notice it ever.

The benefit of fixing this problem is outstanding for our **pre-commit build**.
Pre-commit build is a build runs a test suite before developer pushs changes to trunk repository.
Our pre-commit build runs all unit and functional tests.
The **build time reduced from 18 minutes to 10 minutes** with 20 processes on 3 machines.

Conclusion
=================

We thought we could not improve build time without throwing in more hardwares.
We thought 1 to 2 hours build time were the best we could get with limited resources.
We thought we had too many selenium tests.
We thought long time build was what we had to pay for more coverage in test.

But we got opportunities to improve when we find out more build time details.

**So when you face a tough problem, don't assume, make details visible.**


[Mingle]:                                           https://www.thoughtworks.com/mingle
[Go]:                                               https://go.cd
[Jeff Norris]:                                      http://www.thoughtworks.com/profiles/jeff-norris
[TLB]:                                              https://github.com/test-load-balancer/tlb
[tlb.rb]:                                           https://github.com/test-load-balancer/tlb.rb
[OpenStack]:                                        https://www.openstack.org/
[GoAPI]:                                            https://github.com/ThoughtWorksStudios/goapi