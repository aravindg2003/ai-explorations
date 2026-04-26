# Constellation DX Components UI Gallery — Complete Developer Guide

> **Scope:** Everything you need to understand, build, extend, test, and publish custom Pega Constellation DX components using this gallery as your reference implementation.

---

## Table of Contents

1. [What Is This Repository?](#1-what-is-this-repository)
2. [Architecture Overview](#2-architecture-overview)
3. [Repository Structure](#3-repository-structure)
4. [The Component Anatomy](#4-the-component-anatomy)
5. [Component Types Reference](#5-component-types-reference)
6. [Complete Component Catalogue](#6-complete-component-catalogue)
7. [config.json Deep Dive](#7-configjson-deep-dive)
8. [index.tsx Patterns](#8-indextsx-patterns)
9. [PCore & PConnect API Reference](#9-pcore--pconnect-api-reference)
10. [Styling with Styled-Components & Design Tokens](#10-styling-with-styled-components--design-tokens)
11. [Storybook: Stories & Docs](#11-storybook-stories--docs)
12. [Testing Strategy](#12-testing-strategy)
13. [Third-Party Library Guide](#13-third-party-library-guide)
14. [Shared Utilities](#14-shared-utilities)
15. [Toolchain & Scripts Reference](#15-toolchain--scripts-reference)
16. [Build, Publish & Deploy](#16-build-publish--deploy)
17. [Best Practices Checklist](#17-best-practices-checklist)
18. [Step-by-Step: Building a New Component](#18-step-by-step-building-a-new-component)
19. [Common Patterns & Code Recipes](#19-common-patterns--code-recipes)
20. [Troubleshooting & FAQ](#20-troubleshooting--faq)

---

## 1. What Is This Repository?

The **Constellation DX Components UI Gallery** is an official Pegasystems open-source collection of ~53 production-quality, ready-to-use custom UI components for the **Pega Constellation** front-end framework. Each component is a React component built on the Constellation design system and the Pega DX API.

### Key facts

| Item | Detail |
|---|---|
| Repository | `github.com/pegasystems/constellation-ui-gallery` |
| License | Apache 2.0 |
| Current version | 4.x (master branch) targets **Pega '25.1** |
| Node requirement | 24.x (npm 10.x+) |
| React version | 18.x |
| Cosmos (design system) | `@pega/cosmos-react-core` 8.x |
| Styled-components | 5.3.11 (exact — version mismatches cause runtime issues) |

### Version / Platform compatibility

| Gallery version | Pega platform | Branch |
|---|---|---|
| 1.x | Pega '23 | release/1.x.x |
| 2.x | Pega '24.1 | release/2.0 |
| 3.x | Pega '24.2 | release/3.0 |
| 4.x | Pega '25.1 | master |

### Deployment modes

Two RAP (zip) files are released for each version:

- **`ConstellationUIGallery_x_x_x.zip`** — Full standalone app ("Computerland") showcasing all components.
- **`ConstellationUIGallery_x_x_x_COMPONENTS_ONLY.zip`** — Only the Rule-UI-Component rules, no demo app.

---

## 2. Architecture Overview

```
Pega Platform (Runtime)
  └── Constellation Orchestration Layer
        ├── PCore  (global services: REST, data, events, containers, locale...)
        ├── PConnect  (per-component API: state, actions, field values...)
        └── withConfiguration()  ←── wraps every DX component
                    │
              Your Component (React TSX)
                    ├── @pega/cosmos-react-core  (Constellation UI primitives)
                    ├── styled-components + design tokens  (theming)
                    └── 3rd-party libs  (optional, MIT-compatible)
```

A **Constellation DX component** is a compiled React component stored as a `Rule-UI-Component` rule in the Pega platform. At runtime, the Constellation orchestration layer instantiates it, injects props via `withConfiguration`, and provides access to Pega state and services through the `getPConnect` prop and the global `window.PCore` object.

---

## 3. Repository Structure

```
constellation-ui-gallery-master/
├── src/
│   ├── components/                   # All 53 DX components
│   │   ├── Pega_Extensions_<Name>/   # One folder per component
│   │   │   ├── index.tsx             # Component implementation (required)
│   │   │   ├── config.json           # Pega Designer metadata (required)
│   │   │   ├── Docs.mdx              # Storybook documentation (required)
│   │   │   ├── demo.stories.tsx      # Storybook story (required)
│   │   │   ├── demo.test.tsx         # Jest unit tests (required)
│   │   │   ├── styles.ts             # Styled-components (optional)
│   │   │   ├── localizations.json    # i18n strings (optional)
│   │   │   └── utils.ts / hooks/     # Helpers (optional)
│   │   └── shared/
│   │       └── create-nonce.ts       # Webpack nonce injector (import in every component)
│   ├── GettingStarted.mdx            # Storybook landing page
│   ├── Libraries.mdx                 # Third-party library documentation
│   └── Support.mdx                   # Support & contribution info
├── .storybook/
│   ├── main.ts                       # Storybook webpack/addon config
│   ├── preview.tsx                   # Global decorators (theme, direction, locale)
│   ├── theme.ts                      # Custom Storybook UI theme
│   └── static/                       # Demo images, sample PDFs
├── .github/
│   ├── agents/
│   │   └── pega-dx-component-builder.agent.md   # AI agent definition
│   ├── skills/pega-dx-component-builder/
│   │   ├── SKILL.md                  # Component builder skill
│   │   ├── references/               # Official & repo guidance
│   │   └── assets/                   # Delivery checklist
│   └── workflows/                    # GitHub Actions CI/CD
├── package.json
├── build.config.json
├── jest.config.js
├── eslint.config.mjs
├── .stylelintrc.json
├── .prettierrc.json
├── Component_Build_Guide.md          # Quick reference for new components
├── best practices.md                 # Design & dev principles
└── README.md
```

---

## 4. The Component Anatomy

Every component folder follows exactly this convention:

### 4.1 Required files

| File | Purpose |
|---|---|
| `index.tsx` | React component implementation |
| `config.json` | Pega Designer metadata — properties, types, defaults |
| `Docs.mdx` | MDX documentation shown in Storybook |
| `demo.stories.tsx` | Storybook story for live preview |
| `demo.test.tsx` | Jest + React Testing Library unit tests |

### 4.2 Optional files (add only when justified)

| File | Purpose |
|---|---|
| `styles.ts` | Styled-components definitions |
| `localizations.json` | i18n string map (keys → translated labels) |
| `utils.ts` | Pure helper functions |
| `hooks/` | Custom React hooks |
| `components/` | Sub-components (for complex components like CPQTree) |
| `types.ts` | Shared TypeScript type definitions |
| `constants.ts` | Enumerated constants |

### 4.3 Naming rules

- **Folder name** must exactly match `config.json` → `name` and `componentKey`.
- **Pattern**: `Pega_Extensions_<PascalCaseName>` (e.g. `Pega_Extensions_KanbanBoard`)
- **Exported React component name**: `PegaExtensions<PascalCaseName>` (e.g. `PegaExtensionsKanbanBoard`)

---

## 5. Component Types Reference

The `type` field in `config.json` determines where the component can be used in Pega App Studio:

| Type | `subtype` values | Use case |
|---|---|---|
| `Field` | `Text`, `Integer`, `Decimal`, `DateTime`, etc. | Renders inside a form as a single data-bound field |
| `Template` | `FORM`, `DETAILS` | Replaces an entire section/view layout |
| `Widget` | `PAGE`, `CASE`, or `["PAGE","CASE"]` | Standalone card placed on a page or case view |

### Field components
Bind to a single Pega property. Must handle `readOnly`, `disabled`, `required`, `displayMode: 'DISPLAY_ONLY'`, `hasSuggestions`, `validatemessage`, `hideLabel`, `helperText`, and `testId`. They use `getPConnect().getActionsApi().updateFieldValue()` and `triggerFieldChange()` to write back to Pega state.

### Template components
Render their `children` (the fields inside the view) using Constellation layout primitives like `<Grid>`, `<Flex>`, `<FieldGroup>`. They call `getPConnect().getInheritedProps()` to get label/layout settings from the parent.

### Widget components
Self-contained UI panels. They load their own data (usually from a data page via `PCore.getDataApiUtils()` or `PCore.getDataPageUtils()`) and can create/edit cases. They do not bind to a single property.

---

## 6. Complete Component Catalogue

### 6.1 Field Components

| Component | Description | Key Props | Notes |
|---|---|---|---|
| `Pega_Extensions_MaskedInput` | Text input with IMask pattern masking (phone, SSN, IBAN, zip, credit card, IP) | `mask`, `label`, `value`, `hideLabel`, `disabled`, `readOnly`, `required` | Reference implementation for all field prop patterns |
| `Pega_Extensions_PasswordInput` | Password input with show/hide toggle and strength indicator | `label`, `value`, `hideLabel`, `disabled`, `readOnly` | Mirrors MaskedInput prop conventions |
| `Pega_Extensions_DateInput` | Enhanced date input with Pega integration | `label`, `value`, `hideLabel`, `disabled`, `readOnly` | Another canonical field pattern reference |
| `Pega_Extensions_StarRatingInput` | Star-based rating input (3, 4, or 5 stars) | `maxRating`, `label`, `value`, `hideLabel`, `readOnly` | Uses `<Rating>` from cosmos-react-core |
| `Pega_Extensions_RangeSlider` | Dual-thumb slider for selecting a min–max range | `min`, `max`, `step`, `minValueProperty`, `maxValueProperty` | Template subtype DETAILS |
| `Pega_Extensions_MarkdownInput` | Markdown-enabled text area with live preview | `label`, `value`, `hideLabel` | Uses `@pega/cosmos-react-rte` |
| `Pega_Extensions_SignatureCapture` | Touch/mouse signature pad saved as base64 data URL | `label`, `value`, `hideLabel`, `readOnly` | Uses `signature_pad` library |
| `Pega_Extensions_BarCode` | Barcode generator (multiple formats) and scanner | `barcodeType`, `value` | Uses `jsbarcode` |
| `Pega_Extensions_QRCode` | QR code generator from field value | `value`, `label` | Canvas-based rendering |
| `Pega_Extensions_CameraCapture` | Device camera → capture photo → attach to case | `buttonText` | Widget/CASE subtype |
| `Pega_Extensions_CheckboxRow` | Row of checkboxes bound to a list property | `label`, `value` | |
| `Pega_Extensions_CheckboxTrigger` | Checkbox that fires a Pega action/data page on toggle | `label`, `value` | |
| `Pega_Extensions_JapaneseInput` | IME-aware text input for Japanese characters | `label`, `value` | |
| `Pega_Extensions_BannerInput` | Combined banner + input field for inline data entry | `label`, `value`, `variant` | |
| `Pega_Extensions_ActionableButton` | Button that opens a local action modal | `label`, `localAction`, `value` | Reads `AVAILABLEACTIONS` from PCore |
| `Pega_Extensions_CaseReference` | Field linking to another case with optional preview | `label`, `selectionProperty`, `refCaseClassName`, `allowPreview` | |
| `Pega_Extensions_DisplayPDF` | Inline PDF viewer bound to a field value | `width`, `height`, `showToolbar`, `dataPage` | |
| `Pega_Extensions_StatusBadge` | Color-coded status badge with regex-based variant mapping | `inputProperty`, `infoStatus`, `warnStatus`, `successStatus`, `pendingStatus`, `urgentStatus` | Uses `<Status>` from cosmos |
| `Pega_Extensions_Meter` | Visual progress/value meter | `value`, `min`, `max` | |

### 6.2 Template Components

| Component | Description | Key Props | Notes |
|---|---|---|---|
| `Pega_Extensions_FormFullWidth` | Expands form to full container width with configurable columns | `NumCols`, `gridTemplateColumns`, `heading` | Template/FORM |
| `Pega_Extensions_FormWithVerticalStepper` | Multi-step form with a vertical navigation stepper sidebar | `NumCols`, `stepperPosition` ('left'/'right') | Includes `VerticalNavbar` and `ActionButtons` sub-components |
| `Pega_Extensions_FieldGroupAsRow` | Renders a field group horizontally in a single row | `heading` | Template/DETAILS |
| `Pega_Extensions_EditableTableLayout` | Embeddable page list as an editable table with add/delete rows | `getPConnect` | Uses `PCore.createPConnect` + `getListActions()` |
| `Pega_Extensions_CompareTableLayout` | Side-by-side comparison table (spreadsheet, financial, radio-card variants) | `displayFormat`, `selectionProperty`, `currencyFormat` | Supports selection via radio buttons |
| `Pega_Extensions_DynamicHierarchicalForm` | Dynamically builds hierarchical form from page structure | `refreshActionLabel`, `enableItemSelection`, `Tabs` | Template/DETAILS |
| `Pega_Extensions_DynamicTemplate` | HTML template with up to 6 configurable field slots (A–F) | `HTMLContent`, `A`–`F` | Template/DETAILS |
| `Pega_Extensions_HierarchicalFormAsTasks` | Hierarchical embedded pages shown as a task checklist | `heading` | Template/FORM |
| `Pega_Extensions_JawLayout` | Interactive collapsible "jaw" accordion layout | `heading` | Template/FORM |
| `Pega_Extensions_RatingLayout` | Template that renders a star/numeric rating display | `minWidth` | Template/DETAILS |

### 6.3 Widget Components

| Component | Description | Key Props | 3rd-party lib |
|---|---|---|---|
| `Pega_Extensions_KanbanBoard` | Drag-and-drop Kanban board from a Pega data page | `dataPage`, `groups`, `groupProperty`, `detailsDataPage`, `detailsViewName`, `createClassname` | `@hello-pangea/dnd` |
| `Pega_Extensions_GanttChart` | Gantt chart for project/task tracking | `dataPage`, `startDateFieldName`, `endDateFieldName`, `progressFieldName`, `categoryFieldName`, `defaultViewMode` | `gantt-task-react` |
| `Pega_Extensions_Calendar` | Month/week/day calendar view with event creation | `dataPage`, `dateProperty`, `defaultViewMode`, `nowIndicator`, `weekendIndicator` | `@fullcalendar/*` |
| `Pega_Extensions_Map` | ArcGIS map with markers, shapes, and free-form drawing | `heading`, `height`, `bFreeFormDrawing`, `bShowSearch`, `apiKey`, `locationInputType` | `@arcgis/core` |
| `Pega_Extensions_NetworkDiagram` | Interactive node-edge graph with auto-layout | `dataPage`, `selectionProperty`, `edgePath`, `showMinimap`, `showControls` | `reactflow`, `@dagrejs/dagre` |
| `Pega_Extensions_OrgBuilder` | Hierarchical organization chart | `dataPage`, `selectionProperty`, `referenceHeading`, `targetHeading` | `reactflow` |
| `Pega_Extensions_CardGallery` | Card gallery display of list data with create action | `dataPage`, `createClassname`, `rendering`, `detailsDataPage` | — |
| `Pega_Extensions_ChatGenAI` | Generative AI chat widget backed by a Pega data page | `dataPage`, `maxHeight`, `sendAllUserContext` | `@pega/cosmos-react-social` |
| `Pega_Extensions_CPQTree` | CPQ (Configure-Price-Quote) product hierarchy tree | `dataPage`, `childrenPropertyName`, `displayPropertyName`, `idPropertyName` | — |
| `Pega_Extensions_LangSwitch` | Language and timezone switcher | `Configuration`, `changeTimezone`, `compactView`, `persistChanges` | — |
| `Pega_Extensions_OAuthConnect` | OAuth2 connect/disconnect flow | `profileName`, `connectLabel`, `showDisconnect` | — |
| `Pega_Extensions_DisplayAttachments` | Case attachment viewer with download, lightbox, and tiles | `dataPage`, `categories`, `useLightBox`, `enableDownloadAll`, `displayFormat` | — |
| `Pega_Extensions_Scheduler` | Appointment/event scheduler | `dataPage` | — |
| `Pega_Extensions_TaskList` | CRUD task list with inline add/complete/delete | `dataPage`, `heading` | — |
| `Pega_Extensions_UtilityList` | Summary list of utility items from a data page | `dataPage`, `primaryField`, `secondaryFields`, `iconName` | — |
| `Pega_Extensions_TrendDisplay` | Trend/sparkline chart | `dataPage` | — |
| `Pega_Extensions_CaseLauncher` | Button to launch a new Pega case | `createClassname`, `label` | — |
| `Pega_Extensions_Banner` | Alert/notification banner loaded from a data page | `variant`, `dataPage`, `dismissible`, `dismissAction` | — |
| `Pega_Extensions_IframeWrapper` | Embeds an external URL in a sandboxed iframe | `url`, `height` | — |
| `Pega_Extensions_ImageCarousel` | Scrollable image carousel with navigation | `dataPage` | — |
| `Pega_Extensions_ImageMagnify` | Hover-magnified image viewer | `value` | `react-image-magnifiers` |
| `Pega_Extensions_Shortcuts` | Keyboard shortcut launcher for Pega actions | `names`, `pages`, `displayType` | — |
| `Pega_Extensions_AutoSave` | Invisible component — auto-saves a field on change | `propertyName` | — |
| `Pega_Extensions_SecureRichText` | Rich text editor with DOMPurify XSS sanitization | `label`, `value` | `dompurify`, `@pega/cosmos-react-rte` |

---

## 7. `config.json` Deep Dive

`config.json` is the Pega Designer metadata file. It drives the component's property panel in App Studio. The folder name, `name`, and `componentKey` **must match exactly**.

### 7.1 Top-level structure

```json
{
  "name": "Pega_Extensions_<ComponentName>",
  "label": "Human Readable Label",
  "description": "Short description shown in App Studio",
  "organization": "Pega",
  "version": "4.0.0",
  "library": "Extensions",
  "allowedApplications": [],
  "componentKey": "Pega_Extensions_<ComponentName>",
  "type": "Field",
  "subtype": "Text",
  "properties": [...],
  "defaultConfig": { ... },
  "buildDate": "2025-09-29T17:46:21.831Z",
  "infinityVersion": "25.1.0-95",
  "packageCosmosVersion": "8.4.1"
}
```

### 7.2 Property formats

Each entry in the `properties` array uses a `format` key to control the UI control shown in App Studio:

| `format` | Widget | Use |
|---|---|---|
| `TEXT` | Text input | Free-text string |
| `BOOLEAN` | Toggle/checkbox | True/false flags |
| `SELECT` | Dropdown | Constrained choice; requires `source: [{key, value}]` |
| `PROPERTY` | Property picker | Lets designer select a Pega property |
| `GROUP` | Collapsible group | Groups related props; contains nested `properties` |
| `CONTENTPICKER` | Content picker | For Template slots (child views/fields) |
| `INSTRUCTIONS` | Read-only text | Inline guidance text |
| `LABEL` | Section header | Visual separator; used for the standard "About" section |

### 7.3 Full property object schema

```json
{
  "name": "myProp",
  "label": "My Property",
  "format": "TEXT",
  "required": true,
  "defaultValue": "default",
  "helperText": "Shown below the field",
  "helperTextVisibility": "$this.someOtherProp != 'hidden'",
  "source": [
    { "key": "optA", "value": "Option A" },
    { "key": "optB", "value": "Option B" }
  ]
}
```

### 7.4 The mandatory "About" footer

Every component ends its `properties` array with these two label properties — they auto-populate with build metadata:

```json
{
  "format": "LABEL",
  "label": "About",
  "name": "PegaAboutLabel",
  "variant": "h3"
},
{
  "format": "LABEL",
  "label": "Pega_Extensions 4.0.4, Tue Apr 21 2026, 25.1.2-397",
  "name": "PegaAboutData",
  "variant": "primary"
}
```

### 7.5 Template slot properties

For Template components, child view slots are declared with a special `key` format like `A`, `B`, `C`, etc.:

```json
{
  "name": "A",
  "label": "Slot A",
  "format": "CONTENTPICKER",
  "allowCreatingGroup": true
}
```

---

## 8. `index.tsx` Patterns

### 8.1 Canonical Field component skeleton

```tsx
import { withConfiguration, Input, Text } from '@pega/cosmos-react-core';
import { useEffect, useState, useRef, type MouseEvent } from 'react';
import '../shared/create-nonce';

export type MyFieldProps = {
  getPConnect?: any;
  label: string;
  value?: string;
  helperText?: string;
  validatemessage?: string;
  hideLabel?: boolean;
  disabled?: boolean;
  readOnly?: boolean;
  required?: boolean;
  testId?: string;
  fieldMetadata?: any;
  additionalProps?: any;
  displayMode?: 'DISPLAY_ONLY' | '';
  hasSuggestions?: boolean;
};

export const PegaExtensionsMyField = (props: MyFieldProps) => {
  const {
    getPConnect,
    label,
    value,
    helperText = '',
    validatemessage = '',
    hideLabel = false,
    testId = '',
    fieldMetadata,
    additionalProps,
    displayMode,
    hasSuggestions,
  } = props;

  const pConn = getPConnect();
  const actions = pConn.getActionsApi();
  const propName = pConn.getStateProps().value;
  const maxLength = fieldMetadata?.maxLength;
  const hasValueChange = useRef(false);

  // Runtime coercion: Pega may pass 'true' as a string
  let { readOnly, required, disabled } = props;
  [readOnly, required, disabled] = [readOnly, required, disabled].map(
    (prop) => prop === true || (typeof prop === 'string' && prop === 'true'),
  );

  const [inputValue, setInputValue] = useState(value);
  const [status, setStatus] = useState(hasSuggestions ? 'pending' : undefined);

  useEffect(() => setInputValue(value), [value]);

  useEffect(() => {
    if (validatemessage !== '') setStatus('error');
    if (hasSuggestions) {
      setStatus('pending');
    } else if (!hasSuggestions && status !== 'success') {
      setStatus(validatemessage !== '' ? 'error' : undefined);
    }
  }, [validatemessage, hasSuggestions, status]);

  // Display-only mode
  if (displayMode === 'DISPLAY_ONLY') {
    return <Text>{value || ''}</Text>;
  }

  return (
    <Input
      {...additionalProps}
      label={label}
      labelHidden={hideLabel}        // Note: internal Cosmos prop is labelHidden
      info={validatemessage || helperText}
      data-testid={testId}
      value={inputValue}
      status={status}
      disabled={disabled}
      readOnly={readOnly}
      required={required}
      maxLength={maxLength}
      onChange={(e: MouseEvent<HTMLInputElement>) => {
        if (hasSuggestions) setStatus(undefined);
        setInputValue(e.currentTarget.value);
        if (value !== e.currentTarget.value) {
          actions.updateFieldValue(propName, e.currentTarget.value);
          hasValueChange.current = true;
        }
      }}
      onBlur={(e: MouseEvent<HTMLInputElement>) => {
        if (e.currentTarget.value !== value) {
          actions.updateFieldValue(propName, e.currentTarget.value);
        }
        if (!value || hasValueChange.current) {
          actions.triggerFieldChange(propName, e.currentTarget.value);
          if (hasSuggestions) pConn.ignoreSuggestion();
          hasValueChange.current = false;
        }
      }}
    />
  );
};

export default withConfiguration(PegaExtensionsMyField);
```

**Critical field component rules:**
- Keep `hideLabel` as the public prop name; pass `labelHidden={hideLabel}` to the Cosmos `<Input>` internally.
- Always coerce `readOnly`, `required`, `disabled` with the `.map(prop => prop === true || ...)` pattern.
- Use `getPConnect?: any` — do not define one-off local PConnect/PActionsApi interfaces.
- Always handle `displayMode === 'DISPLAY_ONLY'` by returning `<Text>`.
- Update `updateFieldValue` on `onChange`; trigger `triggerFieldChange` on `onBlur`.

### 8.2 Canonical Widget component skeleton

```tsx
import { withConfiguration, Card, CardHeader, CardContent, Text, Progress } from '@pega/cosmos-react-core';
import { useEffect, useState } from 'react';
import '../shared/create-nonce';

type MyWidgetProps = {
  heading: string;
  dataPage: string;
  getPConnect: any;
};

export const PegaExtensionsMyWidget = (props: MyWidgetProps) => {
  const { heading = '', dataPage = '', getPConnect } = props;
  const [data, setData] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!dataPage) return;
    const context = getPConnect().getContextName();
    (window as any).PCore.getDataApiUtils()
      .getData(dataPage, {}, context)
      .then((response: any) => {
        setData(response.data.data ?? []);
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, [dataPage, getPConnect]);

  return (
    <Card>
      <CardHeader>{heading && <Text variant='h2'>{heading}</Text>}</CardHeader>
      <CardContent>
        {loading ? <Progress placement='local' /> : /* render data */ null}
      </CardContent>
    </Card>
  );
};

export default withConfiguration(PegaExtensionsMyWidget);
```

### 8.3 Canonical Template component skeleton

```tsx
import { withConfiguration, Grid, FieldGroup } from '@pega/cosmos-react-core';
import '../shared/create-nonce';

type MyTemplateProps = {
  heading?: string;
  showLabel?: boolean;
  label?: string;
  NumCols?: string;
  children?: React.ReactNode[];
  getPConnect: any;
};

export const PegaExtensionsMyTemplate = (props: MyTemplateProps) => {
  const { children = [], NumCols = '1', label, showLabel, getPConnect } = props;
  const propsToUse = { label, showLabel, ...getPConnect().getInheritedProps() };
  const nCols = parseInt(NumCols, 10);

  return (
    <FieldGroup name={propsToUse.showLabel ? propsToUse.label : ''}>
      <Grid container={{ cols: `repeat(${nCols}, minmax(0, 1fr))`, gap: 2 }}>
        {children}
      </Grid>
    </FieldGroup>
  );
};

export default withConfiguration(PegaExtensionsMyTemplate);
```

### 8.4 Icon registration

Always import and register icons before use. Never use icon strings directly:

```tsx
import * as plusIcon from '@pega/cosmos-react-core/lib/components/Icon/icons/plus.icon';
import * as pencilIcon from '@pega/cosmos-react-core/lib/components/Icon/icons/pencil.icon';
import { registerIcon, Icon } from '@pega/cosmos-react-core';

registerIcon(plusIcon, pencilIcon);

// Usage
<Icon name='plus' />
```

### 8.5 The `withConfiguration` wrapper

Every component **must** be exported as `export default withConfiguration(YourComponent)`. This HOC connects it to the Constellation runtime and feeds it the props from the Designer config.

---

## 9. PCore & PConnect API Reference

Access the Constellation platform through these two APIs only. Never reach into internal/private Pega objects.

### 9.1 `getPConnect()` — per-component API

```ts
const pConn = getPConnect();
```

| Method | Returns | Purpose |
|---|---|---|
| `pConn.getActionsApi()` | ActionsAPI | Actions: updateFieldValue, triggerFieldChange, createWork, openLocalAction, saveAssignment, finishAssignment |
| `pConn.getStateProps()` | `{ value: string }` | Get the bound property name (used as key for `updateFieldValue`) |
| `pConn.getContextName()` | `string` | Current container context name |
| `pConn.getValue(path)` | `any` | Read a Pega property by path |
| `pConn.getLocalizedValue(key, ...)` | `string` | Localized string lookup |
| `pConn.getInheritedProps()` | `object` | Label, showLabel, and other parent-inherited settings |
| `pConn.getRawMetadata()` | `object` | The component's raw config metadata |
| `pConn.getContainerName()` | `string` | Container name (e.g. 'primary', 'workarea') |
| `pConn.getContainerManager()` | ContainerManager | Add/remove container items |
| `pConn.ignoreSuggestion()` | `void` | Dismiss pending AI suggestion status |
| `pConn.acceptSuggestion()` | `void` | Accept AI suggestion |
| `pConn.getCaseInfo()` | CaseInfo | Case-level information |
| `pConn.getListActions()` | ListActions | `insert()`, `deleteEntry()` for embedded page lists |
| `pConn.getValidationApi()` | ValidationAPI | Trigger/clear validation |

### 9.2 `window.PCore` — global platform services

```ts
const PCore = (window as any).PCore;
```

| Service | Method | Purpose |
|---|---|---|
| **Constants** | `PCore.getConstants().CASE_INFO.*` | `CASE_INFO_ID`, `ASSIGNMENT_ID`, `ACTIVE_ACTION_ID`, `AVAILABLEACTIONS`, etc. |
| **Data API** | `PCore.getDataApiUtils().getData(dataPage, payload, context)` | Fetch list data from a data page |
| **Data Page** | `PCore.getDataPageUtils().getPageDataAsync(dp, ctx, params, opts)` | Fetch single-page data asynchronously |
| **REST** | `PCore.getRestClient().invokeRestApi(endpoint, payload)` | Call Pega REST APIs (e.g. `'save'`, `'loadView'`) |
| **Messaging** | `PCore.getMessagingServiceManager().subscribe(filter, fn, ctx)` | Subscribe to case/assignment events |
| **Messaging** | `PCore.getMessagingServiceManager().unsubscribe(subId)` | Clean up subscription |
| **PubSub** | `PCore.getPubSubUtils().subscribe(event, fn, id)` | Global pub/sub events |
| **PubSub** | `PCore.getPubSubUtils().unsubscribe(event, id)` | Unsubscribe |
| **Events** | `PCore.getEvents().getCaseEvent()` | Case-level events (e.g. `ASSIGNMENT_SUBMISSION`) |
| **Locale** | `PCore.getLocaleUtils().getLocaleValue(key, category)` | Runtime locale/i18n lookup |
| **Container** | `PCore.getContainerUtils().getContainerItems(path)` | Get container items |
| **Container** | `PCore.getContainerUtils().updateCaseContextEtag(ctx, etag)` | Update case etag after save |
| **Cascade** | `PCore.getCascadeManager().registerFields(ctx, ref, fields, fn, id)` | Watch field changes (used by AutoSave) |
| **Cascade** | `PCore.getCascadeManager().unRegisterFields(...)` | Unregister field watcher |
| **Attachment** | `PCore.getAttachmentUtils().*` | Attachment upload/download |
| **Asset Loader** | `PCore.getAssetLoader().getModuleByType('ContextTree')` | Load modules like ContextTree |
| **Store** | `PCore.getStore().getState()` | Redux-like state store |
| **Env Info** | `PCore.getEnvironmentInfo().getOperatorIdentifier()` | Logged-in operator ID |
| **Components** | `PCore.getComponentsRegistry().getLazyComponent(name)` | Get a registered Constellation component |
| **Create PConnect** | `PCore.createPConnect(messageConfig)` | Create a new pConnect for embedded views (used in EditableTableLayout, Banner) |

### 9.3 Event subscription pattern (with cleanup)

```tsx
useEffect(() => {
  const caseID = getPConnect().getValue(
    (window as any).PCore.getConstants().CASE_INFO.CASE_INFO_ID
  );
  const filter = { matcher: 'TASKLIST', criteria: { ID: caseID } };
  const subId = (window as any).PCore.getMessagingServiceManager().subscribe(
    filter,
    () => { loadData(); },
    getPConnect().getContextName()
  );
  loadData();
  return () => {
    (window as any).PCore.getMessagingServiceManager().unsubscribe(subId);
  };
}, []);
```

### 9.4 Performing case actions

```tsx
// Open a local action in a modal
getPConnect().getActionsApi().openLocalAction('MyActionID', {
  caseID: value,
  containerName: 'modal',
  type: 'express',
  name: 'Action Name',
});

// Create a new case
getPConnect().getActionsApi().createWork('My-App-Work-TaskClass', {
  openCaseViewAfterCreate: false,
});

// Save and then open local action
await getPConnect().getActionsApi().saveAssignment(getPConnect().getContextName());
```

---

## 10. Styling with Styled-Components & Design Tokens

### 10.1 `styles.ts` — the standard pattern

```ts
import styled, { css } from 'styled-components';

const StyledWrapper = styled.div(() => {
  return css`
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;  /* Always rem, never px */
  `;
});

export default StyledWrapper;
```

### 10.2 Using the Cosmos theme

Use the `useTheme()` hook to access the full Pega design token set:

```tsx
import { useTheme } from '@pega/cosmos-react-core';

const theme = useTheme();
// Access tokens:
// theme.base.colors.neutral.light  →  CSS color
// theme.base.spacing[2]            →  spacing unit
// theme.base.typography.fontSize   →  font size
```

Or use it inside styled-components:

```ts
import styled from 'styled-components';

const StyledCard = styled.div`
  background-color: ${({ theme }) => theme.base.colors.neutral.light};
  padding: ${({ theme }) => theme.base.spacing[2]};
  border-radius: ${({ theme }) => theme.base.borderRadius};
`;
```

### 10.3 Rules

- **Always use `rem` units** for font sizes, padding, margin, gap.
- **Never hard-code colors** — use theme tokens or Cosmos component props.
- **No `px`** except for border widths (1px) or third-party library overrides.
- **No third-party design systems** (no Material UI, Ant Design, Bootstrap, etc.).
- **Theme-aware styling** means the component automatically adapts to all themes: Default, Dark, Mantis, Flame, HoneyFlower.

### 10.4 Theme tokens used in this gallery (examples)

```ts
theme.base.colors.neutral.light     // Card backgrounds
theme.base.colors.semantic.success  // Status colors
theme.base.spacing[1..8]            // Spacing scale
theme.base.borderRadius             // Rounded corners
theme.base.typography.fontFamily    // Font stack
```

---

## 11. Storybook: Stories & Docs

### 11.1 `demo.stories.tsx` — story structure

```tsx
import type { StoryObj } from '@storybook/react-webpack5';
import { PegaExtensionsMyComponent, type MyComponentProps } from './index';

// Default export = story metadata
export default {
  title: 'Fields/My Component',  // Category/Name path in Storybook sidebar
  component: PegaExtensionsMyComponent,
  argTypes: {
    // Hide internal/runtime-only props from controls
    getPConnect: { table: { disable: true } },
    fieldMetadata: { table: { disable: true } },
    additionalProps: { table: { disable: true } },
    displayMode: { table: { disable: true } },
    // Use select control for constrained props
    variant: {
      control: { type: 'select' },
      options: ['success', 'info', 'warning', 'urgent'],
    },
  },
  parameters: {
    a11y: {
      context: '#storybook-root',
      config: { rules: [{ id: 'autocomplete-valid', enabled: false }] },
    },
  },
};

// Mock PCore for Storybook (no real Pega runtime)
const setPCore = () => {
  (window as any).PCore = {
    getComponentsRegistry: () => ({ getLazyComponent: (f: string) => f }),
    getEnvironmentInfo: () => ({ getTimeZone: () => 'local' }),
  };
};

// Mock getPConnect — provide all methods the component calls
const setPConnect = () => ({
  getStateProps: () => ({ value: 'myPropName' }),
  getActionsApi: () => ({
    updateFieldValue: () => {},
    triggerFieldChange: () => {},
    openLocalAction: () => {},
    createWork: () => {},
  }),
  getContextName: () => 'primary',
  getValue: () => '',
  ignoreSuggestion: () => {},
  acceptSuggestion: () => {},
  setInheritedProps: () => {},
  resolveConfigProps: () => {},
});

type Story = StoryObj<typeof PegaExtensionsMyComponent>;

// Story factory pattern used across this gallery
const Demo = (inputs: Partial<MyComponentProps>) => ({
  render: (args: MyComponentProps) => {
    setPCore();
    return <PegaExtensionsMyComponent {...args} getPConnect={setPConnect} />;
  },
  args: inputs,
});

export const Default: Story = Demo({
  label: 'My Field',
  value: '',
  hideLabel: false,
  disabled: false,
  readOnly: false,
  required: false,
  testId: 'myfield',
});

export const ReadOnly: Story = Demo({ label: 'Read Only', value: 'Some value', readOnly: true });
```

### 11.2 `Docs.mdx` — documentation template

```mdx
import { Meta, Canvas, Story, ArgTypes } from '@storybook/addon-docs/blocks';
import * as DemoStories from './demo.stories';

<Meta of={DemoStories} />

# My Component

Brief description of what it does and when to use it.

## Configuration

Explain the key configuration options the designer needs to set.

## Examples

<Canvas of={DemoStories.Default} />

<Canvas of={DemoStories.ReadOnly} />

## Props

<ArgTypes of={DemoStories.Default} />
```

### 11.3 Storybook story categories (sidebar titles used in this gallery)

| Category | Components |
|---|---|
| `Fields/` | All Field-type components |
| `Templates/` | All Template-type components |
| `Widgets/` | All Widget-type components |
| `Getting Started` | Landing page |
| `Libraries` | Third-party library documentation |

### 11.4 Global Storybook configuration

The `.storybook/preview.tsx` wraps every story in:
- `<Configuration>` — applies the active Cosmos theme
- `<PopoverManager>`, `<Toaster>`, `<ModalManager>` — required Constellation context providers
- Direction control (LTR/RTL)
- Locale control
- Theme switcher (Default, Dark, Mantis, Flame, HoneyFlower)

---

## 12. Testing Strategy

### 12.1 Unit tests with Jest + React Testing Library

The standard pattern re-uses the Storybook story via `composeStories`:

```tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { composeStories } from '@storybook/react';
import * as DemoStories from './demo.stories';

const { Default } = composeStories(DemoStories);

test('renders My Component with default args', async () => {
  render(<Default />);
  expect(await screen.findByText('My Field')).toBeVisible();
  expect(await screen.findByTestId('myfield')).toBeVisible();
});
```

This approach avoids duplicating the mock setup and keeps tests in sync with stories.

### 12.2 Test focus areas

- **Rendering**: Does the component render without errors with default props?
- **Label visibility**: Is the label text displayed? Hidden when `hideLabel=true`?
- **Read-only display**: Is value shown as text (not an input) in `displayMode='DISPLAY_ONLY'`?
- **Interaction**: Does `onChange` / `onClick` fire the right actions?
- **Accessibility**: Are ARIA labels, roles, and keyboard navigation correct? (Handled by Storybook a11y addon automatically.)

### 12.3 Jest configuration highlights

From `jest.config.js`:
- Environment: `jsdom`
- Transform: `babel-jest` + `ts-jest` for TypeScript/TSX
- `setupFiles: ['./setupFiles.ts']` — initializes canvas mock (`jest-canvas-mock`)
- `setupFilesAfterEnv: ['./setupTests.ts']` — extends Jest matchers (`@testing-library/jest-dom`)
- Coverage: collected from `src/components/**/*.{ts,tsx}` (excluding tests/stories)

### 12.4 End-to-end / accessibility testing

```bash
npm run build-storybook     # Build static Storybook
serve -port 6006 storybook-static
npm run test-storybook      # Playwright + axe-playwright accessibility checks
```

---

## 13. Third-Party Library Guide

Only MIT (or compatible) licensed libraries are permitted except ArcGIS which requires a commercial license.

| Library | Used by | License | Notes |
|---|---|---|---|
| `@fullcalendar/core`, `daygrid`, `timegrid`, `react` | Calendar | MIT | 3 views: month, week, day |
| `@hello-pangea/dnd` | KanbanBoard | Apache 2.0 | Drag-and-drop; fork of react-beautiful-dnd |
| `@arcgis/core` | Map | Commercial | ArcGIS API key required; version 5.0 |
| `@dagrejs/dagre` | NetworkDiagram, OrgBuilder | MIT | Auto-layout for node graphs |
| `gantt-task-react` | GanttChart | MIT | Task-hierarchy Gantt with 5 view modes |
| `imask` (`imaskjs`) | MaskedInput | MIT | Pattern-based input masking |
| `jsbarcode` | BarCode | MIT | SVG/canvas barcode rendering |
| `polished` | GanttChart | MIT | Only `transparentize()` color utility used |
| `react-image-magnifiers` | ImageMagnify | MIT | Hover zoom magnification |
| `reactflow` | NetworkDiagram, OrgBuilder | MIT | Node-edge graph; 11.x |
| `signature_pad` | SignatureCapture | MIT | Touch/mouse signature canvas |
| `dompurify` | SecureRichText | Apache 2.0 | XSS sanitization for HTML content |
| `@pega/cosmos-react-social` | ChatGenAI | Pega | `<Message>`, `<TypeIndicator>` |
| `@pega/cosmos-react-rte` | SecureRichText, MarkdownInput | Pega | Rich text editor |
| `@pega/cosmos-react-work` | Various | Pega | Work/case UI components |

### Important: styled-components version lock

The repo uses **exactly `styled-components@5.3.11`**. Different major versions within the same document cause style injection conflicts. Do not upgrade without testing thoroughly.

---

## 14. Shared Utilities

### 14.1 `src/components/shared/create-nonce.ts`

Every component must import this file:

```ts
import '../shared/create-nonce';
```

It sets `__webpack_nonce__` from `window.__webpack_nonce__` when present. This is required for Content Security Policy (CSP) compliance in Pega environments that inject a nonce for inline scripts.

```ts
// create-nonce.ts
// @ts-ignore
if (window?.__webpack_nonce__) {
  // @ts-ignore
  __webpack_nonce__ = window.__webpack_nonce__;
}
```

### 14.2 Common utility patterns seen across components

**Loading data from a data page (list):**
```ts
PCore.getDataApiUtils()
  .getData(dataPage, { dataViewParameters: [{ pyID: caseKey }] }, contextName)
  .then((response: any) => {
    const items = response.data.data ?? [];
    setData(items);
  });
```

**Loading a single-page record:**
```ts
PCore.getDataPageUtils()
  .getPageDataAsync(dataPage, contextName, { ID: itemId }, { invalidateCache: true })
  .then((record: any) => { /* use record */ });
```

**Creating an embedded child component (for list rows):**
```ts
const messageConfig = {
  meta: props,
  options: {
    context: getPConnect().getContextName(),
    pageReference: 'caseInfo.content',
    referenceList: `.${embedDataRef}`,
    viewName: getPConnect().options.viewName,
  },
};
const c11nEnv = (window as any).PCore.createPConnect(messageConfig);
c11nEnv.index = rowIndex;
c11nEnv.getPConnect().getListActions().insert({}, rowIndex);
```

---

## 15. Toolchain & Scripts Reference

### 15.1 Development commands

| Command | What it does |
|---|---|
| `npm install` | Install all dependencies |
| `npm run start` | Start Storybook dev server on port 6006 |
| `npm run storybook-docs` | Start Storybook in docs-only mode |
| `npm run lint` | Run all linters in parallel (ESLint, Prettier, stylelint, tsc) |
| `npm run fix` | Auto-fix lint/format issues |
| `npm run test` | Run all Jest unit tests |
| `npm run coverage` | Jest with HTML coverage report (output: `coverage/`) |
| `npm run build-storybook` | Build static Storybook to `storybook-static/` |
| `npm run test-storybook` | Run Playwright a11y tests against built Storybook |

### 15.2 Component management commands (via `@pega/custom-dx-components`)

| Command | What it does |
|---|---|
| `npm run buildComponent` | Build one component |
| `npm run buildAllComponents` | Build all components |
| `npm run authenticate` | Authenticate against Pega server |
| `npm run publish` | Publish one component to Pega |
| `npm run publishAll` | Publish all components |
| `npm run validate` | Validate a single component |
| `npm run validateAll` | Validate all components |
| `npm run list` | List components |
| `npm run startDevServer` | Start hot-reload dev server for live testing |

### 15.3 Library management commands

| Command | What it does |
|---|---|
| `npm run createLib` | Create a new library in Pega |
| `npm run createLibVersion` | Create a library version |
| `npm run exportLibVersion` | Export library as archive |
| `npm run importLibVersion` | Import library archive |
| `npm run storeLibVersion` | Store library version |
| `npm run switchLibVersion` | Switch active library version |

### 15.4 Linting configuration

- **ESLint**: `eslint.config.mjs` — uses `@pega/eslint-config`, `eslint-plugin-react`, `eslint-plugin-react-hooks`, `eslint-plugin-jsx-a11y`, `eslint-plugin-import`, `eslint-plugin-jest`, `eslint-plugin-mdx`
- **Stylelint**: `.stylelintrc.json` — lints CSS-in-JS inside `.tsx/.ts` files
- **Prettier**: `.prettierrc.json` — code formatting
- **TypeScript**: `tsc --noEmit` — type checking only (not transpilation)
- **cspell**: `cspell.json` — spell-checking for code identifiers and comments

---

## 16. Build, Publish & Deploy

### 16.1 Prerequisites

1. Install the `keys/` folder provided by the Constellation DX Component Builder package.
2. Edit `tasks.config.json` with your Pega server details:
   ```json
   {
     "rulesetName": "MyRuleset",
     "rulesetVersion": "01-01-01",
     "server": "https://my-pega-server.com",
     "clientID": "my-oauth-client-id",
     "user": "admin",
     "password": "..."
   }
   ```

### 16.2 Build and publish flow

```bash
# 1. Lint and type-check
npm run lint

# 2. Run all tests
npm run test

# 3. Build all component bundles
npm run buildAllComponents

# 4. Authenticate against Pega
npm run authenticate

# 5. Publish all components
npm run publishAll
```

### 16.3 RAP file import (non-developer path)

1. Download the latest RAP zip from the GitHub Releases page.
2. In App Studio: **Application** → **Settings** → **Import**.
3. Choose `ConstellationUIGallery_x_x_x_COMPONENTS_ONLY.zip` to add just the components.
4. Or import the full `ConstellationUIGallery_x_x_x.zip` to get the demo "Computerland" application too.

### 16.4 Pega version compatibility

Always use the matching branch/version for your Pega release. Mixing versions is unsupported.

---

## 17. Best Practices Checklist

Use this before submitting any component:

### Structure & naming
- [ ] Folder named `Pega_Extensions_<n>` in `src/components/`
- [ ] `config.json` `name` and `componentKey` exactly match the folder name
- [ ] All required files present: `index.tsx`, `config.json`, `Docs.mdx`, `demo.stories.tsx`, `demo.test.tsx`
- [ ] Optional files (`styles.ts`, `localizations.json`) added only when needed

### Implementation
- [ ] Component exported as `export default withConfiguration(MyComponent)`
- [ ] TypeScript types defined for all props
- [ ] Functional component with hooks (no class components)
- [ ] `../shared/create-nonce` imported at the top of `index.tsx`
- [ ] Field components keep `hideLabel` as public prop; pass `labelHidden={hideLabel}` to Cosmos controls
- [ ] `disabled`, `readOnly`, `required` typed as `boolean` with string coercion (`prop === true || prop === 'true'`)
- [ ] `getPConnect` typed as `getPConnect?: any` (no duplicate local interface)
- [ ] `displayMode === 'DISPLAY_ONLY'` handled by returning `<Text>` for field components
- [ ] No direct DOM manipulation (use refs only for third-party library requirements)
- [ ] Icons imported from `@pega/cosmos-react-core/lib/components/Icon/icons/...` and registered with `registerIcon()`

### Styling
- [ ] `rem` units used throughout (no `px` for spacing/font-size)
- [ ] No hard-coded colors (use design tokens or Cosmos component props)
- [ ] `styled-components` used for custom styles (in `styles.ts`)
- [ ] No third-party design system imports (Material UI, Ant Design, etc.)
- [ ] Component tested with Default and Dark themes

### API usage
- [ ] PCore accessed only via `(window as any).PCore` or PConnect methods
- [ ] Event subscriptions cleaned up in `useEffect` return function
- [ ] Data page calls include error handling (`.catch()`)
- [ ] No invented Pega APIs — all verified against repo examples or official docs

### Testing & accessibility
- [ ] `npm run lint` passes with no errors
- [ ] `npm run test` passes for the new component
- [ ] WCAG 2.1 compliance — interactive elements have labels, keyboard navigation works
- [ ] RTL layout supported (use `useDirection()` or Flex/Grid for layout)
- [ ] Tested against all 5 built-in themes (Default, Dark, Mantis, Flame, HoneyFlower)

### Storybook
- [ ] Story uses `select` controls for constrained-value props
- [ ] `getPConnect`, `fieldMetadata`, `additionalProps`, `displayMode` hidden from controls panel
- [ ] `hideLabel: false` in story defaults (not `labelHidden`)
- [ ] Multiple stories documented in `Docs.mdx` under an `## Examples` section
- [ ] Accessibility (`a11y`) parameters configured in story

---

## 18. Step-by-Step: Building a New Component

Here is the complete workflow to create a new `Pega_Extensions_MyNewComponent`:

### Step 1 — Find your closest existing component

Pick the gallery component most similar to what you're building:

| What you're building | Reference component |
|---|---|
| Text input field | `MaskedInput` or `PasswordInput` |
| Display-only field | `StatusBadge` or `CaseReference` |
| Data-driven widget | `KanbanBoard` or `CardGallery` |
| Template/layout | `FormFullWidth` or `FieldGroupAsRow` |
| Table with rows | `EditableTableLayout` |
| Chart/diagram | `GanttChart` or `NetworkDiagram` |
| Action button | `ActionableButton` or `CaseLauncher` |

### Step 2 — Create the folder and files

```bash
mkdir src/components/Pega_Extensions_MyNewComponent
cd src/components/Pega_Extensions_MyNewComponent
touch index.tsx config.json Docs.mdx demo.stories.tsx demo.test.tsx styles.ts
```

### Step 3 — Write `config.json`

Copy the structure from your reference component. Update:
- `name` and `componentKey` → `"Pega_Extensions_MyNewComponent"`
- `label` → human-readable name
- `description` → one-sentence description
- `type` and `subtype` → match your component category
- `properties` → define the props you need

### Step 4 — Write `index.tsx`

Use the canonical skeleton from [Section 8](#8-indextsx-patterns) matching your type (Field, Widget, or Template). Follow these rules:
- Import `withConfiguration` and your Cosmos primitives from `@pega/cosmos-react-core`
- Import `'../shared/create-nonce'`
- Define a fully typed `Props` type
- Implement the component logic
- Export: `export default withConfiguration(PegaExtensionsMyNewComponent)`

### Step 5 — Write `styles.ts` (if needed)

```ts
import styled, { css } from 'styled-components';
export default styled.div(() => css`/* your styles in rem */`);
```

### Step 6 — Write `demo.stories.tsx`

1. Mock `PCore` (use the same mock from MaskedInput's story as a template).
2. Mock `getPConnect` with all methods your component calls.
3. Create a `Default` story with all props set to their defaults.
4. Add extra stories for key variants (ReadOnly, Disabled, with data, etc.).

### Step 7 — Write `Docs.mdx`

1. Add `<Meta of={DemoStories} />`.
2. Write a 2–3 paragraph description.
3. Document required Pega-side setup (data pages, classes, activities).
4. Add `<Canvas>` blocks for each story.
5. Add `<ArgTypes>` table.

### Step 8 — Write `demo.test.tsx`

Use `composeStories` to run the Default story. Test:
- Component renders without errors
- Key text is visible
- testId element is accessible
- Read-only variant shows correct display mode

### Step 9 — Validate

```bash
npm run lint       # Must pass with no errors
npm run test       # Your new test must pass
npm run start      # Verify it looks correct in Storybook
```

### Step 10 — Register (if required)

Check if `src/component-list.json` needs updating for the component to appear in the gallery. Treat `src/component-list.js` and `Pega_Extensions/` as generated outputs — do not hand-edit them.

---

## 19. Common Patterns & Code Recipes

### 19.1 Reading data from a Pega Data Page (Widget)

```tsx
useEffect(() => {
  if (!dataPage) return;
  const context = getPConnect().getContextName();
  const caseKey = getPConnect().getValue(
    (window as any).PCore.getConstants().CASE_INFO.CASE_INFO_ID
  );
  const payload = {
    dataViewParameters: [{ pyID: caseKey }],
  };
  (window as any).PCore.getDataApiUtils()
    .getData(dataPage, payload, context)
    .then((response: any) => {
      if (response.data?.data !== null) {
        setItems(response.data.data);
      }
      setLoading(false);
    })
    .catch(() => setLoading(false));
}, [dataPage, getPConnect]);
```

### 19.2 Updating a field value (Field)

```tsx
const pConn = getPConnect();
const actions = pConn.getActionsApi();
const propName = pConn.getStateProps().value;

// On change (real-time update)
actions.updateFieldValue(propName, newValue);

// On blur (trigger refresh/validation)
actions.triggerFieldChange(propName, newValue);
```

### 19.3 Loading a case detail view

```tsx
const loadDetails = async (id: string, classname: string) => {
  const context = getPConnect().getContextName();
  const messageConfig = {
    meta: {
      type: 'View',
      config: {
        template: detailsViewName,
        ruleClass: classname,
        showLabel: 'true',
        label: '',
      },
    },
    options: {
      contextName: context,
      pageReference: `caseInfo.content`,
      context,
    },
  };
  const c11nEnv = (window as any).PCore.createPConnect(messageConfig);
  const detailsDataResponse = await (window as any).PCore.getDataPageUtils()
    .getPageDataAsync(detailsDataPage, context, { pyGUID: id }, { invalidateCache: true });
  return c11nEnv.getPConnect().createComponent(messageConfig.meta);
};
```

### 19.4 Drag-and-drop (KanbanBoard pattern)

```tsx
import { DragDropContext, type DropResult } from '@hello-pangea/dnd';

const onDragEnd = (result: DropResult) => {
  const { destination, source, draggableId } = result;
  if (!destination) return;
  if (destination.droppableId === source.droppableId && destination.index === source.index) return;

  // Update state and call Pega API to persist
  updateGroupValue(draggableId, destination.droppableId, getPConnect);
};

return (
  <DragDropContext onDragEnd={onDragEnd}>
    {/* Droppable columns / Draggable items */}
  </DragDropContext>
);
```

### 19.5 Regex-based status variant mapping (StatusBadge pattern)

```tsx
const getVariant = (value: string, regexList: string[]) => {
  const variants = ['info', 'warn', 'success', 'pending', 'urgent'];
  let variant = 'info';
  regexList.forEach((regex, i) => {
    if (regex && new RegExp(regex, 'i').test(value)) {
      variant = variants[i];
    }
  });
  return variant as StatusProps['variant'];
};
```

### 19.6 Auto-save on field change (AutoSave pattern)

```tsx
useEffect(() => {
  const subId = Date.now();
  (window as any).PCore.getCascadeManager().registerFields(
    pConn.getContextName(),
    pConn.getPageReference(),
    [propertyName],
    () => {
      /* invoke PCore.getRestClient().invokeRestApi('save', payload) */
    },
    subId
  );
  return () => {
    (window as any).PCore.getCascadeManager().unRegisterFields(
      pConn.getContextName(),
      pConn.getPageReference(),
      [propertyName],
      saveAssignment,
      subId
    );
  };
}, [pConn, propertyName]);
```

### 19.7 Localization support

```tsx
// In component
const localizedLabel = getPConnect().getLocalizedValue(
  'My label key',
  undefined,
  `${getPConnect().getCaseInfo().getClassName().toUpperCase()}!VIEW!MYVIEW`
);

// localizations.json (for static i18n strings)
{
  "fields": {
    "Create new event": "Create new event"
  }
}
```

### 19.8 Dismissable case-wide action (Banner pattern)

This is the most complex pattern — used to trigger a Pega action from within a widget without a form submit:

```tsx
const c11nEnv = (window as any).PCore.createPConnect({
  meta: {
    config: { context: 'caseInfo.content', name: actionID },
  },
  options: {
    contextName: tmpContainerName,
    context: tmpContainerName,
    pageReference: 'caseInfo.content',
  },
});

c11nEnv.getPConnect().getActionsApi()
  .finishAssignment(c11nEnv.getPConnect().getContextName(), {
    outcomeID: '',
    jsActionQueryParams: {},
  })
  .then(() => {
    getPConnect().getContainerManager()
      .removeContainerItem({ target: 'app/primary', containerItemID: tmpContainerName });
  });
```

---

## 20. Troubleshooting & FAQ

### "Git needs to be installed" error when publishing

You downloaded the ZIP instead of cloning. Run `git init` in the project root to create the missing `.git` folder.

### styled-components version conflicts

Ensure your project uses **exactly** `styled-components@5.3.11`. Multiple versions in the same DOM cause style injection order issues and broken theming.

### Component not appearing in App Studio

1. Confirm `config.json` `name` and `componentKey` exactly match the folder name.
2. Check that `type` and `subtype` are valid values.
3. Ensure the component was successfully published via `npm run publishAll`.
4. Verify the RAP was imported into the correct application layer.

### `PCore is not defined` in Storybook

Storybook has no Pega runtime. Always mock `window.PCore` in your story's `render` function before rendering:
```tsx
(window as any).PCore = { /* minimal mock */ };
```

### Build fails with `@pega/cosmos-react-core` import errors

Ensure the cosmos library version in `package.json` matches the `packageCosmosVersion` in `config.json` (currently `8.17.2` for Pega 25.1).

### TypeScript errors on `window.PCore`

Cast as `(window as any).PCore` — the global PCore object is intentionally not typed in this codebase since it's injected by the Pega runtime.

### ArcGIS Map requires commercial license

The `@arcgis/core` library used in `Pega_Extensions_Map` requires an ArcGIS API key for production use. Provide your key via the `apiKey` component prop.

### `Styled-components` theming not applying

Ensure the component is rendered inside a `<Configuration>` provider (which wraps all Storybook stories via the global decorator). In production Pega, this is handled automatically by the Constellation runtime.

### Tests fail with canvas-related errors

The `jest-canvas-mock` package is initialized in `setupFiles.ts`. Ensure your jest config includes `setupFiles: ['./setupFiles.ts']`.

### Component works in Storybook but not in Pega

Common causes:
1. A PCore/PConnect API used in the component doesn't exist on the target Pega version.
2. A data page or case class expected by the component hasn't been created in the Pega application.
3. The component's `type`/`subtype` doesn't match where it was placed (e.g. a `Field` component placed in a Widget slot).
4. CSP nonce mismatch — ensure `../shared/create-nonce` is imported.

---

## Appendix A: Cosmos React Core — Most Used Components

From analysis of all 53 gallery components, these are the most frequently imported Cosmos primitives:

| Component | Uses | Purpose |
|---|---|---|
| `withConfiguration` | 34 | HOC required on every component |
| `Text` | 22 | Typography — body text, headings |
| `Button` | 18 | Action triggers |
| `Flex` | 17 | Flexbox layout container |
| `Icon` | 11 | Icon display (requires `registerIcon` first) |
| `registerIcon` | 9 | Register icon SVG modules |
| `Input` | 9 | Text input control |
| `Card` | 9 | Card container |
| `useTheme` | 8 | Access design tokens |
| `CardContent` | 8 | Card body area |
| `FormField` | 7 | Field wrapper with label |
| `FieldGroup` | 7 | Grouped fields with optional heading |
| `CardHeader` | 7 | Card title area |
| `Link` | 6 | Hyperlink |
| `FormControl` | 5 | Form control wrapper |
| `DateTimeDisplay` | 5 | Formatted date/time display |
| `Progress` | 4 | Loading spinner/indicator |
| `Grid` | 4 | CSS Grid layout |
| `Checkbox` | 4 | Checkbox input |

---

## Appendix B: PCore API Usage Frequency

From analysis across all components:

| PCore API | Call count | Use |
|---|---|---|
| `PCore.getConstants()` | 31 | Case info constants |
| `PCore.getDataApiUtils()` | 22 | List data page calls |
| `PCore.getPubSubUtils()` | 15 | Global events |
| `PCore.getSemanticUrlUtils()` | 14 | URL generation |
| `PCore.getLocaleUtils()` | 14 | Locale/i18n |
| `PCore.getEvents()` | 8 | Event constants |
| `PCore.getContextTreeManager()` | 7 | Context hierarchy |
| `PCore.getViewResources()` | 6 | View metadata |
| `PCore.getMessagingServiceManager()` | 6 | Case/assignment events |
| `PCore.createPConnect()` | 6 | Embedded component creation |

---

*Guide compiled from the `constellation-ui-gallery-master` repository (v4.0.3/4.0.4, April 2026, targeting Pega '25.1). For the latest updates, refer to the official repository at `github.com/pegasystems/constellation-ui-gallery`.*
