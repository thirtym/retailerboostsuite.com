# retailerboostapp.com

Landing page for RetailerBoost agency partnerships.

## Overview

This is a static landing page targeting Shopify agencies, explaining how RetailerBoost works as a fully-funded, commission-only Google Shopping layer that agencies can offer to their clients.

## Deployment

Static HTML - deploy to any web server or hosting platform.

---

## UTM Tracking

The page includes JavaScript that handles UTM parameters for signup links.

### How It Works

1. When a visitor arrives from a campaign with UTM params (e.g., `?utm_campaign=agency-outreach`), those params are captured.
2. When clicking "Get Started", the captured params are passed through to the signup URL.
3. If no UTM params are present, defaults are used.

### Default Values

| Parameter | Default Value |
|-----------|---------------|
| `utm_source` | Current hostname (e.g., `retailerboostapp.com`) |
| `utm_medium` | `outbound` |
| `utm_campaign` | `email` |
| `utm_content` | Button location (e.g., `nav`, `hero`, `footer-cta`) |

### Adding New Signup Links

To add a new signup link that preserves UTM params:

```html
<a href="https://app.retailerboost.com/signup" data-signup="your-location">Get Started</a>
```

The `data-signup` attribute sets the fallback `utm_content` value for that button location.

### Example Flow

1. Visitor clicks email link: `https://retailerboostapp.com/?utm_source=sendgrid&utm_campaign=agency-dec-2025`
2. Visitor clicks "Get Started" in hero
3. Redirects to: `https://app.retailerboost.com/signup?utm_source=sendgrid&utm_medium=outbound&utm_campaign=agency-dec-2025&utm_content=hero`

The incoming `utm_source` and `utm_campaign` are preserved, while `utm_content` uses the button location default.

### Calendly Links

Calendly popup links also pass UTM params using the `openCalendly()` helper:

```html
<a href="#" onclick="openCalendly('calendly-hero');return false;">Book a Call</a>
```

The location identifier becomes the `utm_content` fallback. Incoming UTM params are passed to Calendly automatically.

---

## Target Persona

**Shopify agencies** who manage Google Ads/Shopping for e-commerce clients.

Typically:
- PPC managers or agency owners
- Managing multiple Shopify stores
- Experienced with Google Ads and Merchant Center
- Looking for ways to grow client accounts without increasing risk

---

## Pain Points We Solve

1. **Budget-capped clients** - Client wants growth but won't increase ad spend. Agency is stuck optimizing the same budget.

2. **Underperforming inventory** - Seasonal stock, long-tail SKUs, new products with no data. Can't justify spending client money experimenting on risky inventory.

3. **Difficult clients** - High-maintenance clients who question every decision. Agency wants to offload risk.

4. **Reputational risk** - Agencies want to offer upsells that make them look smart and conservative, not salesy or reckless.

---

## What RetailerBoost Actually Is

- **Ad spend investors** - We fund 100% of ad spend with our own capital
- **Commission-only** - We only charge when we generate sales
- **Incremental** - We target products Google's algorithm is ignoring, generating net-new revenue
- **Non-competitive** - We don't touch existing campaigns, don't compete with agencies

---

## How to Pitch

### Do Say

- "Incremental sales" - every sale we generate is a sale that wasn't happening before
- "We fund the ad spend, you keep the client relationship"
- "No sales = no charge"
- "We target inventory your campaigns aren't reaching"
- "100% incremental - we run campaigns on the inventory you choose"
- "You're in control - you decide which products we can target"
- "You set the scope via categories or tags"
- "It's additive, not competitive"
- "You can white-label it or add your own markup"
- "We're ad spend investors, not an agency"
- "Zero risk for you or your client"

### Don't Say

- "Growth hack" / "Unlock" / "Revolutionary" - avoid hype
- "We'll manage your client's campaigns" - we don't manage their campaigns. 
- "Guaranteed results" - we can't guarantee volume
- Anything that suggests we compete with agencies
- Anything that suggests we want the client relationship

---

## Objection Handling

| Objection | Response |
|-----------|----------|
| "How is this different from what we already do?" | You charge fees for performance you deliver. We fund ad spend ourselves and only take commission on sales we generate. We target inventory your campaigns aren't reaching. Additive, not competitive. |
| "Will this affect our existing campaigns?" | No. We use our own Google Ads account. Your campaigns run exactly as before. Zero overlap. |
| "Do you contact our clients?" | Only the email used to sign up receives communication. If you sign up on behalf of clients, you get all emails. We never pitch, upsell, or market to clients. |
| "What's the catch?" | If we're losing money on a store, we may dial back investment. But if we generate nothing, you pay nothing. |
| "Do you guarantee sales?" | No. Volume varies. But no sales = no charge. |
| "If this works, why isn't everyone doing it?" | Fronting ad spend is irrational at small scale. It only works with portfolio-level diversification across thousands of SKUs and accounts. We apply a hedge-fund-style approach - absorb variance, aim for small margins, scale massively. It requires a different capital structure and risk appetite than agency economics allow for. |
| "How do you make money?" | We keep the difference between commission and ad spend. We're aiming to break even on most orders - it's a volume play. |

---

## Emotional Goals

When an agency reads this page, they should think:

- "Holy shit, this is clever"
- "This makes me look good to clients"
- "This doesn't threaten my role"
- "I stay in control"
- "There is no downside for me or my client"

---

## Tone Guidelines

- Confident, calm, senior
- No hype, no buzzwords
- Prefer clarity over cleverness
- Short paragraphs, clear sections
- Write as if explaining something smart to another professional
