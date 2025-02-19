// Create the button
const button = dv.el("button", "Export to Canvas", { cls: "button" });

button.addEventListener("click", async () => {
  // Prompt the user for a canvas file name
  const userCanvasPath = await new Promise((resolve) => {
    let modal = new PromptModal("Enter canvas filename (e.g., Music Canvas.canvas)", resolve);
    modal.open();
  });

  // Validate user input
  if (!userCanvasPath || userCanvasPath.trim() === "") {
    new obsidian.Notice("Canvas export canceled.");
    return;
  }

  const canvasPath = userCanvasPath.trim().endsWith(".canvas") ? userCanvasPath.trim() : `${userCanvasPath.trim()}.canvas`;
  let canvasFile = app.vault.getAbstractFileByPath(canvasPath);

  let canvasData = {
    nodes: [],
    edges: [],
  };

  if (canvasFile) {
    try {
      const existingData = await app.vault.read(canvasFile);
      canvasData = JSON.parse(existingData);
      if (!Array.isArray(canvasData.nodes)) canvasData.nodes = [];
      if (!Array.isArray(canvasData.edges)) canvasData.edges = [];
    } catch (error) {
      console.error("Error parsing existing canvas JSON:", error);
      canvasData = { nodes: [], edges: [] };
    }
  } else {
    await app.vault.create(canvasPath, JSON.stringify(canvasData, null, 2));
    canvasFile = app.vault.getAbstractFileByPath(canvasPath);
  }

  if (!canvasFile || typeof canvasFile !== "object" || !("path" in canvasFile)) {
    console.error("Canvas file not found or invalid after creation.");
    new obsidian.Notice("Error: Canvas file could not be accessed.");
    return;
  }

  const existingNodes = new Set(canvasData.nodes.map(n => n.text));
  
  // Find the maximum y position to append new items below existing ones
  let maxY = canvasData.nodes.length > 0
    ? Math.max(...canvasData.nodes.map(n => n.y)) + 150 // Leave some space below the last item
    : 0;

  const numColumns = Math.ceil(Math.sqrt(pages.length)); // Define the number of columns
  const spacingX = 300; // Horizontal spacing
  const spacingY = 100; // Vertical spacing
  const centerX = numColumns * spacingX / 2; // Center the grid horizontally

  let newNodes = [];

  pages.forEach((b, index) => {
    const text = `[[${b.file.link.path.replace(/^.*\/([^\/]+)\.md$/, '$1')}]]`;
    const charWidth = 8;
    const minWidth = 250;
    const padding = 40;
    const width = Math.max(minWidth, text.length * charWidth + padding);

    if (!existingNodes.has(text)) {
      const row = Math.floor(index / numColumns);
      const col = index % numColumns;
      
      newNodes.push({
        id: crypto.randomUUID(),
        x: col * spacingX - centerX, // Center the grid horizontally
        y: maxY + row * spacingY, // Append below existing nodes
        width: width,
        height: 60,
        type: "text",
        text: text,
      });
    }
  });

  if (newNodes.length > 0) {
    canvasData.nodes.push(...newNodes);
    const canvasJSON = JSON.stringify(canvasData, null, 2);
    await app.vault.modify(canvasFile, canvasJSON);
    new obsidian.Notice(`New notes added to ${canvasPath}!`);
  } else {
    new obsidian.Notice("No new notes to add.");
  }
});

/**
 * Helper class to create a simple modal for user input.
 */
class PromptModal extends obsidian.Modal {
  constructor(placeholder, callback) {
    super(app);
    this.placeholder = placeholder;
    this.callback = callback;
  }

  onOpen() {
    let { contentEl } = this;
    contentEl.empty();

    contentEl.createEl("h2", { text: "Export to Canvas" });

    let inputEl = contentEl.createEl("input", {
      type: "text",
      placeholder: this.placeholder,
    });

    inputEl.style.width = "100%";
    inputEl.focus();

    let buttonContainer = contentEl.createEl("div", { cls: "modal-button-container" });

    let confirmButton = buttonContainer.createEl("button", { text: "Confirm" });
    let cancelButton = buttonContainer.createEl("button", { text: "Cancel" });

    confirmButton.onclick = () => {
      this.callback(inputEl.value);
      this.close();
    };

    cancelButton.onclick = () => {
      this.callback(null);
      this.close();
    };
  }

  onClose() {
    let { contentEl } = this;
    contentEl.empty();
  }
}