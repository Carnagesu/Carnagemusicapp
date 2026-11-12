# Carnage Music Finder — Android

**Version:** 0.1.0 (source preview, not a verified production release).  
**Language:** Java 17. **Minimum Android:** 10 (API 29). **Build:** AGP 8.13.2 / Gradle 8.13 / JDK 17.

Android playlist-first music discovery and local library. Import *multiple Yandex Music playlists* and search for a likely full original/Explicit performance, review ambiguous candidates and save authorized recordings as MP3, organized per playlist.

## Current features

- Imports `https://music.yandex.ru/users/<owner>/playlists/<kind>` and `/playlists/<owner>:<kind>` links; attempts canonical resolution of other public `/playlists/<slug>` links. `.csv`, `.txt`, and `.m3u` imports are a fallback.
- Unofficial Yandex API returns **metadata only**. Optional OAuth token used in RAM for each import only; not saved to disk. Yandex may require authorization even for public playlists; private playlists require an authorized account. **Do not share OAuth tokens or post them in GitHub issues.**
- Multiple playlist queues; searches tracks with the local yt-dlp Android runtime (YouTube search), ranks titles, artists, and duration; rejects obvious Clean/Radio Edit/remix/short versions.
- Automatic download is conservative: selected candidates must clearly mention Explicit/Uncensored and match duration. Unclear candidates require manual selection; the app does not pretend to detect censorship from the audio waveform.
- MP3 files published via Android MediaStore to `Music/Carnage/<playlist name>/`; player can access them without broad storage permissions. Persistent JSON stores playlists, candidate URLs, progress status and MediaStore URIs.
- Can import lists of `Artist - Title` or `Artist,Title` or `#EXTINF ... Artist - Title`.
- Export status and unresolved matches of all playlists to a CSV report. Search module can be updated from inside the app. Search quality, YouTube access and downloads depend on external providers and geography.

[Инструкция на русском](%D0%A1%D0%91%D0%9E%D0%A0%D0%9A%D0%90_%D0%9D%D0%90_GITHUB.md)

## Building an APK in GitHub — no Android Studio required

1. Create an empty **public or private repository** on GitHub.
2. Upload the **contents of this folder** to repository root (not an extra nested `CarnageMusicFinder` directory). Commit to `main`.
3. Go to **Actions → Build Android APK → Run workflow** (or trigger by pushing commits to main).
4. Open the completed workflow → **Artifacts → CarnageMusicFinder-APK** and download the ZIP containing `app-debug.apk`.
5. Install `app-debug.apk` on Android. Android may request permission to install from your browser or file manager.

Workflow uses `gradle/actions/setup-gradle` to download Gradle and builds without a Gradle wrapper JAR. For local builds, install Gradle 8.13 and JDK 17 and run `gradle :app:assembleDebug`. Android Studio can open the folder as a Gradle project.

## How to use

1. Paste a playlist link. If Yandex rejects access, provide **your own** OAuth token or use the CSV/M3U fallback.
2. Add as many playlists as desired. Tap **Найти во всех** to scan all playlists.
3. Open tracks marked `? Требуется проверка` and pick the likely original. Search title/Explicit label/length **cannot guarantee** the actual audio is uncensored.
4. Only if you have permission to obtain the files, check the authorization checkbox and use **Найти + скачать** or **Скачать выбранные**. Tracks already saved are skipped.
5. Use the resulting MP3 files with your regular Android music player, even offline.

## Restrictions / design decisions

- **Not a Yandex Music downloader**. Never obtains protected Yandex audio, bypasses subscription access, or decrypts DRM.
- yt-dlp is used for discovery and downloading *authorized* publicly accessible content. Follow source terms and copyright law. The app has no licensed catalog of every popular song and **cannot guarantee** it will find or download all tracks.
- No audio-signal recognition of censored vs uncensored speech in this version. Automatic ranking is heuristic, not a correctness guarantee.
- Android application should remain open during active search/download. After interruption, unfinished entries return to retry/review when reopened. No long-running foreground service yet.
- No embedded OAuth client and no official Yandex public API. Short URLs may fail if not resolvable to owner/kind; use old-style links or CSV/M3U.
- No device tests or actual `assembleDebug` were run in the generation environment (Android SDK and Gradle not available). The GitHub workflow is provided to compile and surface any integration errors.
- Bundled `youtubedl-android` native binaries may have platform-specific compatibility issues with Android 16/17 and 16 KB page size; track via the library's issue tracker.
- No APK is included in the source ZIP. The GitHub Actions build produces it.

## Project structure

```
app/src/main/java/com/carnage/musicfinder/
  MainActivity.java       Native dark UI, multi-playlist queue
  PlaylistImporter.java  Yandex metadata + CSV/M3U parser
  PlaylistStore.java     Persist queue to app-private JSON
  Models.java            Playlist/track/candidate models
  MusicMatcher.java      Explicit-variant and duration scoring
  MusicEngine.java       yt-dlp search, MP3 conversion and MediaStore publish
.github/workflows/android.yml  GitHub Actions installable debug APK
```

## Security

No passwords required. Optional Yandex OAuth token is neither saved to disk nor sent to third-party discovery providers. Avoid using untrusted tokens and take care with screen recordings. Only playlist artist/title metadata is submitted to YouTube search.

## Possible follow-up milestones

- Make short playlist URLs more reliable via a dedicated resolver with automated tests.
- Foreground service with retry/backoff, playlist synchronization, parallel configurable searches and robust background resume.
- Real waveform/lyric-level clean-vs-explicit verification only if legally obtained comparison recordings are available.
- Faster metadata matching against a licensed recording database; source adapters for stores and user-owned files.
