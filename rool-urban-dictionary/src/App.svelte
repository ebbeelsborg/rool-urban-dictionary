<script lang="ts">
  import { createRool, type ReactiveSpace } from "@rool-dev/svelte";
  import Splash from "./Splash.svelte";
  import Header from "./Header.svelte";
  import Home from "./Home.svelte";
  import Define from "./Define.svelte";

  const APP_NAME = "Rool Urban Dictionary";
  const COMMUNITY_SPACE_ID = "19aKjg";

  const rool = createRool();
  rool.init();

  let space = $state<ReactiveSpace | null>(null);
  let currentPage = $state<"home" | "define">("home");
  let openingSpace = $state(false);
  let openError = $state("");

  $effect(() => {
    if (rool.authenticated === true && !space && !openingSpace) {
      openSpace();
    }
  });

  async function openSpace() {
    openingSpace = true;
    openError = "";
    try {
      space = await rool.openSpace(COMMUNITY_SPACE_ID, {
        conversationId: "main",
      });

      // Ensure the community space remains joinable by URL/ID.
      // Non-owner/non-admin users may not have permission, which is fine.
      try {
        if (space.linkAccess !== "editor") {
          await space.setLinkAccess("editor");
        }
      } catch (e) {
        console.debug("Could not set link access:", e);
      }
    } catch (e) {
      console.error("Could not open community space:", e);
      openError =
        "Could not open the shared dictionary space. Please refresh or sign in again.";
    } finally {
      openingSpace = false;
    }
  }
</script>

{#if rool.authenticated == null}
  <div class="loading-screen">
    <div class="loading-inner">
      <span class="loading-emoji">🍊</span>
      <p>Loading...</p>
    </div>
  </div>
{:else if rool.authenticated === false}
  <Splash appName={APP_NAME} onLogin={() => rool.login(APP_NAME)} />
{:else if rool.authenticated !== true}
  <div class="loading-screen">
    <div class="loading-inner">
      <span class="loading-emoji">🍊</span>
      <p>Loading...</p>
    </div>
  </div>
{:else}
  <div class="app-shell">
    <Header appName={APP_NAME} {space} onLogout={() => rool.logout()} />

    {#if openError}
      <div class="loading-screen">
        <div class="loading-inner">
          <span class="loading-emoji">⚠️</span>
          <p>{openError}</p>
          <button class="retry-btn" onclick={openSpace}>Retry</button>
        </div>
      </div>
    {:else if !space}
      <div class="loading-screen">
        <div class="loading-inner">
          <span class="loading-emoji">🍊</span>
          <p>Opening dictionary space...</p>
        </div>
      </div>
    {:else if currentPage === "define"}
      <Define {space} onBack={() => (currentPage = "home")} />
    {:else}
      <Home {space} onNavigateDefine={() => (currentPage = "define")} />
    {/if}
  </div>
{/if}

<style>
  .app-shell {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    background: #ffffff;
  }

  .loading-screen {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #fff7ed, #ffffff);
  }

  .loading-inner {
    text-align: center;
  }

  .loading-emoji {
    font-size: 3rem;
    display: block;
    margin-bottom: 12px;
    animation: wiggle 2s ease-in-out infinite;
  }

  .loading-inner p {
    font-family: var(--font-body);
    color: #9ca3af;
    font-weight: 500;
  }

  .retry-btn {
    margin-top: 12px;
    padding: 8px 14px;
    border-radius: 10px;
    border: 1px solid #fed7aa;
    background: #fff7ed;
    color: #c2410c;
    font-family: var(--font-body);
    font-weight: 600;
    cursor: pointer;
  }

  .retry-btn:hover {
    background: #ffedd5;
  }
</style>
