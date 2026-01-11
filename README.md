# Personal Dashboard

A modern, highly customizable personal dashboard built with **SvelteKit** and **TypeScript**. Inspired by the aesthetics of VSCode and Notion, it features a "True Black" theme, a robust file-tree note-taking system, and a clean, distraction-free interface.

## Features

- **🎨 True Black Theme**: A deep, OLED-friendly dark mode (inspired by VSCode).
- **📝 Notion-Style Notes**: 
  - Hierarchical folder structure.
  - Collapsible sidebar with Notion-like interactions.
  - Auto-saving local storage persistence.
  - Rich-text-ready editor interface.
- **⚡ Quick Actions**:
  - Global Search (`Ctrl+K`) for notes.
  - Sidebar Toggle (`Ctrl+B`) for focus mode.
- **📊 Activity Bar**: Quick links to frequent destinations (GitHub, YouTube, X, Discord).

## Tech Stack

- **Framework**: [SvelteKit](https://kit.svelte.dev/)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **Build Tool**: [Vite](https://vitejs.dev/)
- **Styling**: Vanilla CSS (Variables & Design Tokens)

## Project Structure

A visual breakdown of the core application files. Files marked with `*` were created or significantly refactored for this project.

```text
my-app/
├── src/
│   ├── lib/
│   │   └── components/
│   │       └── Notes.svelte *     # Core note-taking engine. Handles folders, navigation, local storage, and the editor.
│   ├── routes/
│   │   ├── +layout.svelte         # Root layout wrapper.
│   │   └── +page.svelte *         # Main Dashboard Shell. managing the Layout, Top Bar, Activity Bar, Search, and Status Bar.
│   ├── app.css *                  # Global styles. Defines the "True Black" theme variables and utility classes.
│   └── app.d.ts                   # TypeScript declarations.
├── static/                        # Static assets (favicons, etc).
├── svelte.config.js               # SvelteKit configuration.
└── vite.config.ts                 # Vite configuration.
```

## File Breakdown

### `src/routes/+page.svelte` *
The entry point of the dashboard. It implements the "App Shell" layout pattern:
- **Activity Bar**: Left-most icons for navigation and sidebar toggling.
- **Top Bar**: Host for the Date/Time display and the global Search input.
- **Main Area**: Renders the `Notes` component.
- **Status Bar**: Display for application status and metadata (e.g., note count).

### `src/lib/components/Notes.svelte` *
The heart of the application. It replicates a Notion-like filesystem experience:
- **State Management**: Uses Svelte's `$state` for reactive folders and notes.
- **Persistence**: automatically syncs all data to the browser's `localStorage`.
- **UI Logic**: Handles recursive folder rendering, drag-and-drop-like interactions (via clicks), and the main text editor.

### `src/app.css` *
Contains the design system tokens:
- **Colors**: Defines `--bg-base` (`#000000`), `--bg-surface` (`#0a0a0a`), and `--accent` colors.
- **Typography**: Sets up a clean, system-ui font stack.
- **Utilities**: Common helpers for layout and transitions.

## Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/personal-dashboard.git
   cd personal-dashboard
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the development server**
   ```bash
   npm run dev
   ```

4. **Build for production**
   ```bash
   npm run build
   ```
