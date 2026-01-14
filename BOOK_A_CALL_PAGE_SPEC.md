# Book a Call Page Specification

## Overview

A standalone page with an inline Calendly embed for booking meetings. This page is designed to be linked directly from outbound emails and campaigns, bypassing the main landing page.

## Route

```
/book-a-call
```

## Purpose

- Direct booking page for email campaigns
- Reduces friction by going straight to scheduling
- Not linked from main navigation or other pages
- Standalone entry point for meeting bookings

---

## Design Requirements

### Colors

| Element | Color |
|---------|-------|
| Background | `#0B1314` |
| Text | `#FFFFFF` |
| Primary accent | `#247CFF` (RetailerBoost blue) |

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    [RetailerBoost Logo]                     │
│                                                             │
│                       Book a Call                           │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │                                                       │  │
│  │                                                       │  │
│  │              Calendly Inline Widget                   │  │
│  │                  (700px height)                       │  │
│  │                                                       │  │
│  │                                                       │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│              Terms | Privacy | Cookies                      │
│                     © 2025 RetailerBoost                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Components

1. **Header**
   - RetailerBoost logo (centered)
   - Logo links to homepage (`/`)
   - Logo asset: `/assets/images/RetailerBoost-Logo-Blue-White.png`

2. **Heading** (optional)
   - Text: "Book a Call" or similar
   - Centered, white text

3. **Calendly Widget**
   - Inline embed (not popup)
   - Full width within container (max-width ~1000px)
   - Minimum height: 700px
   - Responsive

4. **Footer**
   - Minimal footer with copyright
   - Links to Terms, Privacy, Cookies policies

---

## Calendly Configuration

### Base URL

```
https://calendly.com/sturrock/intro
```

### URL Parameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `hide_event_type_details` | `1` | Cleaner UI |
| `hide_gdpr_banner` | `1` | Remove GDPR notice |
| `background_color` | `0b1314` | Match page background |
| `text_color` | `ffffff` | White text |
| `primary_color` | `247CFF` | RetailerBoost blue |

### Full Calendly URL

```
https://calendly.com/sturrock/intro?hide_event_type_details=1&hide_gdpr_banner=1&background_color=0b1314&text_color=ffffff&primary_color=247CFF
```

---

## UTM Passthrough (Required)

The page MUST capture UTM parameters from the URL and pass them to Calendly.

### Supported UTM Parameters

| Parameter | Default (if not in URL) |
|-----------|------------------------|
| `utm_source` | `retailerboostplatform.com` |
| `utm_medium` | `outbound` |
| `utm_campaign` | `email` |
| `utm_content` | `book-a-call-page` |
| `utm_term` | _(none)_ |

### Implementation

Use Calendly's JavaScript API to pass UTM values:

```javascript
// Get UTM params from current page URL
const urlParams = new URLSearchParams(window.location.search);

// Defaults
const defaults = {
  utm_source: 'retailerboostplatform.com',
  utm_medium: 'outbound',
  utm_campaign: 'email'
};

// Capture or use defaults
const utmSource = urlParams.get('utm_source') || defaults.utm_source;
const utmMedium = urlParams.get('utm_medium') || defaults.utm_medium;
const utmCampaign = urlParams.get('utm_campaign') || defaults.utm_campaign;
const utmContent = urlParams.get('utm_content') || 'book-a-call-page';
const utmTerm = urlParams.get('utm_term') || undefined;

// Initialize Calendly inline widget with UTM passthrough
Calendly.initInlineWidget({
  url: 'https://calendly.com/sturrock/intro?hide_event_type_details=1&hide_gdpr_banner=1&background_color=0b1314&text_color=ffffff&primary_color=247CFF',
  parentElement: document.getElementById('calendly-embed'),
  utm: {
    utmSource: utmSource,
    utmMedium: utmMedium,
    utmCampaign: utmCampaign,
    utmContent: utmContent,
    utmTerm: utmTerm
  }
});
```

### Required Calendly Assets

```html
<!-- CSS -->
<link href="https://assets.calendly.com/assets/external/widget.css" rel="stylesheet">

<!-- JavaScript -->
<script type="text/javascript" src="https://assets.calendly.com/assets/external/widget.js" async></script>
```

---

## Signup Attribution Tracking (Required)

Even though this is a booking page, we must also fire the signup attribution pixel. This ensures that if a visitor later signs up (instead of or after booking), we still have their campaign attribution.

### Why?

- UTM data is stored in `app.retailerboost.com`'s localStorage immediately
- If visitor signs up later (days/weeks), attribution is already captured
- Full funnel tracking: campaigns → calls AND campaigns → signups
- Both CTAs are attributed to the same original campaign

### Implementation

On page load (when UTMs or referrer exist), create a hidden iframe to:

```
https://app.retailerboost.com/signup-attribution?utm_source=...&utm_medium=...&...
```

### Parameters to Send

| Parameter | Source |
|-----------|--------|
| `utm_source` | From URL or empty |
| `utm_medium` | From URL or empty |
| `utm_campaign` | From URL or empty |
| `utm_term` | From URL or empty |
| `utm_content` | From URL or empty |
| `referrer` | `document.referrer` |
| `captured_on` | `window.location.hostname` |

### Example Implementation

```javascript
(function() {
  'use strict';
  
  const params = new URLSearchParams(window.location.search);
  const utmParams = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content'];
  const hasUTM = utmParams.some(p => params.has(p));
  const referrer = document.referrer || '';
  
  // Only fire if we have UTMs or a referrer
  if (!hasUTM && !referrer) return;
  
  const trackParams = new URLSearchParams();
  
  utmParams.forEach(p => {
    const value = params.get(p);
    if (value) trackParams.set(p, value);
  });
  
  if (referrer) trackParams.set('referrer', referrer);
  trackParams.set('captured_on', window.location.hostname);
  
  const trackUrl = `https://app.retailerboost.com/signup-attribution?${trackParams.toString()}`;
  
  // Create hidden iframe
  const iframe = document.createElement('iframe');
  iframe.src = trackUrl;
  iframe.style.cssText = 'position:absolute;width:1px;height:1px;opacity:0;pointer-events:none;border:none;';
  iframe.setAttribute('aria-hidden', 'true');
  iframe.setAttribute('tabindex', '-1');
  
  document.body.appendChild(iframe);
})();
```

---

## Additional Tracking Scripts

Include the standard RetailerBoost tracking:

1. **Google Tag Manager**
   - Container ID: `GTM-WPR55HV5`
   - Head snippet + noscript body fallback

2. **RB2B**
   - Key: `QO92DHLLLGN7`
   - Load in head

---

## SEO / Meta

| Meta | Value |
|------|-------|
| Title | `Book a Call \| RetailerBoost` |
| Description | `Schedule a call with RetailerBoost to learn how we can help grow your sales on Google Ads.` |
| Robots | `noindex, nofollow` |

The page should NOT be indexed since it's for direct campaign links only.

---

## Example Campaign Usage

### Email Link

```
https://retailerboostplatform.com/book-a-call?utm_source=apollo&utm_medium=email&utm_campaign=q1-outreach&utm_content=email-footer
```

### What Happens

1. Visitor lands on `/book-a-call?utm_source=apollo&...`
2. Attribution iframe fires → stores UTMs in `app.retailerboost.com` localStorage
3. Calendly widget initializes with same UTMs
4. Visitor books call → Calendly has full attribution
5. OR visitor leaves, signs up later → app already has their attribution

---

## Responsive Behavior

- Widget container: max-width ~1000px, centered
- Widget itself is responsive (min-width: 320px)
- Mobile: full-width with small padding
- No horizontal scroll

---

## Styles Reference

```css
/* Page */
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: #0B1314;
  color: #FFFFFF;
  min-height: 100vh;
}

/* Header */
.header {
  padding: 1.5rem 2rem;
  text-align: center;
}

/* Logo */
.logo {
  height: 40px;
  width: auto;
}

/* Main content */
.main {
  max-width: 1000px;
  margin: 0 auto;
  padding: 0 1rem 2rem;
}

/* Heading */
.heading {
  text-align: center;
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
}

/* Calendly container */
#calendly-embed {
  min-width: 320px;
  height: 700px;
  border-radius: 12px;
  overflow: hidden;
}

/* Footer */
.footer {
  padding: 1.5rem 2rem;
  text-align: center;
  color: #aaa;
  font-size: 0.875rem;
  border-top: 1px solid rgba(36, 124, 255, 0.2);
}

.footer a {
  color: #aaa;
  text-decoration: none;
  margin: 0 0.75rem;
}

.footer a:hover {
  color: #fff;
}
```

---

## Testing Checklist

- [ ] Page loads at `/book-a-call`
- [ ] Calendly widget displays correctly
- [ ] Widget matches dark theme (background matches page)
- [ ] Logo visible and links to homepage
- [ ] UTM parameters pass through to Calendly booking
- [ ] Attribution iframe fires when UTMs present
- [ ] Works on mobile devices
- [ ] GTM fires correctly
- [ ] Footer links work (Terms, Privacy, Cookies)
- [ ] No horizontal scroll on any screen size

---

## Notes

- Page is `noindex, nofollow` — not for organic search
- No navigation menu — keep focus on booking
- Simple, distraction-free design
- Consistent with RetailerBoost brand
