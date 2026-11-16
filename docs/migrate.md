# Moving a project to a newer format

An update never changes your projects. When a release changes how a project is stored, the projects you made before it are in an **older format**, and this release treats them with care:

- A project in an older format **opens read only**. You can look at it, run it and export it. Every attempt to change it (a save, a change from the assistant, Undo, Redo, a restore from the history) is refused with `needs-migration`, and the page shows a notice at the top.
- Nothing is changed until you decide. **Migrating** is a step you take, one project at a time, when you are ready.
- A project in a **newer** format than this release knows is not written at all (`project-too-new`): update the platform first.

The format is the line `format = N` under `[project]` in the project's `platform.toml`. A project with no such line is format 1.

## What migrating does

1. **A backup first.** The project is copied to `<project>/.migrate-backup/format<N>-<date and time>/`. The backup holds everything but the backups.
2. **The steps run**, one for each format between the project's and the current one. A step is described before it runs, in words.
3. **The project is checked.** If a step fails, or the check fails, the project is **put back from the backup** and the message says so.

## On the page

The notice shows what migrating would change (a dry run; nothing is written yet) and a **Migrate** button. When it is done it names the backup and offers **Put the backup back**. Reload the page to work on the migrated project.

## On the command line

```
platform migrate <project> --dry-run      what would change; nothing is written
platform migrate <project>                make the backup, then migrate
platform migrate <project> --restore      put the last backup back
platform migrate --all [--dry-run]        every project of the volume that needs it
```

`<project>` is a folder, or a name inside the volume (`--workspace <folder>`; in the image, `/workspace`). In the product image: `docker run --rm -v <your folder>:/workspace <image> migrate <project> --dry-run`. The command works on the files, so stop the server, or reload the page, before and after it.

## Going back

`platform migrate <project> --restore` (or **Put the backup back** on the page, right after a migration) returns the project to the last backup, including anything you changed after the migration. The backups stay, so you can go back and forth; delete a `.migrate-backup` folder when you no longer need it.
