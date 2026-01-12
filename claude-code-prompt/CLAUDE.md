# CLAUDE.md - Astro + Webflow DevLink + Headless Shopify Project

## Project Overview

This project is a **headless Shopify storefront** built with:
- **Astro** - Static site generator with islands architecture
- **Webflow DevLink** - Visual design tool that exports React components
- **Shopify Storefront API** - GraphQL API for product data, cart, and checkout
- **GitHub Actions** - CI/CD for automated deployment when designs change

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Webflow        │────▶│  DevLink CLI     │────▶│  React          │
│  Designer       │     │  (sync)          │     │  Components     │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                          │
┌─────────────────┐     ┌──────────────────┐     ┌────────▼────────┐
│  Shopify        │────▶│  Storefront API  │────▶│  Astro          │
│  Admin          │     │  (GraphQL)       │     │  Pages          │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                          │
┌─────────────────┐     ┌──────────────────┐     ┌────────▼────────┐
│  Vercel/        │◀────│  GitHub Actions  │◀────│  GitHub         │
│  Netlify        │     │  (CI/CD)         │     │  Repository     │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | Astro 4.x | Static/SSR site generation |
| UI Components | React 18 + Webflow DevLink | Visual design to code |
| Styling | Tailwind CSS | Utility-first CSS |
| State Management | Nanostores | Lightweight cross-framework state |
| E-commerce | Shopify Storefront API | Products, cart, checkout |
| Deployment | Vercel / Netlify / Webflow Cloud | Hosting & edge functions |
| CI/CD | GitHub Actions | Automated builds & deploys |

## Project Structure

```
/
├── .github/
│   └── workflows/
│       ├── deploy.yml           # Main deployment workflow
│       └── webflow-sync.yml     # Webflow DevLink sync workflow
├── src/
│   ├── components/
│   │   ├── webflow/             # DevLink synced components (DO NOT EDIT)
│   │   ├── astro/               # Astro components
│   │   └── react/               # Custom React components
│   ├── layouts/
│   │   └── BaseLayout.astro
│   ├── pages/
│   │   ├── index.astro
│   │   ├── products/
│   │   │   ├── index.astro      # Product listing
│   │   │   └── [handle].astro   # Product detail (dynamic)
│   │   ├── collections/
│   │   │   └── [handle].astro   # Collection pages
│   │   ├── cart.astro
│   │   └── api/
│   │       └── checkout.ts      # Checkout API endpoint
│   ├── lib/
│   │   ├── shopify/
│   │   │   ├── client.ts        # Shopify GraphQL client
│   │   │   ├── queries.ts       # GraphQL queries
│   │   │   ├── mutations.ts     # GraphQL mutations (cart)
│   │   │   └── types.ts         # TypeScript types
│   │   └── utils/
│   │       └── helpers.ts
│   ├── stores/
│   │   └── cart.ts              # Cart state (nanostores)
│   └── styles/
│       └── global.css
├── public/
│   └── assets/
├── devlink/                     # Webflow DevLink config
│   └── devlink.config.js
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
├── package.json
└── .env.example
```

## Environment Variables

Required environment variables (create `.env` file):

```env
# Shopify Configuration
PUBLIC_SHOPIFY_SHOP=your-store.myshopify.com
PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-public-token
SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-private-token
SHOPIFY_API_VERSION=2024-01

# Webflow DevLink
WEBFLOW_TOKEN=your-webflow-api-token
WEBFLOW_SITE_ID=your-site-id

# Deployment (choose one)
VERCEL_TOKEN=your-vercel-token
NETLIFY_AUTH_TOKEN=your-netlify-token

# GitHub (for webhook triggers)
GITHUB_PAT=your-github-personal-access-token
```

## Key Commands

```bash
# Development
pnpm dev                    # Start dev server (localhost:4321)
pnpm build                  # Build for production
pnpm preview                # Preview production build

# Webflow DevLink
pnpm devlink:sync           # Sync components from Webflow
pnpm devlink:watch          # Watch for Webflow changes

# Shopify
pnpm shopify:codegen        # Generate TypeScript types from schema

# Testing
pnpm test                   # Run tests
pnpm lint                   # Lint code
```

## Coding Guidelines

### Astro Components
- Use `.astro` files for static/server-rendered content
- Minimize client-side JavaScript
- Use `client:load` directive only when interactivity is required
- Prefer `client:visible` for below-the-fold interactive components

### React Components (Webflow DevLink)
- **DO NOT manually edit files in `/src/components/webflow/`** - they are auto-generated
- Create wrapper components in `/src/components/react/` to add logic
- Use TypeScript for all custom components
- Props should be typed with interfaces

### Shopify Integration
- Always use the Storefront API (not Admin API) for frontend
- Cache product data where possible using Astro's built-in caching
- Handle cart state client-side with nanostores
- Use optimistic updates for cart operations

### Styling
- Tailwind CSS is the primary styling method
- Webflow components come with their own styles - don't override unless necessary
- Use CSS custom properties for theming
- Mobile-first responsive design

### Performance
- Target Lighthouse scores: 90+ across all metrics
- Use Astro Image component for optimized images
- Implement proper loading states
- Lazy load non-critical components

## GraphQL Patterns

### Fetching Products
```typescript
// Use fragments for reusable product fields
const PRODUCT_FRAGMENT = `
  fragment ProductFields on Product {
    id
    handle
    title
    description
    priceRange {
      minVariantPrice {
        amount
        currencyCode
      }
    }
    images(first: 5) {
      edges {
        node {
          url
          altText
        }
      }
    }
  }
`;
```

### Cart Operations
- Cart is created on first add-to-cart action
- Cart ID is stored in localStorage
- Use mutations: `cartCreate`, `cartLinesAdd`, `cartLinesUpdate`, `cartLinesRemove`

## Deployment

### Vercel (Recommended)
- Auto-deploys on push to `main`
- Preview deployments for PRs
- Edge functions for API routes

### Webflow Cloud (Alternative)
- Native DevLink integration
- Supports Astro with server adapter
- Auto-deploys from connected GitHub branch

## Common Tasks

### Adding a New Page
1. Create `.astro` file in `/src/pages/`
2. Import layout and components
3. Fetch data in frontmatter (server-side)
4. Add to navigation if needed

### Adding Webflow Components
1. Design component in Webflow
2. Run `pnpm devlink:sync`
3. Import from `/src/components/webflow/`
4. Create wrapper if logic needed

### Modifying Cart Behavior
1. Edit `/src/stores/cart.ts`
2. Update mutations in `/src/lib/shopify/mutations.ts`
3. Test cart flow end-to-end

## Troubleshooting

| Issue | Solution |
|-------|----------|
| 401 from Shopify | Check API tokens and scopes |
| DevLink sync fails | Verify WEBFLOW_TOKEN and site access |
| Cart not persisting | Check localStorage and cart ID |
| Build fails on Vercel | Check env vars are set in Vercel dashboard |
| Images not loading | Verify Shopify image URLs are in `astro.config.mjs` domains |

## Resources

- [Astro Documentation](https://docs.astro.build)
- [Shopify Storefront API](https://shopify.dev/docs/api/storefront)
- [Webflow DevLink Docs](https://developers.webflow.com/devlink)
- [Nanostores](https://github.com/nanostores/nanostores)
