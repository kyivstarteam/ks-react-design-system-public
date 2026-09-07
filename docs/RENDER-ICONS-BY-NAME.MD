
# CompleteSvgIconsByName

Render any icon from the complete icon set by name. Use this for dynamic icon rendering when icon names come from APIs or user input.

## Integration with RenderIconByNameContext

String icon names are resolved **only** through this context — see
[Icons by name](/docs/custom-icons-icons-by-name--docs) for the reasoning
and the ad-hoc alternative.

### Recommended Approach: Using React.lazy

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