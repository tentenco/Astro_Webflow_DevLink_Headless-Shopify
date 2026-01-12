# 🛒 Astro + Webflow DevLink + Headless Shopify

<p align="center">
  <img src="https://img.shields.io/badge/Astro-4.x-BC52EE?style=for-the-badge&logo=astro&logoColor=white" alt="Astro" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Shopify-Storefront_API-7AB55C?style=for-the-badge&logo=shopify&logoColor=white" alt="Shopify" />
  <img src="https://img.shields.io/badge/Webflow-DevLink-4353FF?style=for-the-badge&logo=webflow&logoColor=white" alt="Webflow" />
  <img src="https://img.shields.io/badge/Tailwind-3.x-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind" />
</p>

<p align="center">
  A production-ready headless Shopify storefront that syncs UI designs from Webflow DevLink and auto-deploys via GitHub Actions.
</p>

---

## ✨ Features

- **⚡ Blazing Fast** - Static-first with Astro's islands architecture
- **🎨 Visual Design Workflow** - Designers edit in Webflow, code syncs automatically
- **🛍️ Full E-commerce** - Products, collections, cart, and Shopify checkout
- **🔄 Auto-Deploy** - Push to main or publish in Webflow → instant deployment
- **📱 Responsive** - Mobile-first design with Tailwind CSS
- **🔒 Type-Safe** - Full TypeScript coverage
- **💯 Performance** - 90+ Lighthouse scores out of the box

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           DESIGN LAYER                               │
├─────────────────┬───────────────────────────────────────────────────┤
│  Webflow        │  Visual component design & styling                 │
│  Designer       │  → Exports React components via DevLink            │
└────────┬────────┴───────────────────────────────────────────────────┘
         │ CLI Sync
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                            │
├─────────────────┬───────────────────────────────────────────────────┤
│  Astro          │  Static site generation + hybrid SSR               │
│  React          │  Interactive components (cart, variants)           │
│  Nanostores     │  Lightweight state management                      │
└────────┬────────┴───────────────────────────────────────────────────┘
         │ GraphQL
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER                                 │
├─────────────────┬───────────────────────────────────────────────────┤
│  Shopify        │  Products, collections, inventory                  │
│  Storefront API │  Cart management & checkout                        │
└─────────────────┴───────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT LAYER                              │
├─────────────────┬───────────────────────────────────────────────────┤
│  GitHub Actions │  CI/CD pipeline with DevLink sync                  │
│  Vercel         │  Edge deployment + serverless functions            │
└─────────────────┴───────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- Node.js 20+
- pnpm 8+
- Shopify store with [Headless channel](https://apps.shopify.com/headless) installed
- Webflow account with DevLink access (optional)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/astro-webflow-shopify.git
cd astro-webflow-shopify

# Install dependencies
pnpm install

# Copy environment variables
cp .env.example .env

# Start development server
pnpm dev
```

### Environment Setup

Create a `.env` file with your credentials:

```env
# Shopify Storefront API
PUBLIC_SHOPIFY_SHOP=your-store.myshopify.com
PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-public-token
SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-private-token
SHOPIFY_API_VERSION=2024-01

# Webflow DevLink (optional)
WEBFLOW_TOKEN=your-webflow-token
WEBFLOW_SITE_ID=your-site-id
```

---

## 📁 Project Structure

```
/
├── .github/
│   └── workflows/
│       ├── deploy.yml              # Main CI/CD workflow
│       └── preview.yml             # PR preview deployments
│
├── src/
│   ├── components/
│   │   ├── webflow/                # 🔒 Auto-generated (DO NOT EDIT)
│   │   │   ├── ProductCard.jsx
│   │   │   ├── Header.jsx
│   │   │   └── Footer.jsx
│   │   ├── react/                  # Custom React components
│   │   │   ├── CartDrawer.tsx
│   │   │   ├── VariantSelector.tsx
│   │   │   └── AddToCartButton.tsx
│   │   └── astro/                  # Astro components
│   │       ├── ProductGrid.astro
│   │       └── CollectionList.astro
│   │
│   ├── layouts/
│   │   └── BaseLayout.astro
│   │
│   ├── pages/
│   │   ├── index.astro             # Home page
│   │   ├── cart.astro              # Cart page
│   │   ├── products/
│   │   │   ├── index.astro         # Product listing
│   │   │   └── [handle].astro      # Product detail
│   │   ├── collections/
│   │   │   └── [handle].astro      # Collection page
│   │   └── api/
│   │       └── webhook.ts          # Webflow webhook handler
│   │
│   ├── lib/
│   │   └── shopify/
│   │       ├── client.ts           # GraphQL client
│   │       ├── queries.ts          # Product/collection queries
│   │       ├── mutations.ts        # Cart mutations
│   │       └── types.ts            # TypeScript definitions
│   │
│   ├── stores/
│   │   └── cart.ts                 # Cart state (nanostores)
│   │
│   └── styles/
│       ├── global.css
│       └── webflow.css             # DevLink styles
│
├── devlink/
│   └── devlink.config.js           # DevLink configuration
│
├── scripts/
│   └── sync-webflow.js             # DevLink sync script
│
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
└── package.json
```

---

## 🛠️ Development

### Commands

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start development server at `localhost:4321` |
| `pnpm build` | Build for production |
| `pnpm preview` | Preview production build locally |
| `pnpm devlink:sync` | Sync components from Webflow |
| `pnpm devlink:watch` | Watch for Webflow changes |
| `pnpm typecheck` | Run TypeScript checks |
| `pnpm lint` | Lint codebase |

### Webflow DevLink Workflow

1. **Designer** creates/updates components in Webflow
2. **Sync** components to codebase:
   ```bash
   pnpm devlink:sync
   ```
3. **Import** in your Astro/React files:
   ```tsx
   import { ProductCard } from '@/components/webflow/ProductCard';
   ```
4. **Wrap** with business logic if needed:
   ```tsx
   // src/components/react/ProductCardWrapper.tsx
   import { ProductCard } from '@/components/webflow/ProductCard';
   import { addToCart } from '@/stores/cart';
   
   export function ProductCardWrapper({ product }) {
     return (
       <ProductCard
         {...product}
         onAddToCart={() => addToCart(product.variantId, 1)}
       />
     );
   }
   ```

> ⚠️ **Important:** Never manually edit files in `/src/components/webflow/` - they will be overwritten on sync.

---

## 🛍️ Shopify Integration

### Setting Up Shopify

1. Install the [Headless channel](https://apps.shopify.com/headless) in your Shopify admin
2. Create a new Storefront
3. Copy the **Public** and **Private** access tokens
4. Configure API scopes:
   - `unauthenticated_read_product_listings`
   - `unauthenticated_read_product_inventory`
   - `unauthenticated_read_checkouts`
   - `unauthenticated_write_checkouts`

### GraphQL Usage

```typescript
// Fetch products
import { shopifyClient } from '@/lib/shopify/client';
import { GET_PRODUCTS } from '@/lib/shopify/queries';

const { products } = await shopifyClient.request(GET_PRODUCTS, {
  first: 12,
});

// Add to cart
import { addToCart } from '@/stores/cart';

await addToCart(variantId, quantity);
```

### Cart State

Cart is managed client-side with [nanostores](https://github.com/nanostores/nanostores):

```typescript
import { useStore } from '@nanostores/react';
import { $cart, $cartItemCount, $isCartOpen } from '@/stores/cart';

function CartIcon() {
  const itemCount = useStore($cartItemCount);
  const isOpen = useStore($isCartOpen);
  
  return (
    <button onClick={() => $isCartOpen.set(true)}>
      Cart ({itemCount})
    </button>
  );
}
```

---

## 🚢 Deployment

### Vercel (Recommended)

1. Connect your GitHub repository to Vercel
2. Add environment variables in Vercel dashboard
3. Deploy!

```bash
# Or deploy manually
pnpm build
vercel --prod
```

### GitHub Actions

The included workflow (`.github/workflows/deploy.yml`) automatically:

1. ✅ Syncs Webflow DevLink components
2. ✅ Builds the Astro site
3. ✅ Deploys to Vercel on push to `main`
4. ✅ Creates preview deployments for PRs

### Webflow Webhook Auto-Deploy

To trigger deployments when designers publish in Webflow:

1. Set up a webhook in Webflow pointing to:
   ```
   https://your-domain.com/api/webhook
   ```

2. Add GitHub PAT to your environment:
   ```env
   GITHUB_PAT=your-github-personal-access-token
   GITHUB_REPO=your-org/your-repo
   ```

3. Webhooks will trigger the `repository_dispatch` event in GitHub Actions

---

## ⚡ Performance

This template is optimized for performance:

| Metric | Target | How |
|--------|--------|-----|
| **LCP** | < 2.5s | Static generation, optimized images |
| **FID** | < 100ms | Minimal client JS, islands architecture |
| **CLS** | < 0.1 | Proper image dimensions, no layout shifts |
| **TTI** | < 3.5s | `client:visible` for below-fold components |

### Optimization Tips

```astro
<!-- Use client:visible for non-critical interactivity -->
<ProductCard client:visible product={product} />

<!-- Use client:idle for secondary features -->
<RecentlyViewed client:idle />

<!-- Use client:load only when immediately needed -->
<CartDrawer client:load />
```

---

## 🧪 Testing

```bash
# Type checking
pnpm typecheck

# Linting
pnpm lint

# Build test
pnpm build
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

### Code Style

- Use TypeScript for all new files
- Follow existing patterns for Shopify queries/mutations
- Never edit `/src/components/webflow/` directly
- Add JSDoc comments for complex functions
- Run `pnpm lint` before committing

---

## 📚 Resources

- [Astro Documentation](https://docs.astro.build)
- [Shopify Storefront API Reference](https://shopify.dev/docs/api/storefront)
- [Webflow DevLink Documentation](https://developers.webflow.com/devlink)
- [Nanostores Documentation](https://github.com/nanostores/nanostores)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- [thomasKn/astro-shopify](https://github.com/thomasKn/astro-shopify) - Original Astro Shopify starter
- [Webflow](https://webflow.com) - DevLink visual development
- [Shopify](https://shopify.com) - Headless commerce platform

---

<p align="center">
  Built with ❤️ using Astro, React, and Shopify
</p>
