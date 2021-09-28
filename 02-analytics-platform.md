# 2. Analytics platform

**Date**: 2021-09-22

We will adopt a self hosted Rudderstack and PostHog analytics engine. All future
analytics data related to users should be collected via Rudderstack.

The self hosted system will (initially) run on Google Kubernetes Engine (GKE).

## Status

Accepted

## Context

We need more consistent and collected analytics data than what we can currently get
from the separate sources that exist all around.

We need a central place for enriching and filtering analytics data.

We need a solution that is capable of offline event caching and bulk submit when user is online.

## Decision

All future data collection (as far as technically possible) should happen via this stack.

### Related decisions

* Rudderstack is the primary place for suppressing user analytics data collection
* The hosted K8s cluster should not be used for custom applications, and is not 
  available for general use outside of the stated mission.
  
### Related document

More documents related to this can be found in the `02-analytics` folder.
The documents contained therein are partially living documents and may change over time.

## Consequences

* Better and more consistent analytics
* Better control of user data

