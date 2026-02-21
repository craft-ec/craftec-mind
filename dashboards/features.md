---
type: dashboard
tags: [dashboard, features]
---
# Feature Progress

## All Features by Craft

```dataview
TABLE WITHOUT ID
  file.link AS "Craft",
  status AS "Status",
  length(filter(file.tasks, (t) => t.completed)) + "/" + length(file.tasks) AS "Progress"
FROM "features"
WHERE type = "feature"
SORT craft ASC
```

## In Progress

```dataview
LIST
FROM "features"
WHERE status = "in-progress"
SORT updated DESC
```

## Blocked Features

```dataview
TABLE WITHOUT ID
  file.link AS "Feature",
  craft AS "Craft"
FROM "features"
WHERE contains(file.content, "Blockers") AND status != "done"
SORT craft ASC
```
