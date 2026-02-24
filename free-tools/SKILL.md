---
name: free-tools
description: >
  Generate free interactive tool pages for Next.js websites to drive organic SEO traffic.
  Creates complete tool pages with forms, live results, loading states, SEO metadata, JSON-LD,
  rate limiting, and a hub page. Use when the user wants to build free tools, calculators,
  analyzers, or checkers for their website to attract organic traffic and convert visitors.
metadata:
  author: vood
  version: "1.0"
compatibility: >
  Requires Next.js 14+ with App Router, Tailwind CSS, and shadcn/ui components
  (Button, Input, Label, Skeleton). Optional: next-safe-action for server actions,
  recharts for data visualization.
---

# Free SEO Tool Pages Generator

Generate free interactive tool pages that drive organic search traffic and convert visitors into product users. Each tool works without login, returns live results, and funnels to the paid product.

## When to Use

Use this skill when the user wants to:
- Build free tools / calculators / analyzers for their website
- Create lead-generation tool pages for SEO
- Add interactive tool pages that rank for "[topic] + tool/calculator/checker" keywords
- Build a tools hub page with multiple free tools

## Architecture Overview

```
app/tools/
├── layout.tsx                          # Shared tools layout (header/footer)
├── page.tsx                            # Hub page listing all tools
└── [tool-slug]/
    ├── layout.tsx                      # Per-tool metadata + JSON-LD (Server Component)
    └── page.tsx                        # Tool UI + results ('use client')

components/tools/
├── tool-page-layout.tsx                # Wraps each tool page with standard sections
├── faq-section.tsx                     # FAQ accordion + FAQPage JSON-LD
├── cta-banner.tsx                      # Conversion CTA section
└── related-tools.tsx                   # Cross-linking grid

lib/rate-limit.ts                       # IP-based rate limiter (for server-action tools)
lib/actions/[tool-actions].ts           # Server actions (for API-backed tools)
```

## Step-by-Step Process

### Step 1: Gather Requirements

Ask the user:
1. What tools do they want to build? (calculators, analyzers, checkers, finders)
2. What data sources do they have? (APIs, databases, or pure client-side calculations)
3. What's the product name and CTA destination? (sign-up URL, booking URL)
4. What keywords are they targeting? (e.g., "reddit sentiment analyzer", "SEO audit tool")

### Step 2: Create Shared Components

Create these 4 components in `components/tools/`:

#### tool-page-layout.tsx
```tsx
// Props interface:
interface ToolPageLayoutProps {
  howItWorks: { title: string; description: string }[];
  features: { title: string; description: string }[];
  faqs: { question: string; answer: string }[];
  relatedTools: { slug: string; title: string; description: string }[];
  ctaHeadline?: string;
  ctaDescription?: string;
  children: React.ReactNode; // Hero + form + results go here
}
```

Renders in order:
1. `{children}` — hero, form, loading states, results
2. "How it works" — numbered steps in an `<ol>`
3. "What you'll get" — 2-column feature grid
4. FAQ section — collapsible accordion with JSON-LD injection
5. Related tools — 3-column grid linking to sibling tools
6. CTA banner — conversion section with primary + secondary buttons

#### faq-section.tsx
- Render FAQ items using `<details>/<summary>` elements
- Inject `FAQPage` JSON-LD via `<script type="application/ld+json">`
- Each FAQ item has a chevron icon that rotates on open

#### cta-banner.tsx
- Section with headline, description, and two buttons:
  - Primary: "Start Free Trial" → `/sign-up`
  - Secondary: "Book a Call" → booking URL
- Style: `rounded-2xl border border-border/60 bg-muted/30 px-6 py-6`

#### related-tools.tsx
- Grid of tool cards (responsive: 1 col → 2 col → 3 col)
- Each card: title + description + "Try it free →" arrow link
- Links to `/tools/{slug}`

### Step 3: Create Tools Layout

`app/tools/layout.tsx` — Server Component that provides:
- Header with logo, nav links (Tools, Pricing, Log in, Sign up), and CTA button
- Footer with copyright and navigation links
- Mirror the site's existing marketing layout if one exists

### Step 4: Create Hub Page

`app/tools/page.tsx` — defines a `TOOLS` array and renders a grid:

```tsx
const TOOLS = [
  { slug: 'tool-slug', title: 'Display Name', description: '1-2 sentence description' },
  // ...
];
```

Pattern:
- "FREE TOOLS" badge at top
- H1 with colored accent: `<span className="text-primary">Keyword</span>`
- Responsive grid of tool cards (2 columns on sm+)
- Each card links to `/tools/{slug}` with title, description, "Try it free →"

### Step 5: Create Individual Tool Pages

Each tool has two files:

#### layout.tsx (Server Component)
```tsx
export const metadata: Metadata = {
  title: 'Free [Tool Name] | [Brand]',
  description: 'SEO description with target keywords...',
  openGraph: { title, description, type: 'website' },
  alternates: { canonical: 'https://domain.com/tools/[slug]' },
};

// JSON-LD with @graph containing:
// 1. WebApplication (name, url, applicationCategory, offers: price '0')
// 2. BreadcrumbList (Home → Tools → Current Tool)
// 3. HowTo (steps matching the page's "How it works")
```

#### page.tsx ('use client')

Two patterns depending on data source:

**Pattern A: Client-Side Calculator** (no API needed)
```tsx
// useState for inputs, useMemo for calculations
// No server action, results compute instantly
// Show results section when all required inputs have values
```

**Pattern B: Server-Action Tool** (API-backed)
```tsx
// useAction from next-safe-action/hooks
const { execute, status, result } = useAction(serverAction);
const isLoading = status === 'executing';

// Form → Loading skeletons → Results
// Error handling: result.data.error (rate limit) or result.serverError
```

**Page anatomy for both patterns:**
1. Hero: "FREE TOOL" badge → H1 with colored span → description
2. Form section: inputs in a bordered card, full-width submit button
3. Loading state: Skeleton placeholders matching result layout
4. Results section: data cards, charts, tables, lists
5. Wrapped in `<ToolPageLayout>` with howItWorks, features, faqs, relatedTools

**Each page defines these constants:**
```tsx
const HOW_IT_WORKS = [
  { title: 'Step name', description: 'What happens in this step.' },
  // 3 steps
];
const FEATURES = [
  { title: 'Feature name', description: 'What this feature provides.' },
  // 4 features
];
const FAQS = [
  { question: 'Common question?', answer: 'Clear answer.' },
  // 4 FAQs with target keywords
];
const RELATED_TOOLS = [
  { slug: 'other-tool', title: 'Other Tool', description: 'Short description.' },
  // 2-3 related tools
];
```

### Step 6: Rate Limiting (for server-action tools)

Create `lib/rate-limit.ts` — in-memory token bucket:
- 5 tokens per IP per tool per hour
- Key format: `${ip}:${toolName}`
- Lazy cleanup of entries older than 2 hours
- Returns: `{ allowed: boolean, remaining: number, retryAfterMs: number }`

IP detection:
```tsx
async function getClientIp(): Promise<string> {
  const h = await headers();
  return h.get('x-forwarded-for')?.split(',')[0]?.trim() || h.get('x-real-ip') || 'unknown';
}
```

### Step 7: Server Actions (for API-backed tools)

Pattern using next-safe-action:
```tsx
export const actionName = actionClient
  .schema(z.object({ /* zod schema */ }))
  .action(async ({ parsedInput }) => {
    const ip = await getClientIp();
    const rateCheck = checkRateLimit(ip, 'toolName');
    if (!rateCheck.allowed) {
      return { error: `Rate limit exceeded. Try again in ${Math.ceil(rateCheck.retryAfterMs / 60000)} minutes.`, rateLimited: true };
    }

    // Use Promise.allSettled for resilience against API failures
    const [result1, result2] = await Promise.allSettled([apiCall1(), apiCall2()]);
    const data1 = result1.status === 'fulfilled' ? result1.value : [];

    return { /* processed results */ };
  });
```

### Step 8: Update Sitemap

Add all tool URLs to `sitemap.ts`:
```tsx
{ url: `${BASE_URL}/tools`, priority: 0.8 },
{ url: `${BASE_URL}/tools/[slug]`, priority: 0.7 },
```

### Step 9: Update Navigation

Add "Tools" link to the site's marketing header and footer navigation.

## Visual & Styling Conventions

- Container: `max-w-4xl mx-auto px-4 sm:px-6`
- Section spacing: `space-y-12` between major sections
- Cards: `rounded-2xl border border-border/60 bg-muted/30 px-6 py-6`
- H1: `text-4xl md:text-5xl font-bold leading-tight`
- H2: `text-2xl font-semibold`
- Badge: `text-xs font-semibold uppercase tracking-[0.2em] text-muted-foreground`
- Primary metrics: `text-3xl font-bold text-primary`
- Submit button: `<Button type="submit" size="lg" className="w-full">`
- Loading: `<Loader2 className="h-4 w-4 mr-2 animate-spin" />` in button
- Skeletons: `<Skeleton className="h-24 w-full rounded-xl" />`
- Errors: `<p className="text-sm text-destructive">{errorMessage}</p>`
- Links: External links get `target="_blank" rel="noopener noreferrer"`

## Cross-Linking Strategy

- Each tool's `RELATED_TOOLS` should reference 2-3 sibling tools
- If tool A links to B, tool B should link back to A
- Hub page links to all tools
- CTA banners link to sign-up and booking

## SEO Checklist

For each tool page verify:
- [ ] Title includes "Free" + tool name + brand
- [ ] Meta description includes target keyword
- [ ] Canonical URL is set
- [ ] JSON-LD has WebApplication, BreadcrumbList, HowTo, FAQPage schemas
- [ ] H1 uses target keyword with colored accent span
- [ ] FAQs include natural keyword variations
- [ ] Hub page links to all tools
- [ ] Sitemap includes all tool URLs
- [ ] Tools nav link added to site header/footer
