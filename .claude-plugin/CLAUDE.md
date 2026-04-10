# Product Title Optimizer Plugin

This plugin optimizes Google Shopping product titles using real search data from Google Ads.

## Google Ads Connection (Bundled)

This plugin includes a **GoMarble MCP connector** that provides access to Google Ads data. When the user installs this plugin, they are prompted for their GoMarble API key.

To get a GoMarble API key (free):
1. Go to https://apps.gomarble.ai/setup/integrations
2. Sign in with the Google account that has access to Google Ads
3. Connect your Google Ads account
4. Generate and copy your API key

GoMarble tools available: `list_accounts`, `run_gaql`, `run_keyword_planner`

## What This Plugin Does

1. Pulls product data and search terms from Google Ads via GoMarble MCP
2. Researches keywords from three sources (AI product review, search terms, Keyword Planner)
3. Generates optimized product titles (pipe-separated sections, 130-150 chars, brand last)
4. Exports professionally formatted XLSX files for review
5. Generates a supplemental feed CSV for Google Merchant Center upload

## Commands

- `/product-title-optimizer:optimize-titles` — Run the full optimization process

## Output Files (saved to ~/Desktop/)

- `product-title-reference.xlsx` — Before/after comparison (review and edit here)
- `product-title-research.xlsx` — Full product and keyword research
- `product-title-feed.csv` — Supplemental feed for Merchant Center (generated after review)

## XLSX Generation

When generating XLSX files, use Python with `openpyxl`:

```python
import subprocess
subprocess.run(["pip", "install", "openpyxl"], capture_output=True)
```

Then use openpyxl to create formatted workbooks with branded headers, auto-fitted columns, filters, and frozen rows.
