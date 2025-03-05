+++
title =  'In 2025 moment.js is still the best Date/Time Javascript package'
date = 2025-03-05T22:24:54+08:00
+++

I used `moment.js` frequently between 2017 and 2022, and it worked well for all my use cases. After 2022, I started using `Day.js`, a lightweight library designed to replace `moment.js`, with a size of just 2KB. In my experience, it has worked well in most cases and hasn’t caused any major issues—until this week.

This week, I encountered a serious issue with `Day.js`. The feature in question involves copying last week’s calendar events to the current week. However, March 9th marks the Daylight Saving Time (DST) transition. As a result, my newly copied events were shifted by one hour.

Here’s the problematic code:

```
const timeZone = "America/New_York";

const sourceDay = dayjs.tz(
  "2025-11-01 00:30:00",
  "YYYY-MM-DD HH:mm:ss",
  timeZone
);
console.log(sourceDay.toISOString());

console.log(sourceDay.add(7, "d").toISOString());

const targetDay = dayjs.tz(
  "2025-11-08 00:30:00",
  "YYYY-MM-DD HH:mm:ss",
  timeZone
);
console.log(targetDay.toISOString());
```

And here’s the output:

```text
2025-11-01T04:30:00.000Z
2025-11-08T04:30:00.000Z
2025-11-08T05:30:00.000Z
```

After testing, I confirmed that this is a Day.js bug. Upon searching the issue tracker, I found that someone had already reported it back in [2021](https://github.com/iamkun/dayjs/issues/1271#issuecomment-772542205), yet it remains unfixed after four years.

Then, I became curious to see if `moment.js` or `luxon` had the same issue. After testing, I found that both handled the DST transition correctly without any problems.

This confirmed that the bug is specific to `Day.js`, and it’s not just an edge case—it directly affects time calculations when working with recurring events across DST changes.

At this point, I started wondering: should I continue using Day.js, or is it time to switch back to `moment.js` or `luxon`? While `Day.js` is lightweight and generally reliable, a critical bug like this—one that has remained unfixed for years—makes me question its long-term reliability.

For now, I’ve decided to temporarily work around the issue by manually adjusting for the DST offset, but this experience has made me rethink my choice of date-time libraries.

Have you encountered similar issues with `Day.js` or other libraries? Let me know your thoughts!
