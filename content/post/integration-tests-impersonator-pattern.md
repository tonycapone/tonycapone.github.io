+++
date = "2016-02-13T13:41:53-06:00"
#draft = true
title = "To Mock or Not to Mock: Automated Integration Tests and the Impersonator Pattern"
+++

One debate that comes up a lot in teams I've been on in the past is about our "real" our automated tests should be.
<!--more-->
Not unit tests; those everyone seems to agree on. I'm talking those *other* tests which seem to defy definition, but fall under the
various labels of integration/endtoend/functional/acceptance tests.

The the crux of the debate seems to boil down to this:

  *We need a set of tests that test our code end to end, BUT how should our automated tests
  interact with external systems (such as a 3rd party ReST API).
  Should we allow the calls to go through to the real thing, or use a test double?*

Some of the points that usually come up, all valid:

### Reasons not to use test doubles:

If you mock that service out, then the tests aren't really "real." There is a risk that the tests are invalid,
because assumptions made about how the service will respond were incorrect, or could become out of date. There is also considerable time to invest in stubbing out the interactions, as well as maintenance to keep the tests up to date.


### Reasons to use a test double
On the other hand, if the external system has long running tasks that take seconds or minutes to complete,
I might have to add arbitrary sleeps(), which can lead to brittle tests. They may run much slower, which will decrease
how often we run them.

If the external system is flaky, I have a suite of flaky tests,
[which are worse than no tests at all](http://martinfowler.com/articles/nonDeterminism.html).
If the interactions result in stateful behavior, then I have to deal with that as well.
If the calls to the external system make stateful changes, then you have to write extra code to setup and teardown as well.

In retrospect, I think this is a false dichotomy.

### You should do both
If you're building a continuous delivery pipeline, there is room
for both types of tests: A set of fast integration tests with the external systems that interact with stubbed out methods on a test double, and a set of slower running acceptance tests that run against the real thing.

Most of my thinking on this comes from reading and listening to what Mr. Continuous Delivery himself, **Jez Humble**, has to say on the matter.
Jez talks a lot about how your CD pipeline should have several stages, each stage with a set of automated and manual tests that increase confidence that the artifact/code is ready to go to production.

One very interesting, and under-rated, pattern that he talks **Impersonator Pattern**. The Impersonator Pattern answers
the mock-or-not question by *letting you do both.*


### The Impersonator Pattern
The Impersonator Pattern (Martin Fowler calls them [Self Initializing Fakes](http://martinfowler.com/bliki/SelfInitializingFake.html)) works like this:

We will proxy the calls to the external system through a middle man.
The first time the call is made, the request will be passed on to the remote system, and transparently pass the response
back to the caller, *while recording the interaction*. All subsequent interactions will playback the cached interaction.

{{< figure src="/media/impersonator_pattern.png"
   caption="Credit: Martin Fowler Bliki http://martinfowler.com/bliki/SelfInitializingFake.html" >}}




This pattern offers a nice compromise.

First, the interactions are "real." I didn't make assumptions about how the external system would respond, I also saved a ton of time because I didn't have to manually stub out individual calls. My tests will run faster because I'm interacting with a local test double.

Because they're so fast, this set of tests could even be run along side unit tests on the developers laptop, further increasing confidence before checking the code in.

This set of tests would not replace true end-to-end tests running against the live remote service, but rather supplement them.

In an upcoming post, I'll share my solution for implementing the Impersonator Pattern in Java with [Wiremock](http://wiremock.org/).
