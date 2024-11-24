
```dataviewjs
const statuses = ["reading", "read"];
const year = 2024;

const pages = dv
.pages('"Books"')
.where(p => { 

 return p.status === "reading" ||
		p.status === "read"
	    && (p.finished 
	    && p.finished.find(f => f.year == year))
  }
) 
.groupBy(p => p.status)
.sort(p => p.rows.length)
; 

for (let group of pages) {

  dv.header(3, group.key);
  dv.table(
    ["", "Name", "Finished"],
 	group.rows
	.sort(k => k.file.mtime.ts, 'desc')
	.map(k => ["![|60](" + k.cover + ")", k.file.link, 
	 k["finished"] ? k["finished"].map(d => d.year) : "n/a"]))
}
```

### Favorites

```dataviewjs 
const year = 2024;
dv.table(["", "Name"],
dv.pages('"Books"')
		 .where(q => q.tags && q.tags.includes("favorite") &&
		  (q.finished 
		     && q.finished.find(f => f.year == year))
			 )
		 .map(q => ["![|60](" + q.cover + ")", q.file.link])
		)
```


### Review

```dataviewjs 
const year = 2024;
dv.table(["", "Name"],
dv.pages('"Books"')
		 .where(q => q.tags && q.tags.includes("review") &&
		  (q.finished 
		     && q.finished.find(f => f.year == year))
			 )
		 .map(q => ["![|60](" + q.cover + ")", q.file.link])
		)
```

### Maybe Next...

```dataviewjs 
const year = 2024;
dv.table(["", "Name"],
dv.pages('"Books"')
		 .where(q => q.tags && q.tags.includes("next"))
		 .map(q => ["![|60](" + q.cover + ")", q.file.link])
		)
```
