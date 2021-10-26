# Technical details for the tracking plan

## Implementation notes

The analytics should be collected (unless otherwise noted), even when the client
is offline. The data should be cached as far as possible and sent to the server
when the device is back online. If caching more data becomes impossible, the oldest
cached data should be deleted. The deletion is discussed further later in the doc.

### Cached events

We should be able to cache at least 3 weeks of events (Africa use case),
If we assume that the user is watching 1 hours every day on a device like that,
that would mean `21 * 60 = 1260` "playing" events in a week. Adding 20% for other
events we end up at roughly `1500` events per week.

Thus we should be able to cache ca 5000 events.

If data still needs to be deleted from cache, it should be deleted a day at a time,
starting with the oldest. In exchange, to help us track this events, an event, with
the name `analytics_cache_deleted` should be cached.

## Common data

The data specified below should be sent with every event, unless otherwise noted.
If it is not possible for all events then that should be noted in this document.

In general it is better to omit data (if for example it is not accessible on some
os version), and send what you can.

Note that this is for both APP and Web so some things may be possible on one
platform and not on the other.

### Common data table

| Data                | Name           | Comments                                                                                      |
|---------------------|----------------|-----------------------------------------------------------------------------------------------|
| OS                  | os             | `DeviceInfo.Platform`                                                                         |
| OS Locale           | locale         | `CultureInfo.CurrentUICulture.TextInfo`                                                       |
| Online              | wasOnline      | `true` if the device was online at the tracking time                                          |
| Device Info         | deviceInfo     | `DeviceInfo` as json                                                                          |
| Anonymous ID        | anonymousId    | `SetAnonymousId(Guid.NewGuid().ToString())`. If changed a new call to `identify` must be made |
| Channel             | channel        | `mobile`/`web`/`tv`                                                                           |
| Timezone            | timezone       | `new DateTimeOffset(DateTime.Now).Offset`                                                     |
| Screen data         | screen         | `DeviceDisplay.MainDisplayInfo` as json                                                       |
| App Language        | appLanguage    |                                                                                               |
| Release Version     | releaseVersion | App version, or build number/git hash for web                                                 |
| Person analytics ID | presonId       | For logged in users, see below                                                                |

### Person analytics ID

You can get this ID by sending an authenticated request to `https://API_DOMAIN/api/analitycsid`.
The returned value is an hash of the userID + a secret. This ensures that we are
able to link all data to one user from different systems, while preventing anyone
from being able to reverse the process or generate analytics IDs if they do not
posses the secret.

## Identify

*When*: On login
In addition this call should be made when user starts the application and is already
logged in, but has not sent an identify! This is so the existing users do not have to
login again.
*Reason*: Being able to connect events based on the anonymousId (1 per device) to
the user that is logged in.

### API

Use the `/identify` endpoint. Docs:			https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/identify

### Data

#### Signed in users

For the `userId`, pass in the analyticsId. For `traits` send the personId as `personId` and nothing more.
This call accepts much more data, but we do not want to send it here. When you pass the "personId" as a trait, the additional
data will be automatically injected by RudderStack, and delivered to the needed targets.
Example javascript code:

```js
var analyticsId = getAnalyticsId();
var personId = getPersonId();
rudderanalytics.identify(analyticsId, {
  personId: personId
});
```

#### Anonymous users
Calling identify for anonymous users isn't necessary unless we want to give them traits, but we don't need this for now.

## Screen/Page View

*When*: On every page/screen the user opens. If relevant also on popups and tabs
if they can be considered a separate screen (for example Participate tab on live).

*Reason*: It will allow us to see how used the various pages are and estimate how
much time users spend on various screens.

### API

Use the `/page` endpoint on web. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/page
Use the `/screen` endpoint in app. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/screen

### Data

The rest of the relevant additional data is automatically collected by the SDK on web. App does not
deliver additional data.

| Data         | Name        | Comments                                       |
|--------------|-------------|------------------------------------------------|
| Name         | name        | A page identifier. See comments and list below |
| Element Type | elementType | episode, series, ...                           |
| Element ID   | elementId   | series, episode ID                             |

### Page/Screen IDs

This are the currently used screens from the app:

* AboutPage
* AudiencePage
* CalendarPage
* ExplorePage
* InfoPage
* LiveStreamPage
* LoginPage
* NativePlayer
* ProfileEditPage
* ProfilePage
* ProgramPage
* QueuePage
* SearchPage
* SeriesPage
* SettingsListPage
* SupportPage

## Section click (section_clicked)

*When*: On every tap/click on a section element. This is on all sections, regardless
of where they may appear

*Reason*: This will allow us to asses how effective the sections are, as well as
possible debug purposes

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data             | Name            | Comments                                      |
|------------------|-----------------|-----------------------------------------------|
| Event ID         | event           | Hardcoded: `section_clicked`                  |
| Section ID       | sectionId       | ID of the section that the element belongs to |
| Section Name     | sectionName     | For easier identification in tools            |
| Section Position | sectionPosition | int, position in the page's list of sections  |
| Section Type     | sectionType     | slider, featured, etc                         |
| Element Position | elementPosition | int, position in the section's list of items  |
| Element Type     | elementType     | episode, series, ...                          |
| Element ID       | elementId       | id of the clicked element                     |
| See All          | seeAll          | true if the tapped element was "See All"      |
| Page Name        | pageName        | same ID as for page/screen tracking           |

## Liveboard click (liveboard_clicked)

*When*: User clicks an element in the liveboard. This is for all screens where
liveboard appears (or may appear in the future).

*Reason*: Better understanding about how the modules are user, and how changes
in order, etc affect the use. Can also be used for debug purposes.

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data            | Name           | Comments                                           |
|-----------------|----------------|----------------------------------------------------|
| Event ID        | event          | Hardcoded: `liveboard_clicked`                     |
| Module ID       | moduleId       | from Firestore                                     |
| Module Type     | moduleType     | from Firestore                                     |
| Module Position | modulePosition | top module == 1 (easier logic for non-tech people) |
| Event ID        | eventId        |                                                    |
| Event Name      | eventName      | Optional                                           |

## Audio Only (video_toggle)

*When*: User clicks/taps the audio only button

*Reason*: Measure usage for the button

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data        | Name             | Comments                     |
|-------------|------------------|------------------------------|
| Event ID    | event            | Hardcoded: `video_toggle`    |
| Video state | videoStateTarget | Video state *after* tap      |


## Calendar click (calendar_clicked)

*When*: When user taps/clicks an element in the calendar

*Reason*: Assessing usage, spotting problems (taps on things that don't lead anywhere)

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name         | Comments                                      |
|---------------|--------------|-----------------------------------------------|
| Event ID      | event        | Hardcoded: `calendar_clicked`                 |
| Page Name     | pageName     | same ID as for page/screen tracking           |
| Calendar view | calendarView | `week` or `month`                             |
| Calendar date | calendarDate | ISO-8601 (YYYY-MM-DD)                         |
| Element Id    | elementId    | If tap leads to an episode, record episode id |

## Search (search_performed)

*When*: When user performs a search

*Reason*: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data           | Name              | Comments                                           |
|----------------|-------------------|----------------------------------------------------|
| Event ID       | event             | Hardcoded: `search_performed`                      |
| Search Term    | searchText        | exact search term                                  |
| Search Latency | searchLatency     | how long did the search take in milliseconds (int) |
| Result Count   | searchResultCount |                                                    |


## Search Result click (search_result_clicked)

*When*: When user clicks/taps on a search result

*Reason*: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data            | Name                 | Comments                           |
|-----------------|----------------------|------------------------------------|
| Event ID        | event                | Hardcoded: `search_result_clicked` |
| Search Term     | searchText           | exact search term                  |
| Result Position | searchResultPosition | how long did the search take       |
| Element Type    | elementType          | episode, series, ...               |
| Element ID      | elementId            | series, episode ID                 |

## Language Change (language_changed)

*When*: When user changes the language in any context

*Reason*: Spotting issues, Usage info

*Notes*: This is not high priority, if hard to get, can be omitted

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name               | Comments                            |
|---------------|--------------------|-------------------------------------|
| Event ID      | event              | Hardcoded: `language_changed`       |
| Page Name     | pageName           | same ID as for page/screen tracking |
| From Language | languageFrom       |                                     |
| To Language   | languageTo         |                                     |
| Where         | languageChangeType | audio, app, subs                    |

## Share (content_shared)

*When*: When user shares content via the share menu

*Reason*: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name               | Comments                            |
|---------------|--------------------|-------------------------------------|
| Event ID      | event              | Hardcoded: `content_shared`         |
| Page Name     | pageName           | same ID as for page/screen tracking |
| Element Type  | elementType        | episode, series, ...                |
| Element ID    | elementId          | series, episode ID                  |

## Play next (play_next)

*When*: When play next feature selects next episode on the list.

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data               | Name             | Comments                                 |
|--------------------|------------------|------------------------------------------|
| Event ID           | eventID          | Hardcoded: `play_next`                   |
| Current episode id | currentEpisodeId | id of the episode that just has finished |
| Next episode id    | nextEpisodeId    | id of the episode that will be played    |

## Other Tracking events

This tracking events do not have data besides the common fields. The list contains
the `EventID` values to be used

| Event              | Event ID           |
|--------------------|--------------------|
| Log In             | login              |
| Log Out            | logout             |
| Airplay started    | airplay_started    |
| ChromeCast started | chromecast_started |

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
| Audio           | audioLanguage     | null if not possible to determine. "N/A" if audio is not available in selected language |

## API

Use the `/track` API as described here: https://docs.rudderstack.com/rudderstack-api/api-specification/video-specification

## Standard video events

This events contain no extra data

| Event                 | Event ID                   |
|-----------------------|----------------------------|
| Playback started      | playback_started           |
| Playback paused       | playback_paused            |
| Playback end          | playback_ended             |
| Playback interrupted  | playback_interrupted       |
| Playback buffer start | playback_buffering_started |

## Buffering end (playback_buffering_ended)

### Data

| Data                 | Name       | Comments                   |
|----------------------|------------|----------------------------|
| Event ID             | eventID    | `playback_buffering_ended` |
| Time spent Buffering | bufferTime |                            |

## Seek End (playback_seeking_ended)

**Note:** If the user clicks around multiple times as part of "one" seek, 'seekStartPosition' should be based on the first click. 'seekTime' however, should be based on the last click.

### Data

| Data          | Name              | Comments                             |
|---------------|-------------------|--------------------------------------|
| Event ID      | eventID           | `playback_seeking_ended`                |
| Seek start    | seekStartPosition | offset in seconds                    |
| Seek time     | seekTime          | in seconds, including buffering time |

## Playback playing (playback_playing)

This should be sent every 60 seconds as long as the player is playing.

### Data

| Data     | Name    | Commnets           |
|----------|---------|--------------------|
| Event ID | eventID | `playback_playing` |

