# 3. IaC
Date: 28-09-2021

## Status
Accepted

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
* Somewhat steep learning curve for certain aspects

### Pulumi

Pulumi is up and coming, and supports definitions in multiple programming languages
as well as the concept of "stacks" (environments) by default.

Pros:

* Real programming languages (GO, TS, .NET, ...)
* Good tooling support, because most languages are well supported

Cons:

* Somewhat verbose at times
* Sometimes the API does not react as expected
* After using it for a project, it seems very hard to maintain

## Decision

We adopt Terraform for defining IaC.

We adopt a private repository with the terraform code, and for the moment do not distribute
terraform code as open source, due to high risk of accidental disclosure of sensitive
material.

All sensitive data in the repository must be encrypted with git-crypt.

In case sensitive data is committed in clear text, it needs to be removed from the 
repository and rotated/invalidated as fast as possible.

## Consequences

No direct consequences for existing projects.
New projects may need a bit more work to start, but as we develop a library of how
things are done, correct configuration should be accomplished even faster than
by hand.

We will easily be able to maintain identical development environments that can
be spun up and down at will, if such is needed.
