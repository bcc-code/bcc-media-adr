# Technical details for the tracking plan

## Implementation notes

The analytics should be collected (unless otherwise noted), even when the client
is offline. The data should be cached as far as possible and sent to the server
when the device is back online. If caching more data becomes impossible, the oldest
cached data should be deleted. The deletion is discussed further later in the doc.

## Common data

The data specified below should be sent with every event, unless otherwise noted. 
If it is not possible for all events then that should be noted in this document.

In general it is better to omit data (if for example it is not accessible on some
os version), and send what you can.

Note that this is for both APP and Web so some things may be possible on one 
platform and not on the other.

### Common data table

| Data                | Name           | Comments                                      |
|---------------------|----------------|-----------------------------------------------|
| OS                  | os             | Automatically managed by SDKs                 |
| OS Language         | locale         | Automatically managed by SDKs                 |
| Network Info        | networkInfo    | Automatically managed by SDKs                 |
| Device Info         | deviceInfo     | Automatically managed by SDKs                 |
| Anonymous ID        | anonymousId    | Automatically managed by SDKs                 |
| Channel             | channel        | Automatically managed by SDKs                 |
| Timezone            | timezone       | Automatically managed by SDKs                 |
| Screen data         | screen         | Automatically managed by SDKs                 |
| App Language        | appLanguage    |                                               |
| Release Version     | releaseVersion | App version, or build number/git hash for web |
| Person analytics ID | presonId       | For logged in users, see below                |

### Person analytics ID

You can get this ID by sending a token to TODO. The returned value is an hash
of the userID + a secret. This ensures that we are able to link all data to one
user from different systems, while preventing anyone from being able to reverse
the process or generate analytics IDs if they do not posses the secret.

## Identify

*When*: On login + When you request a new refresh token from Auth0
*Reason*: Being able to connect anonymous events to a user

### API

Use the `/identify` endpoint. Docs:			https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/identify

### Data

This call accepts much more data, but we do not want to send it here. The additional
data will be automatically injected by RudderStack, and delivered to the needed targets.

| Data         | Name        | Comments                                                                         |
|--------------|-------------|----------------------------------------------------------------------------------|
| Anonymous ID | anonymousId | Automatically managed by SDKs                                                    |
| PersonID     | id          | This is the actual person ID. It will be used for data lookup and then discarded |

## Screen/Page View

*When*: On every page/screen the user opens. If relevant also on popups if they
can be considered a separate screen.

*Reason*: It will allow us to see how used the various pages are and estimate how
much time users spend on various screens.

### API

Use the `/page` endpoint on web. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/page
Use the `/screen` endpoint in app. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/screen

### Data

The rest of the relevant additional data is automatically collected by the SDK.

| Data         | Name        | Commnets                                       |
|--------------|-------------|------------------------------------------------|
| Name         | name        | A page identifier. See comments and list below |
| Element Type | elementType | episode, series, ...                           |
| Element ID   | elementId   | series, episode ID                             |

### Page/Screen IDs

TODO: Maybe we can gather a list here. I would like to be consistent between platforms
as far as possible, so we can for example compare how many people go to search on
mobile vs app easily.


## Section click (sectionClick)

*When*: On every tap/click on a section element. This is on all sections, regardless
of where they may appear

*Reason*: This will allow us to asses how effective the sections are, as well as
possible debug purposes

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data         | Name        | Commnets                                      |
|--------------|-------------|-----------------------------------------------|
| Event ID     | event       | Hardcoded: `sectionClick`                     |
| section ID   | sectionId   | ID of the section that the element belongs to |
| section Name | sectionName | For easier identification in tools            |
| Element Type | elementType | episode, series, ...                          |
| Element ID   | elementId   | id of the clicked element                     |
| See All      | seeAll      | true if the tapped element was "See All"      |
| Page Name    | pageName    | same ID as for page/screen tracking           |

## Liveboard click (liveboardClick)

*When*: User clicks an element in the liveboard. This is for all screens where
liveboard appears (or may appear in the future).

*Reason*: Better understanding about how the modules are user, and how changes
in order, etc affect the use. Can also be used for debug purposes.

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data            | Name           | Commnets                                           |
|-----------------|----------------|----------------------------------------------------|
| Event ID        | event          | Hardcoded: `liveboardClick`                        |
| Module ID       | moduleId       | from Firestore                                     |
| Module Type     | moduleType     | from Firestore                                     |
| Module Position | modulePosition | top module == 1 (easier logic for non-tech people) |
| Event ID        | eventId        |                                                    |
| Event Name      | eventName      | Optional                                           |

## Audio Only (audioOnlyClick)

*When*: User clicks/taps the audio only button

*Reason*: Measure usage for the button

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data        | Name             | Commnets                    |
|-------------|------------------|-----------------------------|
| Event ID    | event            | Hardcoded: `audioOnlyClick` |
| Video state | videoStateTarget | Video state *after* tap     |


## Calendar click (calendarClick)

*When*: When user taps/clicks an element in the calendar

*Reason*: Assessing usage, spotting problems (taps on things that don't lead anywhere)

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name         | Commnets                                      |
|---------------|--------------|-----------------------------------------------|
| Event ID      | event        | Hardcoded: `calendarClick`                    |
| Page Name     | pageName     | same ID as for page/screen tracking           |
| Calendar view | calendarView | `week` or `month`                             |
| Calendar date | calendarDate | ISO-8601 (YYYY-MM-DD)                         |
| Element Id    | elementId    | If tap leads to an episode, record episode [id](id) |

## Play Clicked (playClick)

*When*: When user taps/clicks "play"

*Reason*: Spotting issues

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name         | Commnets                                      |
|---------------|--------------|-----------------------------------------------|
| Event ID      | event        | Hardcoded: `playClick`                        |
| Page Name     | pageName     | same ID as for page/screen tracking           |
| Element Type  | elementType  | episode, series, ...                          |
| Element ID    | elementId    | series, episode ID                            |

## Search (search)

*When*: When user performs a search

*Reason*: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data           | Name              | Commnets                     |
|----------------|-------------------|------------------------------|
| Event ID       | event             | Hardcoded: `search`          |
| Search Term    | searchText        | exact search term            |
| Search Latency | searchLatency     | how long did the search take |
| Result Count   | searchResultCount |                              |


## Search Result click (searchClick)

*When*: When user clicks/taps on a search result

*Reason*: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data            | Name                 | Commnets                     |
|-----------------|----------------------|------------------------------|
| Event ID        | event                | Hardcoded: `searchClick`     |
| Search Term     | searchText           | exact search term            |
| Result Position | searchResultPosition | how long did the search take |
| Element Type    | elementType          | episode, series, ...         |
| Element ID      | elementId            | series, episode ID           |

## Language Change (languageChange)

*When*: When user changes the language in any context

*Reason*: Spotting issues, Usage info

*Notes*: This is not high priority, if hard to get, can be omitted

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name               | Commnets                            |
|---------------|--------------------|-------------------------------------|
| Event ID      | event              | Hardcoded: `languageChange`         |
| Page Name     | pageName           | same ID as for page/screen tracking |
| From Language | languageFrom       |                                     |
| To Language   | languageTo         |                                     |
| Where         | languageChangeType | audio, app, subs                    |

## Share (share)

*When*: When user changes the language in any context

*Reason*: Spotting issues, Usage info

*Notes*: This is not high priority, if hard to get, can be omitted

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name               | Commnets                            |
|---------------|--------------------|-------------------------------------|
| Event ID      | event              | Hardcoded: `share`                  |
| Page Name     | pageName           | same ID as for page/screen tracking |
| Element Type  | elementType        | episode, series, ...                |
| Element ID    | elementId          | series, episode ID                  |

## Other Tracking events

This tracking events do not have data besides the common fields. The list contains
the `EventID` values to be used

| Event              | Event ID          |
|--------------------|-------------------|
| Log In             | login             |
| Log Out            | logout            |
| Airplay started    | airplayStarted    |
| ChromeCast started | chromecastStarted |

## Video events

The video events should, in addition to the common data, also contain the following.

### Common video data table

The names are specified by Rudderstack and are inconsistent with the rest of the
document. They should be used as they are here (camelCase).

| Data            | Name               | Comments                                                                                                     |
|-----------------|--------------------|--------------------------------------------------------------------------------------------------------------|
| Session ID      | sessionId         | regenerate after 30 minutes of not being used                                                                |
| Is Live         | livestream         | true/false                                                                                                   |
| Episode ID      | contentPodId     |                                                                                                              |
| Positing        | position           | From start of video in seconds                                                                               |
| Total Length    | totalLength       | null if live stream                                                                                          |
| Video Player    | videoPlayer       | Name of the player (VideoJS, ...)                                                                            |
| Is Full screen? | fullScreen        |                                                                                                              |
| Quality         | quality            | if available                                                                                                 |
| Volume          | volume             | in %                                                                                                         |
| Has video       | hasVideo          | false if in "audio only mode"                                                                                |
| Subs            | subtitlesLanguage | null if not possible to determine. "OFF" if turned off. "N/A" if subs are not available in selected language |

## API

Use the `/track` API as described here: https://docs.rudderstack.com/rudderstack-api/api-specification/video-specification

## Standard video events

This events contain no extra data 

| Event                 | Event ID            |
|-----------------------|---------------------|
| Playback started      | playbackStarted     |
| Playback paused       | playbackPaused      |
| Playback interrupted  | playbackInterrupted |
| Playback buffer start | playbackBufferStart |

## Buffering end (playbackBufferEnd)

### Data

| Data                 | Name       | Commnets            |
|----------------------|------------|---------------------|
| Event ID             | eventID    | `playbackBufferEnd` |
| Time spent Buffering | bufferTime |                     |

## Seek Start (playbackSeekStart)

### Data

| Data          | Name         | Commnets            |
|---------------|--------------|---------------------|
| Event ID      | eventID      | `playbackSeekStart` |
| Is Playing    | isPlaying    |                     |
| Seek position | seekPosition | offset in seconds   |

## Seek End (playbackSeekStart)

### Data

| Data          | Name         | Commnets                             |
|---------------|--------------|--------------------------------------|
| Event ID      | eventID      | `playbackSeekStart`                  |
| Seek position | seekPosition | offset in seconds                    |
| Seek time     | seekTime     | in seconds, including buffering time |

## Playback end (playbackEnd)

### Data

| Data            | Name          | Commnets      |
|-----------------|---------------|---------------|
| Event ID        | eventID       | `playbackEnd` |
| Next episode id | nextEpisodeId | if relevant   |

## Playback playing (playbackPlaying)

This should be sent every 60 seconds as long as the player is playing.

### Data

| Data     | Name          | Commnets          |
|----------|---------------|-------------------|
| Event ID | eventID       | `playbackPlaying` |

