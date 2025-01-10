Notice that the query below also includes a "Copy Canvas JSON to Clipboard" button below it. With the content in the clipboard, just open a new canvas file into any text editor and paste the content of the clipboard onto it, and open the canvas in Obsidian.

See [video](https://youtu.be/P0PR-TYxD1U) for more details.  


```dataviewjs
const statuses = ["read"];
const year = 2024;
const month = 12;

// Fetch and group data
const pages = dv
    .pages('"Books"')
    .where(p => {
        return p.status === "read" &&
            (p.finished && p.finished.find(f => f.year == year));
    })
    .groupBy(p => p.status)
    .sort(p => p.rows.length);

// Function to generate canvas JSON with rows of books
function generateCanvasJSON(groups) {
    const canvasContent = { nodes: [], edges: [] }; // Canvas structure
    const spacingX = 300; // Horizontal spacing between books
    const spacingY = 300; // Vertical spacing between rows
    const maxBooksPerRow = 10; // Maximum books per row

    let x = 0;
    let y = 0;

    groups.forEach((group, groupIndex) => {
        // Add a group label node
        canvasContent.nodes.push({
            id: `group-${groupIndex}`,
            type: "text",
            x: x,
            y: y,
            width: 300,
            height: 50,
            text: `Status: ${group.key}`,
            color: "#FFCC00", // Group background color
            borderColor: "#000000"
        });

        // Offset for books in the group
        let bookX = x;
        let bookY = y + spacingY;

        group.rows.forEach((book, bookIndex) => {
            const fileName = book.file.name; // Extract file name
            const bookLink = `![[${fileName}#^cover]]`; // Markdown link

            // Add a node for the book
            canvasContent.nodes.push({
                id: `book-${groupIndex}-${bookIndex}`,
                type: "text",
                x: bookX,
                y: bookY,
                width: 250,
                height: 285,
                text: `${bookLink}`,
                color: book.tags.includes("favorite") ? "4" : "3", // Book background color
                borderColor: "#000000"
            });

            // Update x for the next book in the row
            bookX += spacingX;

            // If the row is full, move to the next row
            if ((bookIndex + 1) % maxBooksPerRow === 0) {
                bookX = x; // Reset x to the start of the row
                bookY += spacingY; // Move down to the next row
            }
        });

        // Move y down for the next group
        y = bookY + spacingY;
    });

    return JSON.stringify(canvasContent, null, 2); // Return as formatted JSON
}

// Display table results
for (let group of pages) {
    dv.header(3, group.key);
    dv.table(
        ["", "Name", "Finished"],
        group.rows
            .sort(k => k.file.mtime.ts, 'desc')
            .map(k => [
                k.cover ? `![|60](${k.cover})` : "No Cover",
                k.file.link,
                k.finished ? k.finished.map(d => d.year).join(", ") : "n/a"
            ])
    );
}

// Add a button for copying the canvas content
const buttonContainer = document.createElement("div");
buttonContainer.style.marginTop = "10px";
dv.container.appendChild(buttonContainer);

const copyButton = document.createElement("button");
copyButton.textContent = "Copy Canvas JSON to Clipboard";
copyButton.style.cursor = "pointer";
buttonContainer.appendChild(copyButton);

copyButton.addEventListener("click", () => {
    const canvasJSON = generateCanvasJSON(pages);
    navigator.clipboard.writeText(canvasJSON).then(() => {
        alert("Canvas JSON copied to clipboard!");
    });
});
```
