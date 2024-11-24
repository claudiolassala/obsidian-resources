
```dataviewjs
dv
.table(["", "File", "Formats"], 
dv.pages('"Books"')
.where(p => !p.formats)
  .sort(p => p.path, 'asc')
  .map(b => ["![|60](" + b.cover + ")" , b.file.link, b.formats]))
```
