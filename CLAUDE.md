# Working on Mangayomi

Decide the structure before writing the code: what each piece is
responsible for, where the data flows, and where the new thing belongs.
Then write the smallest change that fits that structure.

## Where code goes

- **Screens draw; they don't decide.** A widget reads state and calls
  methods. Logic (what happens on a key, which episode is next, how a
  setting is stored) lives in a Riverpod provider or a plain Dart class
  next to the feature, never inside `build()` or a 1,000-line `State`.
- **Data goes through `lib/repositories/`.** Screens and providers don't
  query Isar directly.
- **UI comes from Shiori** (`packages/shiori`): its tokens, components and
  `Symbols` icons. No new hand-styled `Container`s for something Shiori
  already has.
- **Files don't grow past 1,000 lines.** If the change would make one
  bigger, extract the part you touch first (in its own commit), then
  change it. `anime_player_view.dart`, `reader_view.dart` and
  `novel_reader_view.dart` are already past that: shrink them, never grow
  them.
- **Reuse before you add.** Search for an existing helper, provider or
  widget before writing one.

## Errors

- No empty `catch (_) {}`. Catch the specific error you expect and handle
  it; otherwise log it with `AppLogger.log(..., logLevel: LogLevel.error)`
  so a failure leaves a trace. A login or a network call that fails
  silently costs hours to debug.

## Settings

- A new setting is a field on `Settings` (`lib/models/settings.dart`),
  wired in its `fromJson`/`toJson` so backups carry it, plus a provider
  that reads and writes it through `settingsRepository`.
- Add the row to `lib/modules/more/settings/settings_index.dart` so the
  settings search finds it.

## Strings

- Every user-visible string goes in `lib/l10n/app_en.arb` and, translated,
  in every other `app_*.arb`; then run `flutter gen-l10n`. No hard-coded
  text in widgets.

## Generated code

- After changing a `@riverpod` provider or an Isar model, run
  `dart run build_runner build --delete-conflicting-outputs`, then commit
  **only** the generated files of what you changed. The generator may
  rewrite unrelated `.g.dart` / `.freezed.dart` files with a different
  version; restore those with `git checkout`.

## Before every commit

- `dart format` the files you touched (only those: some files aren't
  formatted yet, and reformatting them buries the real change).
- `flutter analyze lib test` has no issues.
- `flutter test` passes, and new logic (a branch, a parser, a key binding)
  has one small test in `test/` that fails if it breaks.

## Pull requests

- One topic per PR. A refactor that moves code and a feature that changes
  behaviour are two PRs: the first changes nothing a user can see.
- Keep it reviewable: if a reviewer can't read the diff in one sitting,
  split it.
- Copied or vendored code goes in its own commit, unchanged, so the next
  commit shows only what was actually modified.
