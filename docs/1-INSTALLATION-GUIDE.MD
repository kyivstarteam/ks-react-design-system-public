
# Installation Guide

How to consume KDS in a product app. Node `>= 20.19`, React `>= 18.2`.

## 1. Install

Releases are published in GitHub Packages as
`@kyivstarteam/ks-react-design-system-public` (registry
`https://npm.pkg.github.com`).


```bash
yarn add ks-react-design-system
```

`react` and `react-dom` are peer dependencies — your app provides them.

## 2. Import the styles

All component styles ship as a single stylesheet. Import it **once** at the app
entry point, otherwise components render unstyled.

```ts
// src/index.tsx
import 'ks-react-design-system/index.css'
import 'ks-react-design-system/Styles/fonts.css' // Averta Kyivstar @font-face
```

## 3. Wrap the app in `ThemeProvider`

`index.css` alone gives you the default light theme. `ThemeProvider` sets the
`data-theme` / `data-theme-mode` attributes that switch between `consumer`,
`business` and dark mode.

```tsx
import ThemeProvider from 'ks-react-design-system/Components/ThemeProvider'

const App = () => (
  <ThemeProvider theme="consumer" shouldApplyToBodyElement>
    <YourRoutes />
  </ThemeProvider>
)
```

## 4. Use components

Components come from the package root; icons are separate component imports.

```tsx
import { Button } from 'ks-react-design-system'
import CloseIcon from 'ks-react-design-system/assets/CloseIcon'

const SendButton = () => (
  <Button variant="primary" buttonSize="s" iconRight={<CloseIcon size="xs" />}>
    Send
  </Button>
)
```

A few APIs are not in the root barrel and are imported by path:

| API             | Import path                                       |
| --------------- | ------------------------------------------------- |
| `ThemeProvider` | `ks-react-design-system/Components/ThemeProvider` |
| `useKDSTheme`   | `ks-react-design-system/Hooks/useKDSTheme`        |
| `DataTableV2`   | `ks-react-design-system/Components/DataTableV2`   |

## 5. Design tokens

After step 2 every `--kds-*` custom property is available in your own CSS. SCSS
token sources and a typed theme object are also published:

```scss
@import 'ks-react-design-system/Styles/palette';
```

```ts
import { theme } from 'ks-react-design-system/Styles/theme'

theme.color.text.primary // 'var(--kds-color-text-primary)'
```

## Create React App notes

CRA 5 runs Jest 27, which is older than two things KDS depends on. Add both to
the consuming app or component tests will fail to run:

```json
// package.json — Jest 27 does not support the `imports` field used by @base-ui/react
{
  "jest": {
    "moduleNameMapper": {
      "^#prehydration/(.*)$": "<rootDir>/node_modules/@base-ui/react/internals/prehydrationScript.stub.js"
    }
  }
}
```

```ts
// src/setupTests.ts — jsdom has no ResizeObserver
class ResizeObserverMock {
  observe() {}
  unobserve() {}
  disconnect() {}
}

global.ResizeObserver = ResizeObserverMock as unknown as typeof ResizeObserver
```
