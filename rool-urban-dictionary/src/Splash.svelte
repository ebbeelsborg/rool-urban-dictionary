<script lang="ts">
  interface Props {
    appName: string;
    onLogin: () => void;
  }

  let { appName, onLogin }: Props = $props();

  // Fruit configuration for floating particles
  const fruits = [
    { emoji: '🍊', x: '8%', size: '2.5rem', duration: '14s', delay: '0s', drift: '7s', scale: 1.1 },
    { emoji: '🍌', x: '20%', size: '2rem', duration: '11s', delay: '2s', drift: '5s', scale: 0.9 },
    { emoji: '🍋', x: '35%', size: '1.8rem', duration: '16s', delay: '1s', drift: '8s', scale: 1 },
    { emoji: '🍇', x: '50%', size: '2.2rem', duration: '13s', delay: '3s', drift: '6s', scale: 1.2 },
    { emoji: '🍎', x: '65%', size: '2rem', duration: '12s', delay: '0.5s', drift: '7.5s', scale: 1 },
    { emoji: '🍑', x: '78%', size: '2.3rem', duration: '15s', delay: '4s', drift: '5.5s', scale: 1.1 },
    { emoji: '🍊', x: '92%', size: '1.6rem', duration: '10s', delay: '1.5s', drift: '6.5s', scale: 0.8 },
    { emoji: '🍌', x: '5%', size: '1.5rem', duration: '17s', delay: '5s', drift: '9s', scale: 0.7 },
    { emoji: '🍋', x: '42%', size: '2rem', duration: '14s', delay: '6s', drift: '4s', scale: 1 },
    { emoji: '🍇', x: '88%', size: '1.8rem', duration: '11s', delay: '3.5s', drift: '7s', scale: 0.9 },
    { emoji: '🍎', x: '15%', size: '2.4rem', duration: '13s', delay: '7s', drift: '5s', scale: 1.3 },
    { emoji: '🍑', x: '55%', size: '1.7rem', duration: '16s', delay: '2.5s', drift: '8s', scale: 0.85 },
    { emoji: '🍊', x: '72%', size: '2.1rem', duration: '12s', delay: '8s', drift: '6s', scale: 1.05 },
    { emoji: '🍌', x: '30%', size: '1.9rem', duration: '18s', delay: '0s', drift: '7.5s', scale: 0.95 },
    { emoji: '🍋', x: '60%', size: '2.6rem', duration: '15s', delay: '4.5s', drift: '5.5s', scale: 1.15 },
  ];
</script>

<div class="splash-screen gradient-bg">
  <!-- Floating Fruit Particles -->
  <div class="fruits-container">
    {#each fruits as fruit, i}
      <div
        class="fruit-particle"
        style="
          --fruit-x: {fruit.x};
          --fruit-size: {fruit.size};
          --fruit-duration: {fruit.duration};
          --fruit-delay: {fruit.delay};
          --drift-duration: {fruit.drift};
          --fruit-scale: {fruit.scale};
        "
      >
        <span>{fruit.emoji}</span>
      </div>
    {/each}
  </div>

  <!-- Main Content -->
  <div class="splash-content">
    <div class="splash-badge">
      <span>🔥</span> Community Dictionary <span>🔥</span>
    </div>

    <h1 class="splash-title glow-title">
      {appName}
    </h1>

    <p class="splash-tagline">
      Where <span class="shimmer-text">rool</span> words come to life ✨
    </p>

    <button
      class="splash-enter-btn"
      onclick={onLogin}
      id="enter-button"
    >
      <span class="btn-text">Enter the Dictionary</span>
      <span class="btn-arrow">→</span>
    </button>

    <p class="splash-subtitle">
      Define. Discover. Get Rool. 🍊
    </p>
  </div>

  <!-- Decorative rings -->
  <div class="deco-ring ring-1"></div>
  <div class="deco-ring ring-2"></div>
  <div class="deco-ring ring-3"></div>
</div>

<style>
  .splash-screen {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
  }

  .fruits-container {
    position: absolute;
    inset: 0;
    overflow: hidden;
    z-index: 1;
  }

  .splash-content {
    position: relative;
    z-index: 10;
    text-align: center;
    max-width: 600px;
    padding: 0 24px;
    animation: bounce-in 0.8s cubic-bezier(0.68, -0.55, 0.27, 1.55);
  }

  .splash-badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 255, 255, 0.3);
    color: white;
    font-family: var(--font-display);
    font-weight: 600;
    font-size: 0.85rem;
    padding: 8px 20px;
    border-radius: 100px;
    margin-bottom: 24px;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    animation: slide-up 0.6s ease 0.2s both;
  }

  .splash-title {
    font-family: var(--font-display);
    font-size: clamp(2.5rem, 8vw, 5rem);
    font-weight: 900;
    color: white;
    line-height: 1.1;
    margin-bottom: 16px;
    letter-spacing: -0.02em;
  }

  .splash-tagline {
    font-family: var(--font-display);
    font-size: clamp(1.1rem, 3vw, 1.5rem);
    color: rgba(255, 255, 255, 0.9);
    font-weight: 500;
    margin-bottom: 40px;
    animation: slide-up 0.6s ease 0.4s both;
  }

  .splash-enter-btn {
    display: inline-flex;
    align-items: center;
    gap: 12px;
    background: white;
    color: var(--orange-600);
    border: none;
    padding: 18px 48px;
    border-radius: 100px;
    font-family: var(--font-display);
    font-weight: 800;
    font-size: 1.2rem;
    cursor: pointer;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15), 0 0 0 0 rgba(255, 255, 255, 0.4);
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    animation: slide-up 0.6s ease 0.6s both;
    position: relative;
    overflow: hidden;
  }

  .splash-enter-btn::after {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, transparent, rgba(249, 115, 22, 0.05), transparent);
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .splash-enter-btn:hover {
    transform: translateY(-4px) scale(1.05);
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2), 0 0 0 4px rgba(255, 255, 255, 0.3);
  }

  .splash-enter-btn:hover::after {
    opacity: 1;
  }

  .splash-enter-btn:hover .btn-arrow {
    transform: translateX(4px);
  }

  .splash-enter-btn:active {
    transform: translateY(-1px) scale(0.98);
  }

  .btn-arrow {
    transition: transform 0.3s ease;
    font-size: 1.4rem;
  }

  .splash-subtitle {
    margin-top: 28px;
    font-family: var(--font-body);
    font-size: 0.95rem;
    color: rgba(255, 255, 255, 0.7);
    font-weight: 500;
    animation: fade-in 1s ease 1s both;
  }

  /* Decorative rings */
  .deco-ring {
    position: absolute;
    border-radius: 50%;
    border: 2px solid rgba(255, 255, 255, 0.08);
    pointer-events: none;
  }

  .ring-1 {
    width: 600px;
    height: 600px;
    top: -200px;
    right: -200px;
    animation: spin-slow 30s linear infinite;
  }

  .ring-2 {
    width: 400px;
    height: 400px;
    bottom: -100px;
    left: -100px;
    animation: spin-slow 25s linear infinite reverse;
  }

  .ring-3 {
    width: 800px;
    height: 800px;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    animation: spin-slow 40s linear infinite;
    border-style: dashed;
    opacity: 0.5;
  }
</style>
