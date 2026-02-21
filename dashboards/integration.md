---
type: dashboard
tags: [dashboard, integration]
---
# Integration Status

## Integration Gaps

```dataview
TABLE WITHOUT ID
  file.link AS "Feature",
  craft AS "Craft",
  status AS "Status"
FROM "features"
WHERE contains(file.content, "❌") OR contains(file.content, "⚠️")
SORT craft ASC
```

## Recent Integration Checks

```dataview
TABLE WITHOUT ID
  file.link AS "Check",
  craft AS "Craft",
  updated AS "Date"
FROM "debug-log"
WHERE contains(tags, "integration")
SORT updated DESC
LIMIT 10
```
