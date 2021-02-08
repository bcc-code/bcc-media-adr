# 2. IaC
Date: 08-02-2021 09:27:51

## Status
Proposed

## Context

Infrastructure as code is the modern way of deploying, scaling, and managing
resources in and across Cloud environments, and provider.

It provides a lot if benefits related to security, cleanup, replicability, and cleanup.

It also allows us to have a record of how the infrastructure has changed over time,
as well as a record who did what and why.

There are currently 2 main providers that do IaC in a declarative format:

* Terraform
* Pulumi

### Terraform

Terraform is currently the market leader and probably the most well known provider.
The configuration is written in a special Terraform language.

*Pros*:

* Largest community
* Most mature

Cons:

* Not a _real_ language
* Refactoring often results in recreating resources

### Pulumi

Pulumi is up and coming, and supports definitions in multiple programming languages
as well as the concept of "stacks" (environments) by default.

Pros:

* Real programming languages (GO, TS, .NET, ...)
* Good tooling support, because most languages are well supported

Cons:

* Somewhat verbose at times
* Sometimes the API does not react as expected

## Decision

We adopt Pulumi for defining IaC.

The Pulumi configuration should by default reside in `./infra` and have at least
a sample stack showing off the available variables.

By default the Pulumi code should be written in the language the project is written
in, using Go as fallback.

## Consequences

No direct consequences for existing projects.
New projects may need a bit more work to start, but as we develop a library of how
things are done, correct configuration should be accomplished even faster than
by hand.

We will easily be able to maintain identical development environments that can
be spun up and down at will, if such is needed.
