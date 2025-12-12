---
version: 1.1.0
created_at: 2025-12-12
updated_at: 2025-12-12
status: updated
exploration_completed: 2025-12-12
---

# Test Plan: Storzy E-commerce Platform

## Overview

Storzy is a **simplified demonstration e-commerce platform** with core shopping functionality. The application supports basic product browsing, cart management, and checkout flow but is missing many standard e-commerce features like user registration, product filtering, guest checkout, and order tracking.

**Base URL:** https://storzy.vercel.app
**Test Credentials:** test_user / password (displayed on login page)

## Features to Test

### User Authentication
- [ ] User login with valid credentials (test_user / password)
- [ ] User login error handling (invalid credentials)
- [ ] User logout functionality
- [ ] Session persists during shopping flow
- [ ] Session ends after checkout completion (auto-logout)
- ~~[ ] User registration~~ **NOT AVAILABLE** - No registration flow
- ~~[ ] Password reset~~ **NOT AVAILABLE** - No forgot password option
- ~~[ ] Guest checkout~~ **NOT AVAILABLE** - Login required for all access

### Product Browsing
- [ ] Homepage redirects to login if not authenticated
- [ ] Product inventory page displays 6 products after login
- [ ] Product listing shows images, names, descriptions, and prices
- [ ] Product sorting by Name (A to Z) - default
- [ ] Product sorting by Name (Z to A)
- [ ] Product sorting by Price (Low to High)
- [ ] Product sorting by Price (High to Low)
- ~~[ ] Product detail pages~~ **NOT AVAILABLE** - No individual product pages
- ~~[ ] Product search functionality~~ **NOT AVAILABLE** - No search feature
- ~~[ ] Product filtering by category~~ **NOT AVAILABLE** - No filters exist
- ~~[ ] Product image gallery/zoom~~ **NOT AVAILABLE** - No detail view

### Shopping Cart
- [ ] Add product to cart from inventory page
- [ ] Cart badge shows item count in header
- [ ] View cart page (/cart)
- [ ] Increase product quantity in cart (+)
- [ ] Decrease product quantity in cart (-)
- [ ] Remove product from cart (trash button)
- [ ] Cart calculates subtotal correctly
- [ ] Cart calculates tax at 8%
- [ ] Cart calculates total correctly (subtotal + tax)
- [ ] Cart persists across page navigation
- [ ] Empty cart state handling
- [ ] Proceed to Checkout button navigates to /checkout

### Checkout Process
- [ ] Checkout page displays "Checkout: Your Information" heading
- [ ] First Name field entry
- [ ] Last Name field entry
- [ ] Postal Code field entry
- [ ] Country dropdown selection (5 countries available)
- [ ] Shipping method: Standard Shipping (5-7 days) - FREE
- [ ] Shipping method: Express Shipping (2-3 days) - $9.99
- [ ] Shipping method: Overnight Shipping (1 day) - $19.99
- [ ] Order summary displays correct totals on checkout
- [ ] Continue button completes order
- [ ] Cancel button returns to cart
- [ ] Form validation (required fields)
- ~~[ ] Payment method selection~~ **NOT AVAILABLE** - No payment processing
- ~~[ ] Credit card form validation~~ **NOT AVAILABLE** - No payment form
- ~~[ ] Billing address~~ **NOT AVAILABLE** - Only shipping info collected

### Order Completion
- [ ] Order confirmation page displays after checkout
- [ ] "Thank You For Your Order" message shown
- [ ] "Your order has been completed successfully" text present
- [ ] Back Home button visible
- [ ] Back Home logs user out and redirects to login
- [ ] Cart resets to 0 items after order completion
- ~~[ ] Order confirmation number~~ **NOT AVAILABLE** - No order number generated
- ~~[ ] Confirmation email~~ **NOT AVAILABLE** - No email system

### Order Management
- ~~[ ] Order history page~~ **NOT AVAILABLE** - No order history
- ~~[ ] Order detail view~~ **NOT AVAILABLE** - No order details
- ~~[ ] Order status tracking~~ **NOT AVAILABLE** - No tracking system

### User Account
- ~~[ ] Profile information management~~ **NOT AVAILABLE** - No account section
- ~~[ ] Address book management~~ **NOT AVAILABLE** - No saved addresses
- ~~[ ] Password change functionality~~ **NOT AVAILABLE** - No password management

## Out of Scope

- Mobile native app testing (web only)
- Backend API direct testing (UI-focused)
- Load/performance testing
- Payment gateway integration (no payment processing exists)
- Email delivery verification (no email system)
- User registration testing (feature doesn't exist)
- Product filtering/search (features don't exist)
- Order tracking/history (features don't exist)

## Test Environment

- **URL**: https://storzy.vercel.app
- **Credentials**: test_user / password
- **Browser**: Chromium (Playwright default)
- **Viewport**: 1280x720

## Discovered UI Selectors

### Login Page (/)
```
Username: getByRole('textbox', { name: 'Username' })
Password: getByRole('textbox', { name: 'Password' })
Login Button: getByRole('button', { name: 'Login' })
```

### Product Inventory (/inventory)
```
Sort Dropdown: getByRole('combobox')
Add to Cart: getByRole('button', { name: 'Add to Cart' }) - per product card
Cart Link: getByRole('link') with cart count
```

### Cart Page (/cart)
```
Quantity Decrease: button "-"
Quantity Increase: button "+"
Remove Item: button with trash icon
Proceed to Checkout: getByRole('button', { name: 'Proceed to Checkout' })
```

### Checkout Page (/checkout)
```
First Name: first textbox
Last Name: second textbox
Postal Code: third textbox
Country: getByRole('combobox')
Standard Shipping: getByRole('radio', { name: /Standard Shipping/ })
Express Shipping: getByRole('radio', { name: /Express Shipping/ })
Overnight Shipping: getByRole('radio', { name: /Overnight Shipping/ })
Continue: getByRole('button', { name: 'Continue' })
Cancel: getByRole('button', { name: 'Cancel' })
```

### Checkout Complete (/checkout-complete)
```
Back Home: getByRole('button', { name: 'Back Home' })
```

## Products Available (6 total)

| Product | Price |
|---------|-------|
| Baby Onesie | $7.99 |
| Bold Graphic Tee | TBD |
| Cozy Fleece Jacket | TBD |
| Explorer Backpack | TBD |
| Flight Jacket | TBD |
| LED Bike Light | TBD |

## Automation Priority

| Priority | Features |
|----------|----------|
| High | Login, Add to Cart, Complete Checkout Flow |
| Medium | Product Sorting, Cart Quantity Controls, Shipping Method Selection |
| Low | Form Validation, Cancel Checkout, Edge Cases |

## Notes

- Application auto-logs out after checkout completion
- Tax is fixed at 8% regardless of location
- No payment processing - orders complete instantly
- Credentials are displayed on login page for testing convenience
- See `./exploration-reports/` for detailed UI element discovery
- See `.bugzy/runtime/knowledge-base.md` for technical patterns
