<script lang="ts">
  import { onMount } from 'svelte';

  interface Note {
    id: number;
    title: string;
    content: string;
    folderId: number;
    updatedAt: number;
  }

  interface Folder {
    id: number;
    name: string;
    expanded: boolean;
  }

  let { sidebarVisible = true }: { sidebarVisible?: boolean } = $props();

  export function selectNoteById(id: number) {
    selectedNoteId = id;
    const note = notes.find(n => n.id === id);
    if (note) {
      // Expand the folder containing this note
      folders = folders.map(f => 
        f.id === note.folderId ? { ...f, expanded: true } : f
      );
      saveFolders();
    }
  }

  let folders = $state<Folder[]>([]);
  let notes = $state<Note[]>([]);
  let selectedNoteId = $state<number | null>(null);
  let editingFolderId = $state<number | null>(null);
  let editingNoteId = $state<number | null>(null);
  let newFolderName = $state('');
  let newNoteName = $state('');

  onMount(() => {
    const savedFolders = localStorage.getItem('dashboard-folders');
    const savedNotes = localStorage.getItem('dashboard-notes');
    
    if (savedFolders) {
      folders = JSON.parse(savedFolders);
      // Ensure expanded property exists
      folders = folders.map(f => ({ ...f, expanded: f.expanded ?? true }));
    }
    if (savedNotes) notes = JSON.parse(savedNotes);
    
    if (folders.length === 0) {
      folders = [{ id: Date.now(), name: 'Personal', expanded: true }];
      saveFolders();
    }
  });

  function saveFolders() {
    localStorage.setItem('dashboard-folders', JSON.stringify(folders));
  }

  function saveNotes() {
    localStorage.setItem('dashboard-notes', JSON.stringify(notes));
  }

  function toggleFolder(id: number) {
    folders = folders.map(f => 
      f.id === id ? { ...f, expanded: !f.expanded } : f
    );
    saveFolders();
  }

  function createFolder() {
    const newFolder = { id: Date.now(), name: 'New Folder', expanded: true };
    folders = [...folders, newFolder];
    editingFolderId = newFolder.id;
    newFolderName = newFolder.name;
    saveFolders();
  }

  function renameFolder(id: number, name: string) {
    folders = folders.map(f => f.id === id ? { ...f, name: name || 'Untitled' } : f);
    editingFolderId = null;
    saveFolders();
  }

  function deleteFolder(id: number) {
    if (folders.length <= 1) return;
    folders = folders.filter(f => f.id !== id);
    notes = notes.filter(n => n.folderId !== id);
    if (selectedNote?.folderId === id) selectedNoteId = null;
    saveFolders();
    saveNotes();
  }

  function createNote(folderId: number) {
    const newNote: Note = {
      id: Date.now(),
      title: 'Untitled',
      content: '',
      folderId,
      updatedAt: Date.now()
    };
    notes = [newNote, ...notes];
    selectedNoteId = newNote.id;
    // Expand the folder
    folders = folders.map(f => f.id === folderId ? { ...f, expanded: true } : f);
    // Start editing the title
    editingNoteId = newNote.id;
    newNoteName = newNote.title;
    saveFolders();
    saveNotes();
  }

  function renameNote(id: number, name: string) {
    notes = notes.map(n => n.id === id ? { ...n, title: name || 'Untitled', updatedAt: Date.now() } : n);
    editingNoteId = null;
    saveNotes();
  }

  function deleteNote(id: number) {
    notes = notes.filter(n => n.id !== id);
    if (selectedNoteId === id) selectedNoteId = null;
    saveNotes();
  }

  function updateNoteContent(value: string) {
    notes = notes.map(n => 
      n.id === selectedNoteId 
        ? { ...n, content: value, updatedAt: Date.now() } 
        : n
    );
    saveNotes();
  }

  function getFolderNotes(folderId: number) {
    return notes
      .filter(n => n.folderId === folderId)
      .sort((a, b) => b.updatedAt - a.updatedAt);
  }

  const selectedNote = $derived(notes.find(n => n.id === selectedNoteId));

  function getWordCount(content: string) {
    return content.trim() ? content.trim().split(/\s+/).length : 0;
  }

  function formatDate(timestamp: number) {
    const date = new Date(timestamp);
    return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
  }
</script>

<div class="notes-app" class:sidebar-hidden={!sidebarVisible}>
  <!-- Sidebar -->
  <aside class="sidebar">
    <div class="sidebar-inner">
      <div class="sidebar-header">
        <span class="sidebar-title">Notes</span>
        <button class="add-folder-btn" onclick={createFolder} title="New folder">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"/>
            <line x1="12" y1="11" x2="12" y2="17"/>
            <line x1="9" y1="14" x2="15" y2="14"/>
          </svg>
        </button>
      </div>

      <nav class="tree">
        {#each folders as folder (folder.id)}
          <div class="tree-folder">
            <!-- Folder row -->
            <div class="tree-item folder-row">
              <button class="chevron" onclick={() => toggleFolder(folder.id)}>
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                  style="transform: rotate({folder.expanded ? 90 : 0}deg); transition: transform 0.15s ease">
                  <polyline points="9 18 15 12 9 6"/>
                </svg>
              </button>
              <svg class="folder-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"/>
              </svg>
              {#if editingFolderId === folder.id}
                <input
                  type="text"
                  class="inline-edit"
                  bind:value={newFolderName}
                  onblur={() => renameFolder(folder.id, newFolderName)}
                  onkeydown={(e) => e.key === 'Enter' && renameFolder(folder.id, newFolderName)}
                />
              {:else}
                <span class="item-name" ondblclick={() => { editingFolderId = folder.id; newFolderName = folder.name; }}>
                  {folder.name}
                </span>
              {/if}
              <div class="item-actions">
                <button class="action-btn" onclick={() => createNote(folder.id)} title="New note">
                  <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <line x1="12" y1="5" x2="12" y2="19"/>
                    <line x1="5" y1="12" x2="19" y2="12"/>
                  </svg>
                </button>
                {#if folders.length > 1}
                  <button class="action-btn delete" onclick={() => deleteFolder(folder.id)} title="Delete folder">
                    <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <polyline points="3 6 5 6 21 6"/>
                      <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/>
                    </svg>
                  </button>
                {/if}
              </div>
            </div>
            
            <!-- Notes under folder -->
            {#if folder.expanded}
              <div class="tree-children">
                {#each getFolderNotes(folder.id) as note (note.id)}
                  <div 
                    class="tree-item note-row" 
                    class:active={selectedNoteId === note.id}
                    onclick={() => selectedNoteId = note.id}
                    role="button"
                    tabindex="0"
                  >
                    <svg class="note-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                      <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
                      <polyline points="14 2 14 8 20 8"/>
                    </svg>
                    {#if editingNoteId === note.id}
                      <input
                        type="text"
                        class="inline-edit"
                        bind:value={newNoteName}
                        onblur={() => renameNote(note.id, newNoteName)}
                        onkeydown={(e) => e.key === 'Enter' && renameNote(note.id, newNoteName)}
                      />
                    {:else}
                      <span class="item-name" ondblclick={() => { editingNoteId = note.id; newNoteName = note.title; }}>
                        {note.title || 'Untitled'}
                      </span>
                    {/if}
                    <div class="item-actions">
                      <button class="action-btn delete" onclick={(e) => { e.stopPropagation(); deleteNote(note.id); }} title="Delete note">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                          <line x1="18" y1="6" x2="6" y2="18"/>
                          <line x1="6" y1="6" x2="18" y2="18"/>
                        </svg>
                      </button>
                    </div>
                  </div>
                {:else}
                  <div class="tree-empty">No notes</div>
                {/each}
              </div>
            {/if}
          </div>
        {/each}
      </nav>
    </div>
  </aside>

  <!-- Editor -->
  <main class="editor">
    {#if selectedNote}
      <div class="editor-header">
        <input
          type="text"
          class="editor-title"
          placeholder="Untitled"
          value={selectedNote.title}
          oninput={(e) => {
            notes = notes.map(n => 
              n.id === selectedNoteId 
                ? { ...n, title: e.currentTarget.value, updatedAt: Date.now() } 
                : n
            );
            saveNotes();
          }}
        />
        <div class="editor-meta">
          <span>{getWordCount(selectedNote.content)} words</span>
          <span>•</span>
          <span>{formatDate(selectedNote.updatedAt)}</span>
        </div>
      </div>
      <textarea
        class="editor-content"
        placeholder="Start writing..."
        value={selectedNote.content}
        oninput={(e) => updateNoteContent(e.currentTarget.value)}
      ></textarea>
    {:else}
      <div class="editor-empty">
        <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="0.5">
          <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
          <polyline points="14 2 14 8 20 8"/>
        </svg>
        <div class="empty-title">Select a note</div>
        <div class="empty-text">Or create a new one from the sidebar</div>
      </div>
    {/if}
  </main>
</div>

<style>
  .notes-app {
    display: grid;
    grid-template-columns: 260px 1fr;
    height: 100%;
    background: var(--bg-base);
    transition: grid-template-columns 0.2s cubic-bezier(0.16, 1, 0.3, 1);
  }

  .notes-app.sidebar-hidden {
    grid-template-columns: 0px 1fr;
  }

  /* Sidebar */
  .sidebar {
    background: var(--bg-surface);
    border-right: 1px solid var(--border-subtle);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    min-width: 0;
    /* Removed fixed width from here, let grid handle it */
  }

  .sidebar-inner {
    width: 260px; /* Fixed width for content */
    display: flex;
    flex-direction: column;
    height: 100%;
  }

  .sidebar-header {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    padding: var(--space-3) var(--space-3);
    border-bottom: 1px solid var(--border-subtle);
  }

  .add-folder-btn {
    width: 28px;
    height: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-muted);
    border-radius: var(--radius-sm);
    transition: all var(--transition);
  }

  .add-folder-btn:hover {
    background: var(--bg-hover);
    color: var(--text-primary);
  }

  .sidebar-title {
    flex: 1;
    font-size: 12px;
    font-weight: 600;
    color: var(--text-secondary);
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  /* Tree */
  .tree {
    flex: 1;
    overflow-y: auto;
    padding: var(--space-2);
  }

  .tree-folder {
    margin-bottom: var(--space-1);
  }

  .tree-item {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    padding: var(--space-2) var(--space-2);
    border-radius: var(--radius-md);
    cursor: pointer;
    transition: background var(--transition);
    position: relative;
  }

  .tree-item:hover {
    background: var(--bg-hover);
  }

  .folder-row {
    font-weight: 500;
  }

  .note-row {
    padding-left: 28px;
  }

  .note-row.active {
    background: var(--accent-subtle);
  }

  .note-row.active .note-icon {
    color: var(--accent);
  }

  .chevron {
    width: 16px;
    height: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-muted);
    flex-shrink: 0;
  }

  .folder-icon {
    color: var(--text-muted);
    flex-shrink: 0;
  }

  .note-icon {
    color: var(--text-muted);
    flex-shrink: 0;
  }

  .item-name {
    flex: 1;
    font-size: 13px;
    color: var(--text-primary);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .inline-edit {
    flex: 1;
    padding: 2px 6px;
    background: var(--bg-base);
    border: 1px solid var(--accent);
    border-radius: var(--radius-sm);
    font-size: 13px;
    color: var(--text-primary);
  }

  .item-actions {
    display: flex;
    gap: 2px;
    opacity: 0;
    transition: opacity var(--transition);
  }

  .tree-item:hover .item-actions {
    opacity: 1;
  }

  .action-btn {
    width: 22px;
    height: 22px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-muted);
    border-radius: var(--radius-sm);
    transition: all var(--transition);
  }

  .action-btn:hover {
    background: var(--bg-active);
    color: var(--text-primary);
  }

  .action-btn.delete:hover {
    background: rgba(248, 81, 73, 0.15);
    color: var(--error);
  }

  .tree-children {
    margin-top: 2px;
  }

  .tree-empty {
    padding: var(--space-2) var(--space-2);
    padding-left: 44px;
    font-size: 12px;
    color: var(--text-faint);
    font-style: italic;
  }

  /* Editor */
  .editor {
    background: var(--bg-base);
    display: flex;
    flex-direction: column;
  }

  .editor-header {
    padding: var(--space-5) var(--space-6);
    border-bottom: 1px solid var(--border-subtle);
  }

  .editor-title {
    width: 100%;
    font-size: 24px;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: var(--space-2);
    background: transparent;
    border: none;
  }

  .editor-title:focus {
    outline: none;
  }

  .editor-title::placeholder {
    color: var(--text-faint);
  }

  .editor-meta {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-size: 12px;
    color: var(--text-muted);
  }

  .editor-content {
    flex: 1;
    padding: var(--space-5) var(--space-6);
    font-size: 15px;
    line-height: 1.8;
    color: var(--text-primary);
    resize: none;
    background: transparent;
    border: none;
  }

  .editor-content:focus {
    outline: none;
  }

  .editor-content::placeholder {
    color: var(--text-faint);
  }

  .editor-empty {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: var(--space-3);
    color: var(--text-faint);
  }

  .empty-title {
    font-size: 16px;
    font-weight: 500;
    color: var(--text-secondary);
  }

  .empty-text {
    font-size: 13px;
    color: var(--text-muted);
  }
</style>
