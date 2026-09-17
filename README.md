# 🎨 rwp-page-builder

**rwp-page-builder** is a visual, drag-and-drop page builder plugin designed specifically for the **react-wp** CMS framework[cite: 2, 3]. Inspired by modern visual editing tools like Elementor, it enables creators to build rich, responsive page layouts and e-commerce templates without touching code[cite: 2, 3].

Instead of storing unmaintainable raw HTML strings, `rwp-page-builder` serializes page layouts into structured, version-controllable JSON trees stored directly in Supabase PostgreSQL (`builder_data` JSONB)[cite: 2, 3].

---

## 🚀 Key Features

* **3-Panel WYSIWYG Workspace (`PageBuilderLayout.tsx`)**:
  * **Left Sidebar**: Categorized widget panel, inspector panel (Content, Style, Advanced tabs), and global layout tree navigator[cite: 3].
  * **Live Canvas**: Real-time visual editing, responsive column resizing, and hover handles for easy reordering[cite: 2, 3].
  * **Top Controls**: Device viewport switcher (Desktop, Tablet, Mobile), undo/redo action history, draft/publish controls, and template revision manager[cite: 2, 3].
* **Modular Drag-and-Drop System**: Built on top of `@dnd-kit` / `@hello-pangea/dnd`, supporting nested section, column, and widget structures[cite: 2, 3].
* **Lightweight Frontend Renderer (`BuilderRenderer.tsx`)**: Parses layout JSON payloads directly into clean Tailwind CSS and semantic HTML on public routes without loading heavy editor JavaScript bundles[cite: 2, 3].
* **E-Commerce & Dynamic Data Integration**: Full compatibility with `rwp-shop` for building store catalogs, single product viewports, cart screens, and checkout layouts using dynamic data tags[cite: 2, 3].
* **Global Design System & Templates**: Centralized global font/color variables and reusable layout blocks saved in Supabase (`elementor_templates`)[cite: 2, 3].

---

## 🛠️ Tech Stack & Dependencies

| Area | Component |
| :--- | :--- |
| **UI Engine** | React, TypeScript, Tailwind CSS[cite: 3] |
| **Drag & Drop** | `@dnd-kit/core` / `@hello-pangea/dnd`[cite: 2, 3] |
| **Icons** | Lucide-React[cite: 2, 3] |
| **Backend & Storage** | Supabase PostgreSQL (`builder_data` JSONB, Auth, RLS)[cite: 2, 3] |
| **CMS Core** | `react-wp` Plugin Registry Engine[cite: 2] |

---

## 📂 Plugin Directory Structure

```text
plugins/rwp-page-builder/
├── index.ts                   # Plugin entry point & hook registration
├── PageBuilderLayout.tsx       # Main 3-panel workspace interface
├── components/
│   ├── WidgetPanel.tsx        # Left sidebar: Categorized widget panel
│   ├── InspectorPanel.tsx     # Content, Style, & Advanced property controls
│   ├── Canvas.tsx             # WYSIWYG workspace & drop targets
│   ├── Navigator.tsx          # Hierarchical tree overview of sections/widgets
│   └── TopBar.tsx             # Viewport switcher & save controls
├── renderers/
│   └── BuilderRenderer.tsx    # Public site execution engine
└── widgets/                   # Widget component definitions
    ├── Container.tsx          # Flexbox / Grid layout section
    ├── Heading.tsx            # Dynamic header text
    ├── TextEditor.tsx         # Rich-text block
    ├── Image.tsx              # Media & lightbox block
    ├── Button.tsx             # CTA & action link button
    ├── PostsGrid.tsx          # Dynamic Supabase posts feed
    ├── FormBuilder.tsx        # Interactive forms with submission persistence
    └── SidebarWidget.tsx      # Category & post sidebar block
```[cite: 2]

---

## 🧱 Widget Catalog

### 📦 Basic Widgets (Core)
* **Section / Container**: Layout wrapper supporting flex alignment, grid splits, backgrounds, and padding controls[cite: 3].
* **Heading & Text Editor**: Title tags (H1-H6), rich text formatting, typography controls, and inline styling[cite: 3].
* **Image & Video**: Media displays with lightbox support, aspect ratios, responsive scaling, and video embeds[cite: 3].
* **Button, Divider, & Spacer**: Action buttons with hover animations, adjustable blank heights, and icon line breaks[cite: 3].
* **Sidebar Area**: Dynamic block rendering post categories and recent posts feeds[cite: 2].

### ⚡ Pro Widgets
* **Posts Grid Feed**: Queries published posts from Supabase with layout customization (Grid, List, Masonry), category filters, and pagination[cite: 3].
* **Form Builder**: Drag-and-drop input fields that write submissions directly to the `form_submissions` table[cite: 3].
* **Accordions & FAQ**: Collapsible content blocks with dynamic open/close animations[cite: 3].
* **Hero Carousel & Call to Action (CTA)**: Slide banners with background overlays and interactive zoom effects[cite: 3].

### 🛒 E-Commerce Widgets (`rwp-shop`)
* Product Catalog Grid (`[rwp_products]`)[cite: 2, 3]
* Add-to-Cart Action Button (`[rwp_add_to_cart]`)[cite: 2, 3]
* Mini-Cart Header Indicator (`[rwp_cart_link]`)[cite: 2, 3]
* Single Product Gallery, Pricing Block, Cart Table, and Checkout Form Containers[cite: 2, 3].

---

## 🗄️ Database Setup (Supabase SQL)

To enable page builder capabilities in your `react-wp` instance, run the following SQL script in your Supabase SQL Editor:

```sql
-- 1. Extend pages table to support builder JSON storage & shop template types
ALTER TABLE pages 
ADD COLUMN IF NOT EXISTS builder_data JSONB DEFAULT '{}'::jsonb,
ADD COLUMN IF NOT EXISTS is_builder_enabled BOOLEAN DEFAULT false,
ADD COLUMN IF NOT EXISTS is_shop_page BOOLEAN DEFAULT false,
ADD COLUMN IF NOT EXISTS shop_page_type VARCHAR(50) DEFAULT NULL;

-- 2. Create templates table for reusable sections and page layouts
CREATE TABLE IF NOT EXISTS elementor_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  type VARCHAR(50) DEFAULT 'page', -- 'page' or 'section'
  builder_data JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Create form submissions table for Form Builder widgets
CREATE TABLE IF NOT EXISTS form_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  form_id TEXT NOT NULL,
  page_id BIGINT REFERENCES pages(id) ON DELETE CASCADE,
  fields_data JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```[cite: 2, 3]

---

## ⚡ Integration & Public Rendering

### 1. Enabling the Builder in Admin
Inside your admin dashboard routes, load `PageBuilderLayout.tsx` whenever `is_builder_enabled` is active or when an admin clicks **"Edit with Page Builder"** on any page row[cite: 2, 3].

### 2. Rendering on Public Routes
Public pages check `is_builder_enabled` to determine whether to output standard HTML or parse the JSON tree using `BuilderRenderer`:

```tsx
import React from 'react';
import { BuilderRenderer } from './plugins/rwp-page-builder/renderers/BuilderRenderer';

export const PublicPage = ({ page }) => {
  if (page.is_builder_enabled && page.builder_data) {
    return <BuilderRenderer data="{page.builder_data}"/>;
  }

  return (
    <article className="prose max-w-4xl mx-auto py-8">
      <h1>{page.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: page.content }} />
    </article>
  );
};
```[cite: 2, 3]

---

## 📄 License
Distributed under the MIT License. Part of the **react-wp** CMS ecosystem[cite: 1, 2].
