# Bloog Design System (Ghost Admin UI Extraction)

This document contains the foundational design tokens, layout architecture, and a core form component reverse-engineered from the Ghost Admin UI, specifically tailored for a minimal Svelte + Tailwind B2B dashboard.

---

### 1. Extracted Design System Tokens (Tailwind Config)

Ghost uses a very precise set of grays, an "Inter" font stack, and distinct accent colors. These exact CSS variable values have been mapped into a `tailwind.config.js` extension.

Ghost's signature primary accent color (their core green) is `#30cf43`. The exact border radius they use for inputs and most structural elements is `6px`.

**`tailwind.config.js`**
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{html,js,svelte,ts}'],
  theme: {
    extend: {
      colors: {
        // Ghost's primary light-theme accent color
        primary: {
          DEFAULT: '#30cf43', // var(--green)
          light: '#59d968',   // var(--green-l1) / l(+5%)
          dark: '#1eb830',    // var(--green-d1) / l(-5%)
        },
        // Ghost's foundational grayscale
        ghost: {
          black: '#15171A',
          darkgrey: '#394047',
          middarkgrey: '#626D79',
          midgrey: '#7C8B9A',
          midlightgrey: '#ABB4BE',
          lightgrey: '#CED4D9',
          whitegrey: '#EBEEF0',

          // Layout Backgrounds
          bg: '#f5f6f6',      // var(--main-color-content-greybg)
          border: '#EBEEF0',  // var(--main-color-area-divider)
        },
        // Base red for error states
        error: '#f50b23',     // var(--red)
      },
      fontFamily: {
        sans: [
          'Inter',
          '-apple-system',
          'BlinkMacSystemFont',
          '"Segoe UI"',
          'Roboto',
          'Oxygen',
          'Ubuntu',
          '"Droid Sans"',
          '"Helvetica Neue"',
          'sans-serif'
        ],
        mono: [
          'Consolas',
          '"Liberation Mono"',
          'Menlo',
          'Courier',
          'monospace'
        ]
      },
      borderRadius: {
        ghost: '6px', // var(--border-radius)
      },
      boxShadow: {
        // Ghost's signature shadow layers
        'ghost-1': '0 0 1px rgba(0,0,0,.14), 0 1px 6px rgba(0,0,0,0.05), 0 6px 10px -8px rgba(0,0,0,.14)',
        'ghost-2': '0 0 1px rgba(0,0,0,.05), 0 5px 18px rgba(0,0,0,.08)',
        'ghost-3': '0 0 1px rgba(0,0,0,.05), 0 8px 28px rgba(0,0,0,.12)',
        'ghost-m': '0 0 7px rgba(0, 0, 0, 0.08), 0 2.1px 2.2px -5px rgba(0, 0, 0, 0.011), 0 5.1px 5.3px -5px rgba(0, 0, 0, 0.016), 0 9.5px 10px -5px rgba(0, 0, 0, 0.02), 0 17px 17.9px -5px rgba(0, 0, 0, 0.024), 0 31.8px 33.4px -5px rgba(0, 0, 0, 0.029), 0 76px 80px -5px rgba(0, 0, 0, 0.04)',
      },
      spacing: {
        'ghost-side': '24px', // var(--main-layout-content-sidepadding)
      },
      maxWidth: {
        'ghost-content': '1200px', // var(--main-layout-content-maxwidth)
      }
    }
  },
  plugins: []
};
```

---

### 2. Layout Architecture Skeleton

Ghost achieves its spacious layout using a side nav (fixed around `280px` max-width on desktop) and a main content area (`.gh-canvas`) that is center-aligned with a max-width of `1200px` and `24px` horizontal padding.

**`+layout.svelte`**
```svelte
<script>
  // This layout mimics the flex/grid behavior of .gh-viewport and .gh-main
</script>

<!-- The outermost wrapper mimicking .gh-app (full viewport) -->
<div class="flex flex-col h-screen overflow-hidden font-sans text-ghost-darkgrey bg-ghost-bg text-[1.4rem]">

  <!-- Viewport mimicking .gh-viewport -->
  <div class="flex flex-grow overflow-hidden max-h-full">

    <!-- Sidebar (Ghost traditionally uses a fixed/max-width sidebar, roughly 280px) -->
    <nav class="flex flex-col w-full max-w-[280px] bg-white border-r border-ghost-border flex-shrink-0">
      <div class="p-4 font-bold text-ghost-black text-[1.5rem]">
        Bloog
      </div>
      <div class="p-4">
        <!-- Sidebar items -->
        <ul class="space-y-2">
          <li class="cursor-pointer font-semibold text-ghost-darkgrey hover:bg-ghost-whitegrey px-3 py-2 rounded">Dashboard</li>
          <li class="cursor-pointer font-semibold text-ghost-darkgrey hover:bg-ghost-whitegrey px-3 py-2 rounded">Posts</li>
        </ul>
      </div>
    </nav>

    <!-- Main Content Area mimicking .gh-main -->
    <main class="relative flex flex-col flex-grow overflow-y-auto bg-white">

      <!-- Ghost Canvas (.gh-canvas) - The centrally constrained column -->
      <div class="flex-grow w-full max-w-ghost-content mx-auto px-ghost-side pb-ghost-side">

        <!-- Ghost Header (.gh-canvas-header) -->
        <header class="sticky top-0 z-40 flex items-center justify-between py-ghost-side bg-white bg-opacity-95 backdrop-blur-sm -mx-ghost-side px-ghost-side mb-8">
          <h2 class="text-[2.8rem] leading-tight font-bold text-ghost-black tracking-tight">Dashboard</h2>
          <div class="flex items-center gap-2">
            <button class="bg-primary hover:bg-primary-dark text-white px-4 py-2 rounded-ghost font-medium transition-colors">
              Create New
            </button>
          </div>
        </header>

        <!-- Dynamic Content injected here -->
        <slot />

      </div>
    </main>

  </div>
</div>

<style>
  /* Base HTML setup to match Ghost's 62.5% trick (1rem = 10px) */
  :global(html) {
    font-size: 62.5%;
    line-height: 1.65;
    letter-spacing: 0.01em;
  }
</style>
```

---

### 3. Reusable Svelte Form Input Component

Ghost’s inputs have a height of `36px`, a light border (`#ebeeF0`), and a very distinctive focus ring that uses `oklab` color mixing to achieve a soft halo `2px` around the element.

**`TextInput.svelte`**
```svelte
<script>
  export let value = '';
  export let id = '';
  export let label = '';
  export let type = 'text';
  export let placeholder = '';
  export let error = false;
  export let errorMessage = '';
</script>

<div class="relative w-full max-w-[620px] mb-6 text-[1.3rem]">
  {#if label}
    <!-- Label mimicking form-group label -->
    <label
      for={id}
      class="block mb-1 font-medium {error ? 'text-error' : 'text-ghost-black'}"
    >
      {label}
    </label>
  {/if}

  <!--
    The Input:
    - h-[36px] matches Ghost's standard input height.
    - Default border: Ghost's whitegrey.
    - Focus state: changes border to midgrey and applies a 2px box-shadow halo mixed with transparent.
    - Error state: red border and red focus halo.
  -->
  <input
    {id}
    {type}
    bind:value
    {placeholder}
    class="
      block w-full h-[36px] px-3 py-1.5
      bg-white text-ghost-darkgrey
      border border-ghost-whitegrey rounded-ghost
      transition-shadow duration-150 outline-none
      placeholder:text-ghost-midlightgrey placeholder:font-normal
      {error
        ? 'border-error focus:border-error focus:ring-2 focus:ring-error/25'
        : 'focus:border-ghost-midgrey focus:ring-2 focus:ring-ghost-midgrey/25'
      }
    "
  />

  {#if error && errorMessage}
    <p class="mt-1 text-[1.2rem] text-error">{errorMessage}</p>
  {/if}
</div>
```