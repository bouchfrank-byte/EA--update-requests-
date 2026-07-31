# Easy Assign — Change Audit

**Baseline:** Easy Assignments `0.7.3` (2026-07-08)
**Current build:** `Easy_Assign_SnapshotSync_FINAL17`
**Purpose:** An accurate, verified summary of everything changed on top of the 0.7.3 release, prepared to share with the original author.

---

## At a glance (verified facts)

| | 0.7.3 baseline | Current (FINAL17) |
|---|---|---|
| Loaded Lua (`EasyAssign.lua`) | 36,974 lines | 42,298 lines (**+5,324**) |
| Additional library | — | `LibDeflate.lua` (3,609 lines, new) |
| Function definitions | 999 | 1,139 (**+140 added, 0 removed**) |
| Interface version (TOC) | 20506 | 20506 (unchanged) |
| SavedVariables | `EasyAssignDB` | `EasyAssignDB` (unchanged — backward compatible) |

Every change is **additive**: no functions from 0.7.3 were removed or replaced out, and the saved-data schema is preserved, so an existing 0.7.3 user's data loads unchanged. The two files' function inventories were compared directly, and the "new system" claims below were each confirmed absent from 0.7.3 in source (e.g. the snapshot protocol strings, the visual-highlight fields, LibDeflate, and the backup panel all have zero occurrences in 0.7.3).

---

## What the 0.7.3 foundation already provided (preserved intact)

Credit where due — 0.7.3 was already a large, capable addon, and all of this was kept working:

- The full planning workspace (Home / Raids / My Assignments / Settings), Raid Builder, and Saved Raid Teams.
- Boss maps with draggable symbols/icons, freehand drawing, and text annotations.
- General raid assignments, per-raid trash assignments, and per-boss assignment templates (including encounter-specific logic such as Netherspite beams).
- Assignment Profiles with full import/export (profiles, segments, single assignments).
- A working addon-comm layer and **scoped live sync** (`EasyAssignQueueRaidStateSync` / `…FlushQueuedRaidStateSync` / `…QueueScopedSync`), a chunked `START/PART/END` share-transfer protocol, **Request Sync**, raid-leader lock/policy controls, and sync authority handling.
- Missing-assignment **warning text** (chat summaries and preset-load warnings).
- JSON roster import and a My Assignments popup.

The work below builds on that foundation rather than replacing it.

---

## What was added (new systems)

### 1. Full-raid Snapshot push/pull sync — `EasyAssignSnapshot.*`
A second, snapshot-oriented sync layer alongside the original scoped sync. Adds a `SNAP` / `SNAPGET` / `SNAPREQ` message protocol with:
- Chunked transmit and reassembly of a complete current-raid snapshot (`Transmit`, `OnChunk`, `Send`, `SendTo`).
- **Missing-chunk recovery** — a receiver detects dropped parts and asks the sender to resend just the gaps (`ResendChunks`).
- **Late-joiner auto-catch-up** — a joiner requests the current state and the raid leader answers, debounced per requester (`RequestCatchup`, `OnCatchupRequest`).
- Self-echo handling so a client doesn't act on its own broadcasts.

### 2. Live visual missing-assignment highlighting — `EasyAssignMissingCheck.*`
0.7.3 produced missing-assignment *text*; this adds the live *visual* layer:
- A two-level amber highlight: boss tabs and individual assignment dropdowns glow when they hold a player who is not in the raid.
- Highlight sets recomputed at render time (`RecomputeSets`, `IsNameMissing`, `RefreshHighlightData`) so they clear/appear the moment an assignment changes.
- A dedicated, clickable "who's missing" popup that lists missing players and jumps to the affected boss (`Create`, `Open`, `Populate`, `Jump`, `StyleRow`).

### 3. Saved Setups
Local, officer-only save / load / delete of full raid setups (`SaveSetup`, `LoadSetup`, `DeleteSetup`, `GetSavedSetupsStore`) with a toolbar menu.

### 4. Payload compression — LibDeflate
Bundled `LibDeflate` (new file) and wired it into the sync/share paths (`EasyAssignGetLibDeflate`, `…MaybeCompressSyncText`, `…DecompressSyncText`, `…IsSyncCompressionEnabled`) to shrink large snapshots over the addon channel.

### 5. Sync hardening
- **Content-fingerprint echo suppression** and order-independent apply, so a genuine edit is applied while exact re-shares are ignored (`EasyAssignComputeRaidAssignmentSignature`, `…ComputeScopeSignature`, `…ScopeContentHashString`).
- A **roster-change re-sync nudge** (`SYNCNOW`) and auto-sync attempt so state re-settles when people join/leave (`EasyAssignBroadcastSyncNow`, `…HandleSyncNowNudge`, `…MaybeBroadcastSyncNowOnRosterChange`, `…AutoSyncAttempt`).

### 6. Cross-client profile field-identity remapping
Stable key maps so profile-generated field ids line up between the sender and receiver, preventing a received assignment from landing on the wrong row (`EasyAssignBuildProfileFieldStableKeyMap`, `…RemapIncomingProfileFieldKeys`, `…FindReceiverProfileFieldId`, `…IndexReceiverProfileFieldIdentities`).

### 7. Personal-window quality of life
- **Auto-open** the personal My Assignments window when assignments arrive (and, in the current build, for the officer who pushed) — `EasyAssignMaybeAutoOpenMyAssignmentsPopup`.
- **Minimize** states for both the My Assignments popup and the Raid Leader window (`…Minimized` getters/setters and apply helpers).

### 8. Backup / Share panel — `EasyAssignBackupShare.*`
A string-based export/import and backup UI with selectable scopes (team, settings, everything, boss assignments, raid assignments, boss map) and integrity checksums (`Adler32`, `ChecksumHex`), separate from the existing assignment-profile import/export.

### 9. Developer tooling
An in-game sync debug log window (`EasyAssignSyncLog`, `…EnsureSyncLogWindow`, `…ShowSyncLogWindow`, `…ToggleSyncLogWindow`) and a slash command handler (`SlashCmdList.EASYASSIGN`).

### 10. Roster unit context menu
Right-click actions on roster units (`EasyAssignBuildRosterUnitMenu`, `EasyAssignShowRosterUnitMenu`).

*(A handful of the 140 new definitions are small local helpers — `walk`, `run`, `relayout`, `sendNext`, `landOnPushedScope`, `emitList`, `emitSingle`, `OnAccept`, `OnShow` — that support the systems above rather than being features in their own right.)*

---

## Recent refinements (this session, builds FINAL15–17)

- **FINAL15** — the missing-assignment highlight now recomputes and re-renders synchronously on a reassign, a setup load, and a snapshot apply, instead of clearing/appearing one action late (it previously waited for the next boss-tab click or raid switch).
- **FINAL16** — sync status messages default **off** (opt-in via a settings checkbox); the missing-assignment highlight was recolored from yellow to amber so it no longer blends with the yellow selected-tab border (the selection border color is unchanged).
- **FINAL17** — a manual push now also opens the pusher's own personal window for review, since a client filters out its own broadcasts and so never gets the receive-side auto-open.

---

## Compatibility & method notes

- **Additive:** 0 functions removed; `EasyAssignDB` schema unchanged; Interface version unchanged — a 0.7.3 profile upgrades in place.
- **Third-party:** `LibDeflate` (Zlib/MIT-licensed) is bundled to provide sync/share compression.
- **How this audit was produced:** the loaded `EasyAssign.lua` from each build was compared function-by-function (999 vs 1,139 definitions), and each new-system claim was verified by searching the 0.7.3 source for its defining identifiers and confirming zero matches.

---

## Suggested housekeeping before sharing / distributing

These are cosmetic/metadata items, not code:

- The TOC still reads `Version: 0.7.3` and `Author: OpenAI`. Consider bumping the version and adding an attribution line that credits the original author alongside your own handle.
- Note the bundled `LibDeflate` and its license in your credits/readme.
