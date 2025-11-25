# CLAUDE.md - AI Assistant Guide for Cycle Paradise

> **Last Updated**: 2025-11-25
> **Project**: Cycle Paradise - Sri Lanka Cycling Tours
> **Framework**: Astro 4.15+ with TypeScript, React, TailwindCSS, Prisma ORM

This document provides AI assistants with comprehensive guidance for working with the Cycle Paradise codebase. It covers architecture, conventions, workflows, and critical rules.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Tech Stack & Dependencies](#tech-stack--dependencies)
4. [Critical Rules & Conventions](#critical-rules--conventions)
5. [Database & Data Layer](#database--data-layer)
6. [Development Workflows](#development-workflows)
7. [Testing Strategy](#testing-strategy)
8. [Common Tasks & Recipes](#common-tasks--recipes)
9. [Quality Gates](#quality-gates)
10. [File References](#file-references)

---

## 🎯 Project Overview

**Cycle Paradise** is a production-ready, SEO-optimized cycling tour website for Sri Lanka. It's built with Astro's islands architecture for optimal performance and features a unique dual-mode operation that works with or without a database.

### Key Features
- **Dual-mode operation**: Works with PostgreSQL OR static fallback data
- **SEO-optimized**: Static site generation with rich meta tags
- **Performance-first**: Astro's partial hydration, service worker caching
- **Offline-ready**: Service worker with fallback content
- **Rich content**: Tour packages, cycling guides, galleries, Instagram feeds
- **Admin panel**: Session-based authentication for content management
- **Email integration**: NodeMailer with multiple SMTP provider support
- **Image optimization**: Sharp-powered responsive images

### Project Philosophy
From `.specify/memory/constitution.md`:
- **Code Quality Standards**: Zero tolerance for linting/type errors, self-documenting code
- **Test-Driven Development**: Tests before implementation (target: 90%+ coverage)
- **User Experience Consistency**: WCAG 2.1 AA compliance mandatory
- **Performance Requirements**: <2s page loads, <200ms API responses (95th percentile)
- **Documentation-First**: Every feature starts with comprehensive documentation

---

## 📁 Codebase Structure

```
/home/user/cycleparadise/
├── .github/                    # GitHub configuration
│   ├── prompts/               # SpecKit AI workflow prompts (8 files)
│   └── copilot-instructions.md # Copilot guidelines
├── .githooks/                  # Pre-commit hooks (bash + PowerShell)
├── .specify/                   # SpecKit project management framework
│   ├── memory/                # Project constitution & guidelines
│   ├── scripts/               # PowerShell automation scripts
│   └── templates/             # Feature spec templates
├── .vscode/                    # VS Code settings (ESLint, Prettier, etc.)
├── prisma/                     # Database schema & migrations
│   ├── schema.prisma          # 9 models, 5 enums
│   └── seed.ts                # Database seeding script
├── public/                     # Static assets
│   ├── sw.js                  # Service worker (offline support)
│   └── images/                # Static images
├── scripts/                    # Custom validation scripts
│   └── validate.mjs           # Reserved word & syntax checker
├── specs/                      # Feature specifications
│   └── 001-cycling-tour-website/ # Main spec (50k+ lines docs)
├── src/                        # Application source code
│   ├── components/            # Reusable UI components
│   │   ├── analytics/         # PerformanceMonitor.astro
│   │   ├── media/             # InstagramFeed, YouTubeEmbed, PackageGallery
│   │   └── ui/                # PackageCard, OptimizedImage
│   ├── data/                  # **FALLBACK STATIC DATA** (works without DB)
│   │   ├── fallback-tour-packages.ts     # 3 sample packages
│   │   └── fallback-cycling-guides.ts    # 3 sample guides
│   ├── layouts/               # Page layout templates
│   │   └── BaseLayout.astro   # Main layout with SEO, meta tags
│   ├── lib/                   # Core library code
│   │   ├── auth/              # Session management
│   │   ├── db/                # Database client & repositories
│   │   │   ├── client.ts      # Prisma client wrapper
│   │   │   ├── connection.ts  # Singleton connection
│   │   │   └── repositories/  # Data access layer
│   │   ├── email/             # NodeMailer email service
│   │   ├── errors/            # Custom error classes
│   │   ├── media/             # Sharp image optimizer
│   │   ├── services/          # External APIs (Instagram)
│   │   └── utils/             # Utility functions
│   ├── pages/                 # Astro pages (file-based routing)
│   │   ├── api/               # API endpoints
│   │   ├── guides/            # Cycling guides
│   │   │   ├── [slug].astro   # Dynamic guide pages
│   │   │   └── index.astro    # Guide listing
│   │   ├── packages/          # Tour packages
│   │   │   ├── [slug].astro   # Dynamic package pages
│   │   │   └── index.astro    # Package listing
│   │   ├── about.astro        # About page
│   │   ├── contact.astro      # Contact form
│   │   ├── index.astro        # Homepage
│   │   ├── offline.astro      # Offline fallback
│   │   └── search.astro       # Site search
│   └── types/                 # TypeScript type definitions
│       └── models.ts          # Domain models
├── tests/                      # Test directory
│   └── e2e/                   # Playwright E2E tests
├── .eslintrc.json             # ESLint configuration (CRITICAL: reserved word rules)
├── .prettierrc                # Prettier code formatting
├── astro.config.mjs           # Astro framework config
├── docker-compose.yml         # Docker services
├── Dockerfile                 # Multi-stage production build
├── package.json               # Dependencies & scripts (27 scripts)
├── playwright.config.ts       # E2E test configuration
├── tailwind.config.mjs        # Tailwind CSS config
├── tsconfig.json              # TypeScript strict mode
├── vitest.config.ts           # Unit test configuration
├── ADMIN_GUIDE.md             # Admin operations manual (657 lines)
├── ERROR_PREVENTION.md        # Error prevention guide (211 lines)
└── README.md                  # Project documentation (557 lines)
```

### Key Directories Explained

#### `/src/data/` - Fallback Data System ⭐
**CRITICAL**: This directory contains static TypeScript data that allows the site to function WITHOUT a database connection. Always maintain parity between fallback data structure and Prisma schema.

#### `/src/lib/db/repositories/` - Repository Pattern ⭐
Data access layer that abstracts Prisma from business logic. All database queries go through repositories, never direct Prisma calls in pages.

#### `/src/pages/` - File-based Routing ⭐
Astro's file-based routing. Dynamic routes like `[slug].astro` **MUST** export `getStaticPaths()` for static builds.

---

## 🛠️ Tech Stack & Dependencies

### Core Framework (Astro Islands Architecture)
```json
{
  "astro": "^4.15.0",           // Static site generator with islands
  "typescript": "^5.2.0",        // Strict mode enabled
  "node": "20+"                  // Required Node.js version
}
```

### Frontend Layer
```json
{
  "@astrojs/react": "^3.6.0",   // React integration for islands
  "react": "^18.3.0",            // Interactive components only
  "react-dom": "^18.3.0",
  "@astrojs/tailwind": "^5.1.0", // Tailwind integration
  "tailwindcss": "^3.4.0",       // Utility-first CSS
  "@tailwindcss/forms": "^0.5.10",      // Form styling
  "@tailwindcss/typography": "^0.5.19"  // Rich text styling
}
```

### Backend & Database
```json
{
  "@prisma/client": "^5.20.0",   // Prisma ORM client
  "prisma": "^5.20.0",            // Prisma CLI & migrations
  "bcrypt": "^5.1.1",             // Password hashing (admin auth)
  "nodemailer": "^6.9.0",         // Email service (SMTP)
  "sharp": "^0.33.0"              // Image optimization
}
```

### Testing & Quality Tools
```json
{
  "vitest": "^1.6.0",                    // Unit testing
  "@vitest/ui": "^1.6.0",                // Test UI
  "@vitest/coverage-v8": "^1.6.0",       // Coverage reports
  "@playwright/test": "^1.40.0",         // E2E testing
  "eslint": "^8.57.0",                   // Linting
  "@typescript-eslint/*": "^6.21.0",     // TypeScript linting
  "eslint-plugin-astro": "^0.29.0",      // Astro linting
  "prettier": "^3.1.0",                  // Code formatting
  "prettier-plugin-astro": "^0.12.0"     // Astro formatting
}
```

### Build & Deployment
```json
{
  "tsx": "^4.6.0",              // TypeScript execution (seeding, scripts)
  "@astrojs/sitemap": "^3.1.0"  // Auto sitemap generation
}
```

---

## 🚨 Critical Rules & Conventions

### 1. Reserved Word Usage - **ABSOLUTELY FORBIDDEN** ⛔

**NEVER use `package` as a variable name.** This is a JavaScript reserved word and will cause build failures.

#### ❌ WRONG - Will Fail Build
```typescript
// NEVER DO THIS
packages.map((package) => <PackageCard package={package} />)
const package = await repository.findBySlug(slug);
```

#### ✅ CORRECT - Use These Alternatives
```typescript
// Use descriptive alternatives
packages.map((tourPackage) => <PackageCard package={tourPackage} />)
packages.map((pkg) => <PackageCard package={pkg} />)
packages.map((item) => <PackageCard package={item} />)

const tourPackage = await repository.findBySlug(slug);
const pkg = await repository.findBySlug(slug);
```

**Enforcement**:
- ESLint rule: `no-restricted-syntax` catches `package` usage
- Custom validation: `scripts/validate.mjs` scans for patterns
- Pre-commit hook: Prevents commits with reserved words
- See: `ERROR_PREVENTION.md:7-22`

### 2. Dynamic Routes Require `getStaticPaths()` - **MANDATORY** ⚠️

Any Astro file with `[slug].astro` or `[param].astro` **MUST** export `getStaticPaths()` for static builds.

#### ✅ Required Pattern
```typescript
// src/pages/packages/[slug].astro
import type { GetStaticPaths } from 'astro';

export const getStaticPaths: GetStaticPaths = async () => {
  // Check if database is available
  if (!import.meta.env.DATABASE_URL) {
    // Use fallback data
    const { fallbackTourPackages } = await import('../../data/fallback-tour-packages');
    return fallbackTourPackages.map((pkg) => ({
      params: { slug: pkg.slug },
    }));
  }

  // Use database
  const repository = new TourPackageRepository();
  try {
    const result = await repository.findMany({ limit: 100 });
    return result.items.map((pkg) => ({
      params: { slug: pkg.slug },
    }));
  } catch (error) {
    console.error('Error generating static paths:', error);
    return [];
  }
};
```

**Key Points**:
- Always handle both database and fallback data modes
- Return empty array on error (graceful degradation)
- Use descriptive variable names (NOT `package`)
- See: `ERROR_PREVENTION.md:46-62`

### 3. Dual-Mode Data Loading - **CRITICAL PATTERN** 🔄

The site operates in two modes based on `DATABASE_URL` presence:

#### Mode 1: Database Mode (PostgreSQL + Prisma)
```typescript
// Check for database availability
if (import.meta.env.DATABASE_URL) {
  const repository = new TourPackageRepository();
  const result = await repository.findMany({ limit: 10 });
  packages = result.items;
}
```

#### Mode 2: Fallback Mode (Static TypeScript Data)
```typescript
else {
  const { fallbackTourPackages } = await import('../data/fallback-tour-packages');
  packages = fallbackTourPackages;
}
```

#### Combined Pattern (Use This)
```typescript
// src/pages/packages.astro
let packages: TourPackage[] = [];

if (import.meta.env.DATABASE_URL) {
  try {
    const repository = new TourPackageRepository();
    const result = await repository.findMany({ featured: true, limit: 10 });
    packages = result.items;
  } catch (error) {
    console.error('Database error, falling back to static data:', error);
    const { fallbackTourPackages } = await import('../data/fallback-tour-packages');
    packages = fallbackTourPackages;
  }
} else {
  const { fallbackTourPackages } = await import('../data/fallback-tour-packages');
  packages = fallbackTourPackages;
}
```

**Why This Matters**:
- Site works without database setup (developer experience)
- Deployment flexibility (static hosting or full stack)
- Graceful degradation on database errors
- No code changes needed between modes

### 4. Naming Conventions

#### Files
- **Components**: PascalCase - `PackageCard.astro`, `YouTubeEmbed.astro`
- **Pages**: kebab-case - `packages.astro`, `cycling-guides.astro`
- **Utilities**: kebab-case - `prisma-converters.ts`, `image-optimizer.ts`
- **Types**: kebab-case - `models.ts`, `api-types.ts`

#### Variables & Functions
- **camelCase**: `tourPackage`, `findBySlug()`, `validateEmail()`
- **PascalCase**: Classes, interfaces, types - `TourPackageRepository`, `BookingStatus`
- **SCREAMING_SNAKE_CASE**: Constants - `MAX_IMAGE_SIZE`, `SMTP_PORT`

#### Database Fields
- **Prisma schema**: snake_case - `base_price`, `short_description`, `created_at`
- **TypeScript/JavaScript**: camelCase - `basePrice`, `shortDescription`, `createdAt`
- **Automatic conversion**: Prisma handles mapping between conventions

### 5. TypeScript Strict Mode - **ENFORCED** ✅

TypeScript strict mode is enabled in `tsconfig.json`. All code must:
- Explicitly type function parameters and return values
- Avoid `any` type (use `unknown` or proper types)
- Handle null/undefined with proper checks
- Use type guards for runtime validation

```typescript
// ✅ Good
async function findBySlug(slug: string): Promise<TourPackage | null> {
  if (!slug || typeof slug !== 'string') {
    throw new ValidationError('Invalid slug parameter');
  }
  // ...
}

// ❌ Bad - Missing types
async function findBySlug(slug) {
  // ...
}
```

### 6. Code Style (Prettier + ESLint)

**Prettier Configuration** (`.prettierrc`):
```json
{
  "semi": true,
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

**Key ESLint Rules** (`.eslintrc.json`):
- No `console.log` (use `console.error`, `console.warn`)
- Prefer `const` over `let`, never use `var`
- Strict equality (`===`, `!==`)
- All functions need braces
- Max line length: 100 characters

**Commands**:
```bash
npm run lint          # Check for issues
npm run lint:fix      # Auto-fix issues
npm run format        # Format with Prettier
```

---

## 💾 Database & Data Layer

### Database Technology
- **PostgreSQL** with Prisma ORM
- **Schema Location**: `/home/user/cycleparadise/prisma/schema.prisma`
- **Connection**: Singleton pattern in `src/lib/db/connection.ts`

### Database Models (9 Entities)

#### Core Content Models

**1. TourPackage** - Cycling tour offerings
```prisma
model TourPackage {
  id                String    @id @default(cuid())
  title             String
  slug              String    @unique
  shortDescription  String?
  description       String
  itinerary         Json      // Array of day-by-day itinerary
  duration          Int       // Days
  difficultyLevel   DifficultyLevel
  region            String
  basePrice         Float
  priceIncludes     Json      // Array of inclusions
  priceExcludes     Json      // Array of exclusions
  featured          Boolean   @default(false)
  heroImage         String?
  images            Json      // Array of image URLs
  videoUrl          String?
  isActive          Boolean   @default(true)
  // ... SEO fields, timestamps, relations
}
```

**2. CyclingGuide** - Regional cycling guides
```prisma
model CyclingGuide {
  id              String          @id @default(cuid())
  title           String
  slug            String          @unique
  region          String
  description     String
  content         String          // Rich markdown content
  difficultyRating Int           // 1-5 scale
  routeMap        Json            // GPS coordinates, elevation data
  safetyTips      Json            // Array of safety guidelines
  images          Json            // Array of image URLs
  // ... timestamps
}
```

**3. Accommodation** - Lodging options
```prisma
model Accommodation {
  id          String             @id @default(cuid())
  name        String
  type        AccommodationType  // HOTEL, GUESTHOUSE, RESORT, etc.
  location    String
  amenities   Json               // Array of amenities
  pricePerNight Float
  rating      Float?
  images      Json
  // ... relations
}
```

#### Booking System Models

**4. Booking** - Customer reservations
```prisma
model Booking {
  id              String         @id @default(cuid())
  bookingNumber   String         @unique
  tourPackage     TourPackage    @relation(...)
  customerName    String
  customerEmail   String
  customerPhone   String
  participants    Int
  startDate       DateTime
  endDate         DateTime
  status          BookingStatus  // PENDING, CONFIRMED, etc.
  paymentStatus   PaymentStatus  // PENDING, PAID, etc.
  paymentMethod   PaymentMethod? // CASH, BANK_TRANSFER, CARD
  totalAmount     Float
  paidAmount      Float          @default(0)
  // ... relations, timestamps
}
```

#### Supporting Models
- **BookingAccommodation** - Junction table
- **PackageAccommodation** - Junction table
- **AdminUser** - Authentication (bcrypt hashed passwords)
- **MediaAsset** - Uploaded media metadata
- **Session** - Session management

### Enums
```typescript
enum DifficultyLevel {
  BEGINNER, INTERMEDIATE, ADVANCED, EXPERT
}

enum AccommodationType {
  HOTEL, GUESTHOUSE, RESORT, HOMESTAY, CAMPING
}

enum BookingStatus {
  PENDING, CONFIRMED, CANCELLED, COMPLETED
}

enum PaymentStatus {
  PENDING, PAID, PARTIAL, REFUNDED
}

enum PaymentMethod {
  CASH, BANK_TRANSFER, CARD, OTHER
}

enum AdminRole {
  ADMIN, EDITOR
}
```

### Repository Pattern - **USE THIS** ⭐

**Location**: `src/lib/db/repositories/`

All database operations go through repositories. **NEVER** use Prisma client directly in pages/components.

#### TourPackageRepository API
```typescript
class TourPackageRepository {
  // Search with filters
  async findMany(params: PackageSearchParams): Promise<PackageSearchResult>

  // Featured packages (homepage)
  async findFeatured(limit: number): Promise<TourPackage[]>

  // Single package by slug
  async findBySlug(slug: string): Promise<TourPackage | null>

  // Filter by region
  async findByRegion(region: string): Promise<TourPackage[]>

  // Filter by difficulty
  async findByDifficulty(level: DifficultyLevel): Promise<TourPackage[]>

  // Full-text search
  async search(query: string): Promise<TourPackage[]>

  // Get unique regions
  async getRegions(): Promise<string[]>

  // Get price range
  async getPriceRange(): Promise<{ min: number; max: number }>
}
```

#### Example Usage in Pages
```typescript
// src/pages/packages.astro
---
import { TourPackageRepository } from '../lib/db/repositories/tour-packages';

let packages: TourPackage[] = [];

if (import.meta.env.DATABASE_URL) {
  const repository = new TourPackageRepository();
  const result = await repository.findMany({
    featured: true,
    limit: 10
  });
  packages = result.items;
} else {
  const { fallbackTourPackages } = await import('../data/fallback-tour-packages');
  packages = fallbackTourPackages;
}
---

{packages.map((pkg) => (
  <PackageCard package={pkg} />
))}
```

### Fallback Data System

**Location**: `src/data/`

- **`fallback-tour-packages.ts`** - 3 sample tour packages with full data
- **`fallback-cycling-guides.ts`** - 3 sample cycling guides with full data

**Structure matches Prisma models exactly**:
```typescript
// src/data/fallback-tour-packages.ts
export const fallbackTourPackages: TourPackage[] = [
  {
    id: 'tp-001',
    title: 'Cultural Triangle Explorer',
    slug: 'cultural-triangle-explorer',
    duration: 7,
    difficultyLevel: 'INTERMEDIATE',
    basePrice: 1299,
    itinerary: [
      { day: 1, title: 'Arrival in Colombo', description: '...' },
      // ... full itinerary
    ],
    priceIncludes: ['Accommodation', 'Meals', 'Guide', 'Bike rental'],
    // ... complete data matching TourPackage type
  },
  // ... 2 more packages
];
```

**When to Update Fallback Data**:
- ✅ When changing Prisma schema structure
- ✅ When adding new required fields
- ✅ When updating sample content for demos
- ❌ Not needed for normal content updates (those go in database)

---

## 🔄 Development Workflows

### Initial Setup

```bash
# 1. Clone repository
git clone <repo-url>
cd cycleparadise

# 2. Install dependencies
npm install

# 3. Setup git hooks (IMPORTANT - enables pre-commit validation)
npm run setup-hooks

# 4. Configure environment (OPTIONAL - works without DB)
cp .env.example .env
# Edit .env with your settings

# 5. Setup database (OPTIONAL)
npm run db:generate   # Generate Prisma client
npm run db:push       # Push schema to database
npm run db:seed       # Seed with sample data

# 6. Start development server
npm run dev
# Visit http://localhost:4321
```

### Daily Development Workflow

```bash
# 1. Pull latest changes
git pull

# 2. Start dev server (with hot reload)
npm run dev

# 3. Make changes, test locally

# 4. Run quality checks before commit
npm run check-all
# This runs: type-check + lint + validate + build

# 5. Commit (pre-commit hook runs automatically)
git add .
git commit -m "feat: descriptive commit message"

# 6. Push
git push
```

### Pre-commit Hook Execution

When you run `git commit`, the pre-commit hook automatically runs:

1. **TypeScript Type Checking** - `tsc --noEmit --skipLibCheck`
2. **ESLint Validation** - `npm run lint` (max-warnings: 0)
3. **Custom Validation** - `npm run validate` (reserved words, syntax)
4. **Full Build Test** - `npm run build` (catches runtime errors)

If ANY step fails, the commit is **blocked**. Fix errors and try again.

**See**: `.githooks/pre-commit` and `.githooks/pre-commit.ps1`

### Database Workflows

```bash
# Generate Prisma client (after schema changes)
npm run db:generate

# Push schema changes to database (development)
npm run db:push

# Create migration (production-ready)
npm run db:migrate

# Open Prisma Studio (GUI database browser)
npm run db:studio

# Seed database with sample data
npm run db:seed
```

### Testing Workflows

```bash
# Unit tests (Vitest)
npm test                  # Run tests
npm run test:ui           # Visual test UI
npm run test:coverage     # Coverage report

# E2E tests (Playwright)
npm run test:e2e          # Run E2E tests
npx playwright test --ui  # Interactive mode
npx playwright test --debug # Debug mode

# Run all tests
npm test && npm run test:e2e
```

### Build & Deployment

```bash
# Development build
npm run build
npm run preview   # Preview production build

# Type checking only
npm run type-check

# Full validation (recommended before deploy)
npm run check-all

# Docker build
docker build -t cycleparadise .
docker run -p 4321:4321 cycleparadise

# Docker with database
docker-compose up -d
```

---

## 🧪 Testing Strategy

### Unit Testing (Vitest)

**Configuration**: `vitest.config.ts`
```typescript
{
  environment: 'jsdom',  // Browser-like environment
  setupFiles: './src/test/setup.ts',
  include: 'src/**/*.{test,spec}.{js,ts,jsx,tsx}',
  coverage: {
    provider: 'v8',
    reporter: ['text', 'json', 'html'],
    exclude: ['**/*.config.*', '**/types/*']
  }
}
```

**Test File Structure**:
```typescript
// src/lib/utils/__tests__/validators.test.ts
import { describe, it, expect } from 'vitest';
import { validateEmail, validatePhone } from '../validators';

describe('validators', () => {
  describe('validateEmail', () => {
    it('should validate correct email format', () => {
      expect(validateEmail('test@example.com')).toBe(true);
    });

    it('should reject invalid email format', () => {
      expect(validateEmail('invalid')).toBe(false);
    });
  });
});
```

**Commands**:
```bash
npm test              # Run tests (watch mode)
npm run test:ui       # Visual test UI
npm run test:coverage # Generate coverage report
```

**Target**: 90%+ code coverage (from constitution.md)

### E2E Testing (Playwright)

**Configuration**: `playwright.config.ts`
```typescript
{
  testDir: './tests/e2e',
  use: {
    baseURL: 'http://localhost:4321',
    screenshot: 'only-on-failure',
    trace: 'retain-on-failure'
  },
  projects: [
    { name: 'chromium' },
    { name: 'firefox' },
    { name: 'webkit' },
    { name: 'Mobile Chrome' },  // Pixel 5
    { name: 'Mobile Safari' }   // iPhone 12
  ]
}
```

**Test File Structure**:
```typescript
// tests/e2e/packages.spec.ts
import { test, expect } from '@playwright/test';

test('should display tour packages on homepage', async ({ page }) => {
  await page.goto('/');

  // Check for package cards
  const packageCards = await page.locator('[data-testid="package-card"]');
  await expect(packageCards).toHaveCount(3);

  // Click first package
  await packageCards.first().click();
  await expect(page).toHaveURL(/\/packages\/[a-z-]+/);
});
```

**Commands**:
```bash
npm run test:e2e              # Run all E2E tests
npx playwright test --ui      # Interactive UI
npx playwright test --debug   # Debug mode
npx playwright codegen        # Generate tests
```

### Test-Driven Development (TDD)

From constitution.md, the project follows **strict Red-Green-Refactor cycles**:

1. **Red**: Write failing test that defines expected behavior
2. **Green**: Write minimal code to make test pass
3. **Refactor**: Improve code while keeping tests green

**Example TDD Cycle**:
```typescript
// Step 1: Red - Write failing test
test('should calculate package price with discount', () => {
  const price = calculateDiscountedPrice(1000, 10);
  expect(price).toBe(900);
});

// Step 2: Green - Implement minimal solution
function calculateDiscountedPrice(basePrice: number, discount: number): number {
  return basePrice - (basePrice * discount / 100);
}

// Step 3: Refactor - Improve with validation
function calculateDiscountedPrice(basePrice: number, discount: number): number {
  if (basePrice < 0 || discount < 0 || discount > 100) {
    throw new ValidationError('Invalid price or discount');
  }
  return basePrice - (basePrice * discount / 100);
}
```

---

## 📝 Common Tasks & Recipes

### 1. Adding a New Tour Package (Database Mode)

```typescript
// Using Prisma Studio (easiest)
// 1. Run: npm run db:studio
// 2. Open browser at http://localhost:5555
// 3. Navigate to TourPackage table
// 4. Click "Add record" and fill in fields
// 5. Save

// Or using Prisma client directly
import { db } from './src/lib/db/client';

await db.prisma.tourPackage.create({
  data: {
    title: 'New Adventure Tour',
    slug: 'new-adventure-tour',
    shortDescription: 'An exciting new cycling adventure',
    description: 'Full description...',
    duration: 5,
    difficultyLevel: 'INTERMEDIATE',
    region: 'Hill Country',
    basePrice: 999,
    itinerary: [
      { day: 1, title: 'Day 1', description: '...' }
    ],
    priceIncludes: ['Accommodation', 'Meals'],
    priceExcludes: ['Flights'],
    featured: false,
    isActive: true,
    images: ['image1.jpg', 'image2.jpg']
  }
});
```

### 2. Adding a New Tour Package (Fallback Mode)

```typescript
// Edit: src/data/fallback-tour-packages.ts
export const fallbackTourPackages: TourPackage[] = [
  // ... existing packages
  {
    id: 'tp-004',  // Increment ID
    title: 'New Adventure Tour',
    slug: 'new-adventure-tour',
    shortDescription: 'An exciting new cycling adventure',
    description: 'Full description...',
    duration: 5,
    difficultyLevel: 'INTERMEDIATE' as DifficultyLevel,
    region: 'Hill Country',
    basePrice: 999,
    currency: 'USD',
    itinerary: [
      {
        day: 1,
        title: 'Arrival & Welcome',
        description: 'Meet in Colombo...',
        distance: 0,
        elevation: 0,
        highlights: ['City tour']
      }
    ],
    priceIncludes: ['Accommodation', 'Meals'],
    priceExcludes: ['Flights'],
    featured: false,
    heroImage: '/images/packages/new-adventure.jpg',
    images: ['/images/packages/new-adventure-1.jpg'],
    videoUrl: null,
    createdAt: new Date('2025-11-25'),
    updatedAt: new Date('2025-11-25'),
    // ... all required fields
  }
];
```

### 3. Creating a New Page

```typescript
// src/pages/new-page.astro
---
import BaseLayout from '../layouts/BaseLayout.astro';

const pageTitle = 'New Page';
const pageDescription = 'Description for SEO';
---

<BaseLayout title={pageTitle} description={pageDescription}>
  <main class="container mx-auto px-4 py-8">
    <h1 class="text-4xl font-bold">{pageTitle}</h1>
    <p class="mt-4">{pageDescription}</p>
  </main>
</BaseLayout>
```

### 4. Creating a Dynamic Route Page

```typescript
// src/pages/blog/[slug].astro
---
import type { GetStaticPaths } from 'astro';
import BaseLayout from '../../layouts/BaseLayout.astro';

// REQUIRED: Export getStaticPaths for static builds
export const getStaticPaths: GetStaticPaths = async () => {
  // Fetch all blog posts
  const posts = await fetchBlogPosts();

  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post }  // Pass as props to page
  }));
};

// Get the post from props
const { post } = Astro.props;
---

<BaseLayout title={post.title} description={post.excerpt}>
  <article>
    <h1>{post.title}</h1>
    <div set:html={post.content} />
  </article>
</BaseLayout>
```

### 5. Creating a New Component

```astro
// src/components/ui/InfoCard.astro
---
interface Props {
  title: string;
  description: string;
  icon?: string;
  link?: string;
}

const { title, description, icon, link } = Astro.props;
---

<div class="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
  {icon && <div class="text-3xl mb-4">{icon}</div>}
  <h3 class="text-xl font-semibold mb-2">{title}</h3>
  <p class="text-gray-600 mb-4">{description}</p>
  {link && (
    <a href={link} class="text-primary hover:underline">
      Learn More →
    </a>
  )}
</div>
```

**Usage**:
```astro
---
import InfoCard from '../components/ui/InfoCard.astro';
---

<InfoCard
  title="Cycling Tours"
  description="Explore Sri Lanka by bike"
  icon="🚴"
  link="/packages"
/>
```

### 6. Adding Image Optimization

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/images/hero.jpg';
---

<!-- Optimized image with responsive sizes -->
<Image
  src={heroImage}
  alt="Cycling in Sri Lanka"
  width={1200}
  height={600}
  loading="lazy"
  format="webp"
  quality={80}
/>
```

### 7. Sending Email (Contact Form)

```typescript
// src/pages/api/contact.ts
import type { APIRoute } from 'astro';
import { sendEmail } from '../../lib/email/service';

export const POST: APIRoute = async ({ request }) => {
  const data = await request.json();
  const { name, email, message } = data;

  try {
    await sendEmail({
      to: import.meta.env.CONTACT_EMAIL,
      subject: `Contact Form: ${name}`,
      text: `From: ${name} <${email}>\n\n${message}`,
      html: `<p><strong>From:</strong> ${name} &lt;${email}&gt;</p><p>${message}</p>`
    });

    return new Response(JSON.stringify({ success: true }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    });
  } catch (error) {
    console.error('Email error:', error);
    return new Response(JSON.stringify({ error: 'Failed to send email' }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    });
  }
};
```

### 8. Modifying Prisma Schema

```bash
# 1. Edit prisma/schema.prisma
# Add new field or model

# 2. Generate Prisma client
npm run db:generate

# 3. Push to database (development)
npm run db:push

# OR create migration (production)
npm run db:migrate

# 4. Update TypeScript types if needed
# Types are auto-generated from schema

# 5. Update fallback data to match schema
# Edit src/data/fallback-tour-packages.ts
```

### 9. Adding Custom Validation

```typescript
// src/lib/utils/validators.ts
export class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function validatePhone(phone: string): boolean {
  // Sri Lankan phone format
  const phoneRegex = /^(?:\+94|0)?[0-9]{9}$/;
  return phoneRegex.test(phone);
}

export function validateSlug(slug: string): boolean {
  // Kebab-case format
  const slugRegex = /^[a-z0-9]+(?:-[a-z0-9]+)*$/;
  return slugRegex.test(slug);
}
```

### 10. Environment Variable Usage

```typescript
// Access environment variables in Astro
---
// Server-side only (not exposed to browser)
const databaseUrl = import.meta.env.DATABASE_URL;
const smtpPassword = import.meta.env.SMTP_PASSWORD;

// Public variables (exposed to browser, prefixed with PUBLIC_)
const siteUrl = import.meta.env.PUBLIC_SITE_URL;
---

<script define:vars={{ siteUrl }}>
  // Client-side access to public variables
  console.log('Site URL:', siteUrl);
</script>
```

**Environment Variable Types**:
- **Private** (server-only): `DATABASE_URL`, `SMTP_PASSWORD`
- **Public** (client-exposed): `PUBLIC_SITE_URL`, `PUBLIC_CONTACT_FORM_ENDPOINT`

---

## ✅ Quality Gates

### Pre-Commit Requirements (Automated)

When you run `git commit`, these checks **must pass**:

1. ✅ **TypeScript Type Checking** - `npm run type-check`
   - No type errors allowed
   - Strict mode enforced
   - All exported functions have return types

2. ✅ **ESLint Validation** - `npm run lint`
   - Zero warnings allowed
   - Reserved word detection
   - Code style compliance

3. ✅ **Custom Validation** - `npm run validate`
   - No `package` variable usage
   - Dynamic routes have `getStaticPaths`
   - No syntax errors (extra brackets, etc.)

4. ✅ **Full Build Test** - `npm run build`
   - Site builds successfully
   - All pages generate static HTML
   - No runtime errors

**Setup**: `npm run setup-hooks` (one-time setup)

### Manual Quality Checks

Before pushing to production:

```bash
# Run comprehensive checks
npm run check-all

# Run tests
npm test
npm run test:e2e

# Check test coverage
npm run test:coverage

# Format code
npm run format

# Preview production build
npm run build && npm run preview
```

### Code Review Checklist

When reviewing code (AI or human):

- [ ] **No reserved words used** (`package`, `interface`, etc.)
- [ ] **Dynamic routes have `getStaticPaths()`**
- [ ] **Dual-mode data loading** (database + fallback)
- [ ] **TypeScript types explicit** (no `any`)
- [ ] **Error handling present** (try-catch, validation)
- [ ] **Fallback data updated** if schema changed
- [ ] **Tests written** for new functionality
- [ ] **Documentation updated** if needed
- [ ] **Accessibility considered** (ARIA labels, keyboard nav)
- [ ] **Performance optimized** (lazy loading, image optimization)
- [ ] **SEO metadata included** (title, description, OpenGraph)

### Performance Requirements

From constitution.md:

- ⚡ **Page load time**: <2 seconds on standard connections
- ⚡ **API response time**: <200ms for 95th percentile requests
- ⚡ **Lighthouse score**: 90+ across all metrics
- ⚡ **Memory usage**: Monitored and optimized continuously

### Accessibility Requirements

- ♿ **WCAG 2.1 AA compliance** mandatory
- ♿ **Keyboard navigation** for all interactive elements
- ♿ **Screen reader compatibility** validated
- ♿ **Color contrast** meets AA standards
- ♿ **Focus indicators** visible and clear

---

## 📚 File References

### Essential Reading

| File | Purpose | Lines | Priority |
|------|---------|-------|----------|
| `README.md` | Project overview, setup, features | 557 | ⭐⭐⭐ |
| `ADMIN_GUIDE.md` | Content management guide | 657 | ⭐⭐⭐ |
| `ERROR_PREVENTION.md` | Error prevention strategies | 211 | ⭐⭐⭐ |
| `.specify/memory/constitution.md` | Project philosophy & standards | 73 | ⭐⭐ |
| `specs/001-cycling-tour-website/spec.md` | Feature specification | 11,452 | ⭐⭐ |
| `specs/001-cycling-tour-website/data-model.md` | Database schema details | 9,408 | ⭐⭐ |

### Configuration Files

| File | Purpose | Key Settings |
|------|---------|--------------|
| `package.json` | Dependencies & scripts | 27 npm scripts, 28 dependencies |
| `astro.config.mjs` | Astro framework config | Static output, Sharp images, sitemap |
| `tsconfig.json` | TypeScript compiler | Strict mode, Astro base config |
| `tailwind.config.mjs` | CSS framework | Custom colors, fonts, animations |
| `.eslintrc.json` | Linting rules | Reserved word prevention, TypeScript rules |
| `.prettierrc` | Code formatting | Single quotes, 100 char width |
| `vitest.config.ts` | Unit testing | jsdom environment, coverage settings |
| `playwright.config.ts` | E2E testing | Multi-browser, mobile viewports |
| `prisma/schema.prisma` | Database schema | 9 models, 5 enums, relations |

### Key Implementation Files

**Repositories** (Data Access Layer):
- `src/lib/db/repositories/tour-packages.ts` - TourPackage data operations
- `src/lib/db/repositories/cycling-guides.ts` - CyclingGuide data operations

**Fallback Data** (Static Mode):
- `src/data/fallback-tour-packages.ts` - 3 sample tour packages
- `src/data/fallback-cycling-guides.ts` - 3 sample cycling guides

**Core Services**:
- `src/lib/email/service.ts` - NodeMailer email sending
- `src/lib/services/instagram.ts` - Instagram API integration
- `src/lib/media/optimizer.ts` - Sharp image optimization
- `src/lib/auth/session.ts` - Session-based authentication

**Utilities**:
- `src/lib/utils/prisma-converters.ts` - Prisma to domain model converters
- `src/lib/errors/index.ts` - Custom error classes

**Validation**:
- `scripts/validate.mjs` - Custom code validation
- `.githooks/pre-commit` - Pre-commit validation hook

### Page Examples

**Static Pages**:
- `src/pages/index.astro` - Homepage (featured packages, Instagram feed)
- `src/pages/about.astro` - About page with mission & team
- `src/pages/contact.astro` - Contact form with email integration

**Dynamic Pages**:
- `src/pages/packages/[slug].astro` - Tour package details
- `src/pages/guides/[slug].astro` - Cycling guide details

**Listing Pages**:
- `src/pages/packages.astro` - Tour packages with filters
- `src/pages/guides.astro` - Cycling guides with regions

### Component Examples

**Media Components**:
- `src/components/media/InstagramFeed.astro` - Instagram post gallery
- `src/components/media/YouTubeEmbed.astro` - Responsive YouTube player
- `src/components/media/PackageGallery.astro` - Image carousel

**UI Components**:
- `src/components/ui/PackageCard.astro` - Tour package card
- `src/components/ui/OptimizedImage.astro` - Lazy-loaded images

**Analytics**:
- `src/components/analytics/PerformanceMonitor.astro` - Performance tracking

---

## 🎓 Learning Path for New AI Assistants

### Step 1: Understand the Architecture (30 min)
1. Read this CLAUDE.md file completely
2. Review `README.md` for project overview
3. Understand dual-mode operation (database vs. fallback)
4. Review repository pattern in `src/lib/db/repositories/`

### Step 2: Critical Rules (15 min)
1. **MEMORIZE**: Never use `package` as variable name
2. **MEMORIZE**: Dynamic routes need `getStaticPaths()`
3. Review `ERROR_PREVENTION.md` thoroughly
4. Understand pre-commit hook validation

### Step 3: Explore the Codebase (45 min)
1. Browse `src/pages/` structure (file-based routing)
2. Examine `src/components/` (UI components)
3. Review `src/lib/` (business logic)
4. Check `src/data/` (fallback data)
5. Study `prisma/schema.prisma` (database schema)

### Step 4: Practical Examples (30 min)
1. Trace a page request: `/packages` → `packages.astro` → repository → Prisma/fallback
2. Understand component composition: `BaseLayout` → page → components
3. Follow data flow: Prisma schema → repository → converter → page → component

### Step 5: Quality & Testing (20 min)
1. Review `.eslintrc.json` and `.prettierrc`
2. Understand pre-commit hook workflow
3. Review test setup: `vitest.config.ts` and `playwright.config.ts`
4. Run `npm run check-all` to see validation in action

### Quick Reference Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Production build
npm run preview          # Preview build

# Database
npm run db:generate      # Generate Prisma client
npm run db:studio        # Open Prisma Studio GUI
npm run db:seed          # Seed database

# Quality
npm run check-all        # All checks + build
npm run lint             # ESLint check
npm run format           # Format code
npm run type-check       # TypeScript check
npm run validate         # Custom validation

# Testing
npm test                 # Unit tests
npm run test:e2e         # E2E tests
npm run test:coverage    # Coverage report

# Git
npm run setup-hooks      # Setup pre-commit hooks
npm run pre-commit       # Manual pre-commit check
```

---

## 🆘 Common Issues & Solutions

### Issue: "package is a reserved word"

**Error**: `SyntaxError: Unexpected token 'package'`

**Solution**: Replace all instances of `package` variable with alternatives
```bash
# Find instances
grep -r "\.map((package)" src/

# Use alternatives: tourPackage, pkg, item
packages.map((pkg) => <PackageCard package={pkg} />)
```

### Issue: "getStaticPaths is required"

**Error**: `Error: getStaticPaths() function required for dynamic routes`

**Solution**: Add getStaticPaths export to `[slug].astro` files
```typescript
export const getStaticPaths: GetStaticPaths = async () => {
  const items = await fetchItems();
  return items.map((item) => ({
    params: { slug: item.slug }
  }));
};
```

### Issue: Build fails with Prisma error

**Error**: `PrismaClient is unable to run in the browser`

**Solution**: Ensure Prisma is excluded from browser bundle
```javascript
// astro.config.mjs
export default defineConfig({
  vite: {
    optimizeDeps: {
      exclude: ['@prisma/client']
    }
  }
});
```

### Issue: Database connection fails

**Error**: `Can't reach database server`

**Solution**: Site should fall back to static data automatically
```typescript
// Check your page code has fallback
if (import.meta.env.DATABASE_URL) {
  // Try database
} else {
  // Use fallback data
}
```

### Issue: Pre-commit hook not running

**Solution**: Setup git hooks
```bash
npm run setup-hooks
chmod +x .githooks/*
git config core.hooksPath .githooks
```

---

## 🚀 Next Steps

After understanding this guide:

1. **Read ERROR_PREVENTION.md** - Critical error prevention strategies
2. **Read ADMIN_GUIDE.md** - Content management workflows
3. **Explore specs/** - Detailed feature specifications
4. **Review constitution.md** - Project philosophy and standards
5. **Run the project locally** - Best way to learn is by doing

---

## 📞 Getting Help

- **Documentation**: Check README.md, ADMIN_GUIDE.md, ERROR_PREVENTION.md
- **Specs**: Review specs/001-cycling-tour-website/ for detailed docs
- **Code Comments**: Most files have inline documentation
- **Git History**: Check recent commits for context
- **Prisma Studio**: Visual database browser (`npm run db:studio`)

---

**Remember**: This is a production-grade codebase with strict quality standards. Follow the conventions, use the validation tools, and always test thoroughly before committing.

**Version**: 1.0.0
**Last Updated**: 2025-11-25
**Maintainer**: CycleParadise Team
