The Twilio Programmable Video SDKs use [Semantic Versioning](http://www.semver.org).

### 5.0.0-beta4

#### Network Quality API

- Implemented the Network Quality functionality for Group Rooms:
- The Network Quality feature is disabled by default, to enable it set the `ConnectOptions.Builder.enableNetworkQuality` property to `true` when connecting to a Group Room.
    - To determine the current network quality level for your Local Participant, call `LocalParticipant.getNetworkQualityLevel()`. Note, this will return `NETWORK_QUALITY_LEVEL_UNKNOWN` if:
        - The `ConnectOptions.networkQualityEnabled` property was set to `false`
        - Using a Peer-to-Peer room
        - The network quality level has not yet been computed
    - Network Quality Level for Remote Participants will be available in a future release
    - Implementing the `onNetworkQualityLevelChanged` method on your `LocalParticipant.Listener` will allow you to receive callbacks when the network quality level changes

```.java
// Enable network quality
ConnectOptions connectOptions =
                new ConnectOptions.Builder(token)
                        .roomName(roomName)
                        .enableNetworkQuality(true)
                        .build();

// Override onNetworkLevelChanged to observe network quality level changes
LocalParticipant.Listener localParticipantListener = new LocalParticipant.Listener() {
    ...

    @Override
    public void onNetworkQualityLevelChanged(
        @NonNull LocalParticipant localParticipant,
        @NonNull NetworkQualityLevel networkQualityLevel) {}
}

// Connect to room and register listener
Room room = Video.connect(context, connectOptions, roomListener);
LocalParticipant localParticipant = room.getLocalParticipant();
localParticipant.setListener(localParticipantListener);

// Get current network quality
localParticipant.getNetworkQualityLevel();
```

API Changes

- When a Participant connects to a `Room`, the WebSocket handshake is required to complete in 15 seconds or less, otherwise `TwilioException.SIGNALING_CONNECTION_ERROR_EXCEPTION` is raised.
- Added `LocalParticipant.signalingRegion`. You can use this property to determine where your Participant connected to a Room, especially when using the default value of "gll" for `ConnectOptions.Builder.region`.
- Added `Room.mediaRegion`. You can use this property to determine where media is being processed in a Group Room.
- `EncodingParameters` now expresses maximum bitrates in Kilobits per second (Kbps) instead of bits per second (bps).

```
// Before
EncodingParameters encodingParameters = new EncodingParameters(64000, 800000);
```

```
// After
EncodingParameters encodingParameters = new EncodingParameters(64, 800);
```

Enhancements

- Increased the reliability of `LocalDataTrack` by monitoring its send buffer. Sending too many messages, or sending single messages larger than 16 KB no longer causes the underlying channel(s) to close. These requests are ignored instead.

Bug Fixes

- `LocalDataTrack` should no longer end up in an inconsistent state after attempting to send a message larger than 16 KB.

Known issues

- In a two Participant P2P room, the last Participant might not be disconnected when there are terminal media failures.
- Only Constrained Baseline Profile is supported when H.264 is the preferred video codec.
- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Server side deflate compression is disabled due to occasional errors when reading messages.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 5.0.0-beta3

## Dominant Speaker Detection API

The Dominant Speaker Detection API sends events to your application every time the dominant speaker changes. You can use those events to improve the end user's experience by, for example, highlighting which participant is currently talking.

The Dominant Speaker Detection API is only available for Group Rooms.  To enable dominant speaker detection, set the `ConnectOptions.dominantSpeakerEnabled` property to `true`. Use `Room.getDominantSpeaker()` to determine the current dominant speaker. Implement `Room.Listener.onDominantSpeakerChanged()` method to receive callbacks when the dominant speaker changes.

For more information, refer to the [API docs](https://twilio.github.io/twilio-video-android/docs/5.0.0-beta3/) and to the [dominant speaker tutorial](https://www.twilio.com/docs/video/detecting-dominant-speaker)

```.java
 ConnectOptions connectOptions =
                new ConnectOptions.Builder(token)
                        .roomName(roomName)
                        .enableDominantSpeaker(true)
                        .build();
Room room = Video.connect(context, connectOptions, roomListener);

@Override
void onDominantSpeakerChanged(
                @NonNull Room room, @Nullable RemoteParticipant remoteParticipant) {
                // Handle dominant speaker change
        }

```

API Changes

- Introduced `TwilioException.SIGNALING_DNS_RESOLUTION_ERROR_EXCEPTION`, which is now raised instead of `TwilioException.SIGNALING_CONNECTION_ERROR_EXCEPTION` in the following scenarios:
  - The device has misconfigured DNS Server(s) on its active network interface.
  - The region provided in `ConnectOptions` was invalid.
  - The device lost its Internet connection before the query could complete.


Enhancements

- Reduced connection times by removing a round trip when:
  - Reconnecting after a signaling connection failure
  - Connecting with `ConnectOptions.iceOptions`, and overridden Servers
  - Connecting with default ICE servers (us1 only)

Bug Fixes

- Fixed crash that occurred when rapidly connecting and disconnecting from a room.
- Fixed updating `CameraCapturer.State` when error occurs.
- Setting `ConnectOptions.region` to an empty or null value results in the default region being
used.
- Fixed a bug where native memory was leaked after disconnecting from a `Room`.
- Fixed a bug where network monitoring would continue on closed connections in a Peer-to-Peer Room.
- Fixed a bug where Participants remain connected to a Room even after terminal media failures.

Known issues

- In rare cases, the SDK might timeout during a TCP handshake and should be more aggressive at establishing a connection.
- Only Constrained Baseline Profile is supported when H.264 is the preferred video codec.
- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Server side deflate compression is disabled due to occasional errors when reading messages.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 5.0.0-beta2

Improvements

- The Participant's signaling connection now conforms to Twilio's [TLS & Cipher Suite Policy](https://support.twilio.com/hc/en-us/articles/360007724794-Notice-Twilio-REST-API-s-TLS-and-Cipher-Suite-Security-Changes-for-June-2019). Support for TLS versions older than 1.2 has been removed.
- Adjusted the buffer sizes used for signaling messages to reduce network fragmentation.
- Setting `video::LogModule::kSignaling` enables logging of low-level connection events.

Bug Fixes

- WebSocket errors are handled immediately, rather than waiting for a timeout to occur.
- Handle rare exceptions when constructing a WebSocket.

Known issues

- Future 5.0.0-beta releases will reduce the number of round-trips required to connect to a Room.
- In rare cases, the SDK might timeout during a TCP handshake and should be more aggressive at establishing a connection.
- Only Constrained Baseline Profile is supported when H.264 is the preferred video codec.
- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Server side deflate compression is disabled due to occasional errors when reading messages.
- Rapidly connecting and disconnecting from a `Room` may cause a crash.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 5.0.0-beta1

Improvements

- The SDK uses a new WebSocket based signaling transport, and communicates with globally available signaling Servers over IPv4 and IPv6 networks.
- Added a ConnectOptions.region property. By default, the Client will connect to the nearest signaling Server determined by latency based routing. Setting a value other than "gll" bypasses routing and guarantees that signaling traffic will be terminated in the region that you prefer.
- Participants are considered to be reconnecting within 15 seconds, and are disconnected from a Room after 45 seconds of lost connectivity. [#80](https://github.com/twilio/video-quickstart-android/issues/80)
- Added and updated public API nullability annotations.

Bug Fixes

- Participants can send messages that are larger than 16 KB.
- The “roomimpl.worker” thread is no longer needed.

Known issues

- Setting `LogModule.SIGNALING` does not produce any logging.
- Future 5.0.0-beta releases will reduce the number of round-trips required to connect to a Room.
- In rare cases, the SDK might timeout during a TCP handshake and should be more aggressive at establishing a connection.
- Only Constrained Baseline Profile is supported when H.264 is the preferred video codec.
- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 4.4.0

API Changes

#### Network Quality API

- Implemented the Network Quality functionality for Group Rooms:
- The Network Quality feature is disabled by default, to enable it set the `ConnectOptions.Builder.enableNetworkQuality` property to `true` when connecting to a Group Room.
    - To determine the current network quality level for your Local Participant, call `LocalParticipant.getNetworkQualityLevel()`. Note, this will return `NETWORK_QUALITY_LEVEL_UNKNOWN` if:
        - The `ConnectOptions.networkQualityEnabled` property was set to `false`
        - Using a Peer-to-Peer room
        - The network quality level has not yet been computed
    - Network Quality Level for Remote Participants will be available in a future release
    - Implementing the `onNetworkQualityLevelChanged` method on your `LocalParticipant.Listener` will allow you to receive callbacks when the network quality level changes

```.java
// Enable network quality
ConnectOptions connectOptions =
                new ConnectOptions.Builder(token)
                        .roomName(roomName)
                        .enableNetworkQuality(true)
                        .build();

// Override onNetworkLevelChanged to observe network quality level changes
LocalParticipant.Listener localParticipantListener = new LocalParticipant.Listener() {
    ...

    @Override
    public void onNetworkQualityLevelChanged(
        @NonNull LocalParticipant localParticipant,
        @NonNull NetworkQualityLevel networkQualityLevel) {}
}

// Connect to room and register listener
Room room = Video.connect(context, connectOptions, roomListener);
LocalParticipant localParticipant = room.getLocalParticipant();
localParticipant.setListener(localParticipantListener);

// Get current network quality
localParticipant.getNetworkQualityLevel();
```

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 4.3.2

Enhancement

- Upgraded the Gradle plugin used to build the SDK to 3.5.0.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 4.3.1

Bug Fixes

- Fixed updating `CameraCapturer.State` when error occurs.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 4.3.0

API Changes

## Dominant Speaker Detection API

The Dominant Speaker Detection API sends events to your application every time the dominant speaker changes. You can use those events to improve the end user's experience by, for example, highlighting which participant is currently talking.

The Dominant Speaker Detection API is only available for Group Rooms.  To enable dominant speaker detection, set the `ConnectOptions.dominantSpeakerEnabled` property to `true`. Use `Room.getDominantSpeaker()` to determine the current dominant speaker. Implement `Room.Listener.onDominantSpeakerChanged()` method to receive callbacks when the dominant speaker changes.

For more information, refer to the [API docs](https://twilio.github.io/twilio-video-android/docs/latest/) and to the [dominant speaker tutorial](https://www.twilio.com/docs/video/detecting-dominant-speaker)

This release is available [here](https://bintray.com/twilio/releases/video-android/4.3.0)

```.java
 ConnectOptions connectOptions =
                new ConnectOptions.Builder(token)
                        .roomName(roomName)
                        .enableDominantSpeaker(true)
                        .build();
Room room = Video.connect(context, connectOptions, roomListener);

@Override
void onDominantSpeakerChanged(
                @NonNull Room room, @Nullable RemoteParticipant remoteParticipant) {
                // Handle dominant speaker change
        }

```

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.
- Rapidly connecting and disconnecting from a `Room` may cause a crash.
- Using Camera2Capturer with a camera ID that does not support ImageFormat.PRIVATE capture outputs results in a runtime exception. Reference [this](https://github.com/twilio/video-quickstart-android/issues/431) issue for guidance on a temporary work around.

### 4.2.0
 Improvements

- Added a new property `ConnectOptions.enableAutomaticSubscription` to control Track subscription behavior in Group Rooms:
  - Selecting `true` (the default value) causes the Participant to be subscribed to all Tracks that are published in the Room
  - Selecting `false` causes the Participant to be subscribed to none of the Tracks that are published in the Room
  - Selecting `false` has no impact in a Peer-to-Peer Room

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

#### 4.1.2

Bug Fixes

- Fixed stuck aspect ratio when switching orientations with `ScreenCapturer`.
- Fixed a bug where media reconnection might fail when the loopback interface is mistakenly chosen as a preferred network interface.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

#### 4.1.1

Improvements

- Fixed various Lint errors / warnings.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

#### 4.1.0
Improvements

- Added the `AudioSink` interface. When added to a `LocalAudioTrack` or `RemoteAudioTrack`, developers will receive a callback with the raw audio byte buffer from each audio frame. The example below demonstrates adding an `AudioSink` to a `LocalAudioTrack`.

```
AudioSink audioSink = (audioSample, encoding, sampleRate, channels) -> {
    Log.v("AudioSink", "Received audio frame");
};
localAudioTrack.addSink(audioSink);
```
The `AudioSink` can then be removed with the following code.
```
localAudioTrack.removeSink(audioSink);
```

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

#### 4.0.2

Bug Fixes

- Fixed bug in Camera2Capturer.switchCamera which resulted in onCameraSwitched being invoked with a null camera ID [#379](https://github.com/twilio/video-quickstart-android/issues/379)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

#### 4.0.1

Bug Fixes

- `Room.getStats()` will raise a callback to the provided `StatsObserver` if called while `Room.getState() == Room.State.RECONNECTING`.
- Fixed a crash related to stats gathering which could occur when insights reporting is enabled.
- Fixed a crash related to media state summarization which could occur when disconnecting from a Room.
- Fixed a crash related to stats gathering which could occur in the media monitor component.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants. As a result, tracks published after a `Room.State.RECONNECTED` event might not be subscribed to by a `RemoteParticipant`.

###4.0.0

Improvements

- Added new state `Reconnecting` to `Room.State` and new callbacks `onReconnecting`, `onReconnected` to `Room.Listener`. When the `LocalParticipant` experiences a network interruption in signaling or media, the room will transition to `Reconnecting` and `Room.Listener` will notify the developer of this new state via `onReconnecting`. If the connection is successfully restored, `Room.Listener` will notify the developer via `onReconnected`. However, if the connection could not be reestablished `Room.Listener` will notify the developer via `onDisconnected`.
- Added and updated public API nullability annotations.
- Changed `OpusCodec.NAME` and `IsacCodec.NAME` to lowercase

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9
- Unpublishing and republishing a `LocalAudioTrack` or `LocalVideoTrack` might not be seen by Participants.

###3.2.2

Improvements

- Fix Screenshare being negotiated as simulcast without the client actually using simulcast.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###3.2.1
Bug Fixes

- Fixed ICE connection failure after multiple network handovers

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###3.2.0
Improvements

- Added `IceCandidateStats` to `StatsReport` which gives insight into individual ice candidates
- Removed forced ice-restart when synced responses are processed
- Improved the detection of, and recovery from media connection failures. Worst case recovery times have been reduced by up to 10 seconds.
- Added handle to native audio track. This will assist developers when using advanced features like audio track sink.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###3.1.0

Improvements

- Added `IceCandidatePairStats` to `StatsReport` which gives insight into local and remote ice candidates
- Reduced signaling traffic in Group Rooms when communicating with an ICE-lite agent.

 Bug Fixes

- Fixed a bug where the client could send additional ICE candidates after signaling the end of candidates, or mark candidates with a username from a future offer.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###3.0.0

`3.0.0-beta1` has been promoted to `3.0.0`. The `3.x` SDK is now Generally Available (GA). The `1.x` and `2.x` Video Android SDK remain GA but will only receive critical bug fixes moving forward.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###3.0.0-beta1

- `3.0.0-preview3` has been promoted to `3.0.0-beta1`. The `3.0.0` API is now final and will only receive bug fixes and improvements until `3.0.0` is officially released.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

####3.0.0-preview3

Enhancements

- The signaling Client has been updated to support version 2 of the Room Signaling Protocol (RSP).
- Added ECS debugging support at the signaling layer that will help in troubleshooting ice server related issues.
- Introduced a safer threading model in the signaling layer that prevents occasional crashes and missing Track subscription events.
- The SDK no longer waits for pending events when closing a PeerConnection speeding up the teardown process.
- Moved `RoomState` enum to `Room.State`.
- `onDisconnected` will now return a unique error code when the Room is completed via the REST API.

Bug Fixes

- The signaling Client is more lenient when encountering unexpected RSP messages.
- Resolved a scenario where an ICE restart could be ignored after experiencing signaling glare.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

####3.0.0-preview2

Features

- Added flag to IceOptions `abortOnIceServerTimeout` that tells the client whether to continue or abort connecting to a Room when Ice fails to connect.
- Added parameter to IceOptions `iceServersTimeout` that allows control of the timeout period when trying to retrieve Ice servers.
- Added `VideoTextureView`. `VideoTextureView` is similar to `VideoView` but subclasses `TextureView` instead of `SurfaceView`. Unlike SurfaceView, TextureView does not create a separate window but behaves as a regular View. This key difference allows a TextureView to be moved, transformed, animated, etc. For more see the [TextureView documentation](https://developer.android.com/reference/android/view/TextureView.html). If you were previously using [this gist](https://gist.github.com/aaalaniz/bfbc4891c98ef3b23558ff2260cbcc8e), please update your applications to use the `VideoTextureView` provided with the SDK. **NOTE**: `VideoTextureView` can experience dead locking on API Level 19 or below due to a [WebRTC bug](https://bugs.chromium.org/p/webrtc/issues/detail?id=5702). Use with discretion.

Enhancements

- Adding a null video/audio codec to `ConnectOptions` will now throw an exception
- Added `@NonNull` `@Nullable` annotations to a few additional classes.
- Updated `Room.Listener` documentation to provide more clarity about when `onRecording` callbacks are received.
- Switched CI provider for build, test, and release pipeline.

Bug Fixes

- Fixed crash when disconnecting from a room.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

####3.0.0-preview1

Improvements

- Upgraded to WebRTC 67.
- Upgraded to Java 8. Add the following snippet to your `build.gradle` to enable Java 8
features required to build the 3.x Video Android SDK:

        android {
            compileOptions {
                sourceCompatibility 1.8
                targetCompatibility 1.8
            }
        }
- Added @Nullable and @NonNull to all public API fields, methods, and interfaces.
- Updated internal capturer pipeline for `CameraCapturer`, `ScreenCapturer`, and `Camera2Capturer`.
- Upgraded to Android NDK 16.


Bug Fixes

- The SDK is compatible with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- The SDK is compatible with Firefox 63+ in a Peer-to-Peer Room. [#35](https://github.com/twilio/video-quickstart-android/issues/377)


Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Codec preferences do not function correctly in a hybrid codec Group Room with the following
codecs:
    - ISAC
    - PCMA
    - G722
    - VP9

###2.2.1

Improvements

- Adding a null video/audio codec to `ConnectOptions` will now throw an exception
- Switched CI provider for build, test, and release pipeline.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.2.0

Features

- Added flag to IceOptions `abortOnIceServerTimeout` that tells the client whether to continue or abort connecting to a Room when Ice fails to connect.
- Added parameter to IceOptions `iceServersTimeout` that allows control of the timeout period when trying to retrieve Ice servers.
- Added `VideoTextureView`. `VideoTextureView` is similar to `VideoView` but subclasses `TextureView` instead of `SurfaceView`. Unlike SurfaceView, TextureView does not create a separate window but behaves as a regular View. This key difference allows a TextureView to be moved, transformed, animated, etc. For more see the [TextureView documentation](https://developer.android.com/reference/android/view/TextureView.html). If you were previously using [this gist](https://gist.github.com/aaalaniz/bfbc4891c98ef3b23558ff2260cbcc8e), please update your applications to use the `VideoTextureView` provided with the SDK. **NOTE**: `VideoTextureView` can experience dead locking on API Level 19 or below due to a [WebRTC bug](https://bugs.chromium.org/p/webrtc/issues/detail?id=5702). Use with discretion.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.1.1

Improvements

- Updated `Room.Listener` documentation to provide more clarity about when `onRecording` callbacks are received.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.1.0

Features

- Added `simulcast` property to `Vp8Codec`. Enabling simulcast causes the encoder to generate
multiple spatial and temporal layers for the video that is published. Simulcast should only be
enabled in a Group Room.

Bug Fixes

- Fixed a bug where the SDK could crash when unsubscribing from a data track and disconnecting from the room at the same time.
- Fixed a rare crash that occurs when disconnecting from a `Room`.
- Fixed an issue which could cause DTLS roles to be negotiated incorrectly in a multi-party Peer-to-Peer Room.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.2

Improvements

- Updated Android Gradle Plugin to `3.1.1`.

Bug Fixes

- Relaxed state check in `CameraCapturer` when stopCature is called and a camera closed event is
not received.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.1

Improvements

- Updated `ScreenCapturer` to capture at resolution based on the device's screen.

Bug Fixes

- Fixed bug in `VideoCapturer` API where `VideoPixelFormat.RGBA_8888` frames were not rotated
before provided to video broadcaster. This bug would result in frames not being oriented
properly when rendered by participants.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0

We've promoted `2.0.0-beta5` to `2.0.0` as our first General Availability release.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-beta5

Bug Fixes

- Fixed issue where onDisconnected was not called when completing a Room using the REST API. [#277](https://github.com/twilio/video-quickstart-android/issues/277)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-beta4

Improvements

- Removed `trackId` from `BaseTrackStats`. `trackSid` or `trackName` can be used to identify
track stats in a `StatsReport`.
- Removed `getTrackId` from `LocalAudioTrack`, `LocalVideoTrack`, and `LocalDataTrack`.
- Added `getSid` to `RemoteAudioTrack`, `RemoteVideoTrack`, and `RemoteDataTrack`.
- Updated Android Gradle Plugin version to `3.1.0` and Gradle version to `4.4`.
- SDK now defers to WebRTC to validate ice servers and returns an error when a connection attempt
 fails due to invalid servers.
- Reduced the time needed to shutdown the signaling stack while disconnecting from a Room.
- Increased the signaling disconnect timeout interval to 1 second.
- Enable monotonic clock support in the signaling client.
- Initial connect message now includes client version metadata.
- Converted `AudioCodec` and `VideoCodec` from enums to abstract classes with concrete
implementations. The subclasses of `AudioCodec` and `VideoCodec` are the following:
  - `AudioCodec`
     - `IsacCodec`
     - `OpusCodec`
     - `PcmaCodec`
     - `PcmuCodec`
     - `G722Codec`
  - `VideoCodec`
     - `Vp8Codec`
     - `H264Codec`
     - `Vp9Codec`

 The following snippets demonstrate the before and after for setting codec preferences.

        // Setting preferences before 2.0.0-beta4
        ConnectOptions aliceConnectOptions = new ConnectOptions.Builder(aliceToken)
              .roomName(roomName)
              .preferAudioCodecs(Collections.singletonList(VideoCodec.ISAC))
              .preferVideoCodecs(Collections.singletonList(VideoCodec.VP9))
              .build();

        // Setting preferences with 2.0.0-beta4
        ConnectOptions aliceConnectOptions = new ConnectOptions.Builder(aliceToken)
              .roomName(roomName)
              .preferAudioCodecs(Collections.<AudioCodec>singletonList(new IsacCodec()))
              .preferVideoCodecs(Collections.<VideoCodec>singletonList(new Vp9Codec()))
              .build();


Bug Fixes

- Fixed a bug where the SDK hangs if DNS resolution fails and the user does not initiate disconnect.
- Resolved an issue with clock rollover in the Room signaling layer that resulted in high CPU usage
 and disconnects.
- The signaling client no longer logs access tokens.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-beta3

Improvements

- Improved internal logic for retrieving ice servers and resolving outbound DNS.

Bug Fixes

- ICE URIs using the turns and stuns scheme are now supported.
 The SDK will now use turns by default if turn is enabled for your Room.
- Resolved a condition where ICE candidates might not be applied in Peer-to-Peer Rooms.
- Quieted unnecessary warning logs when preferring codecs.
- Fixed a bug where onDisconnected was not getting invoked due to a race condition between a
 network handover and a user initiated disconnect call.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-beta2

Bug Fixes

- Fixed crash when calling `Room#disconnect()` twice. [#255](https://github.com/twilio/video-quickstart-android/issues/255)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-beta1

Improvements

- Updated `targetSdkVersion` to 27
- Updated `buildToolsVersion` to 27.0.3
- Updated Android Gradle plugin to to 3.0.1

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview9

Improvements

- Refactor internal reference counting of internal MediaFactory.

Bug Fixes

- Don't publish Ice Candidate stats unless an active pair is present.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview8

Features

- Added the following callbacks to `RemoteParticipant.Listener`
  - `onAudioTrackSubscriptionFailed` - Notifies listener that an audio track could not be
  subscribed to.
  - `onVideoTrackSubscriptionFailed` - Notifies listener that a video track could not be
  subscribed to.
  - `onDataTrackSubscriptionFailed` - Notifies listener that a data track could not be
  subscribed to.
- Added `trackSid` to `BaseTrackStats`.

Bug Fixes

- Removed public `getEncodingOptions` method from `ConnectOptions`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview7

Improvements

- Added version to javadoc title, header, and bottom.
- `LocalParticipant` throws `IllegalArgumentException` when attempting to publish or unpublish
 a released `Track`.

Bug Fixes

- Fixed crash disconnecting from a `Room` before being connected.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview6

Improvements

- `LocalDataTrack` name is no longer provided via static `create` method argument. DataTrack names
 are now provided via `DataTrackOptions`. Reference snippets below:

    Creating a `LocalDataTrack` with name _before_ `2.0.0-preview6`

        String dataTrackName = "data";
        LocalDataTrack localDataTrack = LocalDataTrack.create(context, dataTrackName);

    Creating a `LocalDataTrack` with name _after_ `2.0.0-preview6`

        String dataTrackName = "data";
        DataTrackOptions dataTrackOptions = new DataTrackOptions.Builder()
             .name("data")
             .build();
        LocalDataTrack localDataTrack = LocalDataTrack.create(context, dataTrackOptions);

- Updated javadoc to include note about `VideoView#setVideoScaleType`. Scale type will only
 be applied to dimensions defined as `WRAP_CONTENT` or a custom value. Setting a width or height to
 `MATCH_PARENT` results in the video being scaled to fill the maximum value of the dimension.
- Add warning log when calling `setVideoScaleType` when width or height is set to `MATCH_PARENT`

Bug Fixes

- Fixed a potential crash when publishing or unpublishing a Data Track.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview5

Features

- Added the following callbacks to `LocalParticipant.Listener`
  - `onAudioTrackPublicationFailed` - Notifies listener that local participant failed to publish
  audio track.
  - `onVideoTrackPublicationFailed` - Notifies listener that local participant failed to publish
  video track.
  - `onVideoTrackPublicationFailed` - Notifies listener that local participant failed to publish
  video track.

Improvements

- Include javadoc and sources jar with artifacts published to Bintray.
- Added support for `DataTrack` API with Group rooms.
- Updated to Build Tools 26.0.2
- Support annotations and Relinker no longer exposed at compile time
- Made `Track` interface public. `Track` is the common interface for an `AudioTrack`, `VideoTrack`,
and `DataTrack`.
- Twilio CDN no longer hosts the Video Android aar artifacts or Javadocs

######Accessing Artifacts
If you are downloading Video Android SDK artifacts from the Twilio CDN then there are two
options available moving forward.

1. Follow our [Downloading Video SDKs Guide for Android](https://www.twilio.com/docs/api/video/download-video-sdks#android-sdk).
2. Download the artifacts [directly from Bintray](https://bintray.com/twilio/releases/video-android#files/com/twilio/video-android).

######Viewing Javadocs
All Javadocs back to `1.0.0-preview1` are now hosted on [Github Pages](https://pages.github.com/)
with the following URL scheme. `https://twilio.github.io/twilio-video-android/docs/{version}`

  - To view `2.0.0-preview5` Javadocs go to [https://twilio.github.io/twilio-video-android/docs/2.0.0-preview5](https://twilio.github.io/twilio-video-android/docs/2.0.0-preview5)

Bug Fixes

- Fixed NPE when calling `takePicture` on `CameraCapturer`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview4

Features

- Added new `DataTrack` API. A data track represents a unidirectional source that allow sharing
string and binary data with all participants of a Room. Data tracks function similarly to audio and
video tracks and can be provided via `ConnectOptions` and published using
`LocalParticipant#publishTrack`. Messages sent on the data track are not guaranteed to be
delivered to all the participants. The following snippets demonstrate how to send and receive
messages with data tracks.

Creating a `LocalDataTrack`

    LocalDataTrack localDataTrack = LocalDataTrack.create(context);

Connecting to a `Room` with a `LocalDataTrack`

    ConnectOptions connectOptions = new ConnectOptions.Builder(token)
            .dataTracks(Collections.singletonList(localDataTrack))
            .build();
    Video.connect(context, connectOptions, roomListener);

Publishing a `LocalDataTrack`

    // ... Connected to room
    LocalParticipant localParticipant = room.getLocalParticipant();

    localParticipant.publish(localDataTrack);

Observing `RemoteDataTrackPublication` and `RemoteDataTrack`

    RemoteParticipant.Listener participantListener = new RemoteParticipant.Listener() {
            // ... complete interface ellided

            // Participant has published data track
            @Override
            public void onDataTrackPublished(RemoteParticipant remoteParticipant,
                                             RemoteDataTrackPublication remoteDataTrackPublication);

            // Participant has unpublished data track
            @Override
            public void onDataTrackUnpublished(RemoteParticipant remoteParticipant,
                                               RemoteDataTrackPublication remoteDataTrackPublication);

            // Data track has been subscribed to and messages can be observed.
            @Override
            public void onDataTrackSubscribed(RemoteParticipant remoteParticipant,
                                              RemoteDataTrackPublication remoteDataTrackPublication,
                                              RemoteDataTrack remoteDataTrack);

            // Data track has been unsubsubscribed from and messages cannot be observed.
            @Override
            public void onDataTrackUnsubscribed(RemoteParticipant remoteParticipant,
                                                RemoteDataTrackPublication remoteDataTrackPublication,
                                                RemoteDataTrack remoteDataTrack);
    };

Sending messages on `LocalDataTrack`

    String message = "Hello DataTrack!";
    ByteBuffer messageBuffer = ByteByffer.wrap(new byte[]{ 0xf, 0xe });

    localDataTrack.send(message);
    localDataTrack.send(messageBuffer);

Observing messages from data track

    RemoteDataTrack.Listener dataTrackListener = new RemoteDataTrack.Listener() {
            @Override
            public void onMessage(String message) {
                // Should print "Hello DataTrack!"
                Log.d(TAG, String.format("Received data track message: %s", message));
            }

            @Override
            public void onMessage(ByteBuffer message) {
                Log.d(TAG, "Received message buffer on data track!");
            }
    };

Improvements

- Moved pre-defined aspect ratios from `VideoConstraints` class to `AspectRatio` class.
- Local audio, video, and data tracks return their track IDs for `getName` if no name was specified.
- Improved threading contract.

Bug Fixes

- Fixed issue that caused track and room names with certain UTF-8 characters to be improperly
encoded. [#179](https://github.com/twilio/video-quickstart-android/issues/179)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview3

Improvements

- Upgraded to Android Oreo from Nougat

Bug Fixes

- Fixed case on some devices where `CameraCapturer` incorrectly reported a failure to close the
camera.
- Improved echo cancellation on Nexus 6P and Nexus 6 by enabling hardware echo canceller and disabling OpenSL ES.
- Fixed crash disconnecting from `Room` that has not connected [#116](https://github.com/twilio/video-quickstart-android/issues/116)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Track and room names with certain UTF-8 characters are not encoded properly [#179](https://github.com/twilio/video-quickstart-android/issues/179)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)
- Data Tracks might not be subscribed with Chrome 76+ in a Peer-to-Peer Room. [#433](https://github.com/twilio/video-quickstart-android/issues/433)
- In a P2P room, participants will not receive any media or data tracks published by participants using Firefox 63 or later. [#377](https://github.com/twilio/video-quickstart-android/issues/377)

####2.0.0-preview2

Bug Fixes

- Fixed crash when disconnecting from a Room immediately after unpublishing a local track.

- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Codec preferences do not function correctly in a hybrid codec Group Room.
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####2.0.0-preview1

Features

- Added `EncodingParameters` which constrains how much bandwidth is used to share audio and
video. This object has been added to `ConnectOptions` and can also be set on `LocalParticipant`
after joining a `Room`.
- Added two static `create` methods to `LocalAudioTrack` and `LocalVideoTrack` that allow creating
named tracks. The following snippet demonstrates how to create a video track named "screen".

      LocalVideoTrack screenVideoTrack = LocalVideoTrack.create(context,
              true,
              screenCapturer,
              "screen");

- Moved `getTrackId` from `Track` to `LocalAudioTrack` and `LocalVideoTrack`.
- Added `AudioCodec` and `VideoCodec` as part of the new codec preferences API. Audio and video
codec preferences can be set in `ConnectOptions`. The following snippet
demonstrates how to prefer the iSAC audio codec and VP9 video codec.

      ConnectOptions aliceConnectOptions = new ConnectOptions.Builder(aliceToken)
              .roomName(roomName)
              .preferAudioCodecs(Collections.singletonList(VideoCodec.ISAC))
              .preferVideoCodecs(Collections.singletonList(VideoCodec.VP9))
              .build();

- Added `RemoteAudioTrack` and `RemoteVideoTrack`. These new objects extend `AudioTrack` and
`VideoTrack` respectively and come with the following new method:
  - `getName` - Returns the name of the track or an empty string if no name is specified.
- Added `enablePlayback` to new `RemoteAudioTrack` which allows developers to mute the audio
 received from a `RemoteParticipant`.
- Added `RemoteAudioTrackPublication` which represents a published `RemoteAudioTrack`. This new
class contains the following methods:
  - `getTrackSid` - Returns the identifier of a remote video track within the scope of a `Room`.
  - `getTrackName` - Returns the name of the track or an empty string if no name was specified.
  - `isTrackEnabled` - Checks if the track is enabled.
  - `getAudioTrack` - Returns the base class object of the remote audio track published.
  - `getRemoteAudioTrack` - Returns the remote audio track published.
- Added `RemoteVideoTrackPublication` which represents a published `RemoteVideoTrack`. This new
class contains the following methods:
  - `getTrackSid` - Returns the identifier of a remote video track within the scope of a `Room`.
  - `getTrackName` - Returns the name of the track or an empty string if no name was specified.
  - `isTrackEnabled` - Checks if the track is enabled.
  - `getAudioTrack` - Returns the base class object of the remote audio track published.
  - `getRemoteAudioTrack` - Returns the remote audio track published.
- Added `LocalAudioTrackPublication` which represents a published `LocalAudioTrack`. This new
class contains the following methods:
  - `getTrackSid` - Returns the identifier of a local video track within the scope of a `Room`.
  - `getTrackName` - Returns the name of the track or an empty string if no name was specified.
  - `isTrackEnabled` - Checks if the track is enabled.
  - `getAudioTrack` - Returns the base class object of the local audio track published.
  - `getLocalAudioTrack` - Returns the local audio track published.
- Added `LocalVideoTrackPublication` which represents a published `LocalVideoTrack`. This new
class contains the following methods:
  - `getTrackSid` - Returns the identifier of a local video track within the scope of a `Room`.
  - `getTrackName` - Returns the name of the track or an empty string if no name was specified.
  - `isTrackEnabled` - Checks if the track is enabled.
  - `getAudioTrack` - Returns the base class object of the local audio track published.
  - `getLocalAudioTrack` - Returns the local audio track published.
- Converted `Participant` to an interface and migrated previous functionality into
`RemoteParticipant`. `LocalParticipant` and the new `RemoteParticipant` implement `Participant`.
- Added `RemoteParticipant#getRemoteAudioTracks` and `RemoteParticipant#getRemoteVideoTracks` which
return `List<RemoteAudioTrackPublication>` and `List<RemoteVideoTrackPublication>` respectively.
- Moved `Participant.Listener` to `RemoteParticipant.Listener` and changed the listener to return
`RemoteParticipant`, `RemoteAudioTrackPublication`, and `RemoteVideoTrackPublication` in callbacks.
- Renamed the following `RemoteParticipant.Listener` callbacks:
  - `onAudioTrackAdded` renamed to `onAudioTrackPublished`.
  - `onAudioTrackRemoved` renamed to `onAudioTrackUnpublished`.
  - `onVideoTrackAdded` renamed to `onVideoTrackPublished`.
  - `onVideoTrackRemoved` renamed to `onVideoTrackUnpublished`.
- Added the following callbacks to `RemoteParticipant.Listener`:
  - `onAudioTrackSubscribed` - Indicates when audio is flowing from a remote particpant's audio
  track. This callback includes the `RemoteAudioTrack` that was subscribed to.
  - `onAudioTrackUnsubscribed` - Indicates when audio is no longer flowing from a remote
  partipant's audio track. This callback includes the `RemoteAudioTrack` that was subscribed to.
  - `onVideoTrackSubscribed` - Indicates when video is flowing from a remote participant's video
  track. This callback includes the `RemoteVideoTrack` that was subscribed to.
  - `onVideoTrackUnsubscribed` - Indicates when video is no longer flowing from a remote
  participant's video track. This callback includes the `RemoteVideoTrack` that was subscribed to.
- Renamed `TrackStats` to `RemoteTrackStats`, `AudioTrackStats` to `RemoteAudioTrackStats`, and
`VideoTrackStats` to `RemoteVideoTrackStats`
- Renamed `LocalParticipant#addAudioTrack` and `LocalParticipant#addVideoTrack` to
`LocalParticipant#publishedTrack`.
- Added `LocalParticipant.Listener` which is provides the following callbacks:
  - `onAudioTrackPublished` - Indicates when a local audio track has been published to a `Room`.
  - `onVideoTrackPublished` - Indicates when a local video track has been published to a `Room`.
- Added `LocalParticipant#getLocalAudioTracks` and `LocalParticipant#getLocalVideoTracks` which
return `List<LocalAudioTrackPublication>` and `List<LocalVideoTrackPublication>` respectively.

Improvements

- Null renderers cannot be added or removed from local or remote video tracks.
- Renderers cannot be added or removed from a `LocalVideoTrack` that has been released.

Bug Fixes

- Change visibility of `LocalParticipant#release()` from public to package.
[#132](https://github.com/twilio/video-quickstart-android/issues/132)
- Codec preferences do not function correctly in a hybrid codec Group Room.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.15

Improvements

- Updated Android Gradle Plugin version to `3.1.0` and Gradle version to `4.4`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.14

Improvements

- Improved internal logic for retrieving ice servers and resolving outbound DNS.

Bug Fixes

- Fixed a bug where onDisconnected was not getting invoked due to a race condition between a
network handover and a user initiated disconnect call.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.13

Bug Fixes

- Fixed crash when calling `Room#disconnect()` twice. [#255](https://github.com/twilio/video-quickstart-android/issues/255)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.12

Bug Fixes

- ICE URIs using the `turns` scheme are now supported. The SDK will now use `turns` by default if turn is enabled for your Room.
- ICE URIs using the `stuns` scheme are now supported.
- Resolved a condition where ICE candidates might not be applied in Peer-to-Peer Rooms.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.11

Improvements

- Updated `targetSdkVersion` to 27
- Updated `buildToolsVersion` to 27.0.3
- Updated Android Gradle plugin to to 3.0.1

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.10

Improvements

- Refactor internal reference counting of internal MediaFactory.

Bug Fixes

- Don't publish Ice Candidate stats unless an active pair is present.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
e VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.9

Improvements

- Added version to javadoc title, header, and bottom.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.8

Improvements

- Updated javadoc to include note about `VideoView#setVideoScaleType`. Scale type will only
 be applied to dimensions defined as `WRAP_CONTENT` or a custom value. Setting a width or height to
 `MATCH_PARENT` results in the video being scaled to fill the maximum value of the dimension.
- Add warning log when calling `setVideoScaleType` when width or height is set to `MATCH_PARENT`

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.7

Improvements

- Twilio CDN no longer hosts the Video Android aar artifacts or Javadocs

######Accessing Artifacts
If you are downloading Video Android SDK artifacts from the Twilio CDN then there are two
options available moving forward.

1. Follow our [Downloading Video SDKs Guide for Android](https://www.twilio.com/docs/api/video/download-video-sdks#android-sdk).
2. Download the artifacts [directly from Bintray](https://bintray.com/twilio/releases/video-android#files/com/twilio/video-android).

######Viewing Javadocs
All Javadocs back to `1.0.0-preview1` are now hosted on [Github Pages](https://pages.github.com/)
with the following URL scheme. `https://twilio.github.io/twilio-video-android/docs/{version}`

  - To view `1.3.7` Javadocs go to [https://twilio.github.io/twilio-video-android/docs/1.3.7](https://twilio.github.io/twilio-video-android/docs/1.3.7)
  - To view the latest Javadocs go to [https://twilio.github.io/twilio-video-android/docs/latest](https://twilio.github.io/twilio-video-android/docs/latest)

Bug Fixes

- Fixed NPE when calling `takePicture` on `CameraCapturer`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.6

Improvements

- Include javadoc and sources jar with artifacts published to Bintray.
- Updated to Build Tools 26.0.2
- Support annotations and Relinker no longer exposed at compile time

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.5

Improvements

- Improved threading contract.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.4

Bug Fixes

- Fixed issue that caused room names with certain UTF-8 characters to be improperly
encoded. [#179](https://github.com/twilio/video-quickstart-android/issues/179)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.3

Improvements

- Internal library update

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- Room names with certain UTF-8 characters are not encoded properly [#179](https://github.com/twilio/video-quickstart-android/issues/179)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.2

Improvements

- Upgraded to Android Oreo from Nougat
- Fixed crash disconnecting from `Room` that has not connected [#116](https://github.com/twilio/video-quickstart-android/issues/116)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.1

Improvements

- Fixed case on some devices where `CameraCapturer` incorrectly reported a failure to close the
camera.
- Improved echo cancellation on Nexus 6P and Nexus 6 by enabling hardware echo canceller and disabling OpenSL ES.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.3.0

Features
- Added static method `CameraCapturer.isSourceAvailable` that validates if a camera source is
available on the device. This method is used when creating a `CameraCapturer` instance and when
calling `CameraCapturer#switchCamera` to validate that a source can be used for capturing frames.

Improvements

- Added javadoc to `Participant.Listener`, `ScreenCapturer.Listener`, `VideoCapturer.Listener`,
and `VideoRenderer.Listener`.

Bug Fixes

- Fixed a bug where multiple participants adding/removing tracks at the same time was not handled
properly.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.2.2

Improvements

- Calling `Participant#setListener` with `null` is no longer allowed.

Bug Fixes

- Removed reference to `LocalMedia` in `CameraCapturer` javadoc.
- Fixed race condition that could result in track events not being raised.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.2.1

Improvements

- Improved safety of asynchronous operations in native core.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.2.0

Features

- The SDK now uses TLS 1.2 in favor of TLS 1.0 to connect to Twilio’s servers.

Improvements

- Deprecated `LocalParticipant#release`. This method is not meant to be called and is now a
no-op until it is removed in `2.0.0-preview1` release. [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- Added more checks and logging to `CameraCapturer` to help identify cases when the camera service cannot be reached. [#126](https://github.com/twilio/video-quickstart-android/issues/126)
- Changed `getSupportedFormats` for `CameraCapturer`, `ScreenCapturer`, and `Camera2Capturer` to
be `synchronized`.

Bug Fixes

- Fixed timing issue where camera was not always available after a video track was released. [#126](https://github.com/twilio/video-quickstart-android/issues/126)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.1.1

Bug Fixes

- Fixed bug in `VideoConstraints` logic where valid VideoCapturer video formats were ignored due to very strict checking of aspect ratios in WebRTC
- Fixed bug in Logger.java where setting certain LogLevel's did not print error logs
- Fixed bug in `LocalVideoTrack` where FPS check was incorrectly marking a constraint as incompatible. [#127](https://github.com/twilio/video-quickstart-android/issues/127)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.1.0

Improvements

- Moved signaling network traffic to port 443.
- Added `Camera2Capturer`. `Camera2Capturer` uses `android.hardware.camera2` to implement
a `VideoCapturer`. `Camera2Capturer` does not yet implement `takePicture` and the ability to modify
camera parameters once `Camera2Capturer` is running.

Create `LocalVideoTrack` with `Camera2Capturer`


    // Check if device supports Camera2Capturer
    if (Camera2Capturer.isSupported(context)) {
        // Use CameraManager.getCameraIdList() for a list of all available camera IDs
        String cameraId = "0";
        Camera2Capturer.Listener camera2Listener = new Camera2Capturer.Listener() {
                @Override
                public void onFirstFrameAvailable() {}

                @Override
                public void onCameraSwitched(String newCameraId) {}

                @Override
                public void onError(Camera2Capturer.Exception exception) {}
        }
        Camera2Capturer camera2Capturer = new Camera2Capturer(context, cameraId, camera2Listener);
        LocalVideoTrack = LocalVideoTrack.create(context, true, camera2Capturer);
    }

- This release adds Insights statistics collection, which reports RTP quality metrics back to
Twilio. In the future, these statistics will be included in customer-facing reports visible in the
Twilio Console. Insights collection is enabled by default, if you wish to disable it reference
the following snippet.


    ConnectOptions connectOptions = new ConnectOptions.Builder(token)
            .enableInsights(false)
            .build();

Bug Fixes

- Improved signaling connection retry logic. In the case of an error, the SDK will continue
to retry with a backoff timer when errors are encountered.
- Fixed a bug in network handoff scenarios where the SDK was not handling the race condition
if network lost or network changed event is received when a network changed event is being processed.
- Fixed bug where audio and video tracks were not available after `onParticipantDisconnected` was
invoked [#125](https://github.com/twilio/video-quickstart-android/issues/125)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.2

Bug Fixes

- Backported fix for Chromium bug [679306](https://codereview.webrtc.org/2879073002).

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.1

Improvements

- Improved internal native Room operations.
- Improved `ScreenCapturer` performance by enabling capturing to a texture.
- Added new error code for case when participant gets disconnected because of duplicate participant
joined the room.

Bug Fixes

- Fixed issue adding audio/video tracks quickly while connected to a room [#90](https://github.com/twilio/video-quickstart-android/issues/90)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0
We've promoted 1.0.0-beta17 to 1.0.0 as our first General Availability release.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Disconnecting from a `Room` that has not connected sometimes results in a crash [#116](https://github.com/twilio/video-quickstart-android/issues/116)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta17

Improvements

- Replaced `LocalMedia` class with Track factories for `LocalVideoTrack` and `LocalAudioTrack`

Working with `LocalVideoTrack` and `LocalAudioTrack` before 1.0.0-beta17

    // Create LocalMedia
    LocalMedia localMedia = LocalMedia.create(context);
    LocalVideoTrack localVideoTrack = localMedia.addVideoTrack(true, videoCapturer);
    LocalAudioTrack localAudioTrack = localMedia.addAudioTrack(true);

    ...

    // Destroy LocalMedia to free native memory resources
    localMedia.release();

Working with `LocalVideoTrack` and `LocalAudioTrack` now

    // Create Tracks
    LocalVideoTrack localVideoTrack = LocalVideoTrack.create(context, true, videoCapturer);
    LocalAudioTrack localAudioTrack = LocalAudioTrack.create(context, true);

    ...

    // Destroy Tracks to free native memory resources
    localVideoTrack.release();
    localAudioTrack.release();

- The `ConnectOptions.Builder` now takes a `List<LocalAudioTrack>` and `List<LocalVideoTrack>` instead of `LocalMedia`

Providing `LocalVideoTrack` and `LocalAudioTrack` before 1.0.0-beta17

    LocalMedia localMedia = LocalMedia.create(context);
    LocalVideoTrack localVideoTrack = localMedia.addVideoTrack(true, videoCapturer);
    LocalAudioTrack localAudioTrack = localMedia.addAudioTrack(true);

    ConnectOptions connectOptions = new ConnectOptions.Builder(accessToken)
        .roomName(roomName)
        .localMedia(localMedia)
        .build();
    VideoClient.connect(context, connectOptions, roomListener);

Providing `LocalVideoTrack` and `LocalAudioTrack` now

    List<LocalVideoTrack> localAudioTracks =
                new ArrayList<LocalVideoTrack>(){{ add(localVideoTrack); }};
    List<LocalAudioTrack> localVideoTracks =
                new ArrayList<LocalAudioTrack>(){{ add(localAudioTrack); }};

    ConnectOptions connectOptions = new ConnectOptions.Builder(accessToken)
        .roomName(roomName)
        .audioTracks(localAudioTracks)
        .videoTracks(localVideoTracks)
        .build();
    VideoClient.connect(context, connectOptions, roomListener);
- Methods `getVideoTracks()` and `getAudioTracks()` moved from `LocalMedia` and `Media` to `LocalParticipant` and `Participant`
- Removed `Media` from `Participant` and migrated `Media.Listener` to `Participant.Listener`.
`AudioTrack` and `VideoTrack` events are raised with the corresponding `Participant` instance.
This allows you to create tracks while connected to a `Room` without immediately adding them to the connected `Room`
- Improved hardware accelerated decoding through the use of surface textures.
- Added `textureId` and `samplingMatrix` fields to `I420Frame` so implementations of `VideoRenderer`
can extract YUV data from frame represented as texture.
- Exposed `org.webrtc.YuvConverter` to facilitate converting a texture to an in memory YUV buffer.
- Invoke `ScreenCapturer.Listener` callbacks on the thread `ScreenCapturer` is created on.
- Fixed an issue where the ConnectivityReceiver was causing a reconnect when connecting to a `Room` for the first time occasionally leading to a 53001 error `onConnectFailure` response.
- `Room#getParticipants` returns `List<Participant>` instead of `Map<String, Participant>`.

Bug Fixes

- On Nexus 9 device, intermittent high decoding times results in delayed video. [#95](https://github.com/twilio/video-quickstart-android/issues/95)
- Unsatisfied link errors for `org.webrtc.voiceengine.WebRtcAudioManager` and `org.webrtc.voiceengine.WebRtcAudioTrack` construction. [#102](https://github.com/twilio/video-quickstart-android/issues/102)
- VideoTrack isEnabled() returning wrong state [#104](https://github.com/twilio/video-quickstart-android/issues/104)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- LocalParticipant release method is public [#132](https://github.com/twilio/video-quickstart-android/issues/132)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta16

Improvements

- Added enum `VideoFrame.RotationAngle` to ensure `VideoFrame` objects are constructed with
valid orientation values.
- Updated `CameraCapturer` to be powered by latest WebRTC camera capturer.
- Updated `CameraCapturer` to allow scheduling a picture to be taken while the capturer is not
running.

Bug Fixes

- Reverted decoding from surface textures. This revert
should fix problems for custom `VideoRenderer`s receiving `null` YUV data for `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- On Nexus 9 device, intermittent high decoding times results in delayed video. [#95](https://github.com/twilio/video-quickstart-android/issues/95)
- Unsatisfied link errors for `org.webrtc.voiceengine.WebRtcAudioManager` and `org.webrtc.voiceengine.WebRtcAudioTrack` construction. [#102](https://github.com/twilio/video-quickstart-android/issues/102)
- VideoTrack isEnabled() returning wrong state [#104](https://github.com/twilio/video-quickstart-android/issues/104)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta15

Improvements

- Upgraded to WebRTC 57.
- Renaming `VideoClient` class to `Video`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta14

Improvements

- Simplified internal data structures that populate `StatsReport`.

Bug Fixes

- Fixed teardown crash that occurred in component that fetches ice servers.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta13

Improvements

- Decreased Room connection time by establishing the signaling connection earlier in the process.
- Removed the final case where we resolve localhost. This also improves connection time to your first Room.

Bug Fixes

- Fixed a regression in 1.0.0-beta12 where a track added event was not raised when the trackId was reused. [#83](https://github.com/twilio/video-quickstart-android/issues/83)
- Fixed crash in `Room#disconnect` when releasing `Participant` media
- Resolved memory corruption issues which could occur in multi-party scenarios.
- Fixed a crash which could occur in signaling stack

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta12

Improvements

- Made `VideoClient` an abstract class.
- We've begun formalizing our error codes. They are divided up into Signaling (530xx), Room (531xx), Participant (532xx), Track (533xx), Media (534xx), Configuration (535xx), and Access Token (201xx) subranges. Instances of `TwilioException` will now carry a numeric code belonging to one of these ranges, an error message, and an optional error explanation.
- Implemented a policy for applying `VideoConstraints`. Adding a `LocalVideoTrack` with no constraints, results in `LocalMedia` applying a set of default constraints based on the closest supported `VideoFormat` to 640x480 at 30 FPS. Adding a `LocalVideoTrack` with custom constraints, results in `LocalMedia` checking if the constraints are compatible with the given `VideoCapturer` before applying. If the constraints are not compatible `LocalMedia` applies default constraints. [#68](https://github.com/twilio/video-quickstart-android/issues/68)

Bug Fixes

- Fixed echo cancellation bug for Nexus 6P [#65](https://github.com/twilio/video-quickstart-android/issues/65).

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant disconnect event can take up to 120 seconds to occur [#80](https://github.com/twilio/video-quickstart-android/issues/80) [#73](https://github.com/twilio/video-quickstart-android/issues/73)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta11

Improvements

- Moved `connect` from instance method to static method on `VideoClient` class. Calling the new static `connect` method requires a `Context` in addition to `ConnectOptions` and a `Room.Listener` . `VideoClient` is no longer an object that can be instantiated and an instance is no longer required to connect to a `Room`.
- Moved access token parameter from `VideoClient` constructor to `ConnectOptions.Builder` constructor.

Connecting to a `Room` before 1.0.0-beta11

    // Create VideoClient
    VideoClient videoClient = new VideoClient(context, accessToken);
    ConnectOptions connectOptions = new ConnectOptions.Builder()
        .roomName(roomName)
        .localMedia(localMedia)
        .build();
    videoClient.connect(connectOptions, roomListener);

Connecting to a `Room` with static `connect`

    ConnectOptions connectOptions = new ConnectOptions.Builder(accessToken)
        .roomName(roomName)
        .localMedia(localMedia)
        .build();
    VideoClient.connect(context, connectOptions, roomListener);

Bug Fixes

- Fixed crash when disconnecting from a `Room` on HTC 10.
- Fixed crash caused by removing a track before calling `Room#disconnect` .
- Use a certificate bundle to validate SSL certificates on the signaling connection.
- Improved compatibility with Group Rooms and track added and removed events.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Missing media when adding audio/video tracks quickly while connected to room [#90](https://github.com/twilio/video-quickstart-android/issues/90)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta10

Improvements

- Network handoff, and subsequent connection renegotiation is now supported for IPv4 networks.

Bug Fixes

- Fixed a regression introduced in 1.0.0-beta8 where tokens with purely numeric identities caused a crash [#64](https://github.com/twilio/video-quickstart-android/issues/64) [#60](https://github.com/twilio/video-quickstart-android/issues/60)
- Participant identities now support UTF-8

Known issues

- Network handoff, and subsequent connection renegotiation is not supported for IPv6 networks [#72](https://github.com/twilio/video-quickstart-android/issues/72)
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta9

Bug Fixes

- Fixed immediate disconnect when using custom ICE servers

Known issues

- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Tokens with purely numeric identities results in a crash
- Participant identities with unicode characters are not supported
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta8

Features

- Added an `isRecording` method to `Room`, and callbacks to `RoomListener`. Please note that recording is only available in our Group Rooms developer preview. `isRecording` will always return `false` in a P2P Room.

Bug Fixes

- Fixed camera freeze when using `CameraCapturer#updateCameraParameters`  API #54
- Fixed crash when caused by calling `getStats()`  immediately after disconnecting from `Room`
- Fixed heap corruption on HTC 10
- Fixed memory leaks parsing signaling messages
- Attempt ICE restarts when a PeerConnection fails

Known issues

- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Tokens with purely numeric identities results in a crash
- Participant identities with unicode characters are not supported
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta7

Improvements

- Clarified documentation for `LocalMedia#addAudioTrack`  enabled parameter

Bug Fixes

- Fixed crash loading library on some devices. [#53](https://github.com/twilio/video-quickstart-android/issues/53)

Known issues

- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Disconnecting from a `Room` immediately after calling `getStats()` results in a crash.
- Participant identities with unicode characters are not supported
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta6

Bug Fixes

- Fixed black frames being rendered after device is rotated.
- Fixed crash in `EglBaseProvider`.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Disconnecting from a `Room` immediately after calling `getStats()` results in a crash.
- Native library fails to load on some devices. [#53](https://github.com/twilio/video-quickstart-android/issues/53)
- Participant identities with unicode characters are not supported
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)


####1.0.0-beta5

New features

- Upgraded to WebRTC 55.
- Upgraded to NDK r12b.
- Improved [quickstart](https://github.com/twilio/video-quickstart-android) README.
- Added a `getStats()` method to `Room` that builds a `StatsReport` with metrics for all the audio and video tracks being shared to a `Room`.
- Improved hardware accelerated decoding through the use of surface textures.
- Standardized error messages and codes for a `Room`.
- Changed `VideoException` to `TwilioException`.

Bug Fixes

- Fixed picture orientation bug for `takePicture`.
- `PictureListener` callbacks are invoked on the calling thread of `takePicture`.
- Reduced previously high decoding times for Nexus 9.

Known issues

- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Disconnecting from a `Room` immediately after calling `getStats()` results in a crash.
- Participant identities with unicode characters are not supported
- Missing YUV data when adding a custom `VideoRenderer` to `VideoTrack`s [#93](https://github.com/twilio/video-quickstart-android/issues/93)
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)


####1.0.0-beta4

New features

- Added new API to `CameraCapturer` for taking a picture.
- Quickstart has been updated to demonstrate the use of APK splits to reduce APK size.

Known issues

- IPv6 is not fully supported.
- Network handoff, and subsequent connection renegotiation is not supported.
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- On Nexus 9 device, intermittent high decoding times results in delayed video.
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta3

New features

- Added new API to `CameraCapturer` for providing custom `Camera.Parameters`.
- Added `isScreencast()` method to `VideoCapturer`. This indicates if a capturer is providing screen content and affects any scaling attempts made while media is flowing.

Bug fixes

- Fixed crashes when RECORD_AUDIO and CAMERA permission are not granted. `LocalMedia` will now return `null` when attempting to add a `LocalAudioTrack` without RECORD_AUDIO permission. `CameraCapturer` will log an error and provide an error code via a new `CameraCapturer.Listener` when trying to capture video without CAMERA permission.

Known issues

- IPv6 is not fully supported
- Network handoff, and subsequent connection renegotiation is not supported
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta2

New features

- Removed dependency on `AccessManager` in `VideoClient` constructor. Only a context and access token are required to create a `VideoClient`.
- Added an `updateToken` method to `VideoClient` that allows for an access token to be updated in case it has expired

Bug fixes

- Fixed crashes on x86 and x86_64 devices

Known issues

- IPv6 is not fully supported
- Network handoff, and subsequent connection renegotiation is not supported
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-beta1

New features

- Added preliminary support for network handover while connected to a Room

Bug fixes

- Provide a `release()` method on the `I420Frame`  object allowing developers to free native memory once they are done using the frame when implementing their own custom renderers

Known issues

- IPv6 is not fully supported
- Network handoff, and subsequent connection renegotiation is not supported
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Using x86 or x86_64 devices results in a crash
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-preview2

New features

- Added support for AudioOptions
- Upgraded to Android Nougat from Marshmallow

Known issues

- IPv6 is not fully supported
- Network handoff, and subsequent connection renegotiation is not supported
- Resource leak when implementing custom `VideoRenderer`
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Using x86 or x86_64 devices results in a crash
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

####1.0.0-preview1

New features

- Adopting Room based communications model. The invite model has been completely removed

Bug fixes

- First developer preview release

Known issues

- IPv6 is not fully supported
- Network handoff, and subsequent connection renegotiation is not supported
- Resource leak when implementing custom `VideoRenderer`
- VP8 is the only supported codec [#71](https://github.com/twilio/video-quickstart-android/issues/71)
- Using x86 or x86_64 devices results in a crash
- Participant identities with unicode characters are not supported
- The SDK is not side-by-side compatible with other WebRTC based libraries [#340](https://github.com/twilio/video-quickstart-android/issues/340)

**Looking for older changlog entries?** [You can find the changelog for the deprecated Programmable Video Conversations API here](https://www.twilio.com/docs/api/video/conversations-api-deprecated/changelogs/android).
