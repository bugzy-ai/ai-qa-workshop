# Knowledge Base

> A curated collection of factual knowledge about this project - what we currently know and believe to be true. This is NOT a historical log, but a living reference that evolves as understanding improves.

**Maintenance Guide**: See `knowledge-maintenance-guide.md` for instructions on how to maintain this knowledge base.

**Core Principle**: This document represents current understanding, not a history. When knowledge evolves, update existing entries rather than appending new ones.

---

## Project Knowledge

### Application Overview

**Storzy** is a simplified demonstration e-commerce platform hosted on Vercel at https://storzy.vercel.app. It provides core shopping functionality (product browsing, cart, checkout) but intentionally lacks many standard e-commerce features.

**Key Characteristics:**
- Authentication required for all access (no guest browsing)
- Test credentials displayed on login page: test_user / password
- Single-page checkout (no multi-step wizard)
- No payment processing (orders complete instantly)
- Auto-logout after checkout completion
- Fixed 8% tax rate

### Domain Knowledge

**Features That Exist:**
| Feature | Details |
|---------|---------|
| Login | Username/password authentication |
| Product Catalog | 6 products displayed in grid |
| Product Sorting | 4 options: Name A-Z, Name Z-A, Price Low-High, Price High-Low |
| Add to Cart | Button on each product card |
| Cart Management | Quantity controls (+/-), remove button, subtotal calculation |
| Cart Badge | Shows item count in header |
| Checkout | Single form: name, postal code, country, shipping method |
| Shipping Methods | Standard (Free), Express ($9.99), Overnight ($19.99) |
| Order Confirmation | Thank you page with "Back Home" button |
| Logout | Button in header |

**Features That DO NOT Exist:**
- Product filtering or search
- Individual product detail pages
- Guest checkout
- User registration
- Account/profile management
- Address book
- Order history or tracking
- Payment processing
- Wishlist
- Product reviews
- Forgot password

### Navigation Structure

```
/ (Login) --> /inventory --> /cart --> /checkout --> /checkout-complete
```

All pages except login require authentication. After checkout completion, clicking "Back Home" logs the user out.

### Testing Patterns

**Priority Test Areas:**
| Priority | Features |
|----------|----------|
| High | Login, Complete Checkout Flow (end-to-end), Cart Operations |
| Medium | Product Sorting, Cart Quantity Controls, Shipping Method Selection |
| Low | Form Validation, Cancel Checkout, Edge Cases |

**Test Data:**
- Credentials: test_user / password
- Sample product: Baby Onesie ($7.99)
- Tax calculation: 8% of subtotal
- Example total: $7.99 + $0.64 tax = $8.63

### UI Selectors (Playwright)

**Login Page:**
```javascript
page.getByRole('textbox', { name: 'Username' })
page.getByRole('textbox', { name: 'Password' })
page.getByRole('button', { name: 'Login' })
```

**Product Inventory:**
```javascript
page.getByRole('combobox') // Sort dropdown
page.getByRole('button', { name: 'Add to Cart' }) // Per product
```

**Cart:**
```javascript
page.getByRole('button', { name: 'Proceed to Checkout' })
// Quantity: "-" and "+" buttons
```

**Checkout:**
```javascript
page.getByRole('combobox') // Country dropdown
page.getByRole('radio', { name: /Standard Shipping/ })
page.getByRole('radio', { name: /Express Shipping/ })
page.getByRole('radio', { name: /Overnight Shipping/ })
page.getByRole('button', { name: 'Continue' })
page.getByRole('button', { name: 'Cancel' })
```

**Order Complete:**
```javascript
page.getByRole('button', { name: 'Back Home' })
```

### Technical Infrastructure

**Environment:**
- Hosting: Vercel
- URL: https://storzy.vercel.app
- Framework: Likely React/Next.js (modern web app)

**Playwright Configuration:**
- Browser: Chromium
- Viewport: 1280x720
- Timeouts: 30s global, 5s assertion

**Console Observations:**
- Warning about input autocomplete attributes
- No critical errors

### Environment Variables

Test data configuration in `.env.testdata`:
- `TEST_BASE_URL=https://storzy.vercel.app`
- `TEST_USER_EMAIL=test_user`
- Password stored in `.env` (not in testdata file)

---

**Remember**: Every entry should answer "Will this help someone working on this project in 6 months?"
