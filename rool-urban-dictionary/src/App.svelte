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

  $effect(() => {
    if (rool.authenticated && rool.spaces && !space) {
      openSpace();
    }
  });

  async function openSpace() {
    space = await rool.openSpace(COMMUNITY_SPACE_ID, {
      conversationId: "main",
    });
  }
</script>

{#if rool.authenticated === undefined}
  <div class="loading-screen">
    <div class="loading-inner">
      <span class="loading-emoji">🍊</span>
      <p>Loading...</p>
    </div>
  </div>
{:else if rool.authenticated === false}
  <Splash appName={APP_NAME} onLogin={() => rool.login(APP_NAME)} />
{:else}
  <div class="app-shell">
    <Header appName={APP_NAME} {space} onLogout={() => rool.logout()} />

    {#if !space}
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
</style>
