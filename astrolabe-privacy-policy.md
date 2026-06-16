# Privacy Policy for Astrolabe

*Last updated: June 16, 2026*

Astrolabe is a music player for Android. It runs entirely on your device. This policy describes what data the app handles, what it does not do, and what to do if you have questions.

## The short version

**Astrolabe does not collect, transmit, or share your personal data.** The app itself sends nothing over the network — it has no internet permission, no account, no analytics, no ads, and no tracking. Everything it does happens on your device.

The only time any information can leave your device is if **you** choose to use the in-app "Send feedback" button, which hands a message to your own email app for you to send (details below). That's optional and entirely under your control.

If you'd rather read a longer version, the rest of this page covers the details.

## What Astrolabe reads on your device

You grant access to specific folders using Android's built-in folder picker (the Storage Access Framework). Astrolabe can only see the folders you choose, and you can change or revoke that access at any time. Within those folders, the app reads:

- **Audio files**, so it can play them.
- **Standard audio metadata** (artist, album, title, duration, etc.) — read from the files themselves, the same way any music player reads them.
- **Cover art embedded in your audio files**, used to show album artwork in the app.
- **Image files that sit alongside your music** (for example, a `cover.jpg`, `folder.jpg`, or `front.jpg` in an album folder). When Astrolabe creates a collection from a folder, it looks for an image like this to use as that collection's artwork.
- **The audio itself**, which the app analyzes on your device to build its constellation map of your library. This analysis is computed locally and never leaves your device.

Separately, if you choose to set custom artwork for a collection, Astrolabe uses **Android's system photo picker** to let you pick one image. The app only receives the single image you select — it does not gain access to your photo library. That image is copied into the app's private storage so it can be shown as the collection's thumbnail.

The app does not read other files on your device, your contacts, your messages, your location, your camera, your microphone, your call history, or anything outside the folders and images you've explicitly chosen.

## What Astrolabe stores on your device

For the app to work, it keeps some information in its private storage on your device:

- **Library data** — the music files it has access to, their metadata, and the analysis results used to build the constellation
- **Collection artwork** — cover images you've selected, copied into the app's storage as thumbnails
- **Your playback history and play counts** — used to show which songs you've listened to and to surface them in the app's "lived-in" features
- **Your walks and collections** — the playlists and groupings you create in the app
- **Notes you write** about individual songs
- **Your settings and preferences** within the app

All of this lives only on your device, in the app's private storage area. None of it is transmitted anywhere. If you uninstall the app, this data is removed from your device automatically.

## Sending feedback (optional)

Astrolabe has a "Send feedback" button in Settings. If you use it:

- It opens **your own email app** with a message addressed to the developer. **Astrolabe does not send anything itself** — nothing happens unless you choose to send the email.
- To help with bug reports, the draft includes a small footer with your **device model, Android version, and the app version**. You can see and edit the entire message before sending, and delete that footer if you prefer.

This is the only feature that can result in information leaving your device, and it only does so when you deliberately send the email.

## What Astrolabe does NOT do

To be explicit, the app does not:

- Send any data over the network on its own — it has no internet access
- Use any analytics or telemetry services
- Show ads or share data with advertisers
- Track you across apps or websites
- Require an account or login
- Require an internet connection (the app works fully offline)
- Use your microphone or camera
- Access your photo library, contacts, messages, location, or any data unrelated to the music and images you've chosen
- Sell, rent, or disclose your data to third parties

## Permissions the app requests, and why

Android requires apps to explicitly request access to certain things. Astrolabe asks for:

- **Read access to audio** (`READ_MEDIA_AUDIO` on Android 13+; `READ_EXTERNAL_STORAGE`, limited to older Android versions) — to read your music. You also grant folder access through Android's folder picker, which limits the app to the folders you choose.
- **Foreground services** (`FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_MEDIA_PLAYBACK`, `FOREGROUND_SERVICE_DATA_SYNC`) — the media-playback service keeps music playing when the app is in the background or your screen is off; the data-sync service lets the one-time, on-device analysis of your library continue while the app is backgrounded. Both run locally; neither sends anything anywhere.
- **Post notifications** (`POST_NOTIFICATIONS`) — to show now-playing controls and library-analysis progress in your notification shade.
- **Wake lock** (`WAKE_LOCK`) — held briefly while a song is being analyzed so the work isn't interrupted, then released. It does not keep your screen on.

The app does **not** request internet access, location, contacts, microphone, or camera. Choosing a custom collection image uses Android's system photo picker, which does not require a broad storage permission.

## Children's privacy

Astrolabe is suitable for general audiences. Because the app does not collect any personal information at all, it does not knowingly collect information from children under 13 — or from anyone of any age. The app reads only the music and images you give it access to, and stores nothing outside the device.

## Changes to this policy

If the app's behavior ever changes in a way that affects this policy (for example, if a future version added an optional online artwork lookup or a cloud-sync feature), this page will be updated and the "Last updated" date at the top will be changed. The current version of this policy always applies to the current version of the app.

## Contact

If you have questions about this policy, find something in the app that surprises you, or have a privacy concern, please reach out:

**Email:** trevor.thiess@gmail.com

I read every message. Reasonable response time is a few days.

---

*Astrolabe is developed and maintained by Trevor Thiess, based in Minneapolis, MN, USA. It is currently in active development; feedback is welcome and appreciated.*
