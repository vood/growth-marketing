---
name: free-tool-research
description: >
  Research competitor free tools and generate ideas for free tools to drive organic SEO traffic.
  Identifies free tools that competitors use to attract organic search traffic, generates actionable
  tool ideas, and provides complete best practices for building SEO-optimized free tool pages with
  JSON-LD schemas, conversion strategy, and internal linking architecture.
metadata:
  author: vood
  version: "1.0"
compatibility: >
  Works with any web framework. Page-building best practices reference Next.js App Router patterns
  but the SEO strategy, JSON-LD schemas, and content guidelines apply universally.
---

# Free Tool SEO Research

Research competitor free tools and generate ideas for free tools to drive organic traffic.

## Trigger
Use when user says "research free tools", "free tool ideas", "SEO tools research", "/free-tool-research", or asks about what free tools competitors offer and what tools could be built for marketing.

## Goal
Identify free tools that competitors and adjacent companies use to attract organic search traffic, then generate actionable free tool ideas for our product that would rank well and convert visitors.

## Process

### Step 1: Understand the Product
Before researching, confirm the product context:
- What does the product do? (check CLAUDE.md, homepage, or ask the user)
- Who is the target audience?
- What are the core keywords/topics?

If working on Mentioned.to: the product is a Reddit marketing automation platform. Target audience is marketers, founders, and growth teams who want to monitor Reddit mentions, track competitors, and engage in relevant conversations.

### Step 2: Identify Competitors & Adjacent Companies
Search the web for:
1. **Direct competitors** — tools in the same space
2. **Adjacent tools** — related categories that share the same audience

For each, look for `/tools`, `/free-tools`, `/resources`, or similar sections on their websites.

### Step 3: Research Competitor Free Tools
For each competitor/adjacent company found, catalog their free tools:

| Company | Tool Name | URL | What It Does | Target Keyword | Est. Traffic |
|---------|-----------|-----|-------------|----------------|-------------|

Focus on tools that:
- Have their own landing page (good for SEO)
- Target a specific search query (e.g., "reddit username checker", "subreddit analyzer")
- Are simple enough to build as a single page
- Serve as a lead magnet (free value → signup)

### Step 4: Keyword Research for Tool Ideas
Search for queries like:
- "[topic] free tool"
- "[topic] calculator"
- "[topic] checker"
- "[topic] analyzer"
- "[topic] generator"
- "free [topic] tool online"
- "reddit [X] tool"

Use web search to find:
- Google autocomplete suggestions
- "People also ask" patterns
- Existing tools ranking for these terms
- Search volume indicators (competition level, number of results)

### Step 5: Generate Tool Ideas
For each idea, provide:

```
### [Tool Name]
- **Target keyword**: the primary search query this would rank for
- **What it does**: 1-2 sentence description
- **How it connects to our product**: why this attracts our ideal customer
- **Complexity**: Low / Medium / High (implementation effort)
- **Example competitors doing this**: who already has something similar
- **Differentiation angle**: what we'd do better or differently
```

### Step 6: Prioritize
Rank ideas by:
1. **SEO potential** — search volume and ranking difficulty
2. **Relevance** — how well it attracts our target user
3. **Build effort** — how quickly we can ship it
4. **Conversion path** — how naturally it leads to signup

Present a final prioritized list with top 5-10 recommendations.

## Output Format
Save research results to `research-output/free-tool-research.md` with:
1. Competitor free tool inventory
2. Keyword opportunities found
3. Prioritized tool ideas with full details
4. Recommended top 5 to build first

---

## Free Tool Page Best Practices

When building a free tool page, follow these guidelines. Every tool page should be a self-contained SEO landing page that ranks, converts, and links to the rest of the tool ecosystem.

### Page Anatomy (top to bottom)

1. **Breadcrumbs** — `Home > Free Tools > [Tool Name]` (with BreadcrumbList JSON-LD)
2. **Hero Section**
   - H1: clear, keyword-rich title (e.g., "Free Reddit Username Checker")
   - Subtitle: 1 sentence explaining value ("Check if any Reddit username is available, taken, or shadowbanned")
   - The tool itself (input + button) — immediately usable, no signup wall
3. **Tool UI** — the interactive tool, above the fold if possible
4. **Results area** — where output appears after running the tool
5. **How It Works** — 3-4 numbered steps with icons explaining the process (with HowTo JSON-LD)
6. **Key Features** — 4-6 feature cards explaining what makes this tool useful
7. **Who Is This For** — 3-5 audience personas (Marketers, Founders, Researchers, etc.)
8. **Educational Content** — 300-800 words of genuinely useful content about the topic. This is the SEO body. Use H2/H3 subheadings targeting related long-tail keywords.
9. **FAQ Section** — 5-8 questions (with FAQPage JSON-LD). Mix tool-specific Qs with topic Qs.
10. **Related Tools** — grid/cards linking to 3-6 other free tools (internal linking hub)
11. **CTA Banner** — soft conversion: "Want to automate this? Try [Product] free" with signup button
12. **Footer** — standard site footer

### Required JSON-LD Schema

Every tool page MUST include these structured data types:

```jsonld
// 1. WebApplication — tells Google this is a tool
{
  "@context": "https://schema.org",
  "@type": "WebApplication",
  "name": "Free Reddit Username Checker",
  "description": "Check if a Reddit username is available, taken, or shadowbanned.",
  "url": "https://mentioned.to/tools/reddit-username-checker",
  "applicationCategory": "UtilityApplication",
  "operatingSystem": "Any",
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  },
  "author": {
    "@type": "Organization",
    "name": "Mentioned.to"
  }
}
```

```jsonld
// 2. FAQPage — for the FAQ section
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How do I check if a Reddit username is taken?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Enter the username above and click Check..."
      }
    }
  ]
}
```

```jsonld
// 3. BreadcrumbList — for navigation
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://mentioned.to" },
    { "@type": "ListItem", "position": 2, "name": "Free Tools", "item": "https://mentioned.to/tools" },
    { "@type": "ListItem", "position": 3, "name": "Reddit Username Checker" }
  ]
}
```

```jsonld
// 4. HowTo — for the "How It Works" section
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Check a Reddit Username",
  "step": [
    { "@type": "HowToStep", "name": "Enter username", "text": "Type the Reddit username you want to check" },
    { "@type": "HowToStep", "name": "Click Check", "text": "Press the Check button to run the lookup" },
    { "@type": "HowToStep", "name": "View results", "text": "See if the username is available, taken, or banned" }
  ]
}
```

### SEO Checklist

- [ ] **Title tag**: "Free [Tool Name] — [Brand] | Check/Analyze/Generate [X] Online"
- [ ] **Meta description**: action-oriented, includes primary keyword, under 160 chars
- [ ] **H1**: matches target keyword exactly (one H1 per page)
- [ ] **URL**: `/tools/[keyword-slug]` — clean, short, keyword-rich
- [ ] **Open Graph tags**: title, description, image (for social sharing)
- [ ] **Canonical URL**: self-referencing canonical tag
- [ ] **Page speed**: tool loads under 2 seconds, no heavy JS bundles
- [ ] **Mobile-first**: tool is fully usable on mobile (inputs, buttons, results)
- [ ] **No signup wall**: tool works without login (collect emails optionally after results)
- [ ] **Internal links**: link to 3-6 related tools + main product pages
- [ ] **External links**: 1-2 authoritative references in educational content
- [ ] **Image alt text**: descriptive alt on any screenshots or diagrams
- [ ] **Sitemap**: tool page included in XML sitemap

### Content Guidelines

**Educational section (below the tool)**:
- Write for humans first, not Google — genuinely helpful content
- Target 2-3 secondary long-tail keywords in H2/H3 headings
- Include concrete examples, not generic filler
- Link to related blog posts or guides if they exist
- Keep paragraphs short (2-3 sentences max)

**FAQ section**:
- Lead with the most-searched question (check "People Also Ask")
- Mix question types: "How do I...", "What is...", "Why should I...", "Is it free..."
- Keep answers concise (2-4 sentences) but complete
- Include the target keyword naturally in at least 2 Q&A pairs
- Always include "Is this tool free?" → yes, reinforces the free positioning

### Conversion Strategy

- **No gate on the tool itself** — free tools must be free. No signup to use.
- **Soft CTA after results** — "Want to monitor this automatically? Try Mentioned.to"
- **Email capture** — optional: "Get results emailed to you" (collects leads)
- **Shareable results** — unique URL or downloadable output that earns backlinks
- **Upgrade nudge** — show what the paid product adds (e.g., "Track 50+ competitors continuously")

### Internal Linking Architecture

Build a **tools hub page** at `/tools` that links to all individual tool pages:
```
/tools (hub) ← high-authority page
  ├── /tools/reddit-username-checker
  ├── /tools/subreddit-analyzer
  ├── /tools/reddit-post-title-generator
  └── /tools/share-of-voice-calculator
```

Each tool page should:
- Link back to the hub (`/tools`)
- Link to 3-6 sibling tools ("Related Tools" section)
- Link to 1-2 relevant blog posts
- Link to the main product signup page (CTA)

This creates a topical cluster that signals authority to search engines.

### Technical Implementation Notes

- Use Next.js dynamic `<Script>` or `<script type="application/ld+json">` for JSON-LD
- Put JSON-LD in the page's `<head>` via `metadata` export or `generateMetadata`
- Validate all schema at https://validator.schema.org/ and Google Rich Results Test
- Use `generateMetadata()` for dynamic title/description per tool page
- Implement as static pages (SSG) where possible for speed
- Add loading states for tool results (skeleton, spinner) to avoid layout shift

## Tips
- Look at Product Hunt for inspiration on popular free tools
- Check Ahrefs/SEMrush free tool pages as best-in-class examples of this strategy
- Small, focused tools (one input, one output) tend to rank better than complex apps
- Tools that generate shareable outputs get natural backlinks
- Calculator and checker tools convert well because they imply the user has a problem to solve
- Best-in-class examples to study: Ahrefs (`/free-seo-tools`), SmallSEOTools, PrePostSEO, Omni Calculator
