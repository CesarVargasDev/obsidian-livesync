# Plugin and Configuration Sync Guide

This guide explains how **Customization Sync (Beta3)** works in Self-hosted LiveSync,
why plugins do not auto-install, and the correct workflow to push plugins and settings
from one device to another.

---

## How it actually works

When a replication item arrives from a remote device, LiveSync updates its internal
list of known configurations — it does **not** automatically write anything to disk.
There is no background auto-install of plugins. Delivery to disk requires one of two
explicit setups described below.

The Customization Sync modal is opened from the ribbon icon or via the command palette
(`Self-hosted LiveSync: Open customization sync`).

---

## The two sync strategies

### 1. Selective / Flagged Selective — manual, you decide per-apply

This is the **default** for every item. The item shows a comparison UI with a device
dropdown and status chips. Nothing happens unless you pick a source and click apply.

Use this when you want control over what gets updated and when.

### 2. Automatic — hands-free, handled by HiddenFileSync

When you set an item to Automatic (✨), its files are handed off to the HiddenFileSync
module and copied to disk automatically on every replication. **You must set this mode
up deliberately on each device.** When you select it, you are asked for an initial
action:

| Option | What it does |
|---|---|
| `↑ Overwrite Remote` | Pushes this device's version to the DB immediately |
| `↓ Overwrite Local` | Pulls the remote version to this device immediately |
| `⇅ Use newer` | Compares timestamps and takes whichever is more recent |

> **Note:** Automatic mode copies the files to disk but does not trigger an Obsidian
> plugin reload. After a plugin update via Automatic mode you may need to restart
> Obsidian once.

---

## Per-item mode reference

Click the emoji button on the left of any item row to change its mode.

| Emoji | Mode | Behaviour |
|---|---|---|
| 🔀 | **Selective** | Default. Full comparison UI shown. Nothing syncs unless you manually apply. |
| ✨ | **Automatic** | Handed to HiddenFileSync. Files sync automatically every replication. |
| ⛔ | **Ignore** | Item is completely skipped — not scanned, not shown, not synced. |
| 🚩 | **Flagged Selective** | Same as Selective, but the item is targeted by the "Select Flagged Shiny" bulk button. Use this to mark items you want to bulk-apply in one click. |

---

## Button reference

| Button | What it does |
|---|---|
| **Scan changes** | Reads every file in `.obsidian/` on this device and writes them into the local database tagged with this device's name. Run this on the source device before syncing. |
| **Sync once** | Triggers a one-shot replication between the local database and the remote. Use after Scan changes on the source, and before Refresh on the destination. |
| **Refresh** | Re-reads the local database and rebuilds the display list. Use this after syncing to see updated statuses. Does not read the file system. |
| **Reload** | *(Maintenance mode only)* Clears the entire in-memory list and rebuilds it from scratch. A harder reset than Refresh. |
| **Select All Shiny** | For every item, automatically pre-selects the remote device that has the most recently modified version. |
| **🚩 Select Flagged Shiny** | Same as Select All Shiny, but only for items set to Flagged Selective (🚩) mode. Items in plain Selective mode are left untouched. |
| **Deselect all** | Clears all pre-selected sources. No apply will happen. |
| **Apply All Selected** | Writes the selected remote's files to disk for every item that has a source selected, then hot-reloads affected plugins. |

---

## Status chips explained

When you select a remote device from a row's dropdown, three status chips appear:

| Chip | Label | Meaning |
|---|---|---|
| 📅 | `Local only` | This item exists only on this device; the remote has nothing. |
| 📅 | `Remote only` | The remote has this item but this device does not. Apply is enabled. |
| 📅 | `Newer (Xm Ys)` | The remote version is newer by that time delta. Apply is enabled. |
| 📅 | `Older (Xm Ys)` | The remote version is older. You can still apply if you want to downgrade. |
| 📅 | `Same` | Timestamps are within 10 seconds of each other. |
| 📄 | `Same` | All file contents are byte-identical. |
| 📄 | `Same or local only` | No file content differs; some files may only exist locally. |
| 📄 | `Different` | At least one file's content differs. Apply and compare (⮂) are enabled. |
| 📄 | `Mixed` | Some files match, some differ, some are missing on one side. |
| 🏷️ | `Same` / `Lower` / `Higher` | Plugin manifest version comparison. |

### "All the same or non-existent"

This message appears when there are **no other devices to compare against** for that
item. It means one of:

- No other device has synced this item yet (the remote DB has no entry for it).
- Every other device's copy is identical and "Hide not applicable items" is on.
- The item simply does not exist anywhere in the database yet.

If you see this on the destination device after syncing, the data has not arrived yet —
run **Sync once** again and then **Refresh**.

---

## Workflow: push plugins from desktop to tablet for the first time

### Option A — Manual (one-time transfer)

**On the desktop (source):**

1. Open Customization Sync.
2. Click **Scan changes** — this uploads your `.obsidian/` state to the local DB.
3. Click **Sync once** — this pushes the DB to the remote server.

**On the tablet (destination):**

4. Open Customization Sync.
5. Click **Sync once** — this pulls from the remote server.
6. Click **Refresh** — the list should now show your desktop's device name in each row.
7. Each row should show `Newer (...)` or `Remote only` in the status chips.
8. Click **Select All Shiny** to auto-select the desktop version for every item.
9. Click **Apply All Selected** — plugins are written to disk and hot-reloaded.
10. If any `CONFIG` item was applied, Obsidian will ask you to restart.

### Option B — Automatic (ongoing, hands-free)

Do this once per item on the **tablet**:

1. Open Customization Sync on the tablet.
2. Complete the one-time manual transfer above first so the item exists locally.
3. Click the mode emoji (🔀) on the item's row.
4. Select **✨ Automatic**.
5. Choose **`↓ Overwrite Local`** if the desktop always has the authoritative version,
   or **`⇅ Use newer`** if both devices may independently update the item.

From now on, that item syncs automatically every replication without opening the modal.

---

## Prerequisite checklist

If nothing appears or statuses are wrong, check these first:

- [ ] **Device name is set** on both devices (LiveSync Settings → General).
  Without a device name `scanAllConfigFiles` silently aborts and nothing is uploaded.
- [ ] **Customization Sync is enabled** in LiveSync Settings
  (`usePluginSync: true`).
- [ ] Desktop has run **Scan changes** at least once. Until it does, the remote DB has
  no entries and the tablet sees "All the same or non-existent" for everything.
- [ ] No item on the tablet is set to **⛔ Ignore** when you expect it to sync.

---

## Resetting a device's stored state (when things are corrupted)

If a device's entries in the DB are stale, wrong, or from a renamed device:

1. Open Customization Sync.
2. Enable **Maintenance mode** (checkbox at the bottom of the modal).
3. A "Delete All of [device]" selector appears at the top.
4. Choose the device name to clean up and click 🗑️.
5. On the affected device: click **Scan changes** (re-uploads current disk state),
   then **Sync once**, then **Refresh**.

Only do this if statuses are genuinely wrong. It is not needed for a normal first-time
setup.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "All the same or non-existent" on all items | Desktop hasn't run Scan changes, or sync hasn't completed | Run Scan changes on desktop → Sync once on both → Refresh on tablet |
| Items appear but Apply does nothing | Item mode is ⛔ Ignore | Change mode to 🔀 Selective |
| Plugin files appear but plugin doesn't load | Obsidian needs a restart after file write | Restart Obsidian on the tablet |
| Automatic mode item isn't syncing | Initial action was never run | Delete the mode setting, re-set to Automatic, choose an initial action |
| Device name missing from dropdown | That device has never run Scan changes | Run Scan changes on that device |
