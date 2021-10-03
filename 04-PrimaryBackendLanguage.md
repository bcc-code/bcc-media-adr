# 4. Primary backend language
Date: 1.10.2021

## Status
Accepted

Note: This document is not a hard rule but a record of a decision, for a particular project.

It can also be considered that `Go` as the accepted language is a language that is
officially used by BCC Media Foundation.

## Context

There are currently many languages and frameworks for various languages available.
There are many differences as well as many similarities between them.
In order to use the best tools we can for new systems, we have evaluated several languages
in more or less detail.

Some of the main requirements for the language to be considered in detail are:

* Strongly statically typed, at compile and run time
* Should be accepted in the industry for the task (web backend/api)
* Should be something that "does not scare away contributors"
* Should be suitable for "cloud native" deployment (via Google Cloud Run at the time of writing)

## Evaluation

### Early filtering

The main requirements quickly disqualified some candidates that came up in discussions:

* Node.JS
	* Typescript does not offer run time safety (As far as we were able to tell)
* Python
	* Type hinting is not sufficient
* Erlang/Elixir/Phoenix
	* Not static types (potentially less of an issue here)
	* Not super widely used
	* Some more notes later
* Haskell
	* Not something I would consider industry standard in this area
	* Could be very daunting for a unexperienced dev
	* Potentially hard to find programmers
* Nim
	* Promising, but not "big" enough to consider for a non-hobby project
	* Not really a "standard" for anything

The main contenders evaluated more closely:

* Java
* C# (.NET core)
* Go
* BEAM (Erlang/Elixir)

### BEAM

BEAM is the runtime most well know for running Erlang. The architecture is
uniquely suited for todays "microservice" world and can handle huge numbers
of concurrent connections.

Unfortunately it is (at the time of writing) comparatively obscure and would
likely hinder us more than help, especially when thinking about hiring developers.

There is also no significant experience in the current team so we would probably,
first have to fall into pretty much every pitfall there is before we could become
really productive.

Based on this facts BEAM was disqualified from further discussions.

### Java

There is currently no one on the team that has significant experience with JAVA,
so there was little interest in exploring further. The language otherwise seems
to fit the bill but has a reputation for being memory hungry if used "naively".

There is also the question of [licensing](https://www.oracle.com/java/technologies/javase/jdk-faqs.html) (why do you need a licensing FAQ Oracle?), and potential
complications on that front. This has not been investigated in detail.

Based on verbal discussions JAVA was disqualified from further investigation.

### C# (.NET core)

There is significant history and expertise in the team, the language itself is
well supported (Microsoft) and actively developed, and well suited for the task.

A.G:
TODO: Comments

M.D:
In this case you are not really free to choose a framework. You are more or less
forced into asp.NET core, which to my taste does too many things by "magic".
I had an issue with how opaque DI is in this case. Main argument for it is Testing.
Counter argument: If you are doing unit testing then you should be able to mock the
few things and pass in mocks. For any other test to have a real value it should
be a E2E test started by calling an API and including a real DB server.

### Go

Existing expertise in the team, language is tailored to the task, and has very
wide adoption and support for a young (11 years) language.
It supported by Google and very small in terms of language features, witch
leads to some boilerplate but on average keeps the language fairly easy to read
and very explicit.

A.G:
TODO: Comments

M.D:
I think Go is very well suited for us because of it's simplicity and explicitness.
It should allow anyone with some programming experience to understand the basic
strokes of the program, as well as where the data flows, which in my opinion is
the core of what one needs to understand to be able to productively contribute.

## Go vs. C#

After long verbal discussions about the benefits and drawback of each language,
we decided to conduct a test: Implement a simple GQL API with a Postgres DB backend
in both languages and then evaluate the following points:

* Performance
* Memory usage
* Ease of understanding/Level of Magic involved
* Team opinion

### Performance

In all conducted tests go performed better and answered requests 4-10x faster
than the equivalent requests in C#.

### Memory

There appears to not be any significant difference of how much memory was used
in a large request.

Both languages do not release the memory back to the OS if not required. From
the POV of the OS they do not free the memory. This is a non-issue as the service
will run on dedicated servers/vms/containers and is thus encouraged to use the
available RAM.

### Ease of understanding/Magic

Both languages employ some magic. The main difference is that C# employs the magic
at run time, while go makes heavy use of code generation.

This allows for inspection of the generated code and thus the code once generated
is no longer magic, but can be traced and debugged like any other code.

### Team opinion

The general consensus is that go offers a better way of achieving the goal.
The language feels less indirect, which makes it easier to reason about what is
going on.

## Decision

We adopt [go](https://golang.org/) as the primary language for backends at BCC Media

This does not invalidate other languages and they should be used when they offer
a significant advantage over Go. Such choice should be documented in the project
as to be able to understand the reasoning later.
