# Project Specification: RepoLens

## 1. Executive Summary
RepoLens is a client-side, zero-footprint utility designed to visualize file structures and metadata for both local directories and public GitHub repositories. It provides developers and researchers with a "heat-mapped" view of a codebase, identifying active development areas and structural bottlenecks without requiring a backend or cloud processing.

## 2. Core Functional Requirements

### 2.1 Hybrid Input Sources
* **Local Scanning:** Utilize the `File System Access API` to allow users to select a local directory.
* **Remote Scanning:** Parse GitHub URLs (e.g., `github.com/user/repo`) to fetch repository trees via the GitHub REST API.

### 2.2 Intelligent Visualization
* **Recursive Tree View:** Nested folder structure using native `<details>` and `<summary>` elements or a custom virtualized list for performance.
* **Breadth-Limited Rendering:** If a directory contains >50 items, the UI must render the first 50 and provide a "Load More" button to append subsequent chunks.
* **Media Aggregation:** Automatically group all image files (png, jpg, svg, webp, gif) into a single virtual node: `🖼️ Images ({count})`. Clicking this node should list the images in the sidebar rather than expanding the tree.
* **Active Path Highlighting:** Upon selecting a leaf node (file), the application must programmatically highlight the full path from the root to the selected node, providing immediate spatial context.

### 2.3 Interactive Features & Metadata
* **Details Sidebar:** A slide-out panel triggered by file selection, displaying:
    * Full filename and extension icon.
    * Human-readable file size (e.g., 1.2 MB).
    * Last modification timestamp (Humanized: "2 hours ago" or "Mar 12, 2024").
    * **Action Buttons:** "Copy Relative Path" and "Copy Static URL" (for GitHub sources).
* **Analytical Dashboard:** A toggleable overlay featuring:
    * **File Type Distribution:** A donut or bar chart showing the composition of the repo by extension.
    * **Size Audit:** A list of the top 10 largest files/folders.
    * **Temporal Heatmap:** Folders are color-coded (e.g., varying intensities of blue or orange) based on the most recent edit within that branch.

## 3. Technical Architecture

### 3.1 Frontend Stack
* **Core:** Vanilla JavaScript (ES6+), HTML5, CSS3.
* **Layout:** Tailwind CSS for responsive components and the sidebar.
* **Charts:** Chart.js or D3.js (via lightweight CDN) for the analytical dashboard.
* **State Management:** Reactive state object to track the current tree, filter settings, and UI highlight state.

### 3.2 Developer Modules
1.  **The Scanner (Local):** Implements `window.showDirectoryPicker()`. Uses a recursive async generator to walk the `FileSystemDirectoryHandle`.
2.  **The Scanner (Remote):** Interfaces with `GET /repos/{owner}/{repo}/git/trees/{sha}?recursive=1`. Must handle `truncated: true` flags for massive repositories.
3.  **The Aggregator Middleware:** Intercepts the raw data stream to:
    * Apply `.gitignore` logic.
    * Bundle images into aggregate nodes.
    * Calculate directory-level metadata (total size and latest edit).
4.  **The Path Tracer:** A utility that maps DOM elements to their file-tree nodes to enable upward "trace-back" highlighting when a file is clicked.

## 4. Edge Case Management & Mitigations

| Edge Case | Risk | Mitigation |
| :--- | :--- | :--- |
| **GitHub Rate Limiting** | 403 Forbidden after 60 calls. | Detect rate limit header; prompt user for an optional Personal Access Token (PAT). |
| **Massive Repos (10k+ files)** | Browser UI freeze / DOM bloat. | Mandatory breadth-limit (50 items/chunk) and virtualized scrolling for the tree. |
| **Circular Symlinks** | Infinite recursion in local mode. | Maintain a `Set` of unique folder IDs (inodes) to skip already-visited nodes. |
| **Deep Nesting** | UI horizontal overflow. | Implement a breadcrumb-style navigation or a "Focus on Folder" mode to reset the view root. |
| **No "Last Edit" in API** | GitHub Tree API doesn't return dates. | Fallback: Fetch the latest commit for the root to provide a general date, or allow per-file date fetching on-demand/click. |

## 5. Development Roadmap

* **Milestone 1 (Foundations):** Local directory picker, basic recursive tree rendering, and "Load More" logic.
* **Milestone 2 (External Integration):** GitHub URL parsing and REST API data mapping.
* **Milestone 3 (Insights):** Image aggregation node and the Details Sidebar with copy-path functionality.
* **Milestone 4 (Analytics):** Implementation of the Dashboard charts and the Temporal Heatmap (color-coding).
* **Milestone 5 (Refinement):** Path highlighting logic, dark mode support, and Markdown export.

## 6. UI Interaction Spec
* **Single Click:** Select node, update sidebar, highlight path.
* **Double Click (Folder):** Toggle expand/collapse.
* **Search Bar:** Filter tree in real-time (hiding nodes that don't match).
