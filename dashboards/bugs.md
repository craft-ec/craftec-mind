---
type: dashboard
tags: [dashboard, bugs]
---
# Open Investigations

## Active Investigations

```dataview
TABLE WITHOUT ID
  file.link AS "Investigation",
  craft AS "Craft",
  component AS "Component",
  status AS "Status",
  updated AS "Last Updated"
FROM "debug-log"
WHERE type = "debug-log" AND status != "resolved"
SORT updated DESC
```

## Recently Resolved

```dataview
TABLE WITHOUT ID
  file.link AS "Investigation",
  craft AS "Craft",
  updated AS "Resolved"
FROM "debug-log"
WHERE type = "debug-log" AND status = "resolved"
SORT updated DESC
LIMIT 10
```
