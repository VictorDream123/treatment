---
{"dg-publish":true,"permalink":"/Instant_notes/Notes/Int.0932_Version 1/"}
---

> 此文档描述了version 1的核心需求
## 需求描述 **Project Summary & Requirements: AI Writing Editor**

#### **1. Core Objective & Guiding Philosophy**

The primary goal is to create a writing assistant that acts as a "proofreader" or "content editor," not a creative writer. The target user wants to improve their writing skills by identifying and correcting their own objective errors.

**Key Principles:**
* The tool should only identify objective errors: spelling, grammar, punctuation, style, and basic logic.
* It must **not** rewrite or heavily rephrase content, as the user wants to maintain their own voice and improve their skills.
* The user is wary of over-using AI and wants to avoid tools that diminish their own writing ability.
* The final output should be a higher-quality text, corrected by the user based on the tool's suggestions.

#### **2. Desired User Workflow**

The process begins with notes taken in Zotero and ends with a fully-featured web application for proofreading.

1. **Zotero:** The user writes their initial text in Zotero, which includes Markdown, LaTeX, images, and Zotero-specific HTML for highlights (`<span class="highlight">`) and citations (`<span class="citation">`).
2. **Python Pre-processing:** The user exports the note as Markdown. A separate Python script (already completed) is run on this file. This script uploads all local images to a GitHub repository and replaces the local paths with public CDN URLs.
3. **AI Editor Application:** The user copies the processed Markdown (which now contains public image URLs, but still has the custom Zotero HTML and LaTeX) and pastes it into the web application we are building.

#### **3. Key Application Features**

The web application must be a sophisticated, single-pane editor with the following capabilities:

* **Obsidian-like "Live Preview":** This is the most critical requirement. There should **not** be a separate source and preview pane. The editor must be a single view that displays the text in a rendered, easy-to-read format by default. When the user clicks on a rendered element (like a heading or bold text), the underlying Markdown source (`#`, `**`) should appear for editing.
* **Comprehensive Rendering:** The editor must correctly render all elements from the input file simultaneously:
    * Standard Markdown.
    * Images from public URLs.
    * LaTeX formulas (both inline `$...$` and block `$$...$$`).
    * The custom HTML from Zotero (e.g., displaying text within a `<span class="highlight">` with a yellow background).
* **AI-Powered Highlighting:**
    * The application must send the text content to the Doubao AI API.
    * The AI's prompt requires it to return a JSON object containing a list of errors, with each error including its **`startIndex`** and **`endIndex`** character positions.
    * The application must then highlight these precise error locations directly within the "Live Preview" editor.
* **Supporting UI:** The application should include sidebars and other elements to display a list of detected errors and provide a "Running Log" of the analysis process.

#### **4. Technical Stack & Environment**

* **Frontend:** Plain HTML, CSS, and JavaScript.
* **Core Editor Library:** **CodeMirror 6**. We have determined that this is essential to achieve the "Live Preview" functionality.
* **Rendering Libraries:**
    * Markdown: `markdown-it`
    * LaTeX: `KaTeX`
* **AI Model:** `doubao-seed-1-6-250615` via the Doubao API.
* **Deployment:** The application is run as a set of local files (`index.html`, `style.css`, `script.js`) in a web browser.