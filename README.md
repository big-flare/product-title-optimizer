# Product Title Optimizer

Optimize your Google Shopping product titles using real search data from your Google Ads account. This plugin pulls your actual products and search terms, researches your products and customers, and rewrites your product titles to maximise their traffic — then exports a ready-to-upload file for Google Merchant Center.

## What This Does

Your product titles are the #1 factor in whether your Shopping ads show up for the right searches. Most product titles are written for humans browsing a website, not for Google's matching algorithm. This plugin fixes that.

Here's what happens when you run it:

**1. Product Research** — Claude reviews your store's website and product pages to understand each product: what it is, who it's for, key benefits, and how your store positions it. This research is saved so you can browse and verify it.

**2. Keyword Research (4 sources)** — Claude builds a keyword pool for each product using four different sources:
- **Your actual search term data** from Google Ads — the real queries people typed that triggered your Shopping ads. This is the highest-quality keyword data because it's what your customers actually search for.
- **Google Keyword Planner** — Claude runs Keyword Planner research to discover high-volume keywords you might be missing, including terms your ads aren't currently showing for.
- **Your website** — Claude reviews your product pages and store website to understand how you describe and position your products, pulling keyword ideas from the language you and your customers already use.
- **AI-generated keywords** — Claude uses its understanding of your products to think of additional relevant keywords that searchers might use.

**3. Title Generation** — Using all that research, Claude rewrites each title to pack in as many high-quality, relevant keywords as possible while keeping it readable. Titles are structured so Google can match them to more searches.

Just say "optimize my product titles" and it does everything automatically — pulls your data, researches keywords, generates titles, creates the files. No downloading data, no fiddling with settings, no technical steps.

You get three files:
- **product-title-reference.xlsx** — Your optimized titles side-by-side with the originals. Review and edit here.
- **product-title-research.xlsx** — All the product research and keyword data behind each title. Browse this to see exactly what Claude found and why it chose the keywords it did.
- **product-title-feed.csv** — The final file you upload to Google Merchant Center as a supplemental feed.

A supplemental feed is a file you upload to Google Merchant Center that overrides specific product attributes (in this case, titles) without changing your main product feed. Your original titles stay untouched — the supplemental feed just tells Google to use the optimized versions in Shopping ads.

## What You Need

1. **Claude Desktop** (with Co-Work) — [Download here](https://claude.ai/download)
2. **A Google Ads account** with active Shopping campaigns
3. **5 minutes** for one-time setup

## Setup

### Step 1: Install this plugin

1. Download `product-title-optimizer.zip` from this repository (click "code" at top right > Download ZIP)
2. Open Claude Desktop and switch to **Co-Work** (top center)
3. Click **Customize** in the left sidebar
4. Under Personal plugins, click the **+** button
5. Select the zip file you downloaded
6. Click **Upload**

### Step 2: Connect your Google Ads account

The plugin includes a GoMarble connector that gives Claude access to your Google Ads data. GoMarble is free.

**First, set up GoMarble:**
1. Go to [apps.gomarble.ai](https://apps.gomarble.ai) and create a free account
2. Sign in with the Google account that has access to your Google Ads
3. Connect Google Ads as a data source inside GoMarble

**Then, connect it in Claude:**
4. In Claude Co-Work, go to **Customize** in the left sidebar
5. Find the Product Title Optimizer plugin
6. Click **Connectors**
7. Click **Connect** next to GoMarble
8. Authorize the connection
9. Done — Claude can now read your Google Ads data

## Usage

1. Start a new Co-Work task
2. Type: **"Optimize my product titles"**
3. Claude will pull your products, research keywords, and generate optimized titles
4. Open the reference Excel file on your Desktop to review the titles
5. Edit any titles you want to change directly in the spreadsheet
6. Go back to Claude and say **"export"**
7. Upload the CSV file to Google Merchant Center (instructions provided in the chat)

### Uploading to Google Merchant Center

After Claude generates the final CSV:

1. Go to **Google Merchant Center**
2. Click **Settings** (left sidebar) then **Data sources**
3. Click the **Supplemental sources** tab
4. Click **Add supplemental product data**
5. Keep "Add product data from a file" selected
6. Choose **Upload a file from your computer** and select `product-title-feed.csv`
7. Click **Continue**
8. Select your main product feed from the dropdown
9. Click **Create data source**

Your optimized titles will start appearing in Google Shopping within 24 hours.

## FAQ

**Will this change my actual product listings on my website?**
No. A supplemental feed only affects how your products appear in Google Shopping ads. Your website product pages stay exactly the same.

**How many products can it handle?**
Any size catalogue. For large catalogues (1,000+ products), it focuses on your top products by traffic first, since those have the most impact.

**Do I need to know how to code?**
No. Everything happens through a conversation with Claude. You just talk to it in plain English.

**Is GoMarble free?**
Yes. GoMarble provides free access to your Google Ads data through Claude.

**Can I edit the titles before uploading?**
Yes. The reference Excel file is specifically designed for you to review and edit. Change any title you want, then tell Claude to export the final version.

**How often should I re-run this?**
Every 1-3 months, or whenever you add significant new products. Search trends change over time, so refreshing your titles keeps them aligned with what people are actually searching for.
