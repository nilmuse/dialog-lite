# dialog-lite

Open and close modal dialogs built from your own DOM markup, with focus handling and scroll locking.

[![npm](https://img.shields.io/npm/v/dialog-lite.svg?colorB=brightgreen)](https://www.npmjs.com/package/dialog-lite)
[![NPM Downloads](https://img.shields.io/npm/dm/dialog-lite.svg?style=flat)](https://www.npmjs.com/package/dialog-lite)

[Demo](https://codepen.io/ux-ui/pen/LYoLOjr)

## Installation

```bash
npm install dialog-lite
```

## Quick start

Add the trigger and dialog markup. Use `hidden` for the initial state: the controller toggles that attribute, so `style="display:none"` would keep the dialog hidden when opened.

```html
<main id="main-content">
  <button id="open-dialog" type="button">Open dialog</button>
</main>

<div class="dialog-lite dialog-lite--out" hidden aria-hidden="true" aria-label="Example dialog">
  <div class="dialog-lite__backdrop"></div>
  <div class="dialog-lite__container">
    <div class="dialog-lite__container-inner">
      <button class="dialog-lite-close-button" type="button" aria-label="Close" tabindex="0">
        Close
      </button>
      <p>Your content</p>
    </div>
  </div>
</div>
```

Run this in your browser entry point after the markup is available:

```ts
import { initDialogLite } from 'dialog-lite';
import 'dialog-lite/dialog-lite.css';

const dialog = initDialogLite({
  closingButton: true,
  closingBackdrop: true,
  injectCss: false,
});

const button = document.querySelector<HTMLButtonElement>('#open-dialog');

button?.addEventListener('click', () => {
  dialog.open();
});
```

The button opens the dialog. The close button, backdrop, and Escape key close it. Call `dialog.destroy()` when removing the dialog.

## API

- `new DialogLite(options?)` and `createDialogLite(options?)` create a controller without initializing it.
- `initDialogLite(options?)` injects CSS by default, calls `init()`, and returns a controller.
- `injectDialogLiteCss(options?)` inserts or updates a style element in a document or shadow root. Its options are `target` (default `document`), `cssText` (default packaged CSS), and `id` (default `'dialog-lite-styles'`). It returns the style element.
- `dialogLiteCssText` is the packaged minified CSS string.

### Options

These options apply to the constructor and both controller factories:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `closingButton` | `boolean` | `false` | Close on a matching close button's click. |
| `closingBackdrop` | `boolean` | `false` | Close on a matching backdrop's click. |
| `dialog` | `HTMLElement \| string` | `'.dialog-lite'` | Dialog root element or selector. |
| `mainContent` | `HTMLElement \| string \| null` | `'#main-content'` | Element to set `aria-hidden` on while the dialog is open. See the note below about `null`. |
| `closeButtonSelector` | `string` | `'.dialog-lite-close-button'` | Close button selector inside the dialog. |
| `backdropSelector` | `string` | `'.dialog-lite__backdrop'` | Backdrop selector inside the dialog. |
| `debounceMs` | `number` | `500` | Minimum interval between accepted `open()` and `close()` calls. |
| `hideDelayMs` | `number` | `500` | Delay before applying `hidden` on close. Match it to your CSS transitions. |
| `focusOnOpenSelector` | `string` | `'[tabindex="0"]'` | Element inside the dialog to focus on open. The root receives focus if no match is found. |
| `lockScroll` | `boolean` | `true` | Lock page scroll while open. |
| `trapFocus` | `boolean` | `true` | Cycle Tab focus inside the dialog. |
| `closeOnEscape` | `boolean` | `true` | Close an open dialog with Escape. |
| `role` | `string` | `'dialog'` | Role applied during `init()`. |
| `ariaModal` | `boolean` | `true` | Apply `aria-modal="true"` during `init()`. |
| `emitEvents` | `boolean` | `true` | Dispatch the DOM events listed below. Callbacks run regardless of this option. |
| `onOpen` | `(detail: DialogLiteOpenDetail) => void` | `undefined` | Callback after an accepted `open()`. |
| `onClose` | `(detail: DialogLiteCloseDetail) => void` | `undefined` | Callback after an accepted `close()`. |

In the current implementation, `mainContent: null` falls back to `'#main-content'`, just like an omitted value. It does not disable background `aria-hidden` handling. If the selector matches no element, no background element is updated.

`initDialogLite()` also accepts:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `injectCss` | `boolean` | `true` | Inject packaged CSS during initialization. Set `false` when importing the stylesheet yourself. |
| `cssText` | `string` | `undefined` | CSS text for injection; omission uses packaged CSS. |
| `cssTarget` | `Document \| ShadowRoot` | `undefined` | Injection target; omission uses `document`. |

### Methods

- `init()` resolves the elements and attaches listeners. It throws if the dialog root cannot be found. It also sets the root's role, modal attribute, and a fallback `tabindex` where applicable.
- `open({ stylingClass = '' } = {})` removes `hidden`, applies the open classes and ARIA state, locks scroll when enabled, and moves focus inside. `stylingClass` adds one class token for this opening.
- `close()` applies the closed classes and ARIA state, restores focus to the previously focused element if it is still connected, and unlocks scroll. It sets `hidden` and removes the extra styling class after `hideDelayMs`.
- `isOpened()` returns the controller's current open state.
- `destroy()` removes listeners, clears timers, and unlocks scroll. It does not close the dialog or reset its open state.

`open()` and `close()` initialize the controller if needed and ignore calls inside the `debounceMs` window.

### Events

Events are dispatched on the dialog element. They do not bubble. The matching callbacks receive the same detail objects.

| Event | When | Payload (`event.detail`) |
| --- | --- | --- |
| `dialog-lite:open` | After the open state and focus are updated. | `{ stylingClass: string }` |
| `dialog-lite:close` | After the close state is applied, before the delayed `hidden` update. | `{}` |

## Vue 3

`dialog-lite/vue` exports a composable for custom markup and a component that renders the default wrapper. Vue is an optional peer dependency; the main entry does not require it.

### Custom markup

```vue
<script setup lang="ts">
import { ref } from 'vue';
import { useDialogLite } from 'dialog-lite/vue';
import 'dialog-lite/dialog-lite.css';

const dialogRef = ref<HTMLElement | null>(null);
const { isOpen, open, close } = useDialogLite(dialogRef, {
  closingBackdrop: true,
});
</script>

<template>
  <button type="button" @click="open()">Open dialog</button>
  <p>Dialog open: {{ isOpen }}</p>

  <div
    ref="dialogRef"
    class="dialog-lite dialog-lite--out"
    hidden
    aria-hidden="true"
    aria-label="Example dialog"
  >
    <div class="dialog-lite__backdrop"></div>
    <div class="dialog-lite__container">
      <div class="dialog-lite__container-inner">
        <button type="button" tabindex="0" @click="close">Close</button>
        <p>Your content</p>
      </div>
    </div>
  </div>
</template>
```

`useDialogLite(dialogRef, options?)` accepts an element ref or a function returning an element. It initializes on mount, keeps `isOpen` in sync through callbacks, and destroys the controller when the component scope is disposed.

It returns `dialog`, `isOpen`, `init`, `open`, `close`, and `destroy`. Its options follow the core options, except that the element source supplies `dialog`.

### Default wrapper

```vue
<script setup lang="ts">
import { ref } from 'vue';
import { DialogLiteRoot } from 'dialog-lite/vue';
import 'dialog-lite/dialog-lite.css';

const isOpen = ref(false);
</script>

<template>
  <button type="button" @click="isOpen = true">Open dialog</button>

  <DialogLiteRoot v-model="isOpen" close-on-backdrop aria-label="Example dialog">
    <button type="button" tabindex="0" @click="isOpen = false">Close</button>
    <p>Your content</p>
  </DialogLiteRoot>
</template>
```

`DialogLiteRoot` controls the core instance through `v-model` and emits `update:modelValue`, `open`, and `close`.

Its props are `modelValue`, `stylingClass`, `closeOnBackdrop`, `closeOnButton`, `mainContent`, `debounceMs`, `hideDelayMs`, `focusOnOpenSelector`, `lockScroll`, `trapFocus`, `closeOnEscape`, `role`, and `ariaModal`. `modelValue` defaults to `false`, `stylingClass` to `''`, and the two close-on-click props to `false`. Other defaults follow the core options, except `mainContent`, whose Vue prop defaults to `null` and is subject to the fallback described above.

For Nuxt or another SSR setup, import is safe but initialization is client-only. Use mounted/client-rendered code and wrap rendered dialogs in `<ClientOnly>` when the page is server-rendered.

## Styling

Import `dialog-lite/dialog-lite.css` once, as in the examples, or let `initDialogLite()` inject it. From a stylesheet, use:

```css
@import "dialog-lite/dialog-lite.css";
```

For Shadow DOM, use `injectDialogLiteCss({ target: shadowRoot })` with your shadow root, or use `dialogLiteCssText` in your own style setup.

The default CSS uses `.dialog-lite`, `.dialog-lite--in`, `.dialog-lite--out`, `.dialog-lite__backdrop`, `.dialog-lite__container`, `.dialog-lite__container-inner`, and `.dialog-lite-close-button`.

Override these CSS variables to change backdrop colors and stacking:

- `--z-index-dialog-lite`
- `--z-index-dialog-lite-backdrop`
- `--z-index-dialog-lite-container`
- `--c-dialog-lite-backdrop-in`
- `--c-dialog-lite-backdrop-out`

## License

MIT
