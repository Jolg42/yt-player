# yt-player [![travis][travis-image]][travis-url] [![npm][npm-image]][npm-url] [![downloads][downloads-image]][downloads-url]

[travis-image]: https://img.shields.io/travis/feross/yt-player/master.svg
[travis-url]: https://travis-ci.org/feross/yt-player
[npm-image]: https://img.shields.io/npm/v/yt-player.svg
[npm-url]: https://npmjs.org/package/yt-player
[downloads-image]: https://img.shields.io/npm/dm/yt-player.svg
[downloads-url]: https://npmjs.org/package/yt-player

### Simple, robust, blazing-fast YouTube Player API

## Install

```
npm install yt-player
```

## Why this package?

Most of the existing YouTube API packages on npm are incomplete or overcomplicated.

Note: If you publish a package that requires all of `babel-runtime`, `lodash`, and
20KB of other nonsense to do a simple task, you're an irresponsible programmer.
Using fancy dependencies doesn't make you a better programmer, and it's certainly
not a "best practice" to ship tons of extra code to your website visitors. Less is
more. 🔥

## Features

  - Powered by the [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference)
  - Lightweight - only 2.89kb gzipped and minified!
  - Extremely fast time-to-first-frame
    - YouTube IFrame API `<script>` is automatically loaded on first use
    - For even faster start time, add `<script src='https://www.youtube.com/iframe_api' async></script>` to your page
    - Automatically detects the presence of the API `<script>` so API is never loaded twice
  - API commands are automatically queued (until both the Iframe API and Player instance are ready)
  - Differentiate between *fatal* errors and *unplayable video* errors
  - Adds the all-important 'timeupdate' event, which the YouTube API lacks
  - Well-commented codebase makes it easy to understand what's going on
  - No promises, babel, async-await, large runtime dependencies, or pointless code
  - Note: no support for old browsers (e.g. IE11 and below)

## Usage

```js
const YTPlayer = require('yt-player')
const player = new YTPlayer('#player')

player.load('GKSRyLdjsPA')
player.setVolume(100)

player.on('playing', () => {
  console.log(player.getDuration()) // => 351.521
})
```

## API

### `player = new Player(selector, [opts])`

Create a new YouTube player at the DOM node matched by `selector`. Alternatively,
`selector` can be an instance of `HTMLElement` (i.e. a DOM node).

Examples: `#player`, `.youtube-player`, or a DOM node.

Optionally, provide an options object `opts` to customize the player.

#### `opts.width` (number)

This parameter indicates the width of the player.

#### `opts.height` (number)

This parameter indicates the height of the player.

#### `opts.autoplay` (boolean)

This parameter indicates whether the initial video will automatically start to play
when the player loads. The default value is `false`.

#### `opts.captions` (boolean)

This parameter indicates whether closed captions should be shown, even if the user
has turned captions off. The default behavior is based on user preference.

#### `opts.controls` (boolean)

This parameter indicates whether the video player controls are displayed. The
default value is `true`.

#### `opts.keyboard` (boolean)

This parameter indicates whether the player will respond to keyboard shortcuts. The
default value is `true`.

#### `opts.fullscreen` (boolean)

This parameter indicates whether the player will show a fullscreen button. The
default value is `true`.

#### `opts.annotations` (boolean)

This parameter indicates whether the player will show video annotations. The
default value is `true`.

#### `opts.modestBranding` (boolean)

This parameter lets you use a YouTube player that does not show a YouTube logo.
Even when this option is enabled, a small YouTube text label will still display in
the upper-right corner of a paused video when the user's mouse pointer hovers over
the player. The default value is `false`.

#### `opts.related` (boolean)

This parameter indicates whether the player should show related videos when
playback of the initial video ends. The default value is `true`.

#### `opts.info` (boolean)

This parameter indicates whether the player should display information like the
video title and uploader before the video starts playing. The default value is
`true`.

#### `opts.timeupdateFrequency` (number)

The time between `onTimeupdate` callbacks, in milliseconds. Default is `1000`.

See:
[`YT.Player` parameters](https://developers.google.com/youtube/iframe_api_reference#Loading_a_Video_Player)
and
[`playerVars` parameters](https://developers.google.com/youtube/player_parameters#Parameters)
for additional documentation about these parameters.

### `player.load(videoId)`

This function loads and plays the specified `videoId`. An example of a `videoId`
is `'GKSRyLdjsPA'`.

This should be the first function called on a new `Player` instance.

### `player.play()`

Plays the currently loaded video.

### `player.pause()`

Pauses the currently playing video.

### `player.seek()`

Seeks to a specified time in the video. If the player is paused when the function
is called, it will remain paused. If the function is called from another state
(playing, video cued, etc.), the player will play the video.

- The `seconds` parameter identifies the time to which the player should advance.

  The player will advance to the closest keyframe before that time unless the player has already downloaded the portion of the video to which the user is seeking.

- The `allowSeekAhead` parameter determines whether the player will make a new
  request to the server if the seconds parameter specifies a time outside of the
  currently buffered video data.

  We recommend that you set this parameter to `false` while the user drags the mouse
  along a video progress bar and then set it to `true` when the user releases the
  mouse. This approach lets a user scroll to different points of a video without
  requesting new video streams by scrolling past unbuffered points in the video.
  When the user releases the mouse button, the player advances to the desired point
  in the video and requests a new video stream if necessary.

### `player.setVolume(volume)`

Sets the volume. Accepts an integer between `0` and `100`.

### `player.getDuration()`

Returns the duration in seconds of the currently playing video. Note that
`getDuration()` will return 0 until the video's metadata is loaded, which normally
happens just after the video starts playing.

### `player.getCurrentTime()`

Returns the elapsed time in seconds since the video started playing.

### `player.getState()`

Returns the state of the player. Possible values are: `'unstarted'`, `'ended'`,
`'playing'`, `'paused'`, `'buffering'`, or `'cued'`.

### `player.destroy()`

Removes the `<iframe>` containing the player and cleans up all resources.

### `player.on('error', (err) => {})`

This event fires if a fatal error occurs in the player. This does not include
videos that fail to play, for whatever reason.

### `player.on('unplayable', (videoId) => {})`

This event fires if the YouTube player cannot play the given video. This is not a
fatal error. This event is reported separately from the `'error'` event so
there is an opportunity to play another video gracefully.

Possible reasons for this error:

- The video requested was not found. This error occurs when a video has been
  removed (for any reason) or has been marked as private.

- The owner of the requested video does not allow it to be played in embedded
  players.

- The request contains an invalid parameter value. For example, this error occurs
  if you specify a videoId that does not have 11 characters, or if the videoId contains invalid characters, such as exclamation points or asterisks.

### `player.on('timeupdate', (seconds) => {})`

This event fires when the time indicated by the `getCurrentTime()` method has been
updated.

### `player.on('unstarted', () => {})`
### `player.on('ended', () => {})`
### `player.on('playing', () => {})`
### `player.on('paused', () => {})`
### `player.on('buffering', () => {})`
### `player.on('cued', () => {})`

These events fire when the player enters the respective state. These event names
are the same as the possible return values from `player.getState()`.

### `player.on('playbackQualityChange', (quality) => {})`

This event fires whenever the video playback quality changes. Possible
values are: 'small', 'medium', 'large', 'hd720', 'hd1080', 'highres'.

### `player.on('playbackRateChange', (playbackRate) => {})`

This event fires whenever the video playback rate changes.

## License

MIT. Copyright (c) [Feross Aboukhadijeh](http://feross.org).
