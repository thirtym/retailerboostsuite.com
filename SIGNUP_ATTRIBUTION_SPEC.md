# Signup Attribution Tracking Specification

## Overview

This document specifies how to implement cross-domain signup attribution tracking for RetailerBoost's sending domains and marketing website.

**IMPORTANT: This is NOT ad tracking!** This is for tracking which marketing campaigns led users to sign up for RetailerBoost. It is completely separate from Google Ads conversion tracking or order attribution.

## Purpose

When a user clicks a link in an email (on a sending domain like `getretailerboost.com`) and later signs up on `app.retailerboost.com`, we need to attribute that signup to the original email campaign - even if they visit days later via a Google search.

## How It Works

```
1. User clicks email link → lands on sending domain (e.g., getretailerboost.com)
2. Sending domain captures UTM params + document.referrer
3. Sending domain loads hidden iframe: app.retailerboost.com/signup-attribution?...
4. The iframe stores this data in app.retailerboost.com's localStorage
5. Days later, user visits app.retailerboost.com/signup directly
6. Signup page reads the stored attribution data
7. User is attributed to the original email campaign
```

## Implementation

### 1. Add the Tracking Snippet

Add this JavaScript snippet to **every page** on your domain. Place it just before the closing `</body>` tag:

```html
<!-- RetailerBoost Signup Attribution Tracking -->
<!-- NOT FOR AD TRACKING - This tracks signup attribution only -->
<script>
(function() {
  'use strict';
  
  // Only run if we have UTM params or a referrer worth tracking
  var params = new URLSearchParams(window.location.search);
  var utmParams = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content'];
  var hasUTM = utmParams.some(function(p) { return params.has(p); });
  var referrer = document.referrer || '';
  
  // Only track if we have UTMs or a meaningful referrer
  if (!hasUTM && !referrer) {
    return;
  }
  
  // Build the tracking URL
  var trackUrl = 'https://app.retailerboost.com/signup-attribution?';
  var trackParams = [];
  
  // Add UTM params
  utmParams.forEach(function(p) {
    var value = params.get(p);
    if (value) {
      trackParams.push(p + '=' + encodeURIComponent(value));
    }
  });
  
  // Add referrer
  if (referrer) {
    trackParams.push('referrer=' + encodeURIComponent(referrer));
  }
  
  // Add the domain where this was captured (for debugging)
  trackParams.push('captured_on=' + encodeURIComponent(window.location.hostname));
  
  trackUrl += trackParams.join('&');
  
  // Load the tracking pixel in a hidden iframe
  var iframe = document.createElement('iframe');
  iframe.src = trackUrl;
  iframe.style.cssText = 'position:absolute;width:1px;height:1px;opacity:0;pointer-events:none;border:none;';
  iframe.setAttribute('aria-hidden', 'true');
  iframe.setAttribute('tabindex', '-1');
  
  // Append to body when ready
  if (document.body) {
    document.body.appendChild(iframe);
  } else {
    document.addEventListener('DOMContentLoaded', function() {
      document.body.appendChild(iframe);
    });
  }
})();
</script>
```

### 2. React/Next.js Implementation

If you're using React or Next.js, create a component:

```tsx
// components/SignupAttributionTracker.tsx
'use client';

import { useEffect } from 'react';

/**
 * Cross-domain signup attribution tracking
 * NOT FOR AD TRACKING - This tracks signup attribution only
 * 
 * Loads a hidden iframe to sync UTM params to app.retailerboost.com
 */
export function SignupAttributionTracker() {
  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    const utmParams = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content'];
    const hasUTM = utmParams.some(p => params.has(p));
    const referrer = document.referrer || '';
    
    // Only track if we have UTMs or a meaningful referrer
    if (!hasUTM && !referrer) {
      return;
    }
    
    // Build tracking URL
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
    
    // Cleanup
    return () => {
      if (iframe.parentNode) {
        iframe.parentNode.removeChild(iframe);
      }
    };
  }, []);
  
  return null;
}
```

Then include it in your layout:

```tsx
// app/layout.tsx or pages/_app.tsx
import { SignupAttributionTracker } from '@/components/SignupAttributionTracker';

export default function Layout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SignupAttributionTracker />
      </body>
    </html>
  );
}
```

### 3. Continue Passing UTMs in Links

In addition to the iframe pixel, **always pass UTM params** when linking to `app.retailerboost.com`:

```html
<!-- Good: Always include UTMs in links to app -->
<a href="https://app.retailerboost.com/signup?utm_source=newsletter&utm_medium=email&utm_campaign=jan_2025_promo">
  Get Started
</a>
```

This provides redundancy - if the iframe fails, the URL params still work.

## What Gets Tracked

| Field | Source | Example | Stored in DB? |
|-------|--------|---------|---------------|
| `utm_source` | URL parameter | `google`, `newsletter`, `website` | ✅ User.utmSource |
| `utm_medium` | URL parameter | `cpc`, `email`, `organic` | ✅ User.utmMedium |
| `utm_campaign` | URL parameter | `spring_promo`, `black_friday` | ✅ User.utmCampaign |
| `utm_term` | URL parameter | `ecommerce`, `shopify` | ✅ User.utmTerm |
| `utm_content` | URL parameter | `hero_cta`, `footer_link` | ✅ User.utmContent |
| `referrer` | `document.referrer` | `https://google.com/...` | ✅ User.signupReferrer |
| `referrerSource` | Parsed from referrer | `google_organic`, `facebook` | ✅ User.referrerSource |
| `captured_on` | `window.location.hostname` | `getretailerboost.com` | ❌ (debugging only) |

## Referrer Source Parsing

When no UTM params are present, we still capture the `document.referrer` URL and parse it into a readable source. This enables organic traffic attribution.

| Referrer URL | Parsed Source |
|--------------|---------------|
| `https://www.google.com/search?q=...` | `google_organic` |
| `https://www.bing.com/search?q=...` | `bing_organic` |
| `https://www.yahoo.com/...` | `yahoo_organic` |
| `https://duckduckgo.com/...` | `duckduckgo_organic` |
| `https://www.facebook.com/...` | `facebook` |
| `https://twitter.com/...` or `https://t.co/...` | `twitter` |
| `https://www.linkedin.com/...` | `linkedin` |
| `https://www.reddit.com/...` | `reddit` |
| `https://retailerboost.com/...` | `website` |
| (empty - direct visit) | `direct` |
| Other domains | The hostname (e.g., `example.com`) |

## Attribution Window

- **Cross-domain tracking**: 5-day expiry
- **Same-domain tracking**: 30-day expiry
- **Smart override logic**: Real campaigns override fallbacks, but NOT vice versa (see below)

## Override Logic (IMPORTANT!)

**Fallback UTMs will NOT override real attribution.**

We define "fallback" UTMs as generic website traffic without a specific campaign:
- `utm_source=website` or `utm_source=organic` or `utm_source=landing_page`
- `utm_medium=organic` or `utm_medium=direct` or `utm_medium=none`
- No `utm_campaign` specified

**Scenario Examples:**

| Existing Attribution | New Visit | Result |
|---------------------|-----------|--------|
| None | `utm_source=website&utm_medium=organic` (fallback) | ✅ Stored |
| `utm_source=website` (fallback) | `utm_source=newsletter&utm_campaign=jan_promo` | ✅ Override |
| Referral code stored | `utm_source=website&utm_medium=organic` (fallback) | ❌ NOT overridden |
| `utm_source=newsletter&utm_campaign=promo` | `utm_source=website` (fallback) | ❌ NOT overridden |
| `utm_source=newsletter&utm_campaign=promo` | `utm_source=google&utm_campaign=ppc` | ✅ Override (real > real, last wins) |

**Why?** This prevents regular website visits from erasing valuable referral/campaign attribution.

## Referral Code Interaction (IMPORTANT!)

The referral code (`ref` parameter) and UTM attribution are **TWO COMPLETELY SEPARATE SYSTEMS**:

| System | What it tracks | Database Field | Expiry | Can be overwritten? |
|--------|----------------|----------------|--------|---------------------|
| **Referral** | Who referred them | `referredByUserId` | 30 days | ❌ NEVER (once set) |
| **UTM Attribution** | What marketing drove signup | `utmSource`, `utmCampaign`, etc. | 30 days | ✅ Yes (last meaningful touch) |

### Key Point: Referrals are NEVER Lost

If a user is referred, the referral relationship is **ALWAYS preserved**, even if they later click a different marketing campaign. The referrer ALWAYS gets credit.

### Example: Referred User Converts via Email Campaign

1. **Day 0**: User A shares referral link with User B (`?ref=abc123`)
2. **Day 0**: User B clicks link → Referral code stored (30-day expiry)
3. **Day 15**: User B receives outbound email, clicks link → `utm_source=newsletter&utm_campaign=jan_promo`
4. **Day 15**: User B signs up

**Result in Database:**
```
User B:
  - referredByUserId = User A's ID ✅ (referral preserved!)
  - utmSource = "newsletter"
  - utmMedium = "email" 
  - utmCampaign = "jan_promo"
```

**What this tells us:**
- **Referral Dashboard**: User A gets credit for referring User B ✅
- **Campaign Dashboard**: The "jan_promo" email campaign gets credit for the conversion ✅

Both are true! User A introduced User B to RetailerBoost, but the email campaign is what finally made them sign up.

### Example: Referred User with Only Fallback Visits

1. **Day 0**: User clicks referral link (`?ref=abc123`) → Stored
2. **Day 10**: User visits retailerboost.com homepage → `utm_source=website` (fallback) → **NOT stored** (preserves referral)
3. **Day 20**: User visits again → `utm_source=website` (fallback) → **NOT stored**
4. **Day 25**: User signs up

**Result in Database:**
```
User:
  - referredByUserId = Referrer's ID ✅
  - utmSource = Referrer's email (auto-populated)
  - utmMedium = "referral"
  - utmCampaign = "referral_landing_page"
  - utmTerm = "abc123" (the ref code)
```

Because no real campaign touched this user, we auto-populate the UTM with referral attribution.

## Trusted Domains

Only these domains are allowed to embed the signup attribution pixel:

- `retailerboost.com`
- `getretailerboost.com`
- `joinretailerboost.com`
- `startretailerboost.com`
- `growwithretailerboost.com`
- `retailerboostlabs.com`
- `retailerboosttool.com`
- `tryretailerboost.com`
- `retailerboostdemo.com`
- `retailerboosthub.com`
- `retailerboostai.com`
- `retailerboostnow.com`
- `retailerboostsystem.com`
- `retailerboostapp.com`
- `retailerboosthq.com`
- `retailerboostsuite.com`
- `retailerboostpro.com`
- `retailerboostonline.com`
- `retailerboostplatform.com`

## Testing

1. Visit your domain with UTM params: `https://yourdomain.com/?utm_source=test&utm_medium=test&utm_campaign=test_campaign`
2. Open browser DevTools → Network tab
3. Verify an iframe request to `app.retailerboost.com/signup-attribution?...` is made
4. Open DevTools → Application → localStorage
5. You should NOT see the data here (it's stored on app.retailerboost.com, not your domain)
6. Visit `app.retailerboost.com/signup` and check localStorage there for `retailerboost_signup_attribution`

## FAQ

### Why use an iframe instead of a direct API call?

The iframe allows us to store data in `app.retailerboost.com`'s localStorage from any domain. A direct API call would require cookies with `SameSite=None` and other cross-origin complications.

### What if the iframe is blocked?

We have fallbacks:
1. URL params in links still work
2. Referrer tracking captures organic traffic
3. Referral auto-attribution for referral signups

### Does this affect page performance?

Minimal impact. The iframe loads a tiny page (<1KB) asynchronously after your page content.

### What about ad blockers?

Some aggressive ad blockers might block the iframe. That's okay - we have multiple fallback mechanisms.

## Questions?

Contact the RetailerBoost engineering team or refer to the internal documentation at `/admin/docs/utm-tracking`.

