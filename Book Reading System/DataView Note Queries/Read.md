
```dataviewjs
const pages = dv
    .pages('"Books"') // Get pages from the "Books" folder
    .where(p => p.status == "read") // Exclude books with status "reading"
    .groupBy(p => p.status || "Unknown"); // Group by 'status', default to "Unknown" if undefined

for (let group of pages) {
    const groupKey = group.key ? group.key.toUpperCase() : "UNKNOWN"; // Handle undefined keys

    dv.header(3, groupKey); // Display the group header
    dv.table(
        ["", "Name", "Finished", "Recommendations"], // Table headers
        group.rows
            .sort(k => (Array.isArray(k.finished) ? -k.finished.length : 0), 'asc') // Sort by number of finished entries (descending)
            .map(k => [
                k.cover ? `![|60](${k.cover})` : "", // Check for cover existence
                k.file.link,
                Array.isArray(k.finished) // Check if finished is an array
                    ? k.finished
                          .filter(date => !isNaN(new Date(date))) // Ensure valid dates
                          .map(date => `${new Date(date).toISOString().split("T")[0]}`) // Format each as "- YYYY-MM-DD"
                          .join("<br>") // Separate each entry with a line break
                    : "N/A", // Default if finished is not an array or missing
                k.recommendations || "-" // Display recommendations or default to a placeholder
            ])
    );
}
```
