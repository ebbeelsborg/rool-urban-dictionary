<script lang="ts">
  import type { ReactiveChannel } from "@rool-dev/svelte";

  interface Props {
    appName: string;
    channel: ReactiveChannel | null;
    onLogout: () => void;
  }

  let { appName, channel, onLogout }: Props = $props();
</script>

<header class="app-header">
  <div class="header-left">
    <span class="header-logo">🍊</span>
    <h1 class="header-title">{appName}</h1>
  </div>
  <div class="header-right">
    {#if channel}
      <span class="header-badge">
        <span class="badge-dot"></span>
        Connected
      </span>
    {/if}
    <button class="header-logout" onclick={onLogout} id="logout-button">
      Sign out
    </button>
  </div>
</header>

<style>
  .app-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 20px;
    background: white;
    border-bottom: 1px solid #f3f4f6;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03);
    z-index: 50;
    flex-shrink: 0;
  }

  .header-left {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .header-logo {
    font-size: 1.4rem;
    animation: wiggle 3s ease-in-out infinite;
  }

  .header-title {
    font-family: var(--font-display);
    font-size: 1rem;
    font-weight: 800;
    background: linear-gradient(135deg, var(--orange-500), var(--orange-600));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .header-right {
    display: flex;
    align-items: center;
    gap: 16px;
  }

  .header-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 0.8rem;
    color: #6b7280;
    font-weight: 500;
  }

  .badge-dot {
    width: 7px;
    height: 7px;
    background: #22c55e;
    border-radius: 50%;
    box-shadow: 0 0 6px rgba(34, 197, 94, 0.4);
  }

  .header-logout {
    padding: 6px 16px;
    background: none;
    border: 1px solid #e5e7eb;
    border-radius: 10px;
    font-family: var(--font-body);
    font-size: 0.85rem;
    font-weight: 500;
    color: #6b7280;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .header-logout:hover {
    border-color: #d1d5db;
    background: #f9fafb;
    color: #374151;
  }
</style>
