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

| Data            | Name           | Comments                                                                                    |
| --------------- | -------------- | ------------------------------------------------------------------------------------------- |
| OS              | _os_           | SDK automatically adds this. **Not the currently selected app-language.** See `appLanguage` |
| OS Locale       | _locale_       | SDK automatically adds.                                                                     |
| Device Info     | _deviceInfo_   | SDK automatically adds.                                                                     |
| Anonymous ID    | _anonymousId_  | SDK automatically adds.                                                                     |
| Timezone        | _timezone_     | SDK automatically adds.                                                                     |
| Screen data     | _screen_       | SDK automatically adds.                                                                     |
| Analytics ID.   | _userId_       | SDK adds after identify. For logged in users, see below.                                    |
| Online          | wasOnline      | `true` if the device was online at the tracking time                                        |
| Channel         | channel        | `mobile`/`web`/`tv`                                                                         |
| App Language    | appLanguage    | The currently selected language which can be changed via settings.                          |
| Release Version | releaseVersion | App version, or build number/git hash for web                                               |

### Person analytics ID

You can get this ID by sending an authenticated request to `https://API_DOMAIN/api/analitycsid`.
The returned value is an hash of the userID + a secret. This ensures that we are
able to link all data to one user from different systems, while preventing anyone
from being able to reverse the process or generate analytics IDs if they do not
posses the secret.

## Identify

_When_: On app launch && login
_Reason_: Being able to connect events based on the anonymousId (1 per device) to
the user that is logged in.

### API

Use the `/identify` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/identify

### Data

#### Signed in users

`userId`: Accessible from GQL under `me` endpoint

`traits`

| Data            | Name       | Comments                                                          |
| --------------- | ---------- | ----------------------------------------------------------------- |
| Unique user ID  | `id`       | Same as the `userId`                                              |
| Users exact age | `age`      | Currently not used. Need to figure out how to guarantee anonymity |
| Age Group       | `ageGroup` | See Age group List below                                          |
| Country         | `country`  | Two letter country code                                           |
| Church ID       | `church`   | Numerical church ID as STRING                                     |
| Gender          | `gender`   | male or female                                                    |

#### Age groups

Please use the exact strings below:

- UNKNOWN
- < 10
- 10 - 12
- 13 - 18
- 19 - 25
- 26 - 36
- 37 - 50
- 51 - 64
- 65+

#### Anonymous users

Calling identify for anonymous users isn't necessary unless we want to give them traits, but we don't need this for now.

## Screen/Page View

_When_: On every page/screen the user opens. If relevant also on popups and tabs
if they can be considered a separate screen (for example Participate tab on live).

_Reason_: It will allow us to see how used the various pages are and estimate how
much time users spend on various screens.

### API

Use the `/page` endpoint on web. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/page
Use the `/screen` endpoint in app. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/screen

### Data

The rest of the relevant additional data is automatically collected by the SDK on web. App does not
deliver additional data.

| Data              | Name     | Comments                                                                                        |
| ----------------- | -------- | ----------------------------------------------------------------------------------------------- |
| ID                | Id       | Code of the page if dynamic or from the list below. String                                      |
| Title             | title    | String, native language                                                                         |
| Dynamic page code | pageCode | if the page has a a related dynamic (backend-driven) page (e.g. `btv-search` for `search` page) |
| Additional info   | meta     | JSON. See below for `settings` example, but fields can be added as needed                       |

### Page/Screen IDs

- about
- calendar
- livestream
- login
- signup
- profileEdit
- profile
- search
- settings
- support
- faq
- episode
- signup
  - signup-email
  - signup-password
  - signup-name
  - signup-birthdate
  - signup-done

### Additional info

```json
{
  "setting": "appLanguage"
}
```

```json
{
  "setting": "audioLanguage"
}
```

```json
{
  "setting": "subtitlesLanguage"
}
```

Web:

```json
{
  "setting": "webSettings"
}
```

## Section click (section_clicked)

_When_: On every tap/click on a section element. This is on all sections, regardless
of where they may appear

_Reason_: This will allow us to asses how effective the sections are, as well as
possible debug purposes

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data             | Name            | Comments                                       |
| ---------------- | --------------- | ---------------------------------------------- |
| Event ID         | event           | Hardcoded: `section_clicked`                   |
| Section ID       | sectionId       | ID of the section that the element belongs to  |
| Section Name     | sectionName     | For easier identification in tools             |
| Section Position | sectionPosition | int, position in the page's list of sections   |
| Section Type     | sectionType     | `__typename` from GQL DefaultSection, ...      |
| Element Position | elementPosition | int, position in the section's list of items   |
| Element Type     | elementType     | `item.__typename` from GQL? Episode, Show, ... |
| Element ID       | elementId       | id of the clicked element                      |
| Element Name     | elementName     | localized name of the clicked element          |
| Page Code        | pageCode        | same ID as for page/screen tracking            |

## Audio Only (audioonly_clicked)

_When_: User clicks/taps the audio only button

_Reason_: Measure usage for the button

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data               | Name      | Comments                                    |
| ------------------ | --------- | ------------------------------------------- |
| Event ID           | event     | Hardcoded: `audioonly_clicked`              |
| Audio only enabled | audioOnly | Audio only state _after_ tap. true or false |

## Calendar day click (calendarday_clicked)

_When_: When user taps/clicks a day in the calendar to see transmission list.

_Reason_: Assessing usage, spotting problems (taps on things that don't lead anywhere)

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name         | Comments                            |
| ------------- | ------------ | ----------------------------------- |
| Event ID      | event        | Hardcoded: `calendarday_clicked`    |
| Page Name     | pageCode     | same ID as for page/screen tracking |
| Calendar view | calendarView | `week` or `month`                   |
| Calendar date | calendarDate | ISO-8601 (YYYY-MM-DD)               |

## Calendar click (calendarentry_clicked)

_When_: When user taps/clicks an element in the calendar

_Reason_: Assessing usage, spotting problems (taps on things that don't lead anywhere)

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name         | Comments                                      |
| ------------- | ------------ | --------------------------------------------- |
| Event ID      | event        | Hardcoded: `calendarentry_clicked`            |
| Page Code     | pageCode     | same code as for page/screen tracking         |
| Calendar view | calendarView | `week` or `month`                             |
| Calendar date | calendarDate | ISO-8601 (YYYY-MM-DD)                         |
| Element Id    | elementId    | If tap leads to an episode, record episode id |

## Search (search_performed)

_When_: When user performs a search

_Reason_: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data           | Name              | Comments                                           |
| -------------- | ----------------- | -------------------------------------------------- |
| Event ID       | event             | Hardcoded: `search_performed`                      |
| Search Term    | searchText        | exact search term                                  |
| Search Latency | searchLatency     | how long did the search take in milliseconds (int) |
| Result Count   | searchResultCount |                                                    |

## Search Result click (searchresult_clicked)

_When_: When user clicks/taps on a search result

_Reason_: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data             | Name            | Comments                           |
| ---------------- | --------------- | ---------------------------------- |
| Event ID         | event           | Hardcoded: `search_result_clicked` |
| Search Term      | searchText      | exact search term                  |
| Element Position | elementPosition | position in the list               |
| Element Type     | elementType     | episode, series, ...               |
| Element ID       | elementId       | series, episode ID                 |
| Group            | group           | "shows" or "episodes" currently    |

## Language Change (language_changed)

_When_: When user changes the language in any context

_Reason_: Spotting issues, Usage info

_Notes_: This is not high priority, if hard to get, can be omitted

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data          | Name               | Comments                      |
| ------------- | ------------------ | ----------------------------- |
| Event ID      | event              | Hardcoded: `language_changed` |
| From Language | languageFrom       |                               |
| To Language   | languageTo         |                               |
| Where         | languageChangeType | audio, app, subs              |

## Open (application_opened)

_When_: When the application is forgrounded. This may occur because the user tapped the app (icon) or
for a number of other reasons, such as tapping a deep link or notification.

_Reason_: Usage info

_Notes_: On tvOS there is only reason Default as links and notifications are not handled.

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data       | Name      | Comments                                                                              |
| ---------- | --------- | ------------------------------------------------------------------------------------- |
| Event ID   | event     | Hardcoded: `application_opened`                                                       |
| Reason     | reason    | Describes how application was opened. Possible values: Default, Notification, Link    |
| Cold Start | coldStart | `true` if the application was not running before (user started it). Otherwise `false` |

## Deep link (deep_link_opened)

_When_: When the application processes a deep link

_Reason_: Usage info, campaign tracking

_Notes_: Not applicable for tvOS

Some parameters should be extracted from the url if present:

| URL Parameter | Comment                                                      |
| ------------- | ------------------------------------------------------------ |
| cid           | This is for tracking a campaign ID                           |
| cs            | This is for tracking the source for example ig for instagram |

If not present they should report an empty string, not NULL!

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data        | Name       | Comments                            |
| ----------- | ---------- | ----------------------------------- |
| Event ID    | event      | Hardcoded: `deep_link_opened`       |
| URL         | url        | Full deep link                      |
| Source      | source     | cs parameter as described in notes  |
| Campaign ID | campaignId | cid parameter as described in notes |

## Share (content_shared)

_When_: When user shares content via the share menu

_Reason_: Spotting issues, Usage info

### API

Use `/track` endpoint. Docs: https://docs.rudderstack.com/rudderstack-api/api-specification/rudderstack-spec/track

### Data

| Data         | Name        | Comments                                                        |
| ------------ | ----------- | --------------------------------------------------------------- |
| Event ID     | event       | Hardcoded: `content_shared`                                     |
| Page Name    | pageCode    | same ID as for page/screen tracking                             |
| Element Type | elementType | episode, series, ...                                            |
| Element ID   | elementId   | series, episode ID                                              |
| position     | position    | playback position in **seconds**. `null` if shared without time |

## Social auth clicked (social_auth_clicked)

_When_: When user clicks a social auth button
_Reason_: Spotting issues, Usage info

### Data

| Data            | Name       | Comments                            |
| --------------- | ---------- | ----------------------------------- |
| Page Name       | pageCode   | same ID as for page/screen tracking |
| Auth connection | connection | google, facebook, apple, bcc        |

## Sign up "register" button clicked (signup_submitted)

_When_: When user finishes the sign up form by clicking "register"
_Reason_: Spotting issues, Usage info

### Data

| Data      | Name     | Comments                            |
| --------- | -------- | ----------------------------------- |
| Page Name | pageCode | same ID as for page/screen tracking |

## Other Tracking events

This tracking events do not have data besides the common fields. The list contains
the `EventID` values to be used

| Event              | Event ID           |
| ------------------ | ------------------ |
| Log In             | login              |
| Log Out            | logout             |
| Airplay started    | airplay_started    |
| ChromeCast started | chromecast_started |

## Video events

The video events should, in addition to the common data, also contain the following.

### Common video data table

The names are specified by Rudderstack and are inconsistent with the rest of the
document. They should be used as they are here (camelCase).

| Data             | Name              | Comments                                                                                                     |
| ---------------- | ----------------- | ------------------------------------------------------------------------------------------------------------ |
| Session ID       | sessionId         | regenerate after 30 minutes of not being used                                                                |
| Is Live          | livestream        | true/false                                                                                                   |
| Content ID       | contentPodId      | string                                                                                                       |
| Content Type     | contentPodType    | episode, program                                                                                             |
| Positing         | position          | From start of video in seconds                                                                               |
| Total Length     | totalLength       | null if live stream                                                                                          |
| Video Player     | videoPlayer       | Name of the player (VideoJS, ...)                                                                            |
| Is Full screen?  | fullScreen        |                                                                                                              |
| Quality          | quality           | if available                                                                                                 |
| Volume           | volume            | in %                                                                                                         |
| Has video        | hasVideo          | false if in "audio only mode"                                                                                |
| Subs             | subtitlesLanguage | null if not possible to determine. "OFF" if turned off. "N/A" if subs are not available in selected language |
| Audio            | audioLanguage     | null if not possible to determine. "N/A" if audio is not available in selected language                      |
| Is Chromecasting | isChromecasting   | true/false. null if not possible to determine.                                                               |
| Is Airplaying    | isAirPlay         | true/false. null if not possible to determine.                                                               |

## API

Use the `/track` API as described here: https://docs.rudderstack.com/rudderstack-api/api-specification/video-specification

## Standard video events

This events contain no extra data

| Event                 | Event ID                   |
| --------------------- | -------------------------- |
| Playback started      | playback_started           |
| Playback paused       | playback_paused            |
| Playback end          | playback_ended             |
| Playback interrupted  | playback_interrupted       |
| Playback buffer start | playback_buffering_started |

## Buffering end (playback_buffering_ended)

### Data

| Data                 | Name       | Comments                   |
| -------------------- | ---------- | -------------------------- |
| Event ID             | eventID    | `playback_buffering_ended` |
| Time spent Buffering | bufferTime | in seconds                 |

## Seek End (playback_seeking_ended)

**Note:** If the user clicks around multiple times as part of "one" seek, 'seekStartPosition' should be based on the first click. 'seekTime' however, should be based on the last click.

### Data

| Data       | Name              | Comments                             |
| ---------- | ----------------- | ------------------------------------ |
| Event ID   | eventID           | `playback_seeking_ended`             |
| Seek start | seekStartPosition | offset in seconds                    |
| Seek time  | seekTime          | in seconds, including buffering time |

## Playback playing (playback_playing)

This should be sent every 60 seconds as long as the player is playing.
Bitrate properties are AVPlayer specific feature so are available only for iOS and tvOS.
More info https://developer.apple.com/documentation/avfoundation/avplayeritemaccesslogevent

### Data

| Data                                | Name                             | Commnets           |
| ----------------------------------- | -------------------------------- | ------------------ |
| Event ID                            | eventID                          | `playback_playing` |
| Average audio bitrate               | averageAudioBitrate              |                    |
| Average video bitrate               | averageVideoBitrate              |                    |
| Indicated average bitrate           | indicatedAverageBitrate          |                    |
| Indicated bitrate                   | indicatedBitrate                 |                    |
| Observed bitrate                    | observedBitrate                  |                    |
| Observed bitrate standard deviation | observedBitrateStandardDeviation |                    |
| Switch bitrate                      | switchBitrate                    |                    |

## Notification received (notification_received)

Event occurs when notification is received on device.
It doesn't occur for tvOS where notifications are open immediately or stored by operating system.

### Data

| Data            | Name           | Commnets                         |
| --------------- | -------------- | -------------------------------- |
| Event ID        | eventID        | `notification_received`          |
| Notification id | notificationId | for tracking purposes            |
| action          | action         | `deep_link`, `clear_cache`, null |
| deeplink        | string         | if action is `deep_link`         |

## Notification received (notification_opened)

Event occurs when notification is opened by the user.
It doesn't occur for headless notifications that execute in background without user interaction like commands.

### Data

| Data            | Name           | Commnets              |
| --------------- | -------------- | --------------------- |
| Event ID        | eventID        | `notification_open`   |
| Notification id | notificationId | for tracking purposes |

## Game closed (game_closed)

Collected when a game is closed for any reason.

### Data

| Data       | Name      | Commnets               |
| ---------- | --------- | ---------------------- |
| Game ID    | gameId    | per-game stats         |
| Time spent | timeSpent | seconds. measure usage |

## Interaction event (interaction)

_When_: On granular events that doesnt deserve their own table.

_Reason_: These events are generally used for analyzing how the users use a particular feature of app, to see where we should focus our efforts.
E.g. are any users swiping horizontally where you can't (shorts), or do any users use that "like" button, etc.

### Data

| Data                 | Name               | Commnets                                                          |
| -------------------- | ------------------ | ----------------------------------------------------------------- |
| Interaction          | interaction        | 'play', 'pause', 'share', 'like', 'mute', etc.                    |
| Page code            | pageCode           | static ('shorts', 'episode') or dynamic ('kids-frontpage2', etc.) |
| Context Element Id   | contextElementId   | ID of e.g. the episode or short this was performed on.            |
| Context Element Type | contextElementType | type for the ID above, e.g. 'episode', 'short', 'show', etc.      |
| Additional info      | meta               | Arbitrary extra json meta.                                        |
