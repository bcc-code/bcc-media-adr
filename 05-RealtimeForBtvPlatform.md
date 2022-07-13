# 5. Realtime for BrunstadTV platform
Date: 13.7.2022

## Status
Accepted

## Context

We will need some real-time or near real-time things in the applications going forward.
Since it is somewhat unclear what exactly that will entail, the solution should give us the
most flexibility while not adding significant overhead to the APP(/Website?).

As far as I can see there is no *good* competitor to Firestore real-time subscription out there without us have to run servers 24/7 etc.

## Evaluation

### Considered options

Polling - Would increase number of requests we have to handle/app has to make. Not real time. Basically free to add.

Something like [Supabase](https://supabase.com/) - Not really an option because of the need need to self host, configure, maintain.
No real benefit over Firestore, other than potentially using same DB as the rest of the system.

Hasura subscriptions - Potential option.
Would keep it in GQL-land but would introduce a new system that we are not familiar with.
In addition we have no experience as to how well GQL subscriptions work in APP context.


Firestore "lite" - Public firestore document/collection with no real data like:


```json
{
    "realtimeIds": [2, 17]
}
```

or

```json
{
    "maintenanceMessage": "2022-07-13T12:03:50.000Z",
    "appConfig": "2022-07-13T12:03:50.000Z"
}
```


That document would inform the app that something has changed and the application can look it up.


Full firestore with auth and data - Increases complexity for small benefit



## Decision

We adopt the "Firestore lite" version as the baseline with exceptions where warranted.
The polling solution is available as well for applications/systems that are unable or unwilling
to use firestore. This also allows us to have a fall back if firestore has issues.

More detailed specification can be found [here](https://github.com/bcc-code/brunstadtv/blob/develop/documents/realtime.md).

