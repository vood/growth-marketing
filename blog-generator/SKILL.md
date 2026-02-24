---
name: blog-generator
description: >
  Generates a GitHub Action workflow that uses Claude Code to automatically write blog posts,
  commit them as PRs, and optionally generate hero images with Nano Banana (Gemini image generation).
  Sets up a complete automated content pipeline with prompt engineering, topic queues, and CI/CD.
metadata:
  author: vood
  version: "1.0"
compatibility: >
  Any GitHub repo with markdown or MDX blog content. Works with Next.js, Astro, Hugo, Jekyll,
  or plain markdown blogs. Optional: Gemini API key for hero image generation.
---

# Blog Post Generator

Set up a GitHub Actions workflow that uses Claude Code to automatically write blog posts and open PRs.

## Trigger

Use when user says "set up blog automation", "auto blog generator", "blog post github action", "automated content pipeline", "auto-generate blog posts", or asks about automating blog content creation with CI/CD.

## Goal

Create a complete GitHub Actions workflow that:
1. Runs on schedule (cron) or manual dispatch
2. Uses `anthropics/claude-code-base-action@beta` to generate a blog post
3. Optionally calls Nano Banana (`gemini-2.5-flash-image`) for hero image generation
4. Commits content to a new branch and opens a PR for review

## Process

### Step 1: Gather Requirements

Ask the user about their blog setup. Do not proceed until you have answers for at least items 1-4:

1. **Blog content directory** — where posts live (e.g., `content/blog/`, `src/posts/`, `app/blog/`)
2. **Frontmatter format** — what fields are required:
   - title, date, slug, description, image, tags, author, category, draft
   - Are there custom fields specific to their framework?
3. **Content framework** — what renders the blog:
   - Next.js MDX (`*.mdx` files)
   - Astro (`.md` or `.mdx`)
   - Hugo (`.md` with TOML/YAML frontmatter)
   - Jekyll (`.md` with YAML frontmatter)
   - Plain markdown
4. **Tone and style** — writing guidelines:
   - Formal / casual / technical / conversational
   - Target audience (developers, marketers, founders, general)
   - Typical post length (500-1000, 1000-2000, 2000+ words)
   - Any forbidden patterns (e.g., no fluff, no "in today's world")
5. **Hero image generation** — do they want Nano Banana image gen?
   - Requires `GEMINI_API_KEY` secret
   - Generates 16:9 PNG images for each post
6. **Schedule** — how often:
   - Weekly (e.g., every Monday at 9am UTC)
   - Daily
   - Manual only (`workflow_dispatch`)
   - Custom cron expression
7. **Topic source** — where topics come from:
   - Manual input via workflow dispatch
   - Topic queue file (`content/topics.yml`)
   - Keyword list that Claude picks from

### Step 2: Create the Prompt File

Create `.github/prompts/blog-post.md` containing the blog writing instructions for Claude.

This file is passed to `claude-code-base-action` via the `prompt_file` parameter and controls how Claude writes each post.

```markdown
# Blog Post Generator Prompt

You are writing a blog post for [PRODUCT/COMPANY].

## Audience
[Target audience description from Step 1]

## Tone & Style
- [Tone from Step 1: e.g., "Technical but accessible, conversational"]
- Write in active voice
- Use short paragraphs (2-3 sentences max)
- Include concrete examples and data where possible
- No filler phrases: never use "in today's world", "it's no secret that", "let's dive in", "without further ado"
- No generic intros — start with a hook or a specific claim

## Structure
1. **Title** — compelling, keyword-aware, under 60 characters
2. **Meta description** — action-oriented, under 155 characters
3. **Introduction** (2-3 paragraphs) — state the problem or insight, why it matters now
4. **Body sections** (3-5 H2 sections) — each section covers one key point with:
   - A clear H2 heading
   - Supporting evidence, examples, or code snippets
   - Subheadings (H3) where needed for scannability
5. **Conclusion** — summarize key takeaway, include a soft CTA
6. **Frontmatter** — must include all required fields

## Frontmatter Template
```yaml
---
title: "[Post Title]"
date: "[YYYY-MM-DD]"
slug: "[url-safe-slug]"
description: "[Meta description under 155 chars]"
image: "/blog/[slug]/hero.png"
tags: ["tag1", "tag2"]
author: "[default author]"
draft: false
---
```

## File Naming
Save the post as: `[CONTENT_DIR]/[YYYY-MM-DD]-[slug].mdx`

## SEO Guidelines
- Use the target keyword in the title, first paragraph, and at least one H2
- Include 2-3 internal links to other pages on the site
- Add 1-2 external links to authoritative sources
- Use descriptive alt text for any images referenced
- Keep the slug short and keyword-focused

## What to Read First
Before writing, read:
- `CLAUDE.md` for project-specific writing rules
- Recent posts in the content directory for style reference
- The topic brief (provided as input or from `content/topics.yml`)
```

Adapt this template based on the user's answers from Step 1. Replace all `[PLACEHOLDER]` values with their actual configuration.

### Step 3: Create the Hero Image Script (Optional)

If the user wants hero image generation, create `.github/scripts/generate-hero-image.mjs`:

```javascript
#!/usr/bin/env node

/**
 * Generate a hero image for a blog post using Nano Banana (Gemini image generation).
 *
 * Usage: node generate-hero-image.mjs <blog-post-path>
 *
 * Requires GEMINI_API_KEY environment variable.
 * Falls back gracefully if the key is not set.
 */

import { GoogleGenAI } from "@google/genai";
import { readFileSync, writeFileSync, mkdirSync } from "node:fs";
import { dirname, join, basename } from "node:path";

const postPath = process.argv[2];
if (!postPath) {
  console.error("Usage: node generate-hero-image.mjs <blog-post-path>");
  process.exit(1);
}

const apiKey = process.env.GEMINI_API_KEY;
if (!apiKey) {
  console.log("GEMINI_API_KEY not set — skipping hero image generation.");
  process.exit(0);
}

// Extract frontmatter from the blog post
const content = readFileSync(postPath, "utf-8");
const frontmatterMatch = content.match(/^---\n([\s\S]*?)\n---/);
if (!frontmatterMatch) {
  console.error("No frontmatter found in", postPath);
  process.exit(1);
}

const frontmatter = frontmatterMatch[1];
const title = frontmatter.match(/title:\s*"?([^"\n]+)"?/)?.[1] || "Blog Post";
const description =
  frontmatter.match(/description:\s*"?([^"\n]+)"?/)?.[1] || title;

// Generate the image
const ai = new GoogleGenAI({ apiKey });

const imagePrompt = [
  `Create a professional, modern blog hero image for an article titled "${title}".`,
  `The article is about: ${description}.`,
  "Style: clean, minimal, abstract or conceptual illustration.",
  "Use a professional color palette with good contrast.",
  "Do NOT include any text, words, letters, or typography in the image.",
  "Aspect ratio: 16:9, suitable for a blog header.",
  "The image should feel premium and editorial, like a top-tier tech blog.",
].join(" ");

console.log("Generating hero image for:", title);

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash-preview-05-20",
  contents: [{ role: "user", parts: [{ text: imagePrompt }] }],
  config: {
    responseModalities: ["TEXT", "IMAGE"],
  },
});

// Find the image part in the response
const imagePart = response.candidates?.[0]?.content?.parts?.find(
  (p) => p.inlineData?.mimeType?.startsWith("image/")
);

if (!imagePart) {
  console.error("No image generated in response.");
  process.exit(1);
}

// Determine output path from frontmatter image field or derive from post path
const slug = basename(postPath, ".mdx")
  .replace(/^\d{4}-\d{2}-\d{2}-/, "");
const imageDir = join(dirname(postPath), "..", "public", "blog", slug);
mkdirSync(imageDir, { recursive: true });
const imagePath = join(imageDir, "hero.png");

// Decode and save the image
const imageBuffer = Buffer.from(imagePart.inlineData.data, "base64");
writeFileSync(imagePath, imageBuffer);
console.log("Hero image saved to:", imagePath);

// Update frontmatter with the image path
const relativeImagePath = `/blog/${slug}/hero.png`;
const updatedContent = content.replace(
  /image:\s*"[^"]*"/,
  `image: "${relativeImagePath}"`
);
writeFileSync(postPath, updatedContent);
console.log("Updated frontmatter image path to:", relativeImagePath);
```

The script paths (image output directory, frontmatter image field) should be adapted to match the user's project structure from Step 1. The example above assumes a Next.js-style `public/` directory.

### Step 4: Create the GitHub Action Workflow

Create `.github/workflows/blog-generator.yml`:

```yaml
name: Blog Post Generator

on:
  schedule:
    # Runs weekly on Monday at 9:00 UTC — adjust to user preference
    - cron: "0 9 * * 1"
  workflow_dispatch:
    inputs:
      topic:
        description: "Blog post topic or title"
        required: false
        type: string
      generate_image:
        description: "Generate hero image with Nano Banana"
        required: false
        type: boolean
        default: true

permissions:
  contents: write
  pull-requests: write

concurrency:
  group: blog-generator
  cancel-in-progress: false

jobs:
  generate-post:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate blog post with Claude
        id: claude
        uses: anthropics/claude-code-base-action@beta
        with:
          prompt: |
            Write a new blog post.
            ${{ github.event.inputs.topic && format('Topic: {0}', github.event.inputs.topic) || 'Pick the next unwritten topic from content/topics.yml. If no topics file exists, write about a relevant topic for the product.' }}

            Follow the instructions in .github/prompts/blog-post.md exactly.
            Read existing posts for style reference before writing.
            Create the post file in the correct directory with proper frontmatter.
          prompt_file: ".github/prompts/blog-post.md"
          model: "claude-sonnet-4-6"
          max_turns: 15
          allowed_tools: "Read,Write,Edit,Bash(git:*),Bash(ls:*),Bash(date:*),Glob,Grep"
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: Find new blog post
        id: find-post
        run: |
          # Find the most recently created .mdx or .md file in the content directory
          NEW_POST=$(git diff --name-only --diff-filter=A HEAD | grep -E '\.(mdx?|md)$' | head -1)
          echo "post_path=$NEW_POST" >> "$GITHUB_OUTPUT"
          echo "Found new post: $NEW_POST"

      - name: Generate hero image (Nano Banana)
        if: |
          (github.event.inputs.generate_image == 'true' || github.event_name == 'schedule') &&
          steps.find-post.outputs.post_path != ''
        run: |
          npm install @google/genai
          node .github/scripts/generate-hero-image.mjs "${{ steps.find-post.outputs.post_path }}"
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

      - name: Create Pull Request
        if: steps.find-post.outputs.post_path != ''
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Extract slug from the post filename
          POST_FILE="${{ steps.find-post.outputs.post_path }}"
          SLUG=$(basename "$POST_FILE" | sed 's/\.[^.]*$//' | sed 's/^[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}-//')
          DATE=$(date +%Y-%m-%d)
          BRANCH="blog/${DATE}-${SLUG}"

          git checkout -b "$BRANCH"
          git add -A
          git commit -m "blog: add post — ${SLUG}"
          git push origin "$BRANCH"

          gh pr create \
            --title "Blog: ${SLUG}" \
            --body "$(cat <<'EOF'
          ## New Blog Post

          Auto-generated blog post via Claude Code.

          **Post**: \`${{ steps.find-post.outputs.post_path }}\`

          ### Checklist
          - [ ] Review content for accuracy
          - [ ] Check frontmatter fields
          - [ ] Verify internal/external links
          - [ ] Review hero image (if generated)
          - [ ] Preview in local dev environment

          *Generated by [blog-generator](https://github.com/vood/growth-marketing) workflow*
          EOF
          )" \
            --base main \
            --head "$BRANCH"
```

Adapt the workflow based on Step 1 answers:
- **Schedule**: replace the cron expression with the user's preference
- **Content directory**: adjust the `grep` pattern in the find-post step
- **Model**: `claude-sonnet-4-6` for cost efficiency, or `claude-opus-4-6` for higher quality
- **Hero image step**: remove entirely if user doesn't want image generation
- **Topic source**: modify the prompt to match user's topic strategy

### Step 5: Create CLAUDE.md Additions

Add blog-specific instructions to the project's `CLAUDE.md` (or create one if it doesn't exist). These instructions guide Claude when running in CI:

```markdown
## Blog Content Guidelines

When generating blog posts in CI:

### Writing Rules
- Write for [TARGET AUDIENCE] — assume they are [EXPERTISE LEVEL]
- Tone: [TONE FROM STEP 1]
- Length: [WORD COUNT RANGE] words
- Every claim needs evidence: a link, a stat, or a concrete example
- No filler paragraphs — every section must deliver value

### Forbidden Patterns
- "In today's fast-paced world..."
- "It's no secret that..."
- "Let's dive in / dive deep"
- "Without further ado"
- "In conclusion" (just conclude, don't announce it)
- "Game-changer", "revolutionize", "cutting-edge" (unless literally true)
- Starting paragraphs with "So," or "Now,"

### Required Frontmatter
Every post must include:
- title (under 60 chars)
- date (YYYY-MM-DD)
- slug (URL-safe, keyword-focused)
- description (under 155 chars, for meta description)
- image (path to hero image)
- tags (2-5 relevant tags)
- author

### Image Conventions
- Hero images: `/blog/[slug]/hero.png`
- Inline images: `/blog/[slug]/[descriptive-name].png`
- Always include alt text

### Internal Linking
- Link to at least 2 other pages on the site
- Link to relevant tool pages under `/tools/` when applicable
- Use descriptive anchor text (not "click here")
```

### Step 6: Create Topic Queue File (Optional)

If the user wants a topic queue, create `content/topics.yml`:

```yaml
# Blog Topic Queue
# Claude picks the next topic with status: pending
# After writing, Claude updates status to: written

topics:
  - title: "How to Monitor Reddit Mentions for Your Brand"
    keyword: "reddit brand monitoring"
    brief: "Guide for marketers on setting up Reddit mention tracking. Cover manual methods vs automated tools. Include real examples of brands responding to Reddit mentions."
    status: pending

  - title: "Share of Voice on Reddit: What It Is and How to Measure It"
    keyword: "reddit share of voice"
    brief: "Explain SOV in the context of Reddit discussions. How to calculate it, why it matters for competitive intelligence, and how to improve it."
    status: pending

  - title: "Reddit Marketing Strategy for B2B SaaS Companies"
    keyword: "reddit marketing b2b saas"
    brief: "Tactical guide for B2B SaaS companies. Which subreddits to target, how to engage authentically, measuring ROI. Include dos and don'ts."
    status: pending
```

Populate with topics relevant to the user's product and target keywords. Each topic should have:
- `title` — working title (Claude may refine it)
- `keyword` — primary SEO keyword to target
- `brief` — 2-3 sentences describing what the post should cover
- `status` — `pending` (unwritten), `written` (generated), `published` (live)

## Required Secrets

Remind the user to add these GitHub repository secrets:

| Secret | Required | Purpose |
|--------|----------|---------|
| `ANTHROPIC_API_KEY` | Yes | Powers Claude Code for blog writing |
| `GEMINI_API_KEY` | No | Powers Nano Banana for hero images |

GitHub's `GITHUB_TOKEN` is automatically available and handles PR creation.

## Verification

After creating all files, verify:
1. `.github/workflows/blog-generator.yml` — valid YAML, correct action references
2. `.github/prompts/blog-post.md` — all placeholders replaced with user's config
3. `.github/scripts/generate-hero-image.mjs` — correct paths for user's project structure
4. `CLAUDE.md` — blog guidelines added (not overwriting existing content)
5. `content/topics.yml` — valid YAML with relevant topics (if using topic queue)

Run the workflow manually via `workflow_dispatch` to test before relying on the schedule.

## Tips

- Start with `workflow_dispatch` only (no schedule) until you've reviewed a few generated posts
- Use `claude-sonnet-4-6` for cost-effective generation; switch to `claude-opus-4-6` for flagship content
- Keep the prompt file detailed — the more specific your instructions, the better the output
- Review and merge generated PRs promptly to keep the topic queue moving
- Add a `max_turns: 15` to prevent runaway generation; increase if posts are being cut short
- The hero image script uses `gemini-2.5-flash-preview-05-20` — update the model ID as newer versions release
- Consider adding a linting step (markdownlint, frontmatter validator) before PR creation
