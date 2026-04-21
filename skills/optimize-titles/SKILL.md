---
name: optimize-titles
description: Optimize Google Shopping product titles using real search data from Google Ads. Pulls products and search terms via GoMarble MCP, researches keywords across three sources, generates optimized titles with pipe-separated sections, and exports a supplemental feed CSV for Google Merchant Center. Triggers include "optimize titles", "product titles", "shopping titles", "title optimizer", "supplemental feed", "product feed".
---

# Product Title Optimizer

Optimize Google Shopping product titles using real search data. This skill pulls your products and search terms from Google Ads, runs keyword research across three sources, generates optimized titles, and exports files for review and upload to Google Merchant Center.

## Prerequisites

- **GoMarble MCP** must be installed and connected to the user's Google Ads account. If GoMarble tools are not available, inform the user they need to set it up first (see README).

## Process

### Phase 1: Connect and Pull Data

**1.1 Detect account**

Use GoMarble's `list_accounts` tool to find accessible Google Ads accounts.
- If **one account**: use it automatically, no need to ask.
- If **multiple accounts**: list them and ask the user which one to use.

Store the `customer_id` for all subsequent queries.

**1.2 Pull products**

Use GoMarble's `run_gaql` tool with the `shopping_product` resource (NOT `shopping_performance_view`):

```
SELECT
  shopping_product.item_id,
  shopping_product.title,
  shopping_product.brand,
  shopping_product.category_level1,
  metrics.impressions,
  metrics.clicks,
  metrics.conversions
FROM shopping_product
WHERE segments.date DURING LAST_30_DAYS
ORDER BY metrics.clicks DESC
```

**Why `shopping_product` instead of `shopping_performance_view`:** 
- `shopping_product` only returns products that currently exist in Merchant Center. `shopping_performance_view` returns historical data including products that have been removed from the feed — optimizing titles for delisted products is wasted work and will fail on upload.
- `shopping_product.item_id` returns the ID exactly as it appears in Merchant Center, with correct capitalisation. `segments.product_item_id` from `shopping_performance_view` may return different casing, causing "Offer does not exist" errors on upload.

**Deduplicate** by `item_id` — products may appear multiple times across channels/countries (e.g., AU feed + US feed). Keep the row with the highest clicks.

**1.3 Pull search terms**

Use GoMarble's `run_gaql` tool:

```
SELECT
  search_term_view.search_term,
  metrics.impressions,
  metrics.clicks,
  metrics.conversions
FROM search_term_view
WHERE segments.date DURING LAST_30_DAYS
  AND metrics.impressions > 10
ORDER BY metrics.impressions DESC
LIMIT 5000
```

**1.4 Ask user how many products to optimize**

Calculate what percentage of total clicks the top 50, 100, and 250 products represent. Present:

```
Found [N] products. Your top 50 account for [X]% of your Shopping clicks.

How many would you like to optimize?
1. Top 50 (recommended — covers most of your traffic)
2. Top 100
3. Top 250
4. All [N]
```

### Phase 2: Keyword Research

Build a keyword pool from three sources. Do all three before generating any titles.

**IMPORTANT — research at the CATEGORY level, not per-product.** For stores with large catalogues (100+ products), doing research per-product would take hours. Instead:
- Group products by `product_type_l1` or similar category field from the data
- Do website/page visits, Keyword Planner calls, and keyword generation once per category
- Then distribute category-level keywords to individual products based on relevance

For small catalogues (under ~50 products), per-product research is fine since the total work is manageable.

**Source 1: Product and website review**

First, try to review the store's **main website homepage** to understand the business, brand positioning, target audience, and product range. Then review **one product page per category** (pick the top product by clicks in each category). Don't visit every product page individually — for a store with 1,000 products across 15 categories, that's ~16 page visits total, not 1,000.

**If direct web fetching is blocked** (common in Co-Work due to network egress settings), fall back to **web search** instead. Search for the store name and key products to gather context about the business, brand, and product range. Don't get stuck trying to access blocked URLs — web search provides enough context for keyword research.

For each category, note:
- What the products in this category are and what they do
- Who they're for (target audience)
- Key benefits and features
- How the store positions them
- Common variant info (size, colour, weight, count)

For individual products within a category, use the product data from Google Ads (title, brand, product_type) to understand specifics without visiting each page. The category-level review gives you the context; the product data gives you the specifics.

From this understanding, generate keyword ideas across all three types:
- **Product keywords** — what someone searching specifically for this product type would type
- **Category/broader keywords** — what someone browsing the right category would type
- **Use case keywords** — how someone searching by use case would find these products

**Source 2: Search term data**

From the search terms pulled in Phase 1, identify which terms are relevant to each product. A search term is relevant if it relates to the product's type, brand, category, or use cases. This is data matching — no API calls needed. For each product, extract the top relevant search terms ranked by impressions.

**Source 3: Keyword Planner**

Use GoMarble's `run_keyword_planner` tool **once per product category**, not per product:
- `customer_id`: the account ID
- `keywords`: seed with the category's product type and top 3-5 search terms for that category
- `page_url`: the top product's landing page URL in that category (if available)

A store with 1,000 products across 15 categories = 15 Keyword Planner calls, not 1,000. This returns keyword ideas with average monthly searches and competition level. Prioritise by relevance x search volume. Distribute the results to all products in that category.

**Combine and deduplicate** all keywords from all three sources into a single prioritised list per product, ranked by quality (relevance x search volume). Category-level keywords apply to all products in that category; product-specific keywords (from search term matching and product data) are layered on top.

**Save the research document** — before generating titles, write `~/Desktop/product-title-research.xlsx` with two sheets:

Sheet 1 — **Product & Business Research**: Product ID, Product Title, Product URL, Product Summary, Target Audience, Key Benefits, Positioning Notes. **IMPORTANT: Every column must be filled in for every product. Do NOT leave Product Summary, Target Audience, Key Benefits, or Positioning Notes empty.** This sheet is the written output of the Source 1 product/website review — if these columns are empty, it means the review wasn't actually done. Fill them in using what you learned from reviewing the product pages and website. This is a deliverable the user will read.

Sheet 2 — **Keyword Research**: Product ID, Product Title, Keyword, Source (search_terms / keyword_planner / ai_generated), Monthly Searches, Clicks. One row per keyword per product. Sorted by product, then by monthly searches DESC.

Use Python with `openpyxl` to create the XLSX with professional formatting: branded headers (dark background, white text, bold), auto-fitted column widths, filters enabled, frozen header row.

### Phase 3: Generate Optimized Titles

For each product, write an optimized title using the keyword research from Phase 2. For large catalogues (100+ products), process titles in batches of 20-50 at a time rather than all at once.

**Title structure:** Multiple sections separated by `|`. Each section is a coherent phrase — not a run-on keyword list. Each pipe-separated section should read like something a person might actually type into Google.

**Two types of brand — store brand vs product brand**

- **Store brand** — the brand of the store selling the product (e.g., the retailer's name). Always goes in the LAST section of the title. When we say "brand" without qualification, we mean the store brand.
- **Product brand** — the manufacturer/maker of the product itself, available in `shopping_product.brand` from the Google Ads data. Only relevant when the store is a reseller of third-party brands.

Two cases to handle:

1. **Store sells its own products** (store brand = product brand, or products are unbranded). Only the store brand appears, in the last section.
2. **Store is a reseller of other brands** (product brand ≠ store brand). The product brand MUST appear in the **first half of the title** — ideally in the first or second section, next to the product identity. The store brand still goes LAST. Missing the product brand in this case is a serious issue: searchers looking for that brand won't match.

How to detect which case applies: inspect `shopping_product.brand` values across the pulled products. If the brand field is empty, consistently matches the store name, or is the same value for every product, it's case 1. If `shopping_product.brand` varies across products or differs from the store name, it's case 2 (reseller) — include the product brand in every title for those products.

**Section ordering (front to back):**
- **First section**: The product's core identity and the #1 bullseye keyword in exact word order. No variant info here. **If the store is a reseller (case 2 above), include the product brand here or in the second section at the latest.**
- **Next sections**: Additional high-quality keyword phrases. Front-load exact word order for high-priority keywords. Variant info (size, count, weight, colour) goes here. As you progress, use more efficient combined phrases that target multiple keywords at once.
- **Later sections**: Category/broader keyword phrases.
- **Last section**: Store brand name. ALWAYS last.

**Keyword efficiency:** A section can target multiple search queries at once by having the right words present. Example:

`Best Magnesium Spray for Sleep, Feet & Pain Relief`

This targets: "best magnesium spray", "magnesium spray for sleep", "magnesium spray for feet", "magnesium spray pain relief" — all from one section.

- **High-priority bullseye keywords**: exact word order, front sections.
- **Medium/lower-priority keywords**: combine efficiently, words represented but not necessarily exact order.

**Three keyword types (in priority order):**
1. **Product keywords** (highest) — people clearly searching for this product type. Go in first.
2. **Category/broader keywords** (second) — people in the right category. Fill in after product keywords.
3. **Use case keywords** (lowest) — how people use the product. Only if search volume supports it OR space remains after types 1 and 2.

**Rules:**
- Target 130-150 characters (min 130, max 150). Short titles waste keyword opportunity.
- Every section between pipes should read naturally.
- Section count varies per product — could be 3, could be 8.
- Each section can target one or multiple keyword phrases.
- Variant info examples (not exhaustive — use judgement):
  - Mattress → size (queen, king, single)
  - Supplements → capsule count or powder weight
  - Clothing → colour (use common names: "green" not "emerald")
  - Electronics → storage capacity, screen size, model year
  - Furniture → material and dimensions (e.g., "oak", "2-seater")
  - Only include if it matters to a searcher

**Handling variant-heavy catalogues:**

Many stores have the same base product in multiple variants (colours, sizes, styles). For example, "24K Gold Dipped Rose" in 30+ colours. This needs special handling:

- **Before generating titles**, identify products that share the same base product. Group them by stripping variant info (colour, size) from the title.
- **Preserve variant info exactly as it appears in the original title.** Parse the colour/size/style from the original title and carry it through to the optimized title. NEVER guess, reassign, or invent variant info — if the original says "Red Quartz", the optimized title must say "Red Quartz".
- **Use a consistent template within each variant group.** Products that share the same base should have the same title structure with only the variant swapped. This ensures uniqueness and avoids the need to individually craft 30+ similar titles.
- **Validate uniqueness after generating all titles.** Check for duplicates before presenting to the user. If two products have identical optimized titles, differentiate them using whatever makes them unique in the original title.
- **Variant info goes in the first section** (as part of the product identity), not buried later. Example: `24K Gold Dipped Rose - Red Quartz | ...` not `24K Gold Dipped Rose | Red Quartz | ...`

**Example titles (store brand is always the last section):**

Health supplements (store: Pure Minerals):
`Magnesium Oil Spray 200ml - 1 Bottle | Best Magnesium Spray for Sleep, Feet & Pain Relief | Natural Magnesium Oil for Kids & Adults | Pure Minerals`

Posture wear (store: Align Co):
`Back Brace for Posture | Full Back Support Brace & Straightener for Men and Women | Best Posture Corrector Vest for Back Pain Relief | Align Co`

Memorials (store: Eternal Rest):
`Platinum Elegance Urn for Ashes | Metal Adult Urns | Memorials For Your Loved One | Urns For Ashes | Cremation Urns | Eternal Rest`

Equestrian (store: Trail & Ride):
`Double T Barrel Style Saddle With Teal Gator Patchwork - 12 Inch | Best Western Saddles | Horse Saddles For Sale | Trail & Ride`

Furniture (store: Home Kit NZ):
`Raglan Bed Frame - Oak - Double | Wooden Double Bed Frame & Base with Headboard | Solid Oak Bedroom Furniture New Zealand | Home Kit NZ`

Equestrian reseller (store: Trail & Ride, product brand: Double T — a third-party saddle maker the store resells):
`Double T Barrel Style Saddle - 12 Inch | Best Western Saddles with Teal Gator Patchwork | Horse Saddles For Sale | Trail & Ride`

Note how the product brand "Double T" sits in the first section alongside the product identity, while the store brand "Trail & Ride" remains in the last section.

### Phase 4: Export and Review

**4.1 Save the reference XLSX**

Write `~/Desktop/product-title-reference.xlsx` with columns: Product ID, Original Title, Optimized Title, Character Count. The Product ID column uses `shopping_product.item_id` which already has the correct Merchant Center format from Phase 1.2.

Use Python with `openpyxl` for professional formatting:
- Branded headers (dark background, white text, bold)
- Auto-fitted column widths
- Filters enabled, frozen header row
- Conditional formatting: highlight Character Count cells outside 130-150 range (e.g., red background)
- The Optimized Title column should be editable — this is where the user makes changes

**4.2 Show a preview and point to the file**

In the conversation, show a before/after preview of the **top 10 products** only. Then point the user to the XLSX:

```
Done! I've saved two files to your Desktop:

1. product-title-reference.xlsx — Your optimized titles with before/after comparison
2. product-title-research.xlsx — Your product research and keyword analysis

Open product-title-reference.xlsx to review your new titles.
Edit any titles you'd like to change directly in the spreadsheet.
When you're happy, come back here and say "export" and I'll generate the final feed file for Google Merchant Center.
```

**4.3 Wait for the user to review**

The user opens the XLSX, reviews titles, edits any they want to change, then comes back and says "export" or "looks good" or similar.

**4.4 Generate the final supplemental feed CSV**

Read the edited `~/Desktop/product-title-reference.xlsx`. Use the Optimized Title column (which the user may have edited). Write `~/Desktop/product-title-feed.csv` with two columns: `id,title`. The IDs are already in the correct Merchant Center format from Phase 1.2 (`shopping_product.item_id`).

**IMPORTANT: After exporting the CSV, you MUST provide the Merchant Center upload instructions below. Do not skip this — the user needs to know how to upload the file.**

```
Exported to ~/Desktop/product-title-feed.csv ([N] products).

To apply these in Google Merchant Center:
1. Go to Google Merchant Center
2. Click Settings (left sidebar) → Data sources
3. Click the "Supplemental sources" tab
4. Click "Add supplemental product data"
5. Keep the default "Add product data from a file"
6. Select "Upload a file from your computer" and upload product-title-feed.csv
7. Click Continue
8. Select your primary data source (your main product feed) from the dropdown
9. Click "Create data source"

You'll see a summary screen confirming how many products matched. 
Your optimized titles will start appearing in Google Shopping within 24 hours.
```
