

```dataviewjs
dv
.table(["", "File", "Gifted to"], 
dv.pages('"Books"')
.where(p => p.gift_recipients && p.gift_recipients.length > 0)
  .sort(b => b.gift_recipients.length, 'desc')
  .map(b => ["![|60](" + b.cover + ")" , b.file.link, b.gift_recipients]))
```
