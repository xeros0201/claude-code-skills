---
name: building-with-bits-ui
description: Build accessible Svelte applications using Bits UI components. Use when creating Bits UI components, implementing accessible UI patterns, styling unstyled components, or when user mentions Bits UI, headless components, or accessible Svelte components.
allowed-tools: [WebFetch, Read, Write, Edit, Glob, Grep, Bash]
---

# Building with Bits UI

Build accessible, unstyled Svelte components using Bits UI - a headless component library providing accessible primitives.

## Core Principles

**Unstyled by Design**: Bits UI provides zero styling. You control all visual design through CSS, Tailwind, or CSS-in-JS.

**Accessibility First**: All components follow WAI-ARIA patterns with keyboard navigation, focus management, and screen reader support built-in.

**Svelte 5 Runes**: Use Svelte 5's runes ($state, $derived, $effect) for reactive state management with Bits UI.

**Security**: Sanitize user content, validate input, implement proper CORS and CSP policies.

## Installation

```bash
npm install bits-ui
```

## Quick Start

```typescript
<script lang="ts">
  import { Accordion } from 'bits-ui';

  let value = $state('item-1');
</script>

<Accordion.Root bind:value type="single" collapsible>
  <Accordion.Item value="item-1">
    <Accordion.Header>
      <Accordion.Trigger>What is Bits UI?</Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Content>
      Bits UI is an unstyled component library for Svelte.
    </Accordion.Content>
  </Accordion.Item>
</Accordion.Root>

<style>
  :global([data-accordion-trigger]) {
    width: 100%;
    padding: 1rem;
    border: 1px solid #e5e7eb;
  }

  :global([data-accordion-content]) {
    padding: 1rem;
    border: 1px solid #e5e7eb;
    border-top: none;
  }
</style>
```

## Common Patterns

### Dialog with Form Validation

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';

  let open = $state(false);
  let email = $state('');
  let errors = $state<Record<string, string>>({});

  function validateEmail(value: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(value);
  }

  function handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    errors = {};

    if (!email) {
      errors.email = 'Email is required';
      return;
    }

    if (!validateEmail(email)) {
      errors.email = 'Invalid email format';
      return;
    }

    submitForm({ email });
    open = false;
  }

  async function submitForm(data: { email: string }) {
    try {
      const response = await fetch('/api/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });

      if (!response.ok) {
        throw new Error('Subscription failed');
      }
    } catch (error) {
      errors.email = 'Failed to subscribe. Please try again.';
    }
  }
</script>

<Dialog.Root bind:open>
  <Dialog.Trigger>Subscribe</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Title>Subscribe to Newsletter</Dialog.Title>
      <Dialog.Description>Enter your email to receive updates.</Dialog.Description>

      <form onsubmit={handleSubmit}>
        <label for="email">Email</label>
        <input
          id="email"
          type="email"
          bind:value={email}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {#if errors.email}
          <span id="email-error" role="alert">{errors.email}</span>
        {/if}
        <button type="submit">Subscribe</button>
      </form>

      <Dialog.Close>Cancel</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>

<style>
  :global([data-dialog-overlay]) {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 50;
  }

  :global([data-dialog-content]) {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: white;
    padding: 2rem;
    border-radius: 0.5rem;
    max-width: 28rem;
    width: 90%;
    z-index: 50;
  }

  input[aria-invalid="true"] {
    border-color: #ef4444;
  }

  span[role="alert"] {
    color: #ef4444;
    font-size: 0.875rem;
  }
</style>
```

### Select with Search

```typescript
<script lang="ts">
  import { Select } from 'bits-ui';

  interface Option {
    value: string;
    label: string;
  }

  const countries: Option[] = [
    { value: 'us', label: 'United States' },
    { value: 'ca', label: 'Canada' },
    { value: 'uk', label: 'United Kingdom' }
  ];

  let selectedCountry = $state('');
  let searchQuery = $state('');

  const filteredCountries = $derived(
    countries.filter(c => c.label.toLowerCase().includes(searchQuery.toLowerCase()))
  );

  function handleSelect(value: string | undefined) {
    if (value && countries.some(c => c.value === value)) {
      selectedCountry = value;
    }
  }
</script>

<Select.Root selected={{ value: selectedCountry }} onSelectedChange={(v) => handleSelect(v?.value)}>
  <Select.Trigger aria-label="Select country">
    <Select.Value placeholder="Select a country" />
  </Select.Trigger>

  <Select.Portal>
    <Select.Content>
      <Select.Input bind:value={searchQuery} placeholder="Search..." />
      <Select.Viewport>
        {#each filteredCountries as country (country.value)}
          <Select.Item value={country.value}>
            <Select.ItemText>{country.label}</Select.ItemText>
          </Select.Item>
        {:else}
          <div>No countries found</div>
        {/each}
      </Select.Viewport>
    </Select.Content>
  </Select.Portal>
</Select.Root>
```

### Tabs with Lazy Loading

```typescript
<script lang="ts">
  import { Tabs } from 'bits-ui';

  let activeTab = $state('profile');
  let profileData = $state(null);

  $effect(() => {
    if (activeTab === 'profile' && !profileData) {
      loadProfileData();
    }
  });

  async function loadProfileData() {
    const response = await fetch('/api/profile');
    if (response.ok) {
      profileData = await response.json();
    }
  }
</script>

<Tabs.Root bind:value={activeTab}>
  <Tabs.List>
    <Tabs.Trigger value="profile">Profile</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="profile">
    {#if profileData}
      <h2>{profileData.name}</h2>
      <p>{profileData.email}</p>
    {:else}
      <p>Loading...</p>
    {/if}
  </Tabs.Content>

  <Tabs.Content value="settings">
    <div>Settings content</div>
  </Tabs.Content>
</Tabs.Root>
```

## Styling Approaches

### Data Attributes with CSS

Bits UI adds data attributes to all elements for styling:

```css
:global([data-state="open"]) {
  display: block;
}

:global([data-state="closed"]) {
  display: none;
}

:global([data-disabled]) {
  opacity: 0.5;
  cursor: not-allowed;
}

:global([data-highlighted]) {
  background: #f3f4f6;
}

:global([data-selected]) {
  background: #dbeafe;
}
```

### Tailwind CSS Integration

Apply Tailwind classes directly to components:

```typescript
<Dialog.Trigger class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
  Open Dialog
</Dialog.Trigger>

<Dialog.Content class="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl max-w-md w-full">
  <Dialog.Title class="text-xl font-semibold mb-2">Title</Dialog.Title>
  <Dialog.Description class="text-gray-600 mb-4">Description</Dialog.Description>
</Dialog.Content>
```

### CSS Variables for Theming

```css
:root {
  --color-primary: #3b82f6;
  --color-border: #e5e7eb;
  --color-background: #ffffff;
  --radius: 0.5rem;
}

[data-theme="dark"] {
  --color-primary: #60a5fa;
  --color-border: #374151;
  --color-background: #1f2937;
}

:global([data-dialog-content]) {
  background: var(--color-background);
  border-radius: var(--radius);
}
```

### State-Based Styling

```css
:global([data-select-item][data-highlighted]) {
  background: #f3f4f6;
}

:global([data-select-item][data-selected]) {
  background: #dbeafe;
}

:global([data-accordion-trigger][data-state="open"]) {
  transform: rotate(180deg);
}

:global([data-switch-root][data-state="checked"]) {
  background: #3b82f6;
}
```

## Security Best Practices

### XSS Prevention

Always sanitize user-generated content:

```typescript
<script lang="ts">
  import { sanitize } from 'dompurify';

  let userContent = $state('');
  const sanitizedContent = $derived(sanitize(userContent));
</script>

<Dialog.Content>
  {@html sanitizedContent}
</Dialog.Content>
```

### CSRF Protection

Include CSRF tokens in forms:

```typescript
<script lang="ts">
  import { page } from '$app/stores';

  async function handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    const formData = new FormData(event.target as HTMLFormElement);
    formData.append('csrf_token', $page.data.csrfToken);

    await fetch('/api/submit', {
      method: 'POST',
      body: formData
    });
  }
</script>

<form onsubmit={handleSubmit}>
  <input type="hidden" name="csrf_token" value={$page.data.csrfToken} />
</form>
```

### Input Validation

Validate all user input with Zod:

```typescript
<script lang="ts">
  import { z } from 'zod';

  const formSchema = z.object({
    email: z.string().email(),
    age: z.number().min(18).max(120),
    username: z.string().min(3).max(20).regex(/^[a-zA-Z0-9_]+$/)
  });

  function validateForm(data: unknown) {
    try {
      return formSchema.parse(data);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return { errors: error.flatten() };
      }
      throw error;
    }
  }
</script>
```

### Content Security Policy

Configure CSP headers in SvelteKit hooks:

```typescript
export async function handle({ event, resolve }) {
  const response = await resolve(event);

  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';"
  );

  return response;
}
```

## Component Reference

Common Bits UI components:

- **Accordion**: Expandable content sections
- **Alert Dialog**: Modal dialogs requiring user action
- **Checkbox**: Checkboxes with indeterminate state
- **Combobox**: Searchable select with autocomplete
- **Context Menu**: Right-click context menus
- **Dialog**: Modal dialogs and overlays
- **Dropdown Menu**: Action menus with nesting
- **Popover**: Floating content containers
- **Radio Group**: Radio button groups
- **Select**: Dropdown selection lists
- **Slider**: Range input sliders
- **Switch**: Toggle switches
- **Tabs**: Tabbed content panels
- **Tooltip**: Hover/focus tooltips
- **Date Picker**: Date selection with validation
- **Progress**: Progress indicators
- **Separator**: Visual dividers

For detailed component documentation, see COMPONENTS.md or visit https://bits-ui.com/docs/llms.txt

## Advanced Patterns

### Compound Components

Create reusable compound components:

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';

  export let open = $bindable(false);
  export let title: string;
  export let description: string;
</script>

<Dialog.Root bind:open>
  <slot name="trigger" />
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Title>{title}</Dialog.Title>
      <Dialog.Description>{description}</Dialog.Description>
      <slot />
      <Dialog.Close>Close</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

Usage:

```typescript
<CustomDialog bind:open title="Confirm" description="Are you sure?">
  <button slot="trigger">Open</button>
  <button onclick={handleConfirm}>Confirm</button>
</CustomDialog>
```

### State Management Integration

Integrate with Svelte stores:

```typescript
<script lang="ts">
  import { writable } from 'svelte/store';
  import { Dialog } from 'bits-ui';

  const dialogState = writable({ open: false, title: '', content: '' });

  export function openDialog(title: string, content: string) {
    dialogState.set({ open: true, title, content });
  }
</script>

<Dialog.Root open={$dialogState.open} onOpenChange={(o) => dialogState.update(s => ({ ...s, open: o }))}>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Title>{$dialogState.title}</Dialog.Title>
      <p>{$dialogState.content}</p>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

### Animation with Svelte Transitions

Add smooth transitions:

```typescript
<script lang="ts">
  import { Dialog } from 'bits-ui';
  import { fade, scale } from 'svelte/transition';

  let open = $state(false);
</script>

<Dialog.Root bind:open>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {#if open}
    <Dialog.Portal>
      <Dialog.Overlay transition={fade} />
      <Dialog.Content transition={scale}>
        Content
      </Dialog.Content>
    </Dialog.Portal>
  {/if}
</Dialog.Root>
```

## Testing

### Unit Testing with Vitest

```typescript
import { render, screen, fireEvent } from '@testing-library/svelte';
import { describe, it, expect } from 'vitest';

describe('Dialog', () => {
  it('opens when trigger is clicked', async () => {
    render(DialogComponent);
    const trigger = screen.getByRole('button', { name: /open/i });
    await fireEvent.click(trigger);
    const dialog = screen.getByRole('dialog');
    expect(dialog).toBeInTheDocument();
  });
});
```

### Accessibility Testing

```typescript
import { axe } from 'jest-axe';

it('has no accessibility violations', async () => {
  const { container } = render(DialogComponent);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Documentation Access

For up-to-date component APIs and patterns:

```typescript
const response = await fetch('https://bits-ui.com/docs/llms.txt');
const documentation = await response.text();
```

Component-specific documentation:

```typescript
const accordionDocs = await fetch('https://bits-ui.com/docs/components/accordion/llms.txt');
const dialogDocs = await fetch('https://bits-ui.com/docs/components/dialog/llms.txt');
```

## Additional Resources

- **Full Component Reference**: See COMPONENTS.md for comprehensive component patterns
- **Styling Guide**: See STYLING.md for advanced styling techniques
- **Official Documentation**: https://bits-ui.com/docs/llms.txt
- **Svelte 5 Best Practices**: https://svelte.dev/docs/llms

## Best Practices

When working with Bits UI, always:

1. Validate user input before processing
2. Sanitize HTML content to prevent XSS
3. Use proper ARIA labels for accessibility
4. Implement keyboard navigation
5. Test with screen readers
6. Follow Svelte 5 runes patterns
7. Reference official documentation for latest APIs
8. Use data attributes for styling instead of custom classes
9. Implement proper error handling and loading states
10. Test accessibility with automated tools
