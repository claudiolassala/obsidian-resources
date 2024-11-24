```dataviewjs
dv.table(
  ["", "File", "Read"], 
  dv.pages('"Books"')
    .where(p => p.tags && p.tags.includes("review")) // Check if tags exists and includes "review"
    .map(b => [
      `![|60](${b.cover || ""})`, // Handle cases where cover might be undefined
      b.file.link,
      (b.finished || []).map(f => new Date(f).getFullYear()) // Handle cases where finished might be undefined
    ])
);
```