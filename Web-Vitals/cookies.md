# Cookies

## What is a Cookie?

A cookie is just a small string that the browser **automatically attaches to every request** made to a domain. Nothing more. It was invented in 1994 by Netscape to solve a simple problem — like keeping items in a shopping cart — because HTTP is stateless (it forgets you between requests).

That "auto-attach" behavior is what made cookies both incredibly convenient *and* eventually a surveillance machine at scale.

---

## How Third-Party Cookies Work

A **first-party cookie** is set by the site you're actually visiting.  
A **third-party cookie** is set by someone else's code embedded in that page.

**Example flow:**

1. You visit `nike.com`
2. Nike's HTML includes an `<iframe>` loading from `facebook.com` (a Like button, Share button, comment section — anything)
3. Browser parses the HTML and **immediately fetches it** — you haven't scrolled or clicked anything yet
4. That request goes to `facebook.com` with your cookie auto-attached
5. Facebook now knows you visited `nike.com` — silently, without asking

You never went to Facebook. But Facebook's code ran in your browser, so from the browser's perspective, a request went out and the cookie was attached.

### Tracking happens at load time, not interaction time

This is a common misconception. You do **not** need to click the Like button to be tracked. The tracking happens the moment the browser loads the iframe — which is automatic.

```
You land on nike.com
       ↓
Browser reads HTML top to bottom
       ↓
Sees <iframe src="facebook.com/...">
       ↓
Instantly fires a request to facebook.com  ← tracking happens HERE
       ↓
You're still reading the product title
```

Clicking the button would send *additional* data — like "this user actively liked this product." But for basic cross-site tracking, the page load alone is sufficient.

### The Like button was just an example

The Like button became the poster child for third-party tracking because it was visible and everywhere in the 2010s. But the tracking mechanism is identical regardless of what Facebook embeds:

| What's on the page | Does it track on load? |
|---|---|
| Like button | ✅ Yes |
| Share button | ✅ Yes |
| Comment section | ✅ Yes |
| Facebook Analytics script | ✅ Yes |
| Invisible 1×1 pixel image | ✅ Yes |

Any of these make a request to Facebook's servers at load time. The real payload was always the **request**, not the button.

**Common third-party trackers embedded across the web:**

| Widget | Who tracked you |
|---|---|
| Facebook Like/Share button | Facebook |
| Google Analytics script | Google |
| "Sign in with Google" button | Google |
| YouTube embedded videos | Google |
| Twitter/X share button | Twitter |
| Disqus comments section | Disqus |

---

## The Privacy Abuse

The auto-attach behavior let advertisers silently follow users across the entire web, not just one site.

**What they built with it:**

| Abuse | What it meant |
|---|---|
| Cross-site tracking | Follow you across thousands of websites |
| Behavioral profiles | Know your interests, habits, income level |
| Retargeted ads | "Why is this shoe following me everywhere?" |
| Fingerprinting | Re-identify you even without cookies |
| Data selling | Your profile sold to brokers without consent |

### Timeline

| Year | Event |
|---|---|
| ~1994 | Cookies invented by Netscape for shopping carts |
| ~2010s | Third-party cookie tracking peaks |
| 2018 | GDPR (Europe) — must ask for consent |
| 2020 | California CCPA follows |
| 2023 | Chrome begins phasing out third-party cookies |
| 2024 | Safari & Firefox already blocking them by default |

---

## The Facebook Pixel

The Facebook Pixel is a small JavaScript file that website owners paste into their `<head>`. The moment a user lands on the page, it runs silently.

```html
<script>
  fbq('init', '123456789');   // your unique pixel ID
  fbq('track', 'PageView');   // fires immediately on load
</script>
```

**What it does:**
1. User visits `nike.com`
2. Pixel script loads and reads the user's `facebook.com` cookie
3. Identifies the user — "this is Facebook user ID 987654"
4. Sends a signal back to Facebook: "User 987654 just viewed the Air Max page"

### The Business Model

Facebook doesn't pay Nike cash. The exchange is:

- **Nike gets** — free Like buttons, social sharing tools, and precise ad targeting
- **Facebook gets** — tracking data on all of Nike's visitors
- **Nike then pays Facebook** for ads — and the pixel data is what makes those ads so precise

```
Websites     →  install pixel for free tools
Users        →  get tracked without knowing
Facebook     →  builds profiles on everyone
Advertisers  →  pay Facebook to reach those profiles
```

Facebook's $100B+/year ad business runs on this loop.

---

## What the Pixel Can Track (if the site is built for it)

A basic install only gets page visits. Rich tracking requires a developer to manually wire up events:

```js
// On product page
fbq('track', 'ViewContent', {
  content_name: 'Air Max 2024',
  value: 120,
  currency: 'USD'
});

// On "Add to Cart" button click
fbq('track', 'AddToCart', { value: 120, currency: 'USD' });

// On purchase completion
fbq('track', 'Purchase', { value: 120, currency: 'USD' });
```

**The effort spectrum:**

| Website type | What Facebook gets |
|---|---|
| Small blog with pixel pasted | Just page visits |
| Basic ecommerce, lazy setup | Visits + maybe purchases |
| Nike-level, fully tailored | Every click, scroll, add-to-cart, purchase value, product names |
| No pixel at all | Nothing |

The pixel is just a tool. The website's own data and the developer's effort is what makes it powerful or useless.

---

## Cookie Consent Banners

GDPR was supposed to enforce consent *before* tracking. The ideal flow:

- You visit a site → banner appears → you **Decline** → no tracking ✅
- You visit a site → banner appears → you **Accept** → tracking loads ✅

**What actually happens on many sites:**

Many sites fire tracking scripts *before* you click anything. By the time the banner appears, Google Analytics and the Facebook Pixel have already fired.

### Dark Patterns

| Trick | How |
|---|---|
| Big green "Accept All" | Small grey "Manage preferences" hidden away |
| Pre-ticked boxes | All tracking options already checked |
| 11 clicks to decline | Accept is 1 click, decline takes forever |
| "Legitimate interest" | They claim they don't even need your consent |
| No "Reject All" button | Only "Accept" or a confusing settings menu |

**The honest reality:**

| Action | Result |
|---|---|
| Click Accept | Definitely tracked |
| Click Decline | *Supposed* to not track — many sites ignore it |
| Don't click anything | Many sites track you anyway |

The cookie banner was the law's attempt to give users control. The industry's response was to make declining as difficult as possible while technically "complying."

---

## Summary

> Cookies were an innocent tool invented to solve a stateless HTTP problem. The browser's "auto-attach to every request" behavior — which made them convenient — is the exact same property that turned them into a web-wide surveillance infrastructure. The privacy crisis wasn't a flaw in the technology; it was a deliberate exploit of it.