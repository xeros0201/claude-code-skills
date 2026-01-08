# Bits UI Component Reference

Comprehensive guide to all Bits UI components with production-ready patterns.

## Accordion

Expandable content sections with single or multiple open panels.

### Single Selection

```typescript
<script lang="ts">
  import { Accordion } from 'bits-ui';

  let value = $state('panel-1');
</script>

<Accordion.Root bind:value type="single" collapsible>
  <Accordion.Item value="panel-1">
    <Accordion.Header>
      <Accordion.Trigger>
        Section 1
      </Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Content>
      Content for section 1
    </Accordion.Content>
  </Accordion.Item>

  <Accordion.Item value="panel-2">
    <Accordion.Header>
      <Accordion.Trigger>
        Section 2
      </Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Content>
      Content for section 2
    </Accordion.Content>
  </Accordion.Item>
</Accordion.Root>
```

### Multiple Selection

```typescript
<Accordion.Root value={['panel-1', 'panel-2']} type="multiple">
```

## Alert Dialog

Modal requiring user action before dismissal.

```typescript
<script lang="ts">
  import { AlertDialog } from 'bits-ui';

  let open = $state(false);

  async function handleConfirm() {
    await deleteAccount();
    open = false;
  }

  async function deleteAccount() {
    await fetch('/api/account', { method: 'DELETE' });
  }
</script>

<AlertDialog.Root bind:open>
  <AlertDialog.Trigger>Delete Account</AlertDialog.Trigger>
  <AlertDialog.Portal>
    <AlertDialog.Overlay />
    <AlertDialog.Content>
      <AlertDialog.Title>Are you absolutely sure?</AlertDialog.Title>
      <AlertDialog.Description>
        This action cannot be undone. This will permanently delete your account.
      </AlertDialog.Description>
      <div>
        <AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
        <AlertDialog.Action onclick={handleConfirm}>
          Delete Account
        </AlertDialog.Action>
      </div>
    </AlertDialog.Content>
  </AlertDialog.Portal>
</AlertDialog.Root>
```

## Checkbox

Checkbox with indeterminate state support.

```typescript
<script lang="ts">
  import { Checkbox } from 'bits-ui';

  let checked = $state<boolean | 'indeterminate'>(false);

  function toggleAll() {
    checked = checked === true ? false : true;
  }
</script>

<Checkbox.Root bind:checked>
  <Checkbox.Input />
  <Checkbox.Indicator>
    {#if checked === true}
      ✓
    {:else if checked === 'indeterminate'}
      −
    {/if}
  </Checkbox.Indicator>
</Checkbox.Root>
<label>Accept terms and conditions</label>
```

### Parent-Child Checkboxes

```typescript
<script lang="ts">
  import { Checkbox } from 'bits-ui';

  let parentChecked = $state<boolean | 'indeterminate'>('indeterminate');
  let childChecked = $state([true, false, false]);

  $effect(() => {
    const checkedCount = childChecked.filter(Boolean).length;
    if (checkedCount === 0) {
      parentChecked = false;
    } else if (checkedCount === childChecked.length) {
      parentChecked = true;
    } else {
      parentChecked = 'indeterminate';
    }
  });

  function toggleParent() {
    const newValue = parentChecked !== true;
    childChecked = childChecked.map(() => newValue);
  }
</script>

<Checkbox.Root checked={parentChecked} onCheckedChange={toggleParent}>
  <Checkbox.Input />
  <Checkbox.Indicator>
    {#if parentChecked === true}✓{:else if parentChecked === 'indeterminate'}−{/if}
  </Checkbox.Indicator>
</Checkbox.Root>
<label>Select all</label>

{#each childChecked as checked, i}
  <Checkbox.Root bind:checked={childChecked[i]}>
    <Checkbox.Input />
    <Checkbox.Indicator>{#if checked}✓{/if}</Checkbox.Indicator>
  </Checkbox.Root>
  <label>Option {i + 1}</label>
{/each}
```

## Combobox

Searchable select with autocomplete.

```typescript
<script lang="ts">
  import { Combobox } from 'bits-ui';

  interface Framework {
    value: string;
    label: string;
  }

  const frameworks: Framework[] = [
    { value: 'svelte', label: 'Svelte' },
    { value: 'react', label: 'React' },
    { value: 'vue', label: 'Vue' },
    { value: 'angular', label: 'Angular' }
  ];

  let selectedValue = $state('');
  let inputValue = $state('');

  const filteredFrameworks = $derived(
    frameworks.filter(f =>
      f.label.toLowerCase().includes(inputValue.toLowerCase())
    )
  );
</script>

<Combobox.Root bind:value={selectedValue} bind:inputValue>
  <Combobox.Input placeholder="Search frameworks..." />
  <Combobox.Portal>
    <Combobox.Content>
      {#each filteredFrameworks as framework (framework.value)}
        <Combobox.Item value={framework.value}>
          <Combobox.ItemText>{framework.label}</Combobox.ItemText>
        </Combobox.Item>
      {:else}
        <span>No results found</span>
      {/each}
    </Combobox.Content>
  </Combobox.Portal>
</Combobox.Root>
```

### Multi-select Combobox

```typescript
<script lang="ts">
  import { Combobox } from 'bits-ui';

  let selectedValues = $state<string[]>([]);
  let inputValue = $state('');

  function handleSelect(value: string) {
    if (selectedValues.includes(value)) {
      selectedValues = selectedValues.filter(v => v !== value);
    } else {
      selectedValues = [...selectedValues, value];
    }
  }

  function removeTag(value: string) {
    selectedValues = selectedValues.filter(v => v !== value);
  }
</script>

<div>
  {#each selectedValues as value}
    <button onclick={() => removeTag(value)}>
      {value} ×
    </button>
  {/each}
</div>

<Combobox.Root multiple bind:value={selectedValues} bind:inputValue>
  <Combobox.Input placeholder="Add tags..." />
  <Combobox.Content>
    {#each filteredOptions as option}
      <Combobox.Item value={option.value} selected={selectedValues.includes(option.value)}>
        {option.label}
      </Combobox.Item>
    {/each}
  </Combobox.Content>
</Combobox.Root>
```

## Context Menu

Right-click context menus.

```typescript
<script lang="ts">
  import { ContextMenu } from 'bits-ui';

  function handleCopy() {
    navigator.clipboard.writeText('Selected text');
  }

  function handlePaste() {
    navigator.clipboard.readText().then(text => {
      console.log('Pasted:', text);
    });
  }

  function handleDelete() {
    confirm('Delete selected item?') && deleteItem();
  }

  function deleteItem() {
    console.log('Item deleted');
  }
</script>

<ContextMenu.Root>
  <ContextMenu.Trigger>
    Right-click me
  </ContextMenu.Trigger>
  <ContextMenu.Portal>
    <ContextMenu.Content>
      <ContextMenu.Item onclick={handleCopy}>
        Copy
      </ContextMenu.Item>
      <ContextMenu.Item onclick={handlePaste}>
        Paste
      </ContextMenu.Item>
      <ContextMenu.Separator />
      <ContextMenu.Sub>
        <ContextMenu.SubTrigger>More Actions</ContextMenu.SubTrigger>
        <ContextMenu.SubContent>
          <ContextMenu.Item>Share</ContextMenu.Item>
          <ContextMenu.Item>Duplicate</ContextMenu.Item>
        </ContextMenu.SubContent>
      </ContextMenu.Sub>
      <ContextMenu.Separator />
      <ContextMenu.Item onclick={handleDelete}>
        Delete
      </ContextMenu.Item>
    </ContextMenu.Content>
  </ContextMenu.Portal>
</ContextMenu.Root>
```

## Dropdown Menu

Action menus with keyboard navigation.

```typescript
<script lang="ts">
  import { DropdownMenu } from 'bits-ui';

  let open = $state(false);

  function handleLogout() {
    fetch('/api/logout', { method: 'POST' }).then(() => {
      window.location.href = '/login';
    });
  }
</script>

<DropdownMenu.Root bind:open>
  <DropdownMenu.Trigger>
    Open Menu
  </DropdownMenu.Trigger>
  <DropdownMenu.Portal>
    <DropdownMenu.Content>
      <DropdownMenu.Group>
        <DropdownMenu.Label>My Account</DropdownMenu.Label>
        <DropdownMenu.Item href="/profile">
          Profile
        </DropdownMenu.Item>
        <DropdownMenu.Item href="/settings">
          Settings
        </DropdownMenu.Item>
      </DropdownMenu.Group>

      <DropdownMenu.Separator />

      <DropdownMenu.Group>
        <DropdownMenu.Item>Team</DropdownMenu.Item>
        <DropdownMenu.Sub>
          <DropdownMenu.SubTrigger>
            Invite Users
          </DropdownMenu.SubTrigger>
          <DropdownMenu.SubContent>
            <DropdownMenu.Item>Email</DropdownMenu.Item>
            <DropdownMenu.Item>SMS</DropdownMenu.Item>
          </DropdownMenu.SubContent>
        </DropdownMenu.Sub>
      </DropdownMenu.Group>

      <DropdownMenu.Separator />

      <DropdownMenu.Item onclick={handleLogout}>
        Log out
      </DropdownMenu.Item>
    </DropdownMenu.Content>
  </DropdownMenu.Portal>
</DropdownMenu.Root>
```

### With Checkboxes

```typescript
<script lang="ts">
  import { DropdownMenu } from 'bits-ui';

  let showPanel = $state(true);
  let showSidebar = $state(false);
</script>

<DropdownMenu.Root>
  <DropdownMenu.Trigger>View</DropdownMenu.Trigger>
  <DropdownMenu.Content>
    <DropdownMenu.CheckboxItem bind:checked={showPanel}>
      Show Panel
    </DropdownMenu.CheckboxItem>
    <DropdownMenu.CheckboxItem bind:checked={showSidebar}>
      Show Sidebar
    </DropdownMenu.CheckboxItem>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

## Radio Group

Radio button groups with validation.

```typescript
<script lang="ts">
  import { RadioGroup } from 'bits-ui';

  let selectedPlan = $state('');
  let error = $state('');

  function validateSelection() {
    if (!selectedPlan) {
      error = 'Please select a plan';
      return false;
    }
    error = '';
    return true;
  }

  function handleSubmit() {
    if (validateSelection()) {
      submitPlan(selectedPlan);
    }
  }

  async function submitPlan(plan: string) {
    await fetch('/api/plan', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ plan })
    });
  }
</script>

<RadioGroup.Root bind:value={selectedPlan} aria-invalid={!!error}>
  <RadioGroup.Item value="free">
    <RadioGroup.Input />
    <RadioGroup.Indicator />
    <label>Free - $0/month</label>
  </RadioGroup.Item>

  <RadioGroup.Item value="pro">
    <RadioGroup.Input />
    <RadioGroup.Indicator />
    <label>Pro - $9/month</label>
  </RadioGroup.Item>

  <RadioGroup.Item value="enterprise">
    <RadioGroup.Input />
    <RadioGroup.Indicator />
    <label>Enterprise - Contact sales</label>
  </RadioGroup.Item>
</RadioGroup.Root>

{#if error}
  <span role="alert">{error}</span>
{/if}

<button onclick={handleSubmit}>Continue</button>
```

## Slider

Range input sliders with formatting.

```typescript
<script lang="ts">
  import { Slider } from 'bits-ui';

  let value = $state([50]);
  let rangeValue = $state([25, 75]);

  const formattedValue = $derived(`${value[0]}%`);
</script>

<label>Volume: {formattedValue}</label>
<Slider.Root bind:value min={0} max={100} step={1}>
  <Slider.Track>
    <Slider.Range />
  </Slider.Track>
  <Slider.Thumb />
</Slider.Root>

<label>Price Range: ${rangeValue[0]} - ${rangeValue[1]}</label>
<Slider.Root bind:value={rangeValue} min={0} max={1000} step={10}>
  <Slider.Track>
    <Slider.Range />
  </Slider.Track>
  <Slider.Thumb />
  <Slider.Thumb />
</Slider.Root>
```

### Vertical Slider

```typescript
<Slider.Root bind:value orientation="vertical" min={0} max={100}>
  <Slider.Track>
    <Slider.Range />
  </Slider.Track>
  <Slider.Thumb />
</Slider.Root>
```

## Switch

Toggle switches with labels.

```typescript
<script lang="ts">
  import { Switch } from 'bits-ui';

  let enabled = $state(false);

  $effect(() => {
    if (enabled) {
      enableFeature();
    } else {
      disableFeature();
    }
  });

  function enableFeature() {
    localStorage.setItem('feature', 'enabled');
  }

  function disableFeature() {
    localStorage.removeItem('feature');
  }
</script>

<Switch.Root bind:checked={enabled}>
  <Switch.Input />
  <Switch.Thumb />
</Switch.Root>
<label>Enable notifications</label>
```

### With Loading State

```typescript
<script lang="ts">
  import { Switch } from 'bits-ui';

  let enabled = $state(false);
  let loading = $state(false);

  async function handleToggle(checked: boolean) {
    loading = true;
    try {
      await fetch('/api/setting', {
        method: 'POST',
        body: JSON.stringify({ enabled: checked })
      });
      enabled = checked;
    } catch (error) {
      console.error('Failed to update setting');
    } finally {
      loading = false;
    }
  }
</script>

<Switch.Root checked={enabled} onCheckedChange={handleToggle} disabled={loading}>
  <Switch.Input />
  <Switch.Thumb />
</Switch.Root>
```

## Tooltip

Hover and focus tooltips.

```typescript
<script lang="ts">
  import { Tooltip } from 'bits-ui';

  let open = $state(false);
</script>

<Tooltip.Root bind:open>
  <Tooltip.Trigger>
    Hover me
  </Tooltip.Trigger>
  <Tooltip.Portal>
    <Tooltip.Content>
      Tooltip content
    </Tooltip.Content>
  </Tooltip.Portal>
</Tooltip.Root>
```

### Rich Content Tooltip

```typescript
<script lang="ts">
  import { Tooltip } from 'bits-ui';

  interface User {
    name: string;
    email: string;
    avatar: string;
  }

  export let user: User;
</script>

<Tooltip.Root>
  <Tooltip.Trigger>
    <img src={user.avatar} alt={user.name} />
  </Tooltip.Trigger>
  <Tooltip.Portal>
    <Tooltip.Content>
      <div>
        <strong>{user.name}</strong>
        <p>{user.email}</p>
      </div>
    </Tooltip.Content>
  </Tooltip.Portal>
</Tooltip.Root>
```

## Date Picker

Date selection with validation.

```typescript
<script lang="ts">
  import { DatePicker } from 'bits-ui';

  let selectedDate = $state<Date | undefined>(undefined);

  const minDate = new Date();
  const maxDate = new Date();
  maxDate.setFullYear(maxDate.getFullYear() + 1);

  function isDateDisabled(date: Date): boolean {
    return date.getDay() === 0 || date.getDay() === 6;
  }
</script>

<DatePicker.Root bind:value={selectedDate}>
  <DatePicker.Trigger>
    {selectedDate ? selectedDate.toLocaleDateString() : 'Select date'}
  </DatePicker.Trigger>
  <DatePicker.Portal>
    <DatePicker.Content>
      <DatePicker.Calendar min={minDate} max={maxDate} isDateDisabled={isDateDisabled}>
        <DatePicker.Header>
          <DatePicker.PrevButton />
          <DatePicker.Heading />
          <DatePicker.NextButton />
        </DatePicker.Header>
        <DatePicker.Grid>
          <DatePicker.GridHead>
            <DatePicker.GridRow>
              {#each ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa'] as day}
                <DatePicker.HeadCell>{day}</DatePicker.HeadCell>
              {/each}
            </DatePicker.GridRow>
          </DatePicker.GridHead>
          <DatePicker.GridBody>
            <DatePicker.GridRow>
              <DatePicker.Cell />
            </DatePicker.GridRow>
          </DatePicker.GridBody>
        </DatePicker.Grid>
      </DatePicker.Calendar>
    </DatePicker.Content>
  </DatePicker.Portal>
</DatePicker.Root>
```

## Progress

Progress indicators.

```typescript
<script lang="ts">
  import { Progress } from 'bits-ui';

  let value = $state(0);

  async function uploadFile(file: File) {
    const formData = new FormData();
    formData.append('file', file);

    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (e) => {
      if (e.lengthComputable) {
        value = (e.loaded / e.total) * 100;
      }
    });

    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  }
</script>

<Progress.Root {value} max={100}>
  <Progress.Indicator style="width: {value}%" />
</Progress.Root>
<span>{Math.round(value)}%</span>
```

## Separator

Visual dividers.

```typescript
<script lang="ts">
  import { Separator } from 'bits-ui';
</script>

<div>Section 1</div>
<Separator.Root />
<div>Section 2</div>

<div>
  <span>Left</span>
  <Separator.Root orientation="vertical" />
  <span>Right</span>
</div>
```

## Complete Form Example

Combining multiple components with validation:

```typescript
<script lang="ts">
  import { Dialog, Select, Checkbox, Switch, RadioGroup } from 'bits-ui';
  import { z } from 'zod';

  const schema = z.object({
    name: z.string().min(2).max(50),
    email: z.string().email(),
    country: z.string().min(1),
    plan: z.enum(['free', 'pro', 'enterprise']),
    notifications: z.boolean(),
    terms: z.boolean().refine(val => val === true, 'You must accept the terms')
  });

  let open = $state(false);
  let formData = $state({
    name: '',
    email: '',
    country: '',
    plan: '',
    notifications: false,
    terms: false
  });
  let errors = $state<Record<string, string>>({});

  function handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    errors = {};

    try {
      const validated = schema.parse(formData);
      submitForm(validated);
      open = false;
    } catch (error) {
      if (error instanceof z.ZodError) {
        errors = Object.fromEntries(
          error.errors.map(err => [err.path[0], err.message])
        );
      }
    }
  }

  async function submitForm(data: z.infer<typeof schema>) {
    await fetch('/api/signup', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
  }
</script>

<Dialog.Root bind:open>
  <Dialog.Trigger>Sign Up</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Title>Create Account</Dialog.Title>

      <form onsubmit={handleSubmit}>
        <label for="name">Name</label>
        <input
          id="name"
          bind:value={formData.name}
          aria-invalid={!!errors.name}
        />
        {#if errors.name}
          <span role="alert">{errors.name}</span>
        {/if}

        <label for="email">Email</label>
        <input
          id="email"
          type="email"
          bind:value={formData.email}
          aria-invalid={!!errors.email}
        />
        {#if errors.email}
          <span role="alert">{errors.email}</span>
        {/if}

        <label>Country</label>
        <Select.Root bind:value={formData.country}>
          <Select.Trigger>
            <Select.Value placeholder="Select country" />
          </Select.Trigger>
          <Select.Content>
            <Select.Item value="us">United States</Select.Item>
            <Select.Item value="uk">United Kingdom</Select.Item>
            <Select.Item value="ca">Canada</Select.Item>
          </Select.Content>
        </Select.Root>
        {#if errors.country}
          <span role="alert">{errors.country}</span>
        {/if}

        <label>Plan</label>
        <RadioGroup.Root bind:value={formData.plan}>
          <RadioGroup.Item value="free">
            <RadioGroup.Input />
            <label>Free</label>
          </RadioGroup.Item>
          <RadioGroup.Item value="pro">
            <RadioGroup.Input />
            <label>Pro</label>
          </RadioGroup.Item>
          <RadioGroup.Item value="enterprise">
            <RadioGroup.Input />
            <label>Enterprise</label>
          </RadioGroup.Item>
        </RadioGroup.Root>
        {#if errors.plan}
          <span role="alert">{errors.plan}</span>
        {/if}

        <Switch.Root bind:checked={formData.notifications}>
          <Switch.Input />
          <Switch.Thumb />
        </Switch.Root>
        <label>Enable notifications</label>

        <Checkbox.Root bind:checked={formData.terms}>
          <Checkbox.Input />
          <Checkbox.Indicator>{#if formData.terms}✓{/if}</Checkbox.Indicator>
        </Checkbox.Root>
        <label>I accept the terms and conditions</label>
        {#if errors.terms}
          <span role="alert">{errors.terms}</span>
        {/if}

        <button type="submit">Create Account</button>
      </form>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

## Best Practices

1. **Always validate user input** - Use Zod or similar validation libraries
2. **Provide clear error messages** - Use role="alert" for accessibility
3. **Handle loading states** - Disable buttons and show feedback during async operations
4. **Implement keyboard navigation** - Test all interactions with keyboard
5. **Test with screen readers** - Ensure proper ARIA labels and descriptions
6. **Sanitize HTML content** - Use DOMPurify for user-generated content
7. **Use proper form semantics** - Include labels, fieldsets, and legends
8. **Handle edge cases** - Empty states, loading states, error states

For the latest component APIs and additional examples, visit https://bits-ui.com/docs/llms.txt
