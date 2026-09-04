# Lindblom Current Schedule

Lindblom Current Schedule is a simple web app that makes Lindblom's rotating daily schedule easier to follow. It gives students and staff a quick answer to four questions:

- What type of schedule are we following today?
- What class or period is happening now?
- How much time remains in the current class?
- What class or period comes next?

## Use the app

Open **[Lindblom Current Schedule](https://mark.sidarous.com/lindsched/)** in any modern web browser. It also works on phones, although the current layout is best viewed on a larger screen.

## Features

- Recognizes Gold, Maroon, 1–8, Colloquium, assembly, finals, weekend, and student non-attendance schedules
- Displays the current date and time
- Shows the current class or passing period
- Counts down to the next scheduled block
- Shows progress through the current block
- Lists the complete schedule for the day and highlights the current block
- Recalculates the current schedule and countdown every second

## How it works

The project is a static GitHub Pages site with no backend, framework, build step, or package dependencies. [`index.html`](./index.html) contains the responsive interface and schedule-selection logic. [`schedule-data.json`](./schedule-data.json) contains the school name, footer text, calendar dates, day types, colors, and reusable bell schedules.

The app loads the JSON file, uses its configured school timezone to select the day, and compares the current time with the schedule's start times to calculate the current block, upcoming block, and countdown.

[`schedule-builder.html`](./schedule-builder.html) provides a visual editor that can load an existing version-3 JSON file, edit calendar dates and schedules, validate the result, and download a replacement `schedule-data.json` file. It is linked from the schedule page footer as **Build or edit a schedule**.

Two earlier self-contained versions remain in the repository for comparison and archival purposes:

- [`index-standalone.html`](./index-standalone.html) is the responsive version with its schedule data embedded in the HTML.
- [`index-legacy.html`](./index-legacy.html) is the original layout and implementation.

## Important notes

- The school calendar may change during the year. This site is maintained manually, so a recent change may not appear immediately.
- The app uses the time reported by your device but interprets it in the school timezone configured in `schedule-data.json`. An incorrect device clock can produce an incorrect result.
- The display recalculates every second. Refresh the page to download a newly published copy of `schedule-data.json`.
- This is an independently maintained convenience tool. If its information conflicts with an official Lindblom announcement or calendar, follow the official source.

## Updating the calendar

To prepare the app for a new school year or calendar revision:

1. Open `schedule-builder.html` through a local web server.
2. Load the current `schedule-data.json` file.
3. Edit calendar dates, day types, colors, site text, or bell schedules.
4. Correct any validation errors and download the replacement JSON file.
5. Replace `schedule-data.json` in the repository without renaming it.
6. Test representative dates for every schedule type, including weekends and non-attendance days.
7. Commit and push the change to the branch published by GitHub Pages.

The complete data contract is documented in [`SCHEDULE_DATA_SCHEMA_V3.md`](./SCHEDULE_DATA_SCHEMA_V3.md).

## Feedback and corrections

Questions, corrections, feature ideas, and design suggestions are welcome. Please contact Mr. Sidarous or [open an issue](https://github.com/sidarous/lindsched/issues).

Improvements may take time, but reports of incorrect dates or schedule times are especially helpful.
