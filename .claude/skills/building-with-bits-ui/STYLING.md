# Styling Bits UI Components

Advanced styling techniques and patterns for Bits UI components.

## Data Attributes

Bits UI applies data attributes to all elements for targeted styling. Use these attributes to style components without adding custom classes.

### Common Data Attributes

```css
[data-state="open"]
[data-state="closed"]
[data-disabled]
[data-highlighted]
[data-selected]
[data-orientation="vertical"]
[data-orientation="horizontal"]
```

### Component-Specific Attributes

```css
[data-dialog-overlay]
[data-dialog-content]
[data-dialog-title]
[data-dialog-description]
[data-dialog-close]

[data-accordion-root]
[data-accordion-item]
[data-accordion-trigger]
[data-accordion-content]

[data-select-trigger]
[data-select-content]
[data-select-item]
[data-select-viewport]

[data-popover-trigger]
[data-popover-content]
[data-popover-close]
```

## Styling Approaches

### Scoped CSS with :global()

Use :global() to style Bits UI components in Svelte's scoped CSS:

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';
</script>

<Dialog.Root>
  <Dialog.Trigger>Open</Dialog.Trigger>
  <Dialog.Content>
    Content
  </Dialog.Content>
</Dialog.Root>

<style>
  :global([data-dialog-overlay]) {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(4px);
    z-index: 50;
    animation: fadeIn 150ms ease-out;
  }

  :global([data-dialog-content]) {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: white;
    padding: 2rem;
    border-radius: 0.5rem;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
    max-width: 32rem;
    width: 90%;
    z-index: 50;
    animation: slideIn 150ms ease-out;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  @keyframes slideIn {
    from {
      transform: translate(-50%, -48%);
      opacity: 0;
    }
    to {
      transform: translate(-50%, -50%);
      opacity: 1;
    }
  }

  :global([data-dialog-title]) {
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
    color: #111827;
  }

  :global([data-dialog-description]) {
    color: #6b7280;
    margin-bottom: 1.5rem;
    line-height: 1.5;
  }

  :global([data-dialog-close]) {
    position: absolute;
    top: 1rem;
    right: 1rem;
    padding: 0.5rem;
    border: none;
    background: transparent;
    cursor: pointer;
    border-radius: 0.25rem;
  }

  :global([data-dialog-close]:hover) {
    background: #f3f4f6;
  }
</style>
```

### Tailwind CSS Classes

Apply Tailwind classes directly to components:

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';
</script>

<Dialog.Root>
  <Dialog.Trigger class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
    Open Dialog
  </Dialog.Trigger>

  <Dialog.Portal>
    <Dialog.Overlay class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50" />

    <Dialog.Content class="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 bg-white rounded-lg shadow-xl max-w-md w-[90%] p-6 z-50">
      <Dialog.Title class="text-xl font-semibold mb-2 text-gray-900">
        Dialog Title
      </Dialog.Title>

      <Dialog.Description class="text-gray-600 mb-4">
        Dialog description and content goes here.
      </Dialog.Description>

      <div class="flex gap-3 justify-end">
        <Dialog.Close class="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors">
          Cancel
        </Dialog.Close>
        <button class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
          Confirm
        </button>
      </div>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

### CSS Variables for Theming

Use CSS variables for consistent theming:

```css
:root {
  --color-primary: #3b82f6;
  --color-primary-hover: #2563eb;
  --color-danger: #ef4444;
  --color-danger-hover: #dc2626;
  --color-text: #111827;
  --color-text-muted: #6b7280;
  --color-border: #e5e7eb;
  --color-background: #ffffff;
  --color-overlay: rgba(0, 0, 0, 0.5);
  --radius: 0.5rem;
  --shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

[data-theme="dark"] {
  --color-primary: #60a5fa;
  --color-primary-hover: #3b82f6;
  --color-danger: #f87171;
  --color-danger-hover: #ef4444;
  --color-text: #f9fafb;
  --color-text-muted: #d1d5db;
  --color-border: #374151;
  --color-background: #1f2937;
  --color-overlay: rgba(0, 0, 0, 0.8);
}

:global([data-dialog-content]) {
  background: var(--color-background);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
}

:global([data-dialog-title]) {
  color: var(--color-text);
}

:global([data-dialog-description]) {
  color: var(--color-text-muted);
}

:global([data-select-trigger]) {
  background: var(--color-background);
  border: 1px solid var(--color-border);
  color: var(--color-text);
}
```

### State-Based Styling

Style components based on their state:

```css
:global([data-select-item]) {
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: background-color 150ms;
}

:global([data-select-item][data-state="checked"]) {
  background: #dbeafe;
  color: #1e40af;
}

:global([data-select-item][data-highlighted]) {
  background: #f3f4f6;
  outline: none;
}

:global([data-select-item][data-disabled]) {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

:global([data-accordion-trigger]) {
  transition: transform 150ms;
}

:global([data-accordion-trigger][data-state="open"]) {
  transform: rotate(180deg);
}

:global([data-switch-root][data-state="checked"]) {
  background: #3b82f6;
}

:global([data-switch-root][data-state="unchecked"]) {
  background: #d1d5db;
}

:global([data-switch-thumb]) {
  transition: transform 150ms;
}

:global([data-switch-root][data-state="checked"] [data-switch-thumb]) {
  transform: translateX(1.25rem);
}
```

## Animations and Transitions

### CSS Animations

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideUp {
  from {
    transform: translateY(10px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes scaleIn {
  from {
    transform: scale(0.95);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

:global([data-dialog-overlay]) {
  animation: fadeIn 150ms ease-out;
}

:global([data-dialog-content]) {
  animation: scaleIn 150ms ease-out;
}

:global([data-popover-content]) {
  animation: slideUp 150ms ease-out;
}

:global([data-select-content]) {
  animation: slideUp 150ms ease-out;
}
```

### Svelte Transitions

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';
  import { fade, scale, fly } from 'svelte/transition';

  let open = $state(false);
</script>

<Dialog.Root bind:open>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {#if open}
    <Dialog.Portal>
      <Dialog.Overlay transition={fade} />
      <Dialog.Content transition={scale} transitionConfig={{ duration: 150, start: 0.95 }}>
        Content
      </Dialog.Content>
    </Dialog.Portal>
  {/if}
</Dialog.Root>
```

### Custom Transitions

```typescript
<script lang="ts">
  import { cubicOut } from 'svelte/easing';

  function slideAndFade(node: HTMLElement, { duration = 200 }) {
    return {
      duration,
      css: (t: number) => {
        const eased = cubicOut(t);
        return `
          opacity: ${eased};
          transform: translateY(${(1 - eased) * -10}px);
        `;
      }
    };
  }
</script>

<Dialog.Content transition={slideAndFade}>
  Content
</Dialog.Content>
```

## Responsive Design

### Mobile-First Approach

```css
:global([data-dialog-content]) {
  width: 100%;
  max-width: 90%;
  padding: 1rem;
}

@media (min-width: 640px) {
  :global([data-dialog-content]) {
    max-width: 28rem;
    padding: 1.5rem;
  }
}

@media (min-width: 768px) {
  :global([data-dialog-content]) {
    max-width: 32rem;
    padding: 2rem;
  }
}

:global([data-select-content]) {
  max-height: 300px;
}

@media (min-width: 640px) {
  :global([data-select-content]) {
    max-height: 400px;
  }
}
```

### Mobile Drawer Pattern

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';

  let isMobile = $state(false);

  $effect(() => {
    const checkMobile = () => {
      isMobile = window.innerWidth < 768;
    };
    checkMobile();
    window.addEventListener('resize', checkMobile);
    return () => window.removeEventListener('resize', checkMobile);
  });
</script>

<Dialog.Root>
  <Dialog.Trigger>Open</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content class={isMobile ? 'mobile-drawer' : 'desktop-dialog'}>
      Content
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>

<style>
  :global(.mobile-drawer) {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    top: auto;
    transform: none;
    max-width: 100%;
    border-radius: 1rem 1rem 0 0;
    animation: slideUpFromBottom 200ms ease-out;
  }

  :global(.desktop-dialog) {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border-radius: 0.5rem;
    animation: scaleIn 150ms ease-out;
  }

  @keyframes slideUpFromBottom {
    from {
      transform: translateY(100%);
    }
    to {
      transform: translateY(0);
    }
  }
</style>
```

## Component Composition

### Styled Wrapper Components

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';

  interface Props {
    open?: boolean;
    title: string;
    description?: string;
    children?: import('svelte').Snippet;
    trigger?: import('svelte').Snippet;
  }

  let { open = $bindable(false), title, description, children, trigger }: Props = $props();
</script>

<Dialog.Root bind:open>
  {#if trigger}
    {@render trigger()}
  {:else}
    <Dialog.Trigger class="btn-primary">
      Open
    </Dialog.Trigger>
  {/if}

  <Dialog.Portal>
    <Dialog.Overlay class="dialog-overlay" />
    <Dialog.Content class="dialog-content">
      <Dialog.Title class="dialog-title">
        {title}
      </Dialog.Title>

      {#if description}
        <Dialog.Description class="dialog-description">
          {description}
        </Dialog.Description>
      {/if}

      <div class="dialog-body">
        {@render children?.()}
      </div>

      <Dialog.Close class="dialog-close">×</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>

<style>
  :global(.dialog-overlay) {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(4px);
    z-index: 50;
    animation: fadeIn 150ms;
  }

  :global(.dialog-content) {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: white;
    padding: 2rem;
    border-radius: 0.5rem;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
    max-width: 32rem;
    width: 90%;
    z-index: 50;
    animation: slideIn 150ms;
  }

  :global(.dialog-title) {
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
  }

  :global(.dialog-description) {
    color: #6b7280;
    margin-bottom: 1rem;
  }

  :global(.dialog-body) {
    margin-bottom: 1.5rem;
  }

  :global(.dialog-close) {
    position: absolute;
    top: 1rem;
    right: 1rem;
    width: 2rem;
    height: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border: none;
    background: transparent;
    font-size: 1.5rem;
    cursor: pointer;
    border-radius: 0.25rem;
    color: #6b7280;
  }

  :global(.dialog-close):hover {
    background: #f3f4f6;
    color: #111827;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  @keyframes slideIn {
    from {
      transform: translate(-50%, -48%);
      opacity: 0;
    }
    to {
      transform: translate(-50%, -50%);
      opacity: 1;
    }
  }
</style>
```

Usage:

```typescript
<StyledDialog title="Confirm Delete" description="This action cannot be undone">
  <p>Are you sure you want to delete this item?</p>
  <button>Delete</button>
</StyledDialog>
```

## Design Systems Integration

### Shadcn-style Variants

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';
  import { cva, type VariantProps } from 'class-variance-authority';
  import { cn } from '$lib/utils';

  const dialogVariants = cva(
    'fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 z-50 rounded-lg shadow-xl',
    {
      variants: {
        size: {
          sm: 'max-w-sm',
          md: 'max-w-md',
          lg: 'max-w-lg',
          xl: 'max-w-xl'
        },
        variant: {
          default: 'bg-white',
          dark: 'bg-gray-900 text-white'
        }
      },
      defaultVariants: {
        size: 'md',
        variant: 'default'
      }
    }
  );

  interface Props extends VariantProps<typeof dialogVariants> {
    class?: string;
  }

  let { size, variant, class: className }: Props = $props();
</script>

<Dialog.Content class={cn(dialogVariants({ size, variant }), className)}>
  <slot />
</Dialog.Content>
```

### Theme Provider Pattern

```typescript
<script lang="ts">
  import { setContext } from 'svelte';

  interface Theme {
    primary: string;
    radius: string;
    shadow: string;
  }

  const theme: Theme = {
    primary: '#3b82f6',
    radius: '0.5rem',
    shadow: '0 10px 15px -3px rgba(0, 0, 0, 0.1)'
  };

  setContext('theme', theme);
</script>

<div style="
  --color-primary: {theme.primary};
  --radius: {theme.radius};
  --shadow: {theme.shadow};
">
  <slot />
</div>
```

## Accessibility Considerations

### Focus Styles

```css
:global([data-dialog-content]:focus-visible) {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}

:global([data-select-trigger]:focus-visible) {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}

:global([data-select-item]:focus-visible) {
  outline: none;
  background: #dbeafe;
}
```

### High Contrast Mode

```css
@media (prefers-contrast: high) {
  :global([data-dialog-overlay]) {
    background: rgba(0, 0, 0, 0.8);
  }

  :global([data-dialog-content]) {
    border: 2px solid currentColor;
  }

  :global([data-select-trigger]) {
    border: 2px solid currentColor;
  }
}
```

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  :global([data-dialog-overlay]),
  :global([data-dialog-content]),
  :global([data-popover-content]),
  :global([data-select-content]) {
    animation: none;
    transition: none;
  }
}
```

## Performance Optimization

### CSS Containment

```css
:global([data-dialog-content]) {
  contain: layout style paint;
}

:global([data-select-content]) {
  contain: layout style paint;
}
```

### Will-change Property

```css
:global([data-dialog-content]) {
  will-change: transform, opacity;
}

:global([data-accordion-content]) {
  will-change: height;
}
```

### GPU Acceleration

```css
:global([data-dialog-overlay]),
:global([data-dialog-content]) {
  transform: translateZ(0);
  backface-visibility: hidden;
}
```

## Print Styles

```css
@media print {
  :global([data-dialog-overlay]),
  :global([data-popover-content]),
  :global([data-select-content]) {
    display: none !important;
  }

  :global([data-dialog-content]) {
    position: static;
    transform: none;
    box-shadow: none;
    border: 1px solid black;
    page-break-inside: avoid;
  }
}
```

## Best Practices

1. **Use Data Attributes** - Target elements using data attributes rather than custom classes
2. **Progressive Enhancement** - Ensure basic functionality works without JavaScript
3. **Respect User Preferences** - Support prefers-color-scheme, prefers-reduced-motion, and prefers-contrast
4. **Optimize Animations** - Use transform and opacity for better performance
5. **Mobile-First** - Design for mobile, enhance for desktop
6. **Semantic HTML** - Use proper heading hierarchy and landmark regions
7. **Focus Management** - Ensure visible focus indicators for keyboard navigation
8. **Color Contrast** - Maintain WCAG AA contrast ratios (4.5:1 for text)
9. **CSS Containment** - Use contain property for better rendering performance
10. **Theme Variables** - Use CSS custom properties for consistent theming

For component-specific styling examples, see COMPONENTS.md
