<script lang="ts">
  import type { ReactiveSpace, ReactiveCollection } from "@rool-dev/svelte";

  interface Props {
    space: ReactiveSpace;
    onNavigateDefine: () => void;
  }

  interface Definition {
    id: string;
    term?: string;
    grammar?: string;
    definition?: string;
    article?: string;
    upvotes?: number;
    downvotes?: number;
    createdAt?: string;
  }

  let { space, onNavigateDefine }: Props = $props();

  let searchQuery = $state("");
  let activeSearchQuery = $state("");
  let isSearching = $state(false);
  let hasSearched = $state(false);

  // Track user's own votes in localStorage: { [definitionId]: 'up' | 'down' | null }
  let userVotes = $state<Record<string, "up" | "down" | null>>({});

  $effect(() => {
    const saved = localStorage.getItem("rool_user_votes");
    if (saved) {
      try {
        userVotes = JSON.parse(saved);
      } catch (e) {
        console.error("Failed to parse user votes", e);
      }
    }
  });

  $effect(() => {
    localStorage.setItem("rool_user_votes", JSON.stringify(userVotes));
  });

  // Reactive collection for all definitions
  const definitions = space.collection({ where: { type: "definition" } });
  let words = $derived(definitions.objects as Definition[]);

  // Derived search results from the reactive collection
  let searchResults = $derived(
    activeSearchQuery.trim()
      ? words.filter((w) =>
          w.term
            ?.toLowerCase()
            .includes(activeSearchQuery.trim().toLowerCase()),
        )
      : [],
  );

  // Randomize word properties for animation
  const wordSeeds = $derived(
    words.map((w, i) => ({
      term: w.term,
      x: `${((i * 137) % 90) + 5}%`,
      size: `${0.8 + (i % 5) * 0.2}rem`,
      duration: `${10 + (i % 8)}s`,
      delay: `-${(i * 2.5) % 15}s`,
      drift: `${5 + (i % 4)}s`,
      opacity: 0.15 + (i % 3) * 0.05,
    })),
  );

  async function handleSearch() {
    activeSearchQuery = searchQuery.trim();
    if (!activeSearchQuery) {
      hasSearched = false;
      return;
    }

    isSearching = true;
    hasSearched = true;

    // We don't need to fetch manually anymore as it's reactive,
    // but we add a small delay to simulate search feel if desired,
    // or just end the loading state immediately.
    setTimeout(() => {
      isSearching = false;
    }, 300);
  }

  async function handleVote(definitionId: string, type: "up" | "down") {
    try {
      const def = words.find((r) => r.id === definitionId);
      if (!def) return;

      const currentVote = userVotes[definitionId];
      let upDelta = 0;
      let downDelta = 0;

      if (currentVote === type) {
        // Toggle off (undo)
        if (type === "up") upDelta = -1;
        else downDelta = -1;
        userVotes[definitionId] = null;
      } else {
        // Switching or first time
        if (currentVote === "up") upDelta = -1;
        if (currentVote === "down") downDelta = -1;

        if (type === "up") upDelta += 1;
        if (type === "down") downDelta += 1;

        userVotes[definitionId] = type;
      }

      const updates: any = {};
      if (upDelta !== 0)
        updates.upvotes = Math.max(0, (def.upvotes || 0) + upDelta);
      if (downDelta !== 0)
        updates.downvotes = Math.max(0, (def.downvotes || 0) + downDelta);

      if (Object.keys(updates).length > 0) {
        await space.updateObject(definitionId, {
          data: updates,
        });
      }
    } catch (e) {
      console.error("Vote failed:", e);
    }
  }

  function handleKeydown(e: KeyboardEvent) {
    if (e.key === "Enter") {
      e.preventDefault();
      handleSearch();
    }
  }
</script>

<div class="home-page">
  <!-- Floating Words Background (only when not searching/showing results) -->
  {#if !hasSearched && !isSearching}
    <div class="words-background">
      {#each wordSeeds as word}
        <div
          class="word-particle"
          style="
            --word-x: {word.x};
            --word-size: {word.size};
            --word-duration: {word.duration};
            --word-delay: {word.delay};
            --drift-duration: {word.drift};
            --word-opacity: {word.opacity};
          "
        >
          <span>{word.term}</span>
        </div>
      {/each}
    </div>
  {/if}

  <!-- Hero Section -->
  <div class="home-hero">
    <h1 class="home-title">
      <span class="gradient-text">Rool Urban Dictionary</span>
    </h1>
    <p class="home-subtitle">
      The community-powered dictionary for all things rool 🍊
    </p>
  </div>

  <!-- Search Section -->
  <div class="search-section">
    <div class="search-container">
      <div class="search-input-wrapper">
        <svg
          class="search-icon"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
        >
          <circle cx="11" cy="11" r="8" />
          <path d="m21 21-4.35-4.35" />
        </svg>
        <input
          type="text"
          class="search-input"
          placeholder="Search for a rool word..."
          bind:value={searchQuery}
          onkeydown={handleKeydown}
          id="search-input"
        />
        {#if searchQuery}
          <button
            class="search-clear"
            onclick={() => {
              searchQuery = "";
              activeSearchQuery = "";
              hasSearched = false;
            }}
          >
            ✕
          </button>
        {/if}
      </div>
      <button
        class="btn-primary search-btn"
        onclick={handleSearch}
        disabled={isSearching || !searchQuery.trim()}
        id="search-button"
      >
        {isSearching ? "🔍 Searching..." : "🔍 Search"}
      </button>
    </div>

    <button
      class="define-btn"
      onclick={onNavigateDefine}
      id="define-word-button"
    >
      <span class="define-icon">✏️</span>
      <span>Define Word</span>
    </button>
  </div>

  <!-- Results Section -->
  <div class="results-section">
    {#if isSearching}
      <div class="loading-state">
        <div class="loading-spinner"></div>
        <p>Searching the rool dictionary...</p>
      </div>
    {:else if hasSearched && searchResults.length === 0}
      <div class="empty-state">
        <span class="empty-emoji">🤔</span>
        <h3>No rool words found</h3>
        <p>Be the first to define this word!</p>
        <button class="btn-secondary" onclick={onNavigateDefine}>
          Define It →
        </button>
      </div>
    {:else if searchResults.length > 0}
      <div class="results-grid">
        {#each searchResults as result (result.id)}
          <div class="definition-card">
            <div class="card-header">
              <h3 class="card-term">{result.term}</h3>
              <span class="card-grammar">{result.grammar}</span>
            </div>
            {#if result.article}
              <p class="card-article">{result.article}</p>
            {/if}
            <p class="card-definition">{result.definition}</p>
            <div class="card-footer">
              <div class="vote-buttons">
                <button
                  class="vote-btn upvote"
                  class:active={userVotes[result.id] === "up"}
                  onclick={() => handleVote(result.id, "up")}
                  title="Upvote"
                >
                  <span class="vote-icon">👍</span>
                  <span class="vote-count">{result.upvotes || 0}</span>
                </button>
                <button
                  class="vote-btn downvote"
                  class:active={userVotes[result.id] === "down"}
                  onclick={() => handleVote(result.id, "down")}
                  title="Downvote"
                >
                  <span class="vote-icon">👎</span>
                  <span class="vote-count">{result.downvotes || 0}</span>
                </button>
              </div>
              <span class="card-date"
                >{new Date(result.createdAt || "").toLocaleDateString()}</span
              >
            </div>
          </div>
        {/each}
      </div>
    {:else}
      <div class="welcome-state">
        <div class="welcome-emojis">
          <span>🍊</span><span>🍌</span><span>🍋</span><span>🍇</span><span
            >🍎</span
          >
        </div>
        <p>Search for a rool word or define your own!</p>
      </div>
    {/if}
  </div>
</div>

<style>
  .home-page {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    background: linear-gradient(180deg, #fff7ed 0%, #ffffff 30%, #ffffff 100%);
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0 16px;
    position: relative;
  }

  /* Floating Words Background */
  .words-background {
    position: absolute;
    inset: 0;
    overflow: hidden;
    pointer-events: none;
    z-index: 0;
  }

  .word-particle {
    position: absolute;
    pointer-events: none;
    user-select: none;
    animation: float-word var(--word-duration, 12s) linear var(--word-delay, 0s)
      infinite;
    font-size: var(--word-size, 1rem);
    left: var(--word-x, 50%);
    opacity: var(--word-opacity, 0.2);
    font-family: var(--font-display);
    font-weight: 800;
    color: var(--orange-500);
    white-space: nowrap;
  }

  .word-particle span {
    display: inline-block;
    animation: float-drift var(--drift-duration, 6s) ease-in-out infinite;
  }

  @keyframes float-word {
    0% {
      transform: translateY(110vh) rotate(-5deg);
      opacity: 0;
    }
    10% {
      opacity: var(--word-opacity, 0.2);
    }
    90% {
      opacity: var(--word-opacity, 0.2);
    }
    100% {
      transform: translateY(-20vh) rotate(5deg);
      opacity: 0;
    }
  }

  .home-hero {
    position: relative;
    z-index: 1;
    text-align: center;
    padding-top: 48px;
    padding-bottom: 8px;
    animation: slide-up 0.6s ease;
  }

  .home-title {
    font-family: var(--font-display);
    font-size: clamp(2rem, 6vw, 3.2rem);
    font-weight: 900;
    margin-bottom: 8px;
  }

  .home-subtitle {
    font-family: var(--font-body);
    font-size: 1.05rem;
    color: #6b7280;
    font-weight: 500;
  }

  /* Search */
  .search-section {
    position: relative;
    z-index: 1;
    width: 100%;
    max-width: 620px;
    margin-top: 32px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
    animation: slide-up 0.6s ease 0.1s both;
  }

  .search-container {
    display: flex;
    gap: 12px;
    width: 100%;
  }

  .search-input-wrapper {
    flex: 1;
    position: relative;
    display: flex;
    align-items: center;
  }

  .search-icon {
    position: absolute;
    left: 16px;
    width: 20px;
    height: 20px;
    color: #9ca3af;
    pointer-events: none;
  }

  .search-input {
    width: 100%;
    padding: 16px 44px 16px 48px;
    border: 2px solid var(--orange-200);
    border-radius: 18px;
    font-family: var(--font-body);
    font-size: 1.05rem;
    color: #1a1a1a;
    outline: none;
    transition: all 0.3s ease;
    box-shadow: 0 2px 12px rgba(249, 115, 22, 0.06);
    background: white;
  }

  .search-input::placeholder {
    color: #b5b5b5;
  }

  .search-input:focus {
    border-color: var(--orange-400);
    box-shadow:
      0 0 0 4px rgba(249, 115, 22, 0.1),
      0 4px 20px rgba(249, 115, 22, 0.08);
  }

  .search-clear {
    position: absolute;
    right: 14px;
    background: none;
    border: none;
    font-size: 0.9rem;
    color: #9ca3af;
    cursor: pointer;
    padding: 4px;
    transition: color 0.2s;
  }

  .search-clear:hover {
    color: #4b5563;
  }

  .search-btn {
    border-radius: 18px;
    padding: 16px 28px;
    white-space: nowrap;
    font-size: 1rem;
  }

  .define-btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 12px 32px;
    background: white;
    border: 2px dashed var(--orange-300);
    border-radius: 100px;
    font-family: var(--font-display);
    font-weight: 700;
    font-size: 1rem;
    color: var(--orange-600);
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .define-btn:hover {
    background: var(--orange-50);
    border-color: var(--orange-400);
    transform: translateY(-2px);
    box-shadow: 0 4px 16px rgba(249, 115, 22, 0.12);
  }

  .define-icon {
    font-size: 1.2rem;
  }

  /* Results */
  .results-section {
    position: relative;
    z-index: 1;
    width: 100%;
    max-width: 620px;
    margin-top: 32px;
    padding-bottom: 48px;
    animation: fade-in 0.4s ease 0.2s both;
  }

  .results-grid {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .card-header {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 8px;
  }

  .card-term {
    font-family: var(--font-display);
    font-size: 1.5rem;
    font-weight: 800;
    color: var(--orange-700);
  }

  .card-grammar {
    background: var(--orange-100);
    color: var(--orange-700);
    padding: 4px 12px;
    border-radius: 100px;
    font-size: 0.8rem;
    font-weight: 600;
    font-family: var(--font-display);
    text-transform: lowercase;
    font-style: italic;
  }

  .card-article {
    color: #6b7280;
    font-style: italic;
    font-size: 0.95rem;
    margin-bottom: 8px;
  }

  .card-definition {
    color: #374151;
    font-size: 1rem;
    line-height: 1.6;
    white-space: pre-wrap;
  }

  .card-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
    padding-top: 16px;
    border-top: 1px solid var(--orange-100);
  }

  .vote-buttons {
    display: flex;
    gap: 8px;
  }

  .vote-btn {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 6px 12px;
    border-radius: 100px;
    border: 1px solid var(--orange-200);
    background: white;
    cursor: pointer;
    transition: all 0.2s ease;
    color: var(--orange-700);
    font-weight: 600;
    font-size: 0.9rem;
  }

  .vote-btn:hover {
    background: var(--orange-50);
    border-color: var(--orange-400);
    transform: translateY(-1px);
  }

  .vote-btn.active {
    background: var(--orange-500);
    border-color: var(--orange-600);
    color: white;
  }

  .vote-btn.active:hover {
    background: var(--orange-600);
  }

  .vote-btn:active {
    transform: translateY(0);
  }

  .vote-icon {
    font-size: 1.1rem;
  }

  .vote-count {
    min-width: 12px;
  }

  .card-date {
    font-size: 0.8rem;
    color: #9ca3af;
    font-weight: 500;
  }

  /* States */
  .loading-state {
    text-align: center;
    padding: 48px 0;
    color: #9ca3af;
  }

  .loading-spinner {
    width: 40px;
    height: 40px;
    border: 3px solid var(--orange-100);
    border-top-color: var(--orange-500);
    border-radius: 50%;
    animation: spin-slow 0.8s linear infinite;
    margin: 0 auto 16px;
  }

  .empty-state {
    text-align: center;
    padding: 48px 0;
  }

  .empty-emoji {
    font-size: 3rem;
    display: block;
    margin-bottom: 12px;
  }

  .empty-state h3 {
    font-family: var(--font-display);
    font-size: 1.3rem;
    font-weight: 700;
    color: #374151;
    margin-bottom: 8px;
  }

  .empty-state p {
    color: #6b7280;
    margin-bottom: 20px;
  }

  .welcome-state {
    text-align: center;
    padding: 48px 0;
    color: #9ca3af;
  }

  .welcome-emojis {
    font-size: 2rem;
    display: flex;
    justify-content: center;
    gap: 12px;
    margin-bottom: 16px;
  }

  .welcome-emojis span {
    animation: wiggle 2s ease-in-out infinite;
  }

  .welcome-emojis span:nth-child(2) {
    animation-delay: 0.2s;
  }
  .welcome-emojis span:nth-child(3) {
    animation-delay: 0.4s;
  }
  .welcome-emojis span:nth-child(4) {
    animation-delay: 0.6s;
  }
  .welcome-emojis span:nth-child(5) {
    animation-delay: 0.8s;
  }
</style>
