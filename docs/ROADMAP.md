# Chronos Roadmap (Diary + Goal Tracker), React Native edition

You write the code. This file gives you the structure, the order, and the concepts for each step.
Each step is sized for roughly one session of an hour or less. Steps marked **(weekend)** are bigger.

---

## 1. Stack: React Native with Expo, in TypeScript

- **React Native** lets you build a real Android app using JavaScript and React.
- **Expo** is the official recommended way to start a React Native app. It handles the build setup, gives you ready-made modules (database, camera roll, notifications), and lets you run your app on your phone by scanning a QR code with the **Expo Go** app.
- **TypeScript** is JavaScript with types, like Python type hints except the editor actually enforces them. It's the default in Expo and it catches a lot of mistakes before you even run the app.

**What you keep from the Flutter setup:** JDK 17, Android Studio and the Android SDK, the USB debugging setup. You won't need them on day one (Expo Go runs your app without building anything), but you will when you build a real installable app in later phases.

**The packages you'll use** (add each one only when you reach its step, always with `npx expo install <name>`, never plain `npm install`, so you get versions that match your Expo SDK):

| Need | Package |
|---|---|
| Screens and tabs | `expo-router` (already included) |
| Local database | `expo-sqlite` (SQLite, you write the SQL yourself) |
| Photos | `expo-image-picker`, `expo-file-system` |
| Reminders ("your goal is due today") | `expo-notifications` |
| Date picker | `@react-native-community/datetimepicker` |
| Tests for your logic | `jest-expo`, `jest` |

**Docs to keep open:** https://docs.expo.dev (Expo changes fast, so always check the docs that match the `expo` version in your `package.json` rather than old tutorials or blog posts).

---

## 2. What we're building (the rules, written down)

These are the rules from your message, plus defaults I picked where you didn't say. **Change any default you disagree with**, the code should follow this section.

### Diary entry
- Text is required. Mood, tags and photos are optional.
- Mood: a number 1 to 5 (shown as emoji). *(default)*
- One day can have several entries. *(default; say if you want one per day)*

### Daily to-dos
- Max **3** per day, set by yourself.
- You can set tomorrow's to-dos the night before (or on the day itself).
- Checking a to-do **on its own day** gives points. Checking it later gives nothing. *(default: 10 points each)*
- Finish **all** of a day's to-dos (at least 1 set) and that day counts toward your **streak**. Miss a day and the streak resets to 0.

### Big goals
- A goal has a title, optional description, and a deadline.
- On the deadline day, the app asks you "Did you finish this?".
- You can mark it done earlier.
- Changing the deadline: each change can move it **at most 7 days** from the current deadline. *(my reading of "1 week gap"; tell me if you meant something else)*
- Points for finishing on or before the deadline. *(default: 50)*

### Profile
- Name, profile picture, improver points, current streak, best streak.

---

## 3. Data model (the database tables)

Think of each table like a Python list of dicts where every dict has the same keys.

```
profile        (id = 1 always, name, photo_path)
entries        (id, date, text, mood?, created_at, updated_at)
tags           (id, name UNIQUE)
entry_tags     (entry_id, tag_id)            -- links entries <-> tags (many-to-many)
entry_photos   (id, entry_id, file_path)
daily_todos    (id, for_date, title, position 1..3, done_at?)
goals          (id, title, description?, deadline, created_at, completed_at?, last_deadline_change?)
points_log     (id, amount, reason, created_at)
```

`?` means the column can be empty (NULL).

**Design decisions worth understanding:**
- **Points are a log, not a single number.** Total points = `SUM(amount)` from `points_log`. You can always see *why* someone has 340 points, and fixing a bug never leaves a wrong total stuck in the database.
- **Streak is calculated, not stored.** You compute it from `daily_todos` when you need it. Stored counters drift out of sync; calculated ones can't.
- **Photos are files, not database blobs.** Copy the picked image into the app's own folder and store only its path.
- **Dates:** store a calendar day as text `'2026-10-03'` and a moment in time as an integer (milliseconds since 1970, which is what JavaScript's `Date.now()` gives you).

---

## 4. Folder structure

```
src/
  app/                      -- ROUTES: every file here is a screen (Expo Router)
    _layout.tsx             -- root: opens the database, wraps everything
    (tabs)/
      _layout.tsx           -- the bottom tab bar
      index.tsx             -- Today tab
      diary.tsx
      goals.tsx
      profile.tsx
    entry/
      new.tsx               -- "new entry" screen
      [id].tsx              -- view/edit one entry ([id] = a value from the URL)
  data/
    db.ts                   -- creates tables, migrations
    types.ts                -- Entry, DailyTodo, Goal, Profile types
    entries.ts              -- all SQL for entries lives here
    todos.ts  goals.ts  points.ts  profile.ts
  logic/
    points.ts               -- pure functions: points rules
    streak.ts               -- pure functions: streak calculation
    goalRules.ts            -- pure functions: deadline-change rule
    *.test.ts               -- tests next to the logic they test
  components/               -- small reusable UI pieces (MoodPicker, TagChip...)
```

**The one rule that keeps this project healthy:** screens never write SQL, and rules (points, streaks, deadlines) never touch the screen. Screens call functions in `data/`; `data/` talks to SQLite; `logic/` is plain functions you can test with no phone attached. Only screens go in `src/app/`, everything else lives outside it.

---

## 5. The roadmap

### Phase 0: Setup (weekend)
- [x] Clean out Flutter (see the steps in the thread).
- [x] Install Node.js and npm: `sudo pacman -S nodejs npm`. Check with `node --version` (needs 20 or newer).
- [x] On your phone: install **Expo Go** from the Play Store.
- [x] In your Chronos folder: move `readme.md` out of the way temporarily, run `npx create-expo-app@latest .`, answer **Y** when it asks to skip creating a new git repo, then move your readme back over the generated `README.md`.
- [x] `npx expo start`, then scan the QR code with Expo Go (phone and computer on the same Wi-Fi). The template app opens on your phone.
- [x] Hot reload test: change some text in `src/app/index.tsx`, save. The phone updates by itself, no key press needed.
- [x] `npm run reset-project` to get a blank app (move the example to `/example` so you can peek at it later, or delete it).
- [x] Delete `Chronos.py`, commit, push.

**Done when:** a blank app runs on your phone through Expo Go and updates when you save.

### Phase 1: TypeScript for Python people (2 to 4 sessions)
Do these in a scratch folder outside the app, run them with `npx tsx file.ts`.
- [ ] `let` vs `const`, basic types (`string`, `number`, `boolean`).
- [ ] `null`, `undefined` and optional `?`. This is the biggest new idea for you.
- [ ] Arrays with `map`, `filter`, `find`. Objects (JavaScript's version of dicts).
- [ ] `type` / `interface` to describe the shape of an object.
- [ ] Functions and arrow functions `(x) => x * 2`.
- [ ] `async` / `await` and `Promise` (same idea as Python's asyncio).
- [ ] `import` / `export` between files.

Quick translation table:

| Python | TypeScript |
|---|---|
| `x = 5` | `let x = 5;` (or `const x = 5;` if it never changes, which is most of the time) |
| `name: str \| None = None` | `let name: string \| null = null;` |
| `def add(a: int, b: int) -> int:` | `function add(a: number, b: number): number { ... }` |
| `lambda x: x * 2` | `(x) => x * 2` |
| `[x * 2 for x in nums]` | `nums.map((x) => x * 2)` |
| `[x for x in nums if x > 0]` | `nums.filter((x) => x > 0)` |
| `f"Hi {name}"` | `` `Hi ${name}` `` (backticks) |
| `{"a": 1}` dict | `{ a: 1 }` object |
| `@dataclass` | `type` or `interface` |
| `from x import y` | `import { y } from './x';` |

Example of the kind of type you'll write:

```ts
export type Entry = {
  id: number;
  date: string;         // '2026-10-03'
  text: string;
  mood: number | null;  // 1..5 or null
};
```

**Done when:** you can write the `Entry` type above from memory, plus a function that takes a list of entries and returns only the ones from a given date.

### Phase 2: React basics and the app shell (3 to 4 sessions)
- [ ] Learn: a **component** is a function that returns UI (written in JSX, which looks like HTML inside your code). **Props** are its arguments.
- [ ] **State** with `useState`: when state changes, React redraws that component.
- [ ] Core pieces: `View`, `Text`, `Pressable`, `TextInput`, `FlatList`, `StyleSheet`.
- [ ] Build `src/app/(tabs)/_layout.tsx` with Expo Router's `Tabs`: **Today, Diary, Goals, Profile**. Each tab shows placeholder text.
- [ ] Navigation: open a screen with `router.push('/entry/new')` or a `<Link>`, go back with `router.back()`.

**Done when:** you can switch between 4 empty tabs and open a dummy "new entry" screen.

### Phase 3: Diary, in memory only (2 to 3 sessions)
- [ ] Diary tab shows a list of entries (newest first) from a plain array in state.
- [ ] "+" button opens a form: text input (required), Save button.
- [ ] Validation: can't save empty text.
- [ ] Tapping an entry opens it; you can edit or delete it.

**Done when:** the full create/read/update/delete loop works, even though it's lost on restart. Getting the UI right before the database means you only debug one thing at a time.

**Concept to learn here: sharing data between screens.** The list lives in one place and both screens need it. Learn React **Context** for this. In Phase 4 the database takes over that job.

### Phase 4: Saving to the phone with SQLite (weekend)
- [ ] `npx expo install expo-sqlite`.
- [ ] In `src/app/_layout.tsx`, wrap the app in `<SQLiteProvider databaseName="chronos.db" onInit={migrateDb}>`.
- [ ] Write `migrateDb` in `data/db.ts`: create the `entries` table (version 1).
- [ ] Write `data/entries.ts` with `insertEntry`, `updateEntry`, `deleteEntry`, `getAllEntries`. Screens get the database with `useSQLiteContext()` and pass it in.
- [ ] Swap the in-memory list for the database.

**Done when:** entries survive closing and reopening the app.

**Concept to learn here: migrations.** SQLite keeps a number called `user_version`. Your `migrateDb` reads it, runs every step the database hasn't had yet (version 1 creates entries, version 2 adds tags, and so on), then updates it. Never edit an old step once it has run on your phone; add a new one instead.

### Phase 5: Mood, tags, photos (3 to 5 sessions)
- [ ] `MoodPicker` component (5 emoji in a row). Add `mood` to the form.
- [ ] Tags: `tags` + `entry_tags` tables (migration step 2). Type a tag, it becomes a chip.
- [ ] Photos: `expo-image-picker` to pick, `expo-file-system` to copy the image into the app's document folder, save the path in `entry_photos`. Show thumbnails.
- [ ] Filter the diary list by tag.

**Done when:** an entry can have text + mood + 2 tags + a photo, and all of it survives a restart.

### Phase 6: Daily to-dos (3 to 4 sessions)
- [ ] `daily_todos` table + `data/todos.ts`.
- [ ] **Today** tab: today's to-dos with checkboxes.
- [ ] "Plan tomorrow" button: add up to 3 to-dos for tomorrow. Block a 4th.
- [ ] Checking a to-do sets `done_at`. Unchecking clears it.

**Done when:** you can plan tomorrow tonight, and tomorrow they appear on the Today tab.

### Phase 7: Points and streaks, with tests (3 to 4 sessions)
This is the most "Python-like" part: pure logic.
- [ ] Set up tests: `npx expo install jest-expo jest @types/jest -- --save-dev`, add `"test": "jest"` to `package.json` scripts and `"jest": { "preset": "jest-expo" }`.
- [ ] `logic/points.ts`: a function that decides how many points a check-off is worth (0 if not on its own day).
- [ ] `points_log` table; write a row when points are earned. Remove it if the to-do is unchecked the same day.
- [ ] `logic/streak.ts`: given which days were complete, return the current streak.
- [ ] Write tests in `logic/streak.test.ts` and run `npm test`. Cover: no to-dos, all done, one missed day, a gap of days, today still in progress.

Example of the shape (pure function, easy to test):

```ts
export function currentStreak(dayComplete: Record<string, boolean>, today: string): number {
  // walk backwards from yesterday (today doesn't break the streak until it's over)
  // count consecutive true days; stop at the first false or missing day
  // add 1 if today is already complete
}
```

**Done when:** all tests pass and the Today tab shows "🔥 4 day streak".

### Phase 8: Big goals and reminders (weekend + 2 sessions)
- [ ] `goals` table + `data/goals.ts` + Goals tab (active / completed lists).
- [ ] Create a goal with a date picker for the deadline.
- [ ] "Mark done" (early or on time). Award points if on or before the deadline.
- [ ] `logic/goalRules.ts`: `canChangeDeadline(oldDate, newDate)` returns false if the move is more than 7 days. Test it.
- [ ] `expo-notifications`: schedule a local notification for the deadline day ("Did you finish *X*?"). Re-schedule when the deadline changes; cancel when the goal is done.
- [ ] Bonus: an evening reminder to plan tomorrow's to-dos.

**Done when:** a goal due tomorrow sends you a notification tomorrow morning.

**Heads-up:** if notifications misbehave inside Expo Go, this is the point to switch to a **development build**, which is your own version of Expo Go built from your project with `npx expo run:android`. This is where JDK 17 and the Android SDK come back. We'll set the environment variables together when you get here.

### Phase 9: Profile (2 sessions)
- [ ] `profile` table (single row). Edit name, pick a profile picture.
- [ ] Show total points, current streak, best streak, number of entries.

### Phase 10: Polish and your first real build (weekend)
- [ ] Search diary text.
- [ ] Export a backup (JSON file, shared with `expo-sharing`) so you never lose your diary while testing.
- [ ] App icon and display name in `app.json`.
- [ ] Build an installable APK, either locally (`npx expo run:android --variant release`) or with EAS Build in the cloud. Install it like a normal app.

### Later: Duo Goal
Needs a server and accounts. We'll pick between Supabase and Firebase when we get there. Because your data already lives behind `data/` functions, the screens won't have to change much.

---

## 6. Habits that will save you time
- **Commit after every checkbox** above. Small commits make it easy to undo a bad hour.
- **Read the red error screen from the top.** The first line and the file name under it are usually what matters.
- **Reload:** saving usually updates the phone automatically. If something looks stuck, press `r` in the terminal running `npx expo start` to reload the whole app.
- **Install packages with `npx expo install`**, not `npm install`, so versions match your Expo SDK.
- **Run `npx tsc --noEmit` now and then.** It checks your types across the whole project.
- **Stuck for 20 minutes?** Ask in the thread with the error text and the file you were in. I'll explain, not rewrite.
