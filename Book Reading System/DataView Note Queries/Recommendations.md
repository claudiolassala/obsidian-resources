- Unread
- More than 1 recommendation

```dataviewjs
dv
.table(["", "File", "Recommendations"], 
dv.pages('"Books"')
.where(p => p.recommendations && p.recommendations.length > 1 && p.status !== 'read')
  .sort(b => b.recommendations.length, 'desc')
  .map(b => ["![|60](" + b.cover + ")" , b.file.link, b.recommendations]))
```
