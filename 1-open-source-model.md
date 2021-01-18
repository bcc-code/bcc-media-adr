
# 1. open-source-model
======
Date: 18-01-2021 12:49:25

We adopt an open source model of development.

This means that we fully publish code, documentation, and other materials required to build,
debug, and modify a project without needing to contact BCC.Media employees or contractors.

This implicitly forces us to develop software in a way that is maintainable and does
not rely on security-by-obscurity.

## Status
======
Proposed

## Context
======

Having a closed source model makes it really hard to involve volunteers. In the future we will need
to expand the pool of contributors significantly if we want to keep up.

In addition to that we hope that we will be able to provide inspiration and grow a community of
developers in BCC.

## Decision
======

All software development projects (unless otherwise decided for documented technical or legal reasons) should be 
released under the [GNU AGPLv3](https://www.gnu.org/licenses/agpl-3.0.en.html) license.

The decision whether to accept a proposed modification is with the main developer of the project and 
must be based on technical merit and alignment with the technical and business goals of the organization.

### Related decisions
======

* The language used in the projects as well as in the documentation is English.
* Projects should define coding guidelines (and preferably provide a `.editorconfig` file) so the project is formatted in a consistent manner, as to be more accessible to new contributors
   * This document does not prescribe what the guidelines should be, but they should preferably reflect what the larger community does

## Consequences
======

The consequences are two fold:

 * All future projects should be developed immediately in the open, or at least with the mindset that they will be open-sourced ASAP
 * Current projects need to be cleaned up and improved to the point that we can release them into the public
 * All projects must store secret data out-of-band (i.e. not in the repository)
