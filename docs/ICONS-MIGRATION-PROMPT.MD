
# Icon migration prompt for consumers

Copy the prompt below and run it with Claude Code (or any coding agent) from the
root of a consumer project. It migrates the project off string icon names,
`SvgIcon`, and the deprecated icon props onto the `*Component` props, reading
`node_modules` for the real prop and icon names.

````markdown
# Task: migrate off KDS string icons, `SvgIcon`, and deprecated icon props

This project consumes `ks-react-design-system` (KDS). Icons are now plain React
components. Migrate all call sites.

**Naming an icon with a string is an antipattern and must be avoided.** A string
name is resolved at runtime, is not type-checked, cannot be tree-shaken, and
silently renders nothing when the name is wrong or missing from the registry.
Every icon must be an imported component. The one narrow exception is a name
that arrives from the backend at runtime (case c) — and even there the string
never reaches a KDS prop, it goes through `DynamicIcon`.

Read the real package name from `package.json` and use it below as `<kds>`.
Treat `node_modules/<kds>` as the source of truth — read the `.d.ts` files, never
guess a prop or icon name. First check that
`node_modules/<kds>/Components/Button/index.d.ts` contains `iconLeftComponent`;
if not, stop and report that KDS must be upgraded.

## 1. Prop rename

Rule: **append `Component` to the deprecated icon prop** — `icon` →
`iconComponent`, `iconLeft` → `iconLeftComponent`, `menuIcon` →
`menuIconComponent`, `bulletIcon`, `simpleViewIcon`, `expandIcon`,
`collapseIcon`, `iframeOpenIcon`, `tooltipSettings.icon`, `inputProps.icon`, …
Also applies to nested objects (`Dropdown.list[]`, `Sidenav.list[]`, `Combobox`
options, `AppHeader`/`GlobalHeader` items, toolbar switcher options) and to your
own components that spread KDS `ButtonProps`/`InputProps`/`HyperlinkOwnProps`.

Two exceptions: on **buttons** (`Button`, `LinkButton`, `TextButton`,
`ElevatedButton`, `ProgressButton`) `icon` → `iconRightComponent`; on **`Tab`**
`icon` → `iconLeftComponent`.

New props take a React node only — never a string.

## 2. Value conversion

**a. Already a node** — rename only.
`<Button iconLeft={<CloseIcon />} />` → `<Button iconLeftComponent={<CloseIcon />} />`

**b. Literal string name** — import the icon component:
```tsx
- <Button iconRight="close" buttonSize="s" />
+ import CloseIcon from '<kds>/assets/CloseIcon'
+ <Button iconRightComponent={<CloseIcon size="s" />} buttonSize="s" />
```
Resolve name → component **only** from
`node_modules/<kds>/Components/SvgIcon/experimental/allAssetIcons.d.ts`
(`ICONS_MAP`: kebab name → `iconName`). Do not derive PascalCase yourself —
many differ (`subtract`→`ChartIcon`, `mobile`→`MobilePhoneIcon`,
`newspaper`→`HubNewspaperIcon`, `2-g`→`TwoGIcon`, `unlink`→`UnLinkIcon`,
`history`→`HistiryIcon`, `sim-change`→`Sim2dChangeIcon`,
`chevron-large-right`→`LargeChevronRightIcon`). Name not in the map → leave the
call site, list it in the report.

**c. Name comes from the backend** (CMS content, config/feature-flag JSON, API) —
the only case where a name stays a string, because the value is unknown at build
time. Do not use it for anything you could import instead. Add one shared module
and use it:
```tsx
// src/shared/DynamicIcon.tsx
import React from 'react'
import type { SvgIconBodyProps } from '<kds>/lib/Icon/SvgIconBody'

const CompleteSvgIconsByName = React.lazy(
  () => import('<kds>/Components/SvgIcon/experimental/CompleteSvgIconsByName'),
)

export type DynamicIconProps = { iconName: string } & SvgIconBodyProps

const DynamicIcon = ({ iconName, ...rest }: DynamicIconProps) => (
  <React.Suspense fallback={null}>
    <CompleteSvgIconsByName iconName={iconName} {...rest} />
  </React.Suspense>
)

export default DynamicIcon
```
```tsx
- <Tag icon={cmsBlock.iconName} />
+ <Tag iconComponent={<DynamicIcon iconName={cmsBlock.iconName} size="s" />} />
```
`CompleteSvgIconsByName` covers the full asset set (superset of the old
`renderIcon` registry) and returns `null` for unknown names; `React.lazy` keeps
it out of the main bundle. Never hardcode your own name→component map, never
import the icon set eagerly.

## 3. Remove `SvgIcon` and the name registry

`SvgIcon` is deprecated — no `import SvgIcon from '<kds>/Components/SvgIcon'`
may remain:
- `<SvgIcon icon="close" size="s" className={c} />` → `<CloseIcon size="s" className={c} />`
- `<SvgIcon icon={node} />` → render `node` directly (drop the wrapper `span`; if
  a selector depended on it, put the class on the icon)
- dynamic name → `<DynamicIcon iconName={name} … />`
- `fill` → `color`; `variant`/`disabled` props are dead, drop them

Also drop `useIconResolver`, `RenderIconByNameContextProvider` /
`RenderIconByNameContext`, and any local icon-name maps once nothing consumes
them. If your own components expose a `string | ReactNode` icon prop, change it
to `ReactNode` (or add a `*Component` prop and deprecate the old one if it is
public API).

## 4. Preserve appearance — four traps

1. **Size.** A component forwarded its `size`/`buttonSize` only to *named* icons;
   a node keeps its own size (default `m`). Set `size` explicitly when converting
   b/c (`<Button buttonSize="s">` → `size="s"`; `AccordionSingle` passed `s`/`xs`;
   `TextField`, `Chips`, `Price` pass their own). Check the `.d.ts`/source.
2. **Colour-baked names.** Legacy names that injected a colour need an explicit
   `color`: `success`/`i-success`→`CheckCircleIcon` `--kds-color-support-success`;
   `error`/`i-error`→`WarningCircleIcon` and `cancel-circle`→`CancelCircleIcon`
   `--kds-color-support-error`; `info-circle`→`InfoCircleIcon`
   `--kds-color-key-primary`; `i-info`→`InfoCircleIcon` `--kds-color-support-info`;
   `i-warn`→`WarningTriangleIcon` `--kds-color-support-warning`;
   `i-promo`→`GiftIcon` `--kds-color-support-promo`; `i-advice`→`BulbIcon`
   `--kds-color-support-neutral`; `old-text`→`OldTextIcon`
   `--kds-color-icon-secondary`; `checkbox`/`wallet`/`log` →
   `--kds-color-key-primary`; `logout`→`--kds-color-interactive-danger`;
   `cost`→`--kds-color-support-warning`; `debt`→`--kds-color-support-error`.
   In `AlertBanner`/`InfoBlock`, if the icon only reproduced the default icon for
   that `type`, drop the prop and let the component render its own `StatusIcon`.
3. **`.icon-name--{name}` is no longer emitted.** Grep your CSS/SCSS for
   `icon-name--` and rewrite those selectors against a class you pass via the
   icon's `className`.
4. **`viewBox`/`fill` set on the KDS component are ignored** for node icons —
   move them onto the icon element.

## Out of scope
KDS internals / `node_modules`; non-KDS icons (`<img>`, other libs,
`react-toastify`'s `icon`); props that merely contain "icon" (`iconPosition`,
`onIconClick`, `classes.icon`, `iconClassName`, `closeIconLabel`); unrelated
refactors or dependency bumps.

## Verify (report actual output)
typecheck, lint, tests; then grep for leftovers: `SvgIcon`, `useIconResolver`,
`RenderIconByName`, `iconLeft=`, `iconRight=`, `menuIcon`, `bulletIcon`,
`simpleViewIcon`, `expandIcon`, `collapseIcon`, `iframeOpenIcon`, bare `icon=` on
KDS components. Zero left, or justify each.

## Report
Files changed with call-site counts per case (a/b/c); every remaining string icon
name and the backend source that justifies it; unresolved icon names and why;
every explicit `size`/`color` added; CSS rewritten for `.icon-name--*`; anything
deliberately left alone.
````
