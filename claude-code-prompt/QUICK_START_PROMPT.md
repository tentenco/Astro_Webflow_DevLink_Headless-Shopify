# QUICK_START_PROMPT.md

Use this prompt directly in Claude Code to start building:

---

## Prompt for Claude Code

```
Build a headless Shopify storefront with the following stack:

**Tech Stack:**
- Astro 4.x with hybrid rendering
- React 18 for interactive components
- Tailwind CSS for styling
- Nanostores for cart state
- Shopify Storefront API (GraphQL)
- Webflow DevLink integration (React components)
- GitHub Actions for CI/CD
- Vercel for deployment

**Requirements:**

1. **Shopify Integration:**
   - GraphQL client for Storefront API
   - Queries: products, collections, product by handle
   - Mutations: cart create, add/update/remove items
   - TypeScript types for all Shopify data

2. **Pages to Create:**
   - Home page with featured products
   - /products - Product listing with grid
   - /products/[handle] - Product detail with variant selector
   - /collections/[handle] - Collection pages
   - /cart - Full cart page

3. **Cart Functionality:**
   - Client-side cart state with nanostores
   - Add to cart with optimistic updates
   - Cart drawer component
   - Persist cart ID in localStorage
   - Redirect to Shopify checkout

4. **Webflow DevLink:**
   - Configure DevLink CLI
   - Output components to /src/components/webflow/
   - Create wrapper components for business logic
   - Sync script for CI/CD

5. **GitHub Actions:**
   - Build on push to main
   - Sync DevLink components before build
   - Deploy to Vercel
   - Webhook endpoint to trigger on Webflow publish

6. **Performance:**
   - Static generation where possible
   - client:visible for below-fold interactivity
   - Optimized images with Astro Image
   - Target 90+ Lighthouse score

**Project Structure:**
```
/src
  /components
    /webflow    # DevLink components (auto-generated)
    /react      # Custom React components
    /astro      # Astro components
  /layouts
  /pages
  /lib/shopify  # Client, queries, mutations, types
  /stores       # Nanostores (cart)
  /styles
/.github/workflows
/devlink
```

**Environment Variables Needed:**
- PUBLIC_SHOPIFY_SHOP
- PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN
- SHOPIFY_STOREFRONT_ACCESS_TOKEN
- WEBFLOW_TOKEN
- WEBFLOW_SITE_ID

Start with project setup and Shopify integration first.
```

---

## Alternative: Step-by-Step Prompts

### Step 1: Initialize Project
```
Create a new Astro project with React, Tailwind CSS, and TypeScript. 
Configure for hybrid rendering with Vercel adapter.
Set up the project structure for a headless Shopify storefront.
```

### Step 2: Shopify Client
```
Create a Shopify Storefront API client using graphql-request.
Include queries for products, collections, and single product by handle.
Include mutations for cart operations (create, add, update, remove).
Add TypeScript types for all Shopify data structures.
```

### Step 3: Cart Store
```
Create a cart store using nanostores with:
- Cart state atom
- Cart open/loading states
- Computed values for item count and total
- Actions: initializeCart, addToCart, updateCartLine, removeFromCart
- localStorage persistence for cart ID
```

### Step 4: Pages
```
Create Astro pages for:
- Home page with featured products grid
- Products listing page
- Product detail page with dynamic [handle] route
- Collection page with dynamic [handle] route
- Cart page with line items and checkout button
```

### Step 5: DevLink Integration
```
Set up Webflow DevLink configuration.
Create a sync script for CI/CD.
Add wrapper components pattern for Webflow components.
Configure output directory to /src/components/webflow/
```

### Step 6: GitHub Actions
```
Create GitHub Actions workflow that:
- Runs on push to main and repository_dispatch
- Installs pnpm dependencies
- Syncs Webflow DevLink components
- Builds Astro site
- Deploys to Vercel
Also create webhook API endpoint for Webflow publish events.
```

---

## Minimum Viable Prompt

For a quick start, use this condensed prompt:

```
Create a headless Shopify storefront using Astro + React + Tailwind.

Include:
1. Shopify Storefront API client (GraphQL)
2. Product listing and detail pages
3. Cart with nanostores (add/remove/update)
4. Webflow DevLink component integration
5. GitHub Actions for CI/CD to Vercel

Use TypeScript throughout. Target Lighthouse 90+.
```
