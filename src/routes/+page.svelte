<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import Notes from '$lib/components/Notes.svelte';

  let time = $state(new Date());
  let interval: ReturnType<typeof setInterval>;
  let searchQuery = $state('');
  let showSearch = $state(false);
  let searchInput: HTMLInputElement;
  let notesComponent: Notes;
  let sidebarVisible = $state(true);

  interface SearchResult {
    id: number;
    title: string;
    content: string;
    folderId: number;
    updatedAt: number;
  }

  let allNotes = $state<SearchResult[]>([]);

  onMount(() => {
    interval = setInterval(() => {
      time = new Date();
    }, 1000);

    const savedNotes = localStorage.getItem('dashboard-notes');
    if (savedNotes) {
      allNotes = JSON.parse(savedNotes);
    }

    const pollInterval = setInterval(() => {
      const notes = localStorage.getItem('dashboard-notes');
      if (notes) allNotes = JSON.parse(notes);
    }, 1000);

    const handleKeydown = (e: KeyboardEvent) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        showSearch = true;
        setTimeout(() => searchInput?.focus(), 50);
      }
      if (e.key === 'Escape') {
        showSearch = false;
        searchQuery = '';
      }
      // Toggle sidebar with Ctrl+B
      if ((e.metaKey || e.ctrlKey) && e.key === 'b') {
        e.preventDefault();
        sidebarVisible = !sidebarVisible;
      }
    };

    document.addEventListener('keydown', handleKeydown);
    return () => {
      document.removeEventListener('keydown', handleKeydown);
      clearInterval(pollInterval);
    };
  });

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });

  const formatTime = (date: Date) => {
    return date.toLocaleTimeString('en-US', {
      hour: 'numeric',
      minute: '2-digit',
      hour12: true
    });
  };

  const formatDate = (date: Date) => {
    return date.toLocaleDateString('en-US', {
      weekday: 'short',
      month: 'short',
      day: 'numeric'
    });
  };

  function openSearch() {
    showSearch = true;
    setTimeout(() => searchInput?.focus(), 50);
  }

  const searchResults = $derived(
    searchQuery.trim()
      ? allNotes.filter(note => {
          const query = searchQuery.toLowerCase();
          return note.title.toLowerCase().includes(query) || 
                 note.content.toLowerCase().includes(query);
        }).slice(0, 8)
      : []
  );

  function selectResult(note: SearchResult) {
    showSearch = false;
    searchQuery = '';
    if (notesComponent) {
      notesComponent.selectNoteById(note.id);
    }
  }

  function highlightMatch(text: string, query: string): string {
    if (!query.trim()) return text;
    const regex = new RegExp(`(${query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi');
    return text.replace(regex, '<mark>$1</mark>');
  }

  function formatResultDate(timestamp: number) {
    const date = new Date(timestamp);
    return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
  }
</script>

<svelte:head>
  <title>Dashboard</title>
</svelte:head>

<div class="app">
  <!-- Activity Bar (Left) -->
  <aside class="activity-bar">
    <div class="activity-top">
      <!-- Sidebar Toggle -->
      <button 
        class="activity-item" 
        class:active={sidebarVisible}
        onclick={() => sidebarVisible = !sidebarVisible} 
        title="Toggle sidebar (Ctrl+B)"
      >
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/>
          <line x1="9" y1="3" x2="9" y2="21"/>
        </svg>
      </button>
    </div>
    
    <div class="activity-bottom">
      <!-- GitHub -->
      <a href="https://github.com" target="_blank" rel="noopener noreferrer" class="activity-item" title="GitHub">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
        </svg>
      </a>
      <!-- YouTube -->
      <a href="https://youtube.com" target="_blank" rel="noopener noreferrer" class="activity-item" title="YouTube">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
          <path d="M23.498 6.186a3.016 3.016 0 0 0-2.122-2.136C19.505 3.545 12 3.545 12 3.545s-7.505 0-9.377.505A3.017 3.017 0 0 0 .502 6.186C0 8.07 0 12 0 12s0 3.93.502 5.814a3.016 3.016 0 0 0 2.122 2.136c1.871.505 9.376.505 9.376.505s7.505 0 9.377-.505a3.015 3.015 0 0 0 2.122-2.136C24 15.93 24 12 24 12s0-3.93-.502-5.814zM9.545 15.568V8.432L15.818 12l-6.273 3.568z"/>
        </svg>
      </a>
      <!-- X/Twitter -->
      <a href="https://x.com" target="_blank" rel="noopener noreferrer" class="activity-item" title="X">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
          <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
        </svg>
      </a>
      <!-- Discord -->
      <a href="https://discord.com" target="_blank" rel="noopener noreferrer" class="activity-item" title="Discord">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
          <path d="M20.317 4.3698a19.7913 19.7913 0 00-4.8851-1.5152.0741.0741 0 00-.0785.0371c-.211.3753-.4447.8648-.6083 1.2495-1.8447-.2762-3.68-.2762-5.4868 0-.1636-.3933-.4058-.8742-.6177-1.2495a.077.077 0 00-.0785-.037 19.7363 19.7363 0 00-4.8852 1.515.0699.0699 0 00-.0321.0277C.5334 9.0458-.319 13.5799.0992 18.0578a.0824.0824 0 00.0312.0561c2.0528 1.5076 4.0413 2.4228 5.9929 3.0294a.0777.0777 0 00.0842-.0276c.4616-.6304.8731-1.2952 1.226-1.9942a.076.076 0 00-.0416-.1057c-.6528-.2476-1.2743-.5495-1.8722-.8923a.077.077 0 01-.0076-.1277c.1258-.0943.2517-.1923.3718-.2914a.0743.0743 0 01.0776-.0105c3.9278 1.7933 8.18 1.7933 12.0614 0a.0739.0739 0 01.0785.0095c.1202.099.246.1981.3728.2924a.077.077 0 01-.0066.1276 12.2986 12.2986 0 01-1.873.8914.0766.0766 0 00-.0407.1067c.3604.698.7719 1.3628 1.225 1.9932a.076.076 0 00.0842.0286c1.961-.6067 3.9495-1.5219 6.0023-3.0294a.077.077 0 00.0313-.0552c.5004-5.177-.8382-9.6739-3.5485-13.6604a.061.061 0 00-.0312-.0286zM8.02 15.3312c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9555-2.4189 2.157-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.9555 2.4189-2.1569 2.4189zm7.9748 0c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9554-2.4189 2.1569-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.946 2.4189-2.1568 2.4189z"/>
        </svg>
      </a>
      <!-- Settings -->
      <button class="activity-item" title="Settings">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <circle cx="12" cy="12" r="3"/>
          <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"/>
        </svg>
      </button>
    </div>
  </aside>

  <!-- Main Content -->
  <div class="main-wrapper">
    <!-- Top Bar -->
    <header class="top-bar">
      <div class="top-left">
        <span class="datetime">
          <span class="date">{formatDate(time)}</span>
          <span class="divider">•</span>
          <span class="time">{formatTime(time)}</span>
        </span>
      </div>
      <div class="top-center">
        <button class="search-box" onclick={openSearch}>
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"/>
            <line x1="21" y1="21" x2="16.65" y2="16.65"/>
          </svg>
          <span class="search-text">Search notes...</span>
          <span class="search-shortcut">Ctrl+K</span>
        </button>
      </div>
      <div class="top-right">
        <!-- Empty for now, could add user avatar etc -->
      </div>
    </header>

    <!-- Notes Area -->
    <main class="content">
      <Notes bind:this={notesComponent} {sidebarVisible} />
    </main>
  </div>

  <!-- Status Bar -->
  <footer class="status-bar">
    <div class="status-left">
      <span class="status-item">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="20 6 9 17 4 12"/>
        </svg>
        Ready
      </span>
    </div>
    <div class="status-right">
      <span class="status-item">{allNotes.length} notes</span>
    </div>
  </footer>
</div>

<!-- Search Modal -->
{#if showSearch}
  <div class="search-overlay" onclick={() => { showSearch = false; searchQuery = ''; }}>
    <div class="search-modal" onclick={(e) => e.stopPropagation()}>
      <div class="search-modal-header">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/>
          <line x1="21" y1="21" x2="16.65" y2="16.65"/>
        </svg>
        <input
          bind:this={searchInput}
          type="text"
          class="search-modal-input"
          placeholder="Search notes..."
          bind:value={searchQuery}
        />
        <button class="search-close" onclick={() => { showSearch = false; searchQuery = ''; }}>
          <span class="esc-key">ESC</span>
        </button>
      </div>
      
      <div class="search-results">
        {#if searchQuery.trim()}
          {#if searchResults.length > 0}
            {#each searchResults as result (result.id)}
              <button class="search-result-item" onclick={() => selectResult(result)}>
                <svg class="result-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
                  <polyline points="14 2 14 8 20 8"/>
                </svg>
                <div class="result-content">
                  <div class="result-title">{@html highlightMatch(result.title || 'Untitled', searchQuery)}</div>
                  <div class="result-preview">{@html highlightMatch(result.content.slice(0, 80) || 'Empty note', searchQuery)}</div>
                </div>
                <span class="result-date">{formatResultDate(result.updatedAt)}</span>
              </button>
            {/each}
          {:else}
            <div class="no-results">
              <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1">
                <circle cx="11" cy="11" r="8"/>
                <line x1="21" y1="21" x2="16.65" y2="16.65"/>
              </svg>
              <span>No notes found for "{searchQuery}"</span>
            </div>
          {/if}
        {:else}
          <div class="search-hint">
            <span>Type to search through all your notes</span>
          </div>
        {/if}
      </div>
    </div>
  </div>
{/if}

<style>
  .app {
    display: grid;
    grid-template-columns: var(--activity-bar-width) 1fr;
    grid-template-rows: 1fr var(--status-bar-height);
    height: 100vh;
    overflow: hidden;
  }

  /* Activity Bar */
  .activity-bar {
    grid-row: 1 / 3;
    background: var(--bg-surface);
    border-right: 1px solid var(--border-subtle);
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: var(--space-3) 0;
  }

  .activity-top, .activity-bottom {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: var(--space-2);
  }

  .activity-item {
    width: 36px;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-muted);
    border-radius: var(--radius-md);
    transition: all var(--transition);
    text-decoration: none;
    position: relative;
  }

  .activity-item:hover {
    color: var(--text-primary);
    background: var(--bg-hover);
  }

  .activity-item.active {
    color: var(--text-primary);
  }

  .activity-item.active::before {
    content: '';
    position: absolute;
    left: 0;
    top: 6px;
    bottom: 6px;
    width: 2px;
    background: var(--accent);
    border-radius: 0 2px 2px 0;
  }

  /* Main Wrapper */
  .main-wrapper {
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  /* Top Bar */
  .top-bar {
    height: var(--top-bar-height);
    min-height: var(--top-bar-height);
    background: var(--bg-surface);
    border-bottom: 1px solid var(--border-subtle);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 var(--space-5);
    gap: var(--space-4);
  }

  .top-left, .top-right {
    flex: 1;
  }

  .top-right {
    text-align: right;
  }

  .top-center {
    flex: 2;
    max-width: 480px;
  }

  .datetime {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-size: 12px;
  }

  .date {
    color: var(--text-secondary);
    font-weight: 500;
  }

  .divider {
    color: var(--text-faint);
  }

  .time {
    color: var(--text-primary);
    font-weight: 600;
    font-variant-numeric: tabular-nums;
  }

  .search-box {
    width: 100%;
    display: flex;
    align-items: center;
    gap: var(--space-3);
    padding: var(--space-2) var(--space-4);
    background: var(--bg-elevated);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-lg);
    color: var(--text-muted);
    cursor: pointer;
    transition: all var(--transition);
  }

  .search-box:hover {
    border-color: var(--border-default);
    background: var(--bg-hover);
  }

  .search-text {
    flex: 1;
    font-size: 13px;
    text-align: left;
  }

  .search-shortcut {
    font-size: 11px;
    padding: 3px 8px;
    background: var(--bg-base);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-sm);
    font-family: monospace;
  }

  /* Content */
  .content {
    flex: 1;
    overflow: hidden;
  }

  /* Status Bar */
  .status-bar {
    grid-column: 1 / 3;
    background: var(--accent);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 var(--space-4);
    font-size: 11px;
  }

  .status-left, .status-right {
    display: flex;
    align-items: center;
    gap: var(--space-4);
  }

  .status-item {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    color: white;
    font-weight: 500;
  }

  /* Search Modal */
  .search-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.75);
    display: flex;
    align-items: flex-start;
    justify-content: center;
    padding-top: 12vh;
    z-index: 1000;
    backdrop-filter: blur(4px);
  }

  .search-modal {
    width: 100%;
    max-width: 600px;
    background: var(--bg-elevated);
    border: 1px solid var(--border-default);
    border-radius: var(--radius-lg);
    box-shadow: 0 24px 48px rgba(0, 0, 0, 0.5);
    overflow: hidden;
  }

  .search-modal-header {
    display: flex;
    align-items: center;
    gap: var(--space-3);
    padding: var(--space-4) var(--space-5);
    border-bottom: 1px solid var(--border-subtle);
    color: var(--text-muted);
  }

  .search-modal-input {
    flex: 1;
    font-size: 16px;
    color: var(--text-primary);
    background: transparent;
    border: none;
  }

  .search-modal-input:focus {
    outline: none;
  }

  .search-close {
    color: var(--text-muted);
  }

  .esc-key {
    font-size: 10px;
    padding: 3px 6px;
    background: var(--bg-base);
    border: 1px solid var(--border-subtle);
    border-radius: var(--radius-sm);
    font-family: monospace;
  }

  .search-results {
    max-height: 400px;
    overflow-y: auto;
  }

  .search-hint {
    padding: var(--space-5);
    text-align: center;
    color: var(--text-muted);
    font-size: 13px;
  }

  .no-results {
    padding: var(--space-6);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: var(--space-3);
    color: var(--text-muted);
    font-size: 13px;
  }

  .no-results svg {
    opacity: 0.3;
  }

  .search-result-item {
    width: 100%;
    display: flex;
    align-items: flex-start;
    gap: var(--space-3);
    padding: var(--space-3) var(--space-5);
    text-align: left;
    transition: background var(--transition);
    background: transparent;
    border: none;
    cursor: pointer;
  }

  .search-result-item:hover {
    background: var(--bg-hover);
  }

  .result-icon {
    color: var(--text-muted);
    flex-shrink: 0;
    margin-top: 2px;
  }

  .result-content {
    flex: 1;
    min-width: 0;
  }

  .result-title {
    font-size: 14px;
    font-weight: 500;
    color: var(--text-primary);
    margin-bottom: 2px;
  }

  .result-preview {
    font-size: 12px;
    color: var(--text-muted);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .result-date {
    font-size: 11px;
    color: var(--text-muted);
    flex-shrink: 0;
  }

  :global(.search-result-item mark) {
    background: rgba(0, 120, 212, 0.3);
    color: var(--text-primary);
    border-radius: 2px;
    padding: 0 2px;
  }
</style>
