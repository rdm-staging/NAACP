# Copilot Instructions

## Project Overview

Static HTML/JS donation landing page (no build system, no package manager). This is the **NAACP** monthly donation campaign (National Association for the Advancement of Colored People). Pages are served directly — edit files and open in a browser to test.

## Project Status

Campaign migration from Covenant House template is **complete**. All copy, branding colors (`--mainColor: #002868`), and messaging across all pages now reflect NAACP.

## Architecture

```
index.html         ← 3-step donation form (main entry point)
confirmation.html  ← Post-submission thank-you page
dummy_decline.html ← Shown on declined transactions
js/donation.js     ← Central campaign config (amounts, fees, layout)
js/form.js         ← Multi-step navigation, validation, error display
js/utils.js        ← Modal loading (jQuery), DOM helpers (createNode, insertAfter)
css/base.css       ← All custom styles; CSS vars: --mainColor, --bgColor
```

## Form Flow

3-step form tracked via `sessionStorage("current-step")`:

1. **Step 1** — Donation amount selection (`#step-1`)
2. **Step 2** — Payment/credit card info (`#step-2`)
3. **Step 3** — Personal info + billing address + T&C checkbox + CAPTCHA (`#step-3`)

Submit triggers `submitOrd()` (defined inline in `index.html`) which sets hidden input `submitOrder = 1` and calls `document.forms[0].submit()`. The hidden `pageSubmits` input tracks re-submissions; if `> 1`, the entire form is shown flat (error-review mode) instead of stepped.

Desktop (> 980px) = multi-step; mobile = full form shown at once (`form.js: screenSizeCheck()`).

## Campaign Configuration (`js/donation.js`)

**All donation tuning goes here.** Key `donationParameters` fields:

- `"001"–"004"`: amount, prechecked, inputType — `"001"` must always be the lowest amount
- `donationOrder`: array controlling display order (e.g. `["003","002","001","004"]`)
- `pageTemplate.rowGrid`: columns in donation grid (default `3`)
- `pageTemplate.freeformInsideLabel`: puts "Other" input inside its label
- `pageTemplate.disableFreeformOnPageLoad`: disables freeform until "Other" is chosen
- `processingFees.feePercentage`: decimal (e.g. `0.03` = 3%)
- `minimumDonation` / `maximumDonation`: integers, no `$` or decimals
- `donationStatements`: per-amount copy shown below the grid

To **add/remove a preset amount**, add/remove a key (`"005"` etc.) and include it in `donationOrder`. Leave `amount: ""` for a freeform "Other" option.

## State Management

`sessionStorage` keys: `donation` (code e.g. `"002"`), `amount` (numeric string), `fees-accepted` (`"0"` or `"1"`), `total-donation`, `current-step`.

## Optional vs. Required Fields

Fields marked with the HTML attribute `optional` (not `required`) are skipped by `validateForm()` in `form.js`. Examples: `sAddress2`, `phone`, `giftAmount` (required only when "Other" is selected — handled specially).

## Modals

Privacy, CVV, and Terms modals are Bootstrap dialogs. Their content loads remotely via jQuery `.load()` from `privacy_modal.html`, `cvv_modal.html`. Triggered by `data-remote` + `data-toggle="modal"` attributes.

## External Dependencies

| Dependency                 | Source                                                 |
| -------------------------- | ------------------------------------------------------ |
| Bootstrap 4                | Local (`css/bootstrap.min.css`, `js/bootstrap.min.js`) |
| jQuery 3.5                 | Local (`js/jquery-3.5.0.min.js`)                       |
| `currency.js`              | CDN: `order-safely.com/_css_/currency/currency.js`     |
| Foundation CSS / normalize | CDN: `order-safely.com`                                |
| Confirmation page CSS      | CDN: `order-safely.com/_css_/confirmation.css`         |
| Fonts                      | Typekit `und2uku`, Google Fonts `Montserrat`           |
| CAPTCHA                    | Server-side: `image_validation.php?get_r_img=1`        |
| Analytics                  | Google Tag Manager `GTM-TD9H85F7`, MaxMind device.js   |

## Key Patterns

- `createNode(tagName, attributesObj)` in `utils.js` — use this whenever creating DOM elements programmatically (donation grid items, error messages, etc.)
- Per-amount effect lists (`donationEffect` object) are defined inline in `index.html` at the bottom, separate from `donationParameters`. Update both when changing amounts.
- CSS variables `--mainColor` and `--bgColor` in `base.css` control brand colors — change here first before touching individual selectors.
- `keepValues()` in `index.html` resets select fields (cardType, sState, expYear, expMonth) on load to prevent browser autofill from pre-selecting stale values.

## Remaining TODOs

The following items require assets or information not yet available:

1. **Logo files**: Drop `images/naacp-logo.png` into the `images/` folder — all pages reference this path.
2. **Phone number**: Replace `xxx-xxx-xxxx` placeholder in `index.html` (T&C section), `soft_declines.html`, and `dummy_decline.html` when the NAACP contact number is confirmed.
3. **Contact email**: Replace `xxx@naacp.org` placeholder in `index.html`, `privacy_modal.html` when the actual email is confirmed.
4. **Processing fee %**: Update `processingFees.feePercentage` in `js/donation.js` (currently `0.03`) once the rate is confirmed.
5. **Privacy policy content**: Replace the generic placeholder text in `privacy_modal.html` with NAACP's official privacy policy. The full policy is at https://naacp.org/resources/privacy-policy.
6. **Favicon**: Add NAACP favicon files to `images/favicons/` — the `<head>` in `index.html` already references these paths.
