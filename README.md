# 📊 Active Data Callout System

## Overview

The **Active Data Callout System** injects live product engagement metrics — such as views, orders, units sold, and revenue — into content areas like product detail pages (PDPs) on Salesforce Commerce Cloud (SFCC).

These metrics are rendered as dynamic callouts (e.g., 🔥 *Hot Right Now*, 🛒 *Popular Pick*) that visually communicate product interest tiers. They're fully customizable, driven by backend logic and content asset markup, and optionally gated behind A/B tests.

This system supports:

- ✅ Backend-calculated thresholds using site preferences
- ✅ Metric-specific tier defaults (label + emoji)
- ✅ Full override flexibility via HTML attributes
- ✅ Animation and styling for attention
- ✅ Graceful fallback behavior to avoid rendering issues

## ✅ Key Features

- **Metric Flexibility**  
  Supports multiple product-level metrics:
  - `weeklyViews`
  - `weeklyOrders`
  - `weeklyUnitsSold`
  - `weeklyRevenue`

- **Backend-Driven Tiers**  
  Tiers (`low`, `medium`, `high`) are assigned using site preference thresholds or asset-specific overrides.

- **Fully Customizable Content**  
  Authors can override:
  - Tier thresholds (`data-low`, `data-medium`, `data-high`)
  - Labels (`data-label`, `data-label-high`, etc.)
  - Emojis (`data-emoji`, `data-emoji-low`, etc.)

- **Dynamic Frontend Rendering**  
  Callouts are populated client-side using JSON injected on page load.

- **Safe Fallbacks**  
  If data is missing or malformed, callouts are hidden automatically to avoid UI issues.

- **A/B Test Ready**  
  Optional gating via SFCC A/B testing campaigns.

## 🧠 System Architecture & Data Flow

The Active Data Callout system is composed of three main layers:

### 1. **Backend Formatter (`getFormattedMetrics`)**

- Reads `product.activeData` fields for supported metrics.
- Looks up tier thresholds from **site preferences**.
- Calculates tier (`low`, `medium`, `high`) based on values.
- Formats numbers (e.g., `1200` → `1,200`) for display.
- Injects results into `pdict.metricsJSONString` as JSON.

```json
{
  "weeklyViews": {
    "value": "1,200",
    "tier": "medium",
    "thresholds": {
      "low": 100,
      "medium": 200,
      "high": 400
    }
  }
}
```

### 2. **Controller Integration (`Product-Show`)**

- Checks for `req.querystring.pid` and gets the product.
- Calls `getFormattedMetrics(product)` and assigns it to `pdict.metricsJSONString`.
- Also sets `pdict.activeDataJSONString` for debugging (optional).

```html
<div id="metrics-json" style="display: none;">
  ${pdict.metricsJSONString}
</div>
```

### 3. **Frontend Renderer (`productDetail.js`)**

- Parses `#metrics-json` content on DOM ready.
- Loops through all `.social-proof` blocks.
- Matches `data-key` to the metric JSON.
- Applies tier using:
  - Inline thresholds if provided (`data-low`, etc.)
  - Fallback to site thresholds from backend
- Replaces template placeholders:
  - `${value}`, `${tier}`, `${label}`, `${emoji}`
- Removes the block if the metric is invalid or tier is `no-data`.

## 📏 Supported Metrics & Thresholds

The system supports four primary product engagement metrics, each with its own default threshold configuration and logic.

| Metric Key         | Source Field     | Site Preference Keys                          |
|--------------------|------------------|-----------------------------------------------|
| `weeklyViews`      | `viewsWeek`      | `viewsThresholdLow/Medium/High`               |
| `weeklyOrders`     | `ordersWeek`     | `ordersThresholdLow/Medium/High`              |
| `weeklyUnitsSold`  | `unitsWeek`      | `unitsThresholdLow/Medium/High`               |
| `weeklyRevenue`    | `revenueWeek`    | `revenueThresholdLow/Medium/High`             |

### 📦 Sample Output (Backend JSON)

```json
{
  "weeklyRevenue": {
    "value": "3,500",
    "tier": "medium",
    "thresholds": {
      "low": 1000,
      "medium": 3000,
      "high": 5000
    }
  }
}
```

### ⚙️ Fallback Defaults

If no site preference is set, the system uses hardcoded defaults in the helper:

- **weeklyViews**: 11 / 31 / 66
- **weeklyOrders**: 5 / 10 / 20
- **weeklyUnitsSold**: 15 / 40 / 80
- **weeklyRevenue**: 1000 / 3000 / 5000

## 🎨 Frontend Behavior

The frontend script runs when the DOM is ready and renders `.social-proof` blocks using the injected JSON metrics.

### 🧪 Rendering Flow

1. Reads and parses the hidden `#metrics-json` block.
2. For each `.social-proof` element:
   - Retrieves `data-key` and matches it to a metric in the JSON.
   - Parses the raw number value from `metric.value`.
   - Resolves thresholds from:
     - `data-low`, `data-medium`, `data-high` attributes (if present), **OR**
     - `metric.thresholds` from backend.
   - Determines tier based on thresholds.
   - Picks label and emoji using:
     - `data-label-*` or `data-emoji-*` overrides
     - Or metric-specific defaults for the current tier.
   - Replaces template placeholders: `${value}`, `${tier}`, `${label}`, `${emoji}`.
3. If the value is invalid or tier is `no-data`, the block is removed.

### ✅ Example Tier Logic (in JS)

```js
if (rawNumber >= high) {
  tier = 'high';
} else if (rawNumber >= medium) {
  tier = 'medium';
} else if (rawNumber >= low) {
  tier = 'low';
} else {
  tier = 'no-data';
}
```

### 🧼 Fallback Handling

Block is removed if:

- `data-key` is missing or invalid.
- Metric value is not a valid number.
- Tier is `no-data`.
- JSON block is missing or malformed.

This ensures clean, error-free rendering — only valid callouts appear on the page.

## 🛠️ Customizing Labels, Emojis, and Thresholds

You can customize the behavior and appearance of each callout block using `data-*` attributes in your content asset markup.

### 🧩 Supported Attributes

| Attribute               | Description                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| `data-key`              | Required. Metric key to use (`weeklyViews`, `weeklyRevenue`, etc.)          |
| `data-template`         | Required. HTML template string with placeholders like `${value}`            |
| `data-low/medium/high`  | Optional. Override backend thresholds for this block only                   |
| `data-label`            | Optional. Generic label override (applies to all tiers)                     |
| `data-label-high`       | Optional. Override label only for `high` tier                               |
| `data-label-medium`     | Optional. Override label only for `medium` tier                             |
| `data-label-low`        | Optional. Override label only for `low` tier                                |
| `data-emoji`            | Optional. Generic emoji override                                            |
| `data-emoji-high`       | Optional. Emoji for `high` tier only                                        |
| `data-emoji-medium`     | Optional. Emoji for `medium` tier only                                      |
| `data-emoji-low`        | Optional. Emoji for `low` tier only                                         |

### 💡 Example with Per-Tier Overrides

```html
<div class="social-proof"
     data-key="weeklyRevenue"
     data-low="1000"
     data-medium="3000"
     data-high="5000"
     data-label-low="Needs Love"
     data-label-medium="Doing Well"
     data-label-high="Killing It 💸"
     data-emoji-high="💸"
     data-template="<div class='proof-label-box ${tier}'><span class='emoji-wrapper'>${emoji}</span> <span class='label-text ${tier}'>${label}</span>: ${value} earned</div>">
</div>
```

### 🖼️ Template Placeholders

The following placeholders are supported inside `data-template`:

- `${value}` — formatted number (e.g., 3,200)
- `${tier}` — resolved tier (low, medium, high)
- `${label}` — default or overridden label
- `${emoji}` — default or overridden emoji

## 🎯 Default Tier Labels & Emojis (Per Metric)

If no custom `data-label` or `data-emoji` is provided, the system uses predefined labels and emojis per tier for each metric.

### 🧾 `weeklyViews`

| Tier     | Default Label     | Default Emoji |
|----------|-------------------|---------------|
| High     | Hot Right Now     | 🔥            |
| Medium   | Popular Pick      | ⚡            |
| Low      | Hidden Gem        | 💎            |
| No Data  | No Data           | (none)        |

### 📦 `weeklyOrders`

| Tier     | Default Label     | Default Emoji |
|----------|-------------------|---------------|
| High     | Best Seller       | 🏅            |
| Medium   | Popular Choice    | 📬            |
| Low      | Needs a Push      | 🛒            |
| No Data  | No Data           | (none)        |

### 📈 `weeklyUnitsSold`

| Tier     | Default Label        | Default Emoji |
|----------|----------------------|---------------|
| High     | Flying Off the Shelf | 🚀            |
| Medium   | On the Rise          | 📈            |
| Low      | Under the Radar      | 📉            |
| No Data  | No Data              | (none)        |

### 💰 `weeklyRevenue`

| Tier     | Default Label     | Default Emoji |
|----------|-------------------|---------------|
| High     | Top Performer     | 🏆            |
| Medium   | Solid Seller      | 💰            |
| Low      | Room to Grow      | 🛠️            |
| No Data  | No Data           | (none)        |

## 🧱 Content Asset Usage Examples

Below are examples of how to use the `.social-proof` component inside WYSIWYG content assets or page markup.

### 📌 Example 1: Custom Thresholds & Styling

```html
<div class="social-proof"
     data-key="weeklyViews"
     data-low="100"
     data-medium="160"
     data-high="300"
     data-template="<div class='proof-label-box ${tier}'><span class='emoji-wrapper flame-flicker'>${emoji}</span> <span class='label-text ${tier}'>Custom Thresholds</span>: ${value} people viewed this</div>">
</div>
```

✅ Uses hardcoded thresholds instead of site prefs  
✅ Includes animation (flame-flicker class)

### 🔁 Example 2: Different Metric – weeklyOrders

```html
<div class="social-proof"
     data-key="weeklyOrders"
     data-template="<div class='proof-label ${tier}'><span class='emoji-wrapper'>${emoji}</span> <span class='label-text ${tier} shine'>${label}</span>: ${value} orders this week</div>">
</div>
```

✅ Uses built-in backend thresholds for weeklyOrders  
✅ Applies shine animation to label

### ⚠️ Example 3: Invalid Metric Handling

```html
<div class="social-proof"
     data-key="nonexistentMetric"
     data-template="<div class='proof-label ${tier}'><span class='emoji-wrapper shake'>${emoji}</span> <span class='label-text ${tier} shake'>MISSING</span>: ${value}</div>">
</div>
```

❌ Will not render — nonexistentMetric is not in backend JSON  
✅ Cleanly removed by JS fallback logic

### 🖊️ Example 4: Full Label & Emoji Overrides by Tier

```html
<div class="social-proof"
     data-key="weeklyRevenue"
     data-label-low="Slow Start"
     data-label-medium="Good Traction"
     data-label-high="Profit Machine"
     data-emoji-low="🟡"
     data-emoji-medium="🟢"
     data-emoji-high="💰"
     data-template="<div class='proof-label-box ${tier}'><span class='emoji-wrapper pulse'>${emoji}</span> <span class='label-text ${tier}'>${label}</span> — ${value} revenue</div>">
</div>
```

✅ Overrides both label and emoji for each tier  
✅ Adds pulse animation

## 🧪 A/B Testing & Gating (Optional)

The Active Data Callout system can be conditionally activated for users participating in an A/B test campaign. This allows you to measure the impact of callouts on engagement and conversion without exposing the feature site-wide.

### 🔀 ISML Conditional Wrapper

Wrap the entire Active Data logic in a campaign check to only show it for test participants:

```html
<isif condition="${dw.campaign.ABTestMgr.isParticipant('ActiveData', 'activeData')}">
    <div id="metrics-json" style="display: none;">
        ${pdict.metricsJSONString}
    </div>
    <isslot id="active-data-slot"
            description="Product active data"
            context="global"
            context-object="${pdict.product.raw}" />
</isif>
```

### ⚙️ Business Manager Setup

1. Go to Merchant Tools → Online Marketing → A/B Tests
2. Create or locate a campaign named ActiveData
3. Add a test variation also named activeData
4. Assign participants, targeting rules, and test duration
5. Use dw.campaign.ABTestMgr in ISML to gate frontend logic

### 🧼 Gating Behavior

If a shopper is not part of the test:

- The #metrics-json block is not rendered
- No .social-proof elements are processed
- The site behaves as if the feature doesn't exist (no errors or visual disruption)

This ensures a clean, test-safe implementation of the feature.

## 🧼 Fallbacks, Debugging & Summary

The system includes built-in safety checks and graceful degradation to avoid breaking the user experience.

### 🧪 Fallback Logic (Frontend)

A `.social-proof` block is automatically **removed** from the page if:

- `#metrics-json` is missing or invalid
- `data-key` is not found in the backend JSON
- `metric.value` is not a valid number
- Thresholds are misconfigured (e.g., low ≥ medium)
- Final `tier` is `no-data`

### 🧰 Debugging Tips

- **Check PDP Source:** Confirm `<div id="metrics-json">` is rendered with valid JSON.
- **Inspect Console:** JS errors in `productDetail.js` may indicate malformed data or template issues.
- **Use Raw JSON:** For debugging, `pdict.activeDataJSONString` is available in the controller.
- **Log Rendering Data:** Add `console.log(metric)` in JS to inspect values and thresholds.

### ✅ Implementation Checklist

- [x] Backend helper includes `getFormattedMetrics(product)`
- [x] Controller injects `pdict.metricsJSONString` and optionally `activeDataJSONString`
- [x] ISML template includes `#metrics-json` block
- [x] Content asset contains `.social-proof` blocks with `data-key` and `data-template`
- [x] Frontend JS is included and parses/render correctly
- [x] Site preferences are defined for each metric (or defaults are acceptable)

## 📌 Final Notes

The Active Data Callout system is:

- 🔄 **Flexible** — supports multiple metrics, thresholds, and overrides
- 🔐 **Safe** — fails gracefully on missing or invalid data
- 🎯 **A/B test-ready** — integrates cleanly with SFCC campaigns
- ✏️ **Easy to manage** — content authors can control messaging without deployment

## 🚀 Next Steps (Optional Enhancements)

- [ ] Add more metrics (e.g., impressions, click-through rate, days active)
- [ ] Localize tier labels and emoji for multilingual sites
- [ ] Expand desktop rendering logic (currently mobile-focused)
- [ ] Expose Active Data on other templates (e.g., category, search)
