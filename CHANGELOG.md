# Changelog

## 0.3.0 (the first release)

- **Install and update:** `platform update`, `platform check` and `platform rollback`; a failed update puts the old version back. An update asks for your yes when it changes an interface, needs a migration or brings new tools.
- **Hearing about new releases:** a check once a day, **off until you turn it on**; nothing else is sent. `platform update --install-timer` can also install patch and security updates by itself, and never while you are working.
- **Older projects:** a project in an older format opens read only until you migrate it (`platform migrate`), with a backup first.
- **Work on another machine:** `platform connect HOST`.
- **The platform itself:** long work runs without blocking the page; 3-D scenes and rigid bodies; dashboards with a grid, several blocks per row, and view-only share links; DuckDB for stored data; the user guide inside the page; a disk ceiling you control; a faster, calmer assistant panel.

The full notes are on the release page.
