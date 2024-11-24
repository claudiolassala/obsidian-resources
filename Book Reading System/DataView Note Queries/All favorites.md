```dataviewjs 
dv.table(
    ["", "Name", "Finished"], // Updated to match the data columns
    dv.pages('"Books"')
        .where(q => q.tags && q.tags.includes("favorite"))
        .sort(q => (q["finished"] ? q["finished"].length : 0), 'desc') // Sort by number of finished dates in descending order
        .map(q => [
            "![|60](" + q.cover + ")", // Cover image
            q.file.link, // File link
            q["finished"] ? q["finished"].map(d => d.year).join(", ") : "n/a" // Finished years, joined into a string
        ])
);
```
