
# Icons by name

**Breaking change.** `useIconResolver` — and with it `SvgIcon` and every prop
typed `string | ReactNode` — no longer falls back to the built-in `renderIcon`
registry. A string icon name is now resolved **only** through
`RenderIconByNameContext`. Without a provider the icon renders nothing and dev
builds log a warning.

### Why

The fallback was a static import. A name → component map cannot be tree-shaken,
so every app pulling in any KDS component that renders an icon also pulled in
~300 icon components — including apps that only ever pass icons as React nodes.

Turning the registry into a context dependency moves that cost to the apps that
actually need names, and lets them decide **when** it is loaded.

### Default: import the icon component

Nothing to configure, fully tree-shaken, type-checked:

```tsx
import CloseIcon from 'ks-react-design-system/assets/CloseIcon'

<Button iconRightComponent={<CloseIcon />} />
```

### Global registry

One-line restore of the previous behaviour — eager, the whole set lands in your
main chunk. Fine as a migration step:

```tsx
import RenderIconByNameContextProvider from 'ks-react-design-system/Contexts/RenderIconByName/RenderIconByNameContextProvider'

<RenderIconByNameContextProvider>
  <App />
</RenderIconByNameContextProvider>
```

Lazy variant — the icon set becomes its own chunk, fetched on the first named
icon:

```tsx
const CompleteSvgIconsByName = React.lazy(
  () => import('ks-react-design-system/Components/SvgIcon/experimental/CompleteSvgIconsByName'),
)

const iconRegistry = {
  renderIcon: (iconName, props) => (
    <React.Suspense fallback={null}>
      <CompleteSvgIconsByName iconName={iconName} {...props} />
    </React.Suspense>
  ),
}

<RenderIconByNameContext.Provider value={iconRegistry}>
  <App />
</RenderIconByNameContext.Provider>
```

Keep the context value referentially stable (module constant or `useMemo`).

### Ad-hoc, where the name comes from a CMS or a backend config

If names only appear in a few places, skip the global registry and resolve them
at the call site — the rest of the app stays free of it:

```tsx
const DynamicIcon = ({ iconName, ...rest }) => (
  <React.Suspense fallback={null}>
    <CompleteSvgIconsByName iconName={iconName} {...rest} />
  </React.Suspense>
)

<Tag iconComponent={<DynamicIcon iconName={cmsBlock.iconName} size="s" />} />
```

`CompleteSvgIconsByName` covers the full asset set and returns `null` for an
unknown name.

## CompleteSvgIconsByName

Render any icon from the complete icon set by name. Use this for dynamic icon rendering when icon names come from APIs or user input.

### Integration with RenderIconByNameContext

String icon names are resolved **only** through this context — see
[Icons by name](/docs/custom-icons-icons-by-name--docs) for the reasoning
and the ad-hoc alternative.

#### Recommended Approach: Using React.lazy

Wrap your app to enable rendering icons by name:

```tsx
import React from 'react'
import RenderIconByNameContext from 'ks-react-design-system/Contexts/RenderIconByName/RenderIconByNameContext'
// `baseRenderIcon` includes icons from the SvgIcon component and will be used to render previously available icons
import { renderIcon as baseRenderIcon } from 'ks-react-design-system/Components/SvgIcon/renderIcon'

// This component is large and will be lazy-loaded only when `baseRenderIcon` doesn't find the provided icon name
const CompleteSvgIconsByName = React.lazy(
  () => import('ks-react-design-system/Components/SvgIcon/experimental/CompleteSvgIconsByName')
)

export default function App() {
  return (
    <RenderIconByNameContext.Provider
      value={{
        renderIcon: (iconName, props) =>
          baseRenderIcon(iconName, props) || (
            <React.Suspense fallback={null}>
              <CompleteSvgIconsByName iconName={iconName} {...props} />
            </React.Suspense>
          ),
      }}
    >
      <YourApp />
    </RenderIconByNameContext.Provider>
  )
}
```

**Note**: Using `Suspense` enables code splitting for better performance. The complete icon set is only loaded when needed, not upfront.

### Alternative: Fast Integration (Not Recommended for Production)

```tsx
import React from 'react'
import RenderCompleteIconsByNameContextProvider from 'ks-react-design-system/Contexts/RenderIconByName/RenderCompleteIconsByNameContextProvider'

export default function App() {
  return (
    <RenderCompleteIconsByNameContextProvider>
      <YourApp />
    </RenderCompleteIconsByNameContextProvider>
  )
}
```

## Usage Examples

Icons from the context are now available for all components.

<Canvas of={CompleteSvgIconsByNameStories.WithRenderIconByNameContext} />