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
- Refreshes automatically every five minutes

## How it works

The project is a static GitHub Pages site contained in [`index.html`](./index.html). It has no backend, framework, build step, or package dependencies. The HTML, CSS, JavaScript, school calendar dates, and bell schedules are all stored in that file.

The app compares the current date with the configured calendar to select the day's schedule. It then compares the current local time with the schedule's start times to calculate the current block, the upcoming block, and the countdown.

## Important notes

- The school calendar may change during the year. This site is maintained manually, so a recent change may not appear immediately.
- The app uses the date and time reported by your device. An incorrect device clock or timezone can produce an incorrect result.
- The page refreshes every five minutes so it can select a new schedule after the date changes. If a device was asleep overnight, refresh the page or wait for the automatic refresh.
- This is an independently maintained convenience tool. If its information conflicts with an official Lindblom announcement or calendar, follow the official source.

## Updating the calendar

To prepare the app for a new school year or calendar revision:

1. Update the date arrays near the beginning of the `<script>` section in `index.html`.
2. Add or revise the corresponding bell-schedule definitions when schedule times change.
3. Use dates in `YYYY-MM-DD` format so they match the value produced by the app.
4. Test representative dates for every schedule type, including weekends and non-attendance days.
5. Commit and push the change to the branch published by GitHub Pages.

## Feedback and corrections

Questions, corrections, feature ideas, and design suggestions are welcome. Please contact Mr. Sidarous or [open an issue](https://github.com/sidarous/lindsched/issues).

Improvements may take time, but reports of incorrect dates or schedule times are especially helpful.

