# Duplicate File Checker

A small Windows desktop tool that finds files with identical content in a folder
and lets you remove the extra copies safely.

- Single `.exe`, no installer, no window dressing
- Content-based matching (SHA-256), so renamed copies are still caught
- Deletes go to the **Recycle Bin**, not into the void

---

## Quick start

1. Double-click `DuplicateFileChecker.exe`
2. Choose a folder — or drag a folder onto the window, or drop a folder onto the `.exe` icon
3. Press **Scan**
4. Review the groups, untick anything you want to keep
5. Press **Move checked files to Recycle Bin (n)** and confirm with **Yes**

A scan of a typical folder takes a moment; a large drive takes proportionally longer,
and you can stop it at any time with **Esc**.

---

## Reading the results

Files are displayed in groups — one group per set of identical files. The group
header summarises the set:

```
2 copies  -  300 KB each  -  300 KB can be freed
```

Inside each group:

| Column    | Meaning                                        |
| --------- | ---------------------------------------------- |
| File name | Name of this particular copy                   |
| Size      | Size of the file                               |
| Modified  | Last modified timestamp                        |
| Folder    | Where the copy lives                           |

The **checkbox means "delete me"**. When results appear, the first copy in each
group is left unticked and the other copies are ticked, so a single click on
**Move checked files to Recycle Bin** keeps one copy of everything.

The status line shows how many files were scanned and how long it took, plus how
many folders were skipped for lack of access. The bottom-right corner shows the
totals: number of duplicate sets, number of redundant files, and how much space
could be freed.

Double-click any row to jump to that file in File Explorer.

---

## How matching works

1. **Walk the tree** and record every file's size. Folders that cannot be read
   (protected system folders, for example) are skipped and counted, not fatal.
   Symbolic links and junctions are not followed, so the scan can't loop.
2. **Bucket by size.** A file with a unique size cannot have a duplicate, so it
   is dropped immediately. This is what keeps scans fast on big folders.
3. **Hash the survivors** with SHA-256 and group files whose hashes match.

Only files that share a size with at least one other file are ever hashed, and
only hashes are compared — never file names, dates, or extensions. Two files are
reported as duplicates only if their bytes are identical.

Empty files are ignored by default because they are all "identical"; untick
**Ignore empty files** if you actually want them grouped.

---

## Safety

- Nothing is deleted until you press the delete button **and** answer **Yes** in the
  confirmation dialog. **No** is the default answer.
- Files are moved to the Recycle Bin via the standard Windows shell operation, so
  they can be restored from there.
- Only ticked rows are removed. Unticked rows in the same group are untouched, so
  the tool will not wipe out every copy of a file.
- If a file cannot be recycled (locked, permission denied, too large for the bin),
  it is reported afterwards and left in place.
- After a deletion the list is rebuilt from the files that still exist, so you are
  never looking at stale results.

---

## Requirements

- Windows 10 or 11 (x64 or ARM64)
- .NET Framework 4.8 — included with Windows 10 1903+ and Windows 11

No Python, no runtime download, no admin rights. The app runs as your user, which
also means it can only delete what your account is allowed to delete.

> The `.exe` is unsigned, so SmartScreen may show a warning the first time.
> Choose **More info → Run anyway**.

---

## Command line

Pass a folder to scan it immediately on startup:

```bat
DuplicateFileChecker.exe "D:\Photos"
```

This is handy for a shortcut or for dropping a folder onto the executable.

---

## Files in this folder

| File                       | Purpose                     |
| -------------------------- | --------------------------- |
| `DuplicateFileChecker.exe` | The application             |
| `README.md`                | This document               |

The source is a single C# file, `DuplicateFileChecker.cs`, kept in the Bionic
project workspace.

---

## Rebuilding from source

The app is plain WinForms with no third-party dependencies, so the C# compiler
that ships with Windows is enough:

```bat
csc /target:winexe /platform:anycpu /optimize+ /debug- ^
    /out:DuplicateFileChecker.exe ^
    /reference:System.dll /reference:System.Core.dll ^
    /reference:System.Drawing.dll /reference:System.Windows.Forms.dll ^
    DuplicateFileChecker.cs
```

Run it from a *Developer Command Prompt* (or with `csc.exe` from
`C:\Windows\Microsoft.NET\Framework64\v4.0.30319`). Add `/win32icon:icon.ico`
if you have an icon you want to embed.

---

## Limitations

- Single-folder (and its subtree) scope per scan; no multi-folder or drive-wide queue
- No preview/thumbnail comparison — content equality only
- No exclusion patterns; keep a folder out of the scan by scanning a narrower folder
- Results are not saved between runs
