<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Frontend standards

## Frameworks

| Application | Framework | Output |
|---|---|---|
| `portfolio-site` | Astro | Static site; JavaScript only for interactive islands |
| `nasa-rag`, `outage-atlas` | React with Vite | Single-page applications served through CloudFront |

TypeScript in strict mode throughout. Avoid `any`. Use `@ts-expect-error` with a reason instead of `@ts-ignore`.

## Design system

One small design system, built in-house, gives every application a consistent look.

- **Tokens** are CSS custom properties for color, type scale, spacing, radii, shadows, and motion. Components use tokens only: no raw colors, pixel spacing, or font sizes outside token files.
- **Themes** override color tokens. Light and dark follow `prefers-color-scheme`.
- **Base styles** provide the reset, typography, focus styles, and layout primitives (stack, cluster, grid, container).
- **Components** are built once and extended with props and variants. Check for an existing component before creating one.
- Styles are scoped: `<style>` in Astro components, CSS Modules in React. No runtime CSS-in-JS and no utility CSS framework.

Tokens and base styles are framework-agnostic and shared by both applications. Components are framework-specific: Astro for static content, React for interactive features.

The design system starts in `portfolio-site` under `src/design-system/` and never imports application code. It moves to a shared, versioned package when the first React application begins frontend work.

## Approved dependencies

| Package | Scope |
|---|---|
| `astro` | `portfolio-site` |
| `react`, `react-dom` | `nasa-rag`, `outage-atlas`; `portfolio-site` islands through `@astrojs/react` |
| `vite` | `nasa-rag`, `outage-atlas` |
| `typescript` | All |
| `eslint`, `typescript-eslint`, `eslint-plugin-astro`, `eslint-plugin-jsx-a11y` | Linting |
| `prettier`, `prettier-plugin-astro` | Formatting |
| `vitest`, `@testing-library/react`, `@testing-library/user-event` | Component and unit tests |
| `@playwright/test` | Smoke tests |

Evaluate these by ADR when the need arises:

- Cognito sign-in for the React applications, such as the modular `aws-amplify/auth` entry point
- A headless, accessible primitives library for complex widgets (dialog, menu, combobox) instead of hand-rolled versions
- A Markdown renderer with sanitization for model output

Do not add:

- UI kits with their own visual style
- State management libraries before React state and context prove insufficient
- Date or utility libraries for what `Intl` and the language already provide

## Accessibility

WCAG 2.2 AA is the baseline.

- Use semantic HTML first and ARIA only when no native element fits.
- Every interactive element works from the keyboard and has a visible focus state.
- Contrast meets AA in both themes, and color is never the only signal.
- Honor `prefers-reduced-motion`.
- Images have meaningful `alt` text, or empty `alt` when decorative.
- Form fields have labels, and errors are associated with their fields.

## Performance

- `portfolio-site` scores 95 or higher in every Lighthouse category on mobile.
- React applications ship less than 150 KB of compressed JavaScript on first load and splits code by route.
- Images go through Astro's image pipeline with responsive sizes, modern formats, and explicit dimensions.
- Use a system font stack or at most two self-hosted, subset font files.
- Fingerprinted assets are cached as immutable. HTML is cached briefly and invalidated on deploy.

## Content

Projects and experience in `portfolio-site` are Astro content collections with typed schemas. Adding a project means adding a content file, not editing components.

## Structure and naming

- Components use PascalCase files (`ProjectCard.astro`, `ChatMessage.tsx`), one per file, with styles and tests alongside (`ChatMessage.module.css`, `ChatMessage.test.tsx`).
- Other modules use kebab-case files.
- Organize the React app by feature (`features/chat/`) rather than by file type.

## Testing

- Component tests find elements by role and accessible name, the way users do, not by class or test ID.
- Playwright smoke tests run against the test environment: pages load, navigation works, and, in the React applications, sign-in and the core workflow succeed.
- Static Astro content is covered by the build and smoke tests, not unit tests.
