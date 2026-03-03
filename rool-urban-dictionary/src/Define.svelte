<script lang="ts">
  import type { ReactiveSpace } from "@rool-dev/svelte";

  interface Props {
    space: ReactiveSpace;
    onBack: () => void;
  }

  let { space, onBack }: Props = $props();

  let term = $state("");
  let grammar = $state("");
  let definition = $state("");
  let articlePreview = $state("");
  let isLoadingArticle = $state(false);
  let isSubmitting = $state(false);
  let errorMessage = $state("");
  let successMessage = $state("");

  const grammarOptions = [
    "noun",
    "verb",
    "adjective",
    "adverb",
    "pronoun",
    "preposition",
    "conjunction",
    "interjection",
    "determiner",
    "exclamation",
  ];

  // Generate article preview when term and grammar are set
  async function generateArticlePreview() {
    if (!term.trim() || !grammar) return;

    isLoadingArticle = true;
    articlePreview = "";

    try {
      const { message } = await space.prompt(
        `I need to know how to use the article (a, an, the, or no article) before the word "${term.trim()}" when used as a ${grammar}. Reply with ONLY the article (if any) followed by the word, nothing else. For example, if the word is "rooler" and it's a noun, reply "A rooler". If it's an adjective like "roolish", reply "roolish" with no article. Keep it simple — just the usage form.`,
        {
          readOnly: true,
          effort: "QUICK",
          ephemeral: true,
        },
      );
      articlePreview = message.trim();
    } catch (e) {
      console.error("Article preview failed:", e);
      articlePreview = term.trim();
    } finally {
      isLoadingArticle = false;
    }
  }

  // Watch for changes to term + grammar
  $effect(() => {
    if (term.trim() && grammar) {
      generateArticlePreview();
    } else {
      articlePreview = "";
    }
  });

  async function handleSubmit() {
    errorMessage = "";
    successMessage = "";

    if (!term.trim()) {
      errorMessage = "Yo, you gotta enter a word first! 😤";
      return;
    }

    if (!grammar) {
      errorMessage = "Pick a grammar type, fam! 📝";
      return;
    }

    if (!definition.trim()) {
      errorMessage = "Drop a definition, don't leave us hanging! ✍️";
      return;
    }

    // Validate "rool" is in the term
    if (!term.trim().toLowerCase().includes("rool")) {
      errorMessage =
        'Yo! The word has gotta contain "rool" — that\'s the whole vibe! 🤙 No rool, no entry. Keep it rool!';
      return;
    }

    isSubmitting = true;

    try {
      await space.createObject({
        data: {
          type: "definition",
          term: term.trim().toLowerCase(),
          grammar: grammar,
          article: articlePreview || term.trim(),
          definition: definition.trim(),
          upvotes: 0,
          downvotes: 0,
          createdAt: new Date().toISOString(),
        },
      });
      successMessage = `"${term.trim()}" has been added to the Rool Urban Dictionary! 🎉🍊`;
      // Reset form
      term = "";
      grammar = "";
      definition = "";
      articlePreview = "";
    } catch (e) {
      console.error("Submit failed:", e);
      errorMessage = "Something went wrong. Try again, rooler! 😬";
    } finally {
      isSubmitting = false;
    }
  }
</script>

<div class="define-page">
  <!-- Back Button -->
  <button class="back-btn" onclick={onBack} id="back-button">
    <span class="back-arrow">←</span>
    <span>Back to Dictionary</span>
  </button>

  <div class="define-container">
    <h1 class="define-title">
      <span class="gradient-text">Define a Rool Word</span>
    </h1>
    <p class="define-subtitle">
      Add your term to the dictionary. Make it rool. 🍊
    </p>

    <!-- Term Input Row -->
    <div class="input-group">
      <label class="input-label" for="term-input">Term</label>
      <div class="term-row">
        <input
          type="text"
          class="input-orange term-input"
          placeholder={"e.g. rooler, roolify..."}
          bind:value={term}
          id="term-input"
        />
        <select
          class="input-orange grammar-select"
          bind:value={grammar}
          id="grammar-select"
        >
          <option value="" disabled selected>Grammar</option>
          {#each grammarOptions as opt}
            <option value={opt}>{opt}</option>
          {/each}
        </select>
      </div>
    </div>

    <!-- Article Preview -->
    {#if isLoadingArticle}
      <div class="article-preview loading">
        <div class="article-dot-pulse"></div>
        <span>Generating usage preview...</span>
      </div>
    {:else if articlePreview}
      <div class="article-preview">
        <span class="article-label">Usage:</span>
        <span class="article-text">"{articlePreview}"</span>
      </div>
    {/if}

    <!-- Definition Input -->
    <div class="input-group">
      <label class="input-label" for="definition-input"
        >Definition & Examples</label
      >
      <textarea
        class="input-orange definition-textarea"
        placeholder={"Define the word and provide funny examples. e.g. A rooler is someone who rools hard. Example: That teacher is a total rooler!"}
        bind:value={definition}
        rows="5"
        id="definition-input"
      ></textarea>
    </div>

    <!-- Messages -->
    {#if errorMessage}
      <div class="toast-error" id="error-toast">
        {errorMessage}
      </div>
    {/if}

    {#if successMessage}
      <div class="toast-success" id="success-toast">
        {successMessage}
      </div>
    {/if}

    <!-- Submit -->
    <button
      class="btn-primary submit-btn"
      onclick={handleSubmit}
      disabled={isSubmitting}
      id="submit-button"
    >
      {isSubmitting ? "🍊 Submitting..." : "🍊 Submit to Dictionary"}
    </button>
  </div>
</div>

<style>
  .define-page {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    background: #ffffff;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0 16px;
  }

  .back-btn {
    align-self: flex-start;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 12px 20px;
    margin-top: 16px;
    background: none;
    border: none;
    font-family: var(--font-display);
    font-weight: 600;
    font-size: 0.95rem;
    color: var(--orange-500);
    cursor: pointer;
    transition: all 0.2s ease;
    border-radius: 12px;
  }

  .back-btn:hover {
    background: var(--orange-50);
    color: var(--orange-600);
  }

  .back-arrow {
    font-size: 1.3rem;
    transition: transform 0.2s ease;
  }

  .back-btn:hover .back-arrow {
    transform: translateX(-4px);
  }

  .define-container {
    width: 100%;
    max-width: 580px;
    padding-top: 24px;
    padding-bottom: 48px;
    animation: slide-up 0.5s ease;
  }

  .define-title {
    font-family: var(--font-display);
    font-size: clamp(1.8rem, 5vw, 2.5rem);
    font-weight: 900;
    margin-bottom: 4px;
  }

  .define-subtitle {
    color: #6b7280;
    font-size: 1rem;
    margin-bottom: 36px;
  }

  .input-group {
    margin-bottom: 24px;
  }

  .input-label {
    display: block;
    font-family: var(--font-display);
    font-weight: 700;
    font-size: 0.9rem;
    color: var(--orange-700);
    margin-bottom: 8px;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .term-row {
    display: flex;
    gap: 12px;
  }

  .term-input {
    flex: 1;
    font-size: 1.1rem;
    font-weight: 600;
  }

  .grammar-select {
    width: 160px;
    cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%23f97316' d='M6 8L1 3h10z'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 14px center;
    padding-right: 36px;
  }

  /* Article preview */
  .article-preview {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px 18px;
    background: var(--orange-50);
    border: 1px solid var(--orange-200);
    border-radius: 12px;
    margin-bottom: 24px;
    font-size: 0.95rem;
    animation: fade-in 0.3s ease;
  }

  .article-preview.loading {
    color: #9ca3af;
  }

  .article-dot-pulse {
    width: 8px;
    height: 8px;
    background: var(--orange-400);
    border-radius: 50%;
    animation: pulse-glow 1s ease-in-out infinite;
  }

  .article-label {
    font-weight: 600;
    color: var(--orange-600);
  }

  .article-text {
    font-style: italic;
    color: var(--orange-800);
    font-weight: 500;
  }

  /* Definition textarea */
  .definition-textarea {
    width: 100%;
    min-height: 140px;
    resize: vertical;
    line-height: 1.6;
  }

  /* Submit */
  .submit-btn {
    width: 100%;
    padding: 18px;
    font-size: 1.15rem;
    border-radius: 18px;
    margin-top: 8px;
  }

  /* Messages */
  .toast-error,
  .toast-success {
    margin-bottom: 16px;
    text-align: center;
    font-size: 0.95rem;
  }

  @media (max-width: 480px) {
    .term-row {
      flex-direction: column;
    }

    .grammar-select {
      width: 100%;
    }
  }
</style>
