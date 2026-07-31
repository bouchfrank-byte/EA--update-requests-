# Easy Assign — changes on top of 0.7.3 (for review)

This is a set of changes built on top of **Easy Assignments 0.7.3**. Everything here is plain,
human-readable Lua — nothing compiled, nothing hidden. You don't need to install anything to review
it: read the diff, and if you want any of it, merge it yourself.

## What's in here

- **`EasyAssign_0.7.3_to_FINAL17.diff`** — a full line-by-line unified diff of `EasyAssign.lua`
  (and the one-line `.toc` change) from 0.7.3 to the current build. This is the complete, exact record
  of what changed. Roughly **5,470 lines added, ~150 changed/removed, across 123 hunks** — almost
  entirely additions.
- **`EasyAssign_Change_Audit_0.7.3_to_FINAL17.md`** — a plain-English summary grouping the changes by
  system, so you can decide what's worth reading in the diff.

## Which files actually change

Only three files are touched, and no new art/assets are needed (the new code reuses textures 0.7.3
already ships):

1. **`EasyAssign.lua`** — where essentially every change lives (see the diff).
2. **`Easy Assign.toc`** — one added line so the library below loads first.
3. **`LibDeflate.lua`** — a **new dependency**, *not* my code (see next section).

## About `LibDeflate` (not written by me)

The compression paths use **LibDeflate** — a well-known, widely-used public WoW library
(`_MAJOR = "LibDeflate"`, minor 3), **zlib-licensed**, by Haoqian "SafeteeWoW" He. It is **not**
included in this diff on purpose: you should pull your own copy from the official source rather than
take a library file from a stranger. It's available on CurseForge / WoWAce and GitHub
(search "LibDeflate"). Once you have the official file, the only code from me you'd be running is
`EasyAssign.lua`.

## Compatibility

- **Additive:** no functions were removed (999 → 1,139 definitions; +140, 0 removed).
- **Data-safe:** `SavedVariables` is still `EasyAssignDB` with the same schema, so existing profiles
  load in place — no reset, no migration.
- **Interface version unchanged** (20506).

## How to review or take pieces

- Read `EasyAssign_0.7.3_to_FINAL17.diff` directly (GitHub renders it with highlighting), or
  `git apply` it to your own 0.7.3 tree to see it in context.
- The changes are grouped into fairly independent systems (see the audit), so you can lift just the
  parts you want — e.g. the amber missing-assignment highlight, or the snapshot sync layer — rather
  than taking the whole file.

No pressure to use any of it — it's your project, and this just puts the changes in front of you to
review on your own terms.
