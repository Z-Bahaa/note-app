<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { slide } from 'svelte/transition';
  import { notesStore } from '$lib/stores/notes';
  import { get } from 'svelte/store';

  export let isOpen = true;

  const dispatch = createEventDispatcher<{
    close: void;
  }>();

  let editingTag: string | null = null;
  let editedTagName = '';
  let selectedTag: string | null = null;
  let showNewTagInput = false;
  let newTagName = '';

  $: tags = $notesStore.tags;
  $: activeTags = $notesStore.activeTags;

  function selectAllNotes() {
    notesStore.setActiveTags([]);
    selectedTag = null;
  }

  function selectTag(tag: string) {
    notesStore.setActiveTags([tag]);
    selectedTag = tag;
  }

  function startEditTag(tag: string) {
    editingTag = tag;
    editedTagName = tag;
  }

  function saveEditTag(tag: string) {
    const newTag = editedTagName.trim();
    if (newTag && newTag !== tag) {
      notesStore.update(state => {
        // Update tag in tags array
        const tags = state.tags.map(t => (t === tag ? newTag : t));
        // Update tag in all notes
        const notes = state.notes.map(note => ({
          ...note,
          tags: note.tags.map(t => (t === tag ? newTag : t))
        }));
        return { ...state, tags, notes };
      });
    }
    editingTag = null;
    editedTagName = '';
  }

  function cancelEditTag() {
    editingTag = null;
    editedTagName = '';
  }

  function deleteTag(tag: string) {
    notesStore.removeTag(tag);
    if (selectedTag === tag) {
      selectAllNotes();
    }
  }

  function startNewTag() {
    showNewTagInput = true;
    newTagName = '';
  }

  function saveNewTag() {
    const tag = newTagName.trim();
    if (tag && !$notesStore.tags.includes(tag)) {
      notesStore.addTag(tag);
    }
    showNewTagInput = false;
    newTagName = '';
  }

  function cancelNewTag() {
    showNewTagInput = false;
    newTagName = '';
  }
</script>

<div 
  class="sidebar" 
  class:open={isOpen}
  transition:slide={{ duration: 200 }}
>
  <div class="sidebar-header">
    <h1 class="app-title">Linen</h1>
    <button 
      class="collapse-button"
      on:click={() => dispatch('close')}
      title={isOpen ? 'Collapse sidebar' : 'Expand sidebar'}
    >
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M10 12L6 8L10 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </button>
  </div>

  <div class="sidebar-divider" />

  {#if isOpen}
    <nav class="sidebar-tags">
      <button class="tag-list-item all-notes {activeTags.length === 0 ? 'active' : ''}" on:click={selectAllNotes}>
        All Notes
      </button>
      {#each tags as tag}
        <div class="tag-list-item {activeTags[0] === tag ? 'active' : ''}">
          {#if editingTag === tag}
            <input
              class="edit-tag-input"
              bind:value={editedTagName}
              on:keydown={(e) => {
                if (e.key === 'Enter') saveEditTag(tag);
                if (e.key === 'Escape') cancelEditTag();
              }}
              on:blur={() => saveEditTag(tag)}
              autofocus
            />
            <span class="tag-actions">
              <button class="save-tag-btn" title="Save tag" on:click={() => saveEditTag(tag)}>
                <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M4 8L7 11L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </button>
              <button class="close-tag-btn" title="Cancel" on:mousedown|preventDefault={cancelEditTag}>
                <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M4 4L12 12M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </button>
            </span>
          {:else}
            <span class="tag-name" on:click={() => selectTag(tag)}>{tag}</span>
            <span class="tag-actions">
              <button class="edit-tag-btn" title="Edit tag" on:click={() => startEditTag(tag)}>
                ✎
              </button>
              <button class="delete-tag-btn" title="Delete tag" on:click={() => deleteTag(tag)}>
                🗑
              </button>
            </span>
          {/if}
        </div>
      {/each}
      {#if showNewTagInput}
        <div class="tag-list-item new-tag-item">
          <input
            class="edit-tag-input"
            bind:value={newTagName}
            on:keydown={(e) => {
              if (e.key === 'Enter') saveNewTag();
              if (e.key === 'Escape') cancelNewTag();
            }}
            on:blur={saveNewTag}
            placeholder="New tag name..."
            autofocus
          />
          <span class="tag-actions">
            <button class="save-tag-btn" title="Save tag" on:click={saveNewTag}>
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M4 8L7 11L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
            <button class="close-tag-btn" title="Cancel" on:mousedown|preventDefault={cancelNewTag}>
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M4 4L12 12M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </span>
        </div>
      {:else}
        <div class="tag-list-item new-tag-item" on:click={startNewTag}>
          <span class="tag-name">New Tag</span>
          <span class="tag-actions">
            <button class="add-tag-btn" title="Add tag">
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M8 3.5V12.5M3.5 8H12.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </span>
        </div>
      {/if}
    </nav>
  {/if}
</div>

<style>
  .sidebar {
    width: 224px;
    height: 100vh;
    background-color: var(--color-bg-elevated);
    display: flex;
    flex-direction: column;
    transition: width 0.2s ease;
  }

  .sidebar:not(.open) {
    width: 60px;
  }

  .sidebar-header {
    padding: var(--spacing-md);
    display: flex;
    justify-content: space-between;
    align-items: center;
    min-height: 60px;
    height: 60px;
    box-sizing: border-box;
  }

  .app-title {
    font-family: var(--font-mono);
    font-size: 1.5rem;
    font-weight: 600;
    color: var(--color-text-primary);
    margin: 0;
    margin-right: 20px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .sidebar:not(.open) .app-title {
    display: none;
  }

  .collapse-button {
    background: none;
    border: none;
    color: var(--color-text-muted);
    cursor: pointer;
    padding: var(--spacing-xs);
    border-radius: var(--radius-sm);
    transition: all var(--transition-fast);
    flex-shrink: 0;
  }

  .collapse-button:hover {
    color: var(--color-text-primary);
    background-color: var(--color-bg-tertiary);
  }

  .sidebar:not(.open) .collapse-button {
    transform: rotate(180deg);
    margin: 0 auto;
  }

  .sidebar-divider {
    height: 1px;
    background-color: var(--color-border);
    margin: 0;
  }

  .sidebar-tags {
    display: flex;
    flex-direction: column;
    gap: 2px;
    padding: var(--spacing-md) 0;
  }

  .tag-list-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 6px 20px;
    font-family: var(--font-mono);
    font-size: 1rem;
    color: var(--color-text-secondary);
    background: none;
    border: none;
    border-radius: var(--radius-md);
    cursor: pointer;
    transition: background 0.15s, color 0.15s;
    gap: 0;
  }
  .tag-list-item.active, .tag-list-item:hover {
    background-color: var(--color-bg-tertiary);
    color: var(--color-text-primary);
  }
  .all-notes {
    font-weight: 600;
    color: var(--color-accent-blue);
    margin-bottom: 8px;
  }
  .all-notes.active {
    background-color: var(--color-bg-tertiary);
    color: var(--color-accent-blue);
  }
  .edit-tag-btn, .delete-tag-btn {
    background: none;
    border: none;
    color: var(--color-text-muted);
    cursor: pointer;
    font-size: 1rem;
    padding: 2px 4px;
    border-radius: var(--radius-sm);
    transition: background 0.15s, color 0.15s;
  }
  .edit-tag-btn:hover {
    color: var(--color-accent-blue);
    background: var(--color-bg-tertiary);
  }
  .delete-tag-btn:hover {
    color: var(--color-accent-red);
    background: var(--color-bg-tertiary);
  }
  .edit-tag-input {
    flex: 1;
    font-family: var(--font-mono);
    font-size: 1rem;
    color: var(--color-text-secondary);
    background: var(--color-bg-tertiary);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-sm);
    padding: 2px 6px;
    width: 100%;
    max-width: 100%;
    box-shadow: none;
    outline: none;
    transition: border-color 0.15s;
  }
  .edit-tag-input:focus {
    border-color: var(--color-border-light);
    background: var(--color-bg-tertiary);
    color: var(--color-text-primary);
  }
  .tag-actions {
    display: flex;
    gap: 4px;
    margin-left: 8px;
  }
  .tag-list-item.new-tag-item {
    color: var(--color-accent-blue);
    font-style: italic;
    cursor: pointer;
  }
  .add-tag-btn {
    background: none;
    border: none;
    color: var(--color-accent-blue);
    cursor: pointer;
    font-size: 1rem;
    padding: 2px 4px;
    border-radius: var(--radius-sm);
    transition: background 0.15s, color 0.15s;
  }
  .add-tag-btn:hover {
    background: var(--color-bg-tertiary);
    color: var(--color-accent-blue);
  }
  .close-tag-btn, .save-tag-btn {
    background: none;
    border: none;
    color: var(--color-accent-blue);
    cursor: pointer;
    font-size: 1rem;
    padding: 2px 4px;
    border-radius: var(--radius-sm);
    transition: background 0.15s, color 0.15s;
  }
  .close-tag-btn:hover {
    color: var(--color-accent-red);
    background: var(--color-bg-tertiary);
  }
  .save-tag-btn:hover {
    color: var(--color-accent-green);
    background: var(--color-bg-tertiary);
  }
</style> 