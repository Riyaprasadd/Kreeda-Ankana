# Kreeda-Ankana 🏏 🏐

**Ground & Match Organizer App** for village teams.

A digital notice board for the village ground that lets teams reserve slots, post friendly-match challenges to other villages, and keep a running record of recent match scores. Built as an Android app with Jetpack Compose, MVVM, Room, and Hilt — wired to optionally sync over Firebase Realtime Database.

---

## Quick start

### 1. Open in Android Studio

```bash
# Already cloned/extracted into:
cd ~/projects/Mindu
```

In Android Studio: **File → Open** → select the `Mindu` folder. Studio will index, sync Gradle, and download dependencies the first time (5–10 min).

### 2. Run on an emulator or device

- **Emulator**: open AVD Manager, create or pick any phone with API 24+, and press the green Run button (`Shift+F10`).
- **Physical device**: enable Developer Options → USB Debugging, plug in over USB, then press Run.

The first launch takes you to the Ground tab — today's grid of 6 AM – 8 PM hourly slots. Tap a free slot to book, swipe over to Challenges to post or accept a friendly match, and Score Wall to publish recent results.

### 3. (Optional) Wire up Firebase for cross-village sync

By default the app stores everything locally with Room — perfect for demos and offline use. To turn on real-time sync between phones:

1. Create a Firebase project at <https://console.firebase.google.com> and add an Android app with package `com.kreeda.ankana`.
2. Download the generated `google-services.json` and drop it into `app/`.
3. In `app/build.gradle.kts` add at the top of the `plugins { … }` block:
   ```kotlin
   id("com.google.gms.google-services")
   ```
   and add to the root `build.gradle.kts`:
   ```kotlin
   plugins {
       id("com.google.gms.google-services") version "4.4.2" apply false
   }
   ```
4. Uncomment the two `firebase` lines in `app/build.gradle.kts`'s `dependencies { … }`.
5. Re-sync Gradle. Repository code can then be extended to push/pull from `FirebaseDatabase.getInstance().reference`.

---

## Project map

```
app/src/main/java/com/kreeda/ankana/
├── KreedaApp.kt              # Hilt-enabled Application
├── MainActivity.kt           # Single ComponentActivity hosting Compose
│
├── data/
│   ├── model/                # Sport, Booking, Challenge, Score (Room entities)
│   ├── local/                # Room DB, DAOs, type converters
│   └── repository/           # BookingRepository, ChallengeRepository, ScoreRepository
│
├── di/
│   └── DatabaseModule.kt     # Hilt module providing Room + DAOs
│
└── ui/
    ├── theme/                # Bold green/orange Material 3 theme
    ├── nav/                  # Routes + bottom-nav scaffold
    ├── components/           # Reusable: header, sport picker, search field, empty state
    ├── util/                 # DateUtil + GROUND_HOURS constant
    └── screens/
        ├── calendar/         # Ground grid + Book Slot
        ├── challenges/       # Challenge board, Post & Accept flows
        ├── scores/           # Score Wall + Post Score
        └── settings/         # About + Firebase status
```

## Architecture (MVVM)

```
       UI (Compose)
            │  observes StateFlow
            ▼
      ViewModel (@HiltViewModel)
            │  calls suspend fns
            ▼
      Repository
            │  Flow<…>  /  suspend
            ▼
      DAO  →  Room SQLite
```

- **No two-way binding magic** — every input is plumbed via explicit setter methods on the ViewModel
- **Single source of truth** — Room is the canonical store; UI reactively updates from `Flow`s
- **Dependency injection** — Hilt provides the database, DAOs, and repositories; ViewModels are auto-discovered via `@HiltViewModel`

## Key design decisions

| | |
|---|---|
| **Slot granularity** | 1-hour slots, 6 AM – 8 PM (14 slots/day) |
| **Slot uniqueness** | Database-enforced unique index on `(date, hour)` — concurrent double-booking returns `BookingResult.SlotTaken` |
| **No login** | Team name *is* the identity — keeps friction at zero, matching the PRD's "≤ 5 seconds to book" goal |
| **Single ground per app** | The PRD says "*the* village ground"; multi-ground would inflate scope |
| **Offline-first** | Works without internet; Firebase is opt-in once `google-services.json` is added |

## Tech

- **Kotlin** 2.0.21
- **Jetpack Compose** (Material 3, BOM 2024.12.01)
- **Room** 2.6.1 (KSP)
- **Hilt** 2.52 (KSP)
- **Navigation Compose** 2.8.5
- **kotlinx-datetime** 0.6.1
- **Coroutines** 1.9.0
- **Gradle** 8.14, **AGP** 8.7.3, **JDK** 17
- `compileSdk` 35, `minSdk` 24, `targetSdk` 35

## Building from the command line

```bash
export JAVA_HOME=/home/faris/jdk
export PATH=$JAVA_HOME/bin:$PATH
export ANDROID_HOME=/home/faris/Android/Sdk

./gradlew assembleDebug              # builds app/build/outputs/apk/debug/app-debug.apk
./gradlew installDebug               # installs to a connected device / running emulator
./gradlew lint                       # static analysis
```

## Author

Mohammed Faris Sait • USN 1HK22CS076
*VTU Internship — Android App Development using GenAI — Title #6: Kreeda-Ankana*
