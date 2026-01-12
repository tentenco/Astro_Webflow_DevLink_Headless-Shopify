# PROJECT_PROMPT.md - Build Instructions for Claude Code

## Project Goal

Build a **production-ready headless Shopify storefront** using Astro that integrates with Webflow DevLink for visual design management and deploys automatically via GitHub Actions when designers make changes in Webflow.

---

## Phase 1: Project Setup

### 1.1 Initialize Astro Project

Create a new Astro project with the following configuration:

```bash
pnpm create astro@latest astro-webflow-shopify --template minimal --typescript strict --git
cd astro-webflow-shopify
```

### 1.2 Install Dependencies

```bash
# Core dependencies
pnpm add @astrojs/react @astrojs/tailwind react react-dom nanostores @nanostores/react

# Shopify
pnpm add @shopify/hydrogen-react graphql graphql-request

# DevLink (when available - use placeholder for now)
pnpm add @webflow/webflow-cli

# Dev dependencies
pnpm add -D @types/react @types/react-dom typescript @graphql-codegen/cli @graphql-codegen/typescript
```

### 1.3 Configure Astro

Create `astro.config.mjs`:

```javascript
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';
import vercel from '@astrojs/vercel/serverless';

export default defineConfig({
  output: 'hybrid', // Static by default, opt-in to SSR per page
  adapter: vercel(),
  integrations: [
    react(),
    tailwind(),
  ],
  image: {
    domains: ['cdn.shopify.com'],
  },
  vite: {
    ssr: {
      noExternal: ['@shopify/hydrogen-react'],
    },
  },
});
```

---

## Phase 2: Shopify Integration

### 2.1 Create Shopify Client

Create `/src/lib/shopify/client.ts`:

```typescript
import { GraphQLClient } from 'graphql-request';

const endpoint = `https://${import.meta.env.PUBLIC_SHOPIFY_SHOP}/api/${import.meta.env.SHOPIFY_API_VERSION || '2024-01'}/graphql.json`;

export const shopifyClient = new GraphQLClient(endpoint, {
  headers: {
    'X-Shopify-Storefront-Access-Token': import.meta.env.PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN,
    'Content-Type': 'application/json',
  },
});

// Server-side client with private token
export const shopifyServerClient = new GraphQLClient(endpoint, {
  headers: {
    'X-Shopify-Storefront-Access-Token': import.meta.env.SHOPIFY_STOREFRONT_ACCESS_TOKEN,
    'Content-Type': 'application/json',
  },
});
```

### 2.2 Create GraphQL Queries

Create `/src/lib/shopify/queries.ts` with:

- `GET_PRODUCTS` - Fetch all products with pagination
- `GET_PRODUCT_BY_HANDLE` - Single product by handle
- `GET_COLLECTIONS` - All collections
- `GET_COLLECTION_BY_HANDLE` - Single collection with products
- `GET_CART` - Cart by ID

### 2.3 Create GraphQL Mutations

Create `/src/lib/shopify/mutations.ts` with:

- `CREATE_CART` - Initialize new cart
- `ADD_TO_CART` - Add line items
- `UPDATE_CART` - Update quantities
- `REMOVE_FROM_CART` - Remove line items

### 2.4 Create TypeScript Types

Create `/src/lib/shopify/types.ts` with interfaces for:

- `Product`, `ProductVariant`, `ProductImage`
- `Collection`
- `Cart`, `CartLine`, `CartCost`
- `Money`
- API response types

---

## Phase 3: Cart State Management

### 3.1 Create Cart Store

Create `/src/stores/cart.ts` using nanostores:

```typescript
import { atom, computed } from 'nanostores';
import type { Cart, CartLine } from '@/lib/shopify/types';

// Cart state
export const $cart = atom<Cart | null>(null);
export const $isCartOpen = atom(false);
export const $isCartLoading = atom(false);

// Computed values
export const $cartItemCount = computed($cart, (cart) => {
  if (!cart) return 0;
  return cart.lines.edges.reduce((sum, { node }) => sum + node.quantity, 0);
});

export const $cartTotal = computed($cart, (cart) => {
  if (!cart) return '0.00';
  return cart.cost.totalAmount.amount;
});

// Actions
export async function initializeCart() { /* ... */ }
export async function addToCart(variantId: string, quantity: number) { /* ... */ }
export async function updateCartLine(lineId: string, quantity: number) { /* ... */ }
export async function removeFromCart(lineId: string) { /* ... */ }
```

---

## Phase 4: Webflow DevLink Integration

### 4.1 DevLink Configuration

Create `/devlink/devlink.config.js`:

```javascript
module.exports = {
  host: "https://api.webflow.com",
  siteId: process.env.WEBFLOW_SITE_ID,
  authToken: process.env.WEBFLOW_TOKEN,
  outputDir: "./src/components/webflow",
  cssOutput: "./src/styles/webflow.css",
};
```

### 4.2 Create DevLink Sync Script

Create `/scripts/sync-webflow.js`:

```javascript
#!/usr/bin/env node
const { execSync } = require('child_process');
const path = require('path');

console.log('🔄 Syncing Webflow DevLink components...');

try {
  execSync('npx webflow devlink sync', {
    stdio: 'inherit',
    cwd: path.resolve(__dirname, '..'),
  });
  console.log('✅ DevLink sync complete!');
} catch (error) {
  console.error('❌ DevLink sync failed:', error.message);
  process.exit(1);
}
```

### 4.3 Component Wrapper Pattern

Create wrapper components to add logic to Webflow components:

```typescript
// /src/components/react/ProductCardWrapper.tsx
import { ProductCard } from '@/components/webflow/ProductCard';
import { addToCart } from '@/stores/cart';
import type { Product } from '@/lib/shopify/types';

interface Props {
  product: Product;
}

export function ProductCardWrapper({ product }: Props) {
  const handleAddToCart = () => {
    const variantId = product.variants.edges[0]?.node.id;
    if (variantId) {
      addToCart(variantId, 1);
    }
  };

  return (
    <ProductCard
      title={product.title}
      price={product.priceRange.minVariantPrice.amount}
      imageUrl={product.images.edges[0]?.node.url}
      onAddToCart={handleAddToCart}
    />
  );
}
```

---

## Phase 5: Page Templates

### 5.1 Create Base Layout

Create `/src/layouts/BaseLayout.astro`:

```astro
---
import '@/styles/global.css';
import Header from '@/components/astro/Header.astro';
import Footer from '@/components/astro/Footer.astro';
import CartDrawer from '@/components/react/CartDrawer';

interface Props {
  title: string;
  description?: string;
}

const { title, description = 'Headless Shopify Storefront' } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content={description} />
    <title>{title}</title>
  </head>
  <body>
    <Header />
    <main>
      <slot />
    </main>
    <Footer />
    <CartDrawer client:load />
  </body>
</html>
```

### 5.2 Create Pages

Create these pages:

| Page | Path | Purpose |
|------|------|---------|
| Home | `/src/pages/index.astro` | Featured products, hero, collections |
| Products | `/src/pages/products/index.astro` | Product listing with filters |
| Product Detail | `/src/pages/products/[handle].astro` | Single product with variants |
| Collections | `/src/pages/collections/[handle].astro` | Collection product grid |
| Cart | `/src/pages/cart.astro` | Full cart page |

---

## Phase 6: GitHub Actions CI/CD

### 6.1 Main Deploy Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  repository_dispatch:
    types: [webflow-sync]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Sync Webflow DevLink
        env:
          WEBFLOW_TOKEN: ${{ secrets.WEBFLOW_TOKEN }}
          WEBFLOW_SITE_ID: ${{ secrets.WEBFLOW_SITE_ID }}
        run: pnpm devlink:sync
        continue-on-error: true

      - name: Build
        env:
          PUBLIC_SHOPIFY_SHOP: ${{ secrets.PUBLIC_SHOPIFY_SHOP }}
          PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN: ${{ secrets.PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN }}
          SHOPIFY_STOREFRONT_ACCESS_TOKEN: ${{ secrets.SHOPIFY_STOREFRONT_ACCESS_TOKEN }}
        run: pnpm build

      - name: Deploy to Vercel
        if: github.ref == 'refs/heads/main'
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### 6.2 Webflow Webhook Handler

Create `/src/pages/api/webflow-webhook.ts`:

```typescript
import type { APIRoute } from 'astro';

export const POST: APIRoute = async ({ request }) => {
  const signature = request.headers.get('x-webflow-signature');
  
  // Verify webhook signature (implement your verification)
  if (!verifyWebflowSignature(signature, await request.text())) {
    return new Response('Unauthorized', { status: 401 });
  }

  // Trigger GitHub Actions workflow
  const response = await fetch(
    `https://api.github.com/repos/${import.meta.env.GITHUB_REPO}/dispatches`,
    {
      method: 'POST',
      headers: {
        'Authorization': `token ${import.meta.env.GITHUB_PAT}`,
        'Accept': 'application/vnd.github.v3+json',
      },
      body: JSON.stringify({
        event_type: 'webflow-sync',
        client_payload: {
          timestamp: new Date().toISOString(),
        },
      }),
    }
  );

  if (response.ok) {
    return new Response(JSON.stringify({ success: true }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  return new Response('Failed to trigger deployment', { status: 500 });
};

function verifyWebflowSignature(signature: string | null, body: string): boolean {
  // Implement signature verification
  return true; // Placeholder
}
```

---

## Phase 7: Package.json Scripts

Update `package.json` with these scripts:

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "start": "astro dev",
    "devlink:sync": "node scripts/sync-webflow.js",
    "devlink:watch": "nodemon --watch devlink -e js,json --exec 'pnpm devlink:sync'",
    "shopify:codegen": "graphql-codegen --config codegen.yml",
    "lint": "eslint src --ext .ts,.tsx,.astro",
    "lint:fix": "eslint src --ext .ts,.tsx,.astro --fix",
    "typecheck": "astro check && tsc --noEmit"
  }
}
```

---

## Phase 8: Deployment Configuration

### 8.1 Vercel Configuration

Create `vercel.json`:

```json
{
  "framework": "astro",
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "installCommand": "pnpm install",
  "env": {
    "PUBLIC_SHOPIFY_SHOP": "@shopify-shop",
    "PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN": "@shopify-public-token"
  }
}
```

### 8.2 Environment Variables Setup

Create `.env.example`:

```env
# Shopify Storefront API
PUBLIC_SHOPIFY_SHOP=your-store.myshopify.com
PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN=xxxxx
SHOPIFY_STOREFRONT_ACCESS_TOKEN=xxxxx
SHOPIFY_API_VERSION=2024-01

# Webflow DevLink
WEBFLOW_TOKEN=xxxxx
WEBFLOW_SITE_ID=xxxxx

# GitHub (for webhook)
GITHUB_REPO=your-org/your-repo
GITHUB_PAT=xxxxx

# Deployment
VERCEL_TOKEN=xxxxx
VERCEL_ORG_ID=xxxxx
VERCEL_PROJECT_ID=xxxxx
```

---

## Acceptance Criteria

### Functionality
- [ ] Products load from Shopify Storefront API
- [ ] Product detail pages with variant selection
- [ ] Add to cart works with optimistic updates
- [ ] Cart persists across page refreshes (localStorage)
- [ ] Checkout redirects to Shopify checkout
- [ ] Webflow components render correctly
- [ ] DevLink sync updates components without manual intervention

### Performance
- [ ] Lighthouse Performance score > 90
- [ ] First Contentful Paint < 1.5s
- [ ] Time to Interactive < 3s
- [ ] Images optimized with Astro Image

### CI/CD
- [ ] GitHub Actions builds on push to main
- [ ] Webflow webhook triggers rebuild
- [ ] Vercel deploys successfully
- [ ] Environment variables properly configured

### Code Quality
- [ ] TypeScript strict mode enabled
- [ ] No TypeScript errors
- [ ] ESLint passes
- [ ] Components properly typed

---

## Implementation Order

1. **Setup** - Initialize project, install deps, configure Astro
2. **Shopify** - Create client, queries, mutations, types
3. **Cart** - Implement cart store with nanostores
4. **Layout** - Create base layout and navigation
5. **Pages** - Build home, products, product detail, cart pages
6. **DevLink** - Configure and integrate Webflow components
7. **Styling** - Tailwind setup, global styles
8. **API** - Webhook endpoint for Webflow
9. **CI/CD** - GitHub Actions workflows
10. **Testing** - Manual testing, Lighthouse audit
11. **Deploy** - Vercel production deployment

---

## Notes for Claude Code

- Start with Phase 1 and proceed sequentially
- Ask clarifying questions if Shopify credentials are needed
- Create placeholder Webflow components if DevLink is not accessible
- Use mock data for development if Shopify store is not available
- Prioritize TypeScript types and error handling
- Add comments explaining complex logic
- Follow Astro best practices for client directives
