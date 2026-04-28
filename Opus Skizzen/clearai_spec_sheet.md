# ClearAI Frontend Specification Document

This document outlines the architecture, components, and styling requirements for the ClearAI platform based on the provided HTML prototypes. It is designed to be handed off to a Frontend Developer using React or Vue.

## 1. Global Styling & Theming
ClearAI uses a modern, dark-themed glassmorphism aesthetic with vibrant color coding to indicate AI scores and metrics.

**Color Palette (CSS Variables):**
```css
:root {
  /* Core Backgrounds */
  --bg: #0a0a0f;
  --card: #13131a;
  --card-hover: #181822; /* Or #16161f */
  --border: #1e1e2e;
  
  /* Typography */
  --text: #e2e2e8;
  --muted: #6b6b80;
  
  /* Brand Accents */
  --accent: #6c5ce7;
  --accent-light: #a29bfe;
  
  /* Status / Functional Colors */
  --green: #00b894;   /* Positive / High Score */
  --yellow: #fdcb6e;  /* Warning / Mid Score / Pending */
  --red: #e17055;     /* Negative / Low Score / Missing */
  --blue: #0984e3;
  --cyan: #00cec9;
  --pink: #fd79a8;
  
  /* Comparison Specific */
  --ai-a: #6c5ce7;
  --ai-b: #00cec9;
}
```

**Typography:**
- `font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;`
- Emphasize clear hierarchies. Use `font-weight: 800` for main headers and `400/500/600` for body and UI elements.

## 2. Shared/Reusable Components

To build a scalable SPA, abstract the following UI elements into reusable components:

1. **`Navbar`**: Top navigation with logo and links. On the Comparison view, it becomes a fixed glassmorphism topbar (`backdrop-filter: blur(14px)`).
2. **`Badge` / `Tag`**: 
   - Accepts props for `colorTheme` (green, red, yellow, neutral, purple, blue) and `icon` (e.g., checkmark, dot, cross).
   - Variations include Verification Status (Verified, Pending, Unverified) and Capability Tags.
3. **`ScoreBar`**: A horizontal progress bar representing a score out of 10. Automatically applies `--green`, `--yellow`, or `--red` based on the score threshold (>= 7 is green, >= 5 is yellow, < 5 is red).
4. **`SpiderChart` (Radar Chart)**:
   - A reusable SVG-based chart component to display the 8 core metrics (Transparency, Ethics, Fairness, Accountability, Safety, Privacy, Explainability, Controllability).
   - Needs a `size` prop to render as a "Mini" version on the Browse cards and a "Large/Interactive" version on the Detail panel.
5. **`GradientIcon`**: A rounded box (e.g., `40px` or `48px`) with a linear gradient background and a single letter initial.

## 3. Pages and Views

### 3.1. Browse View (`/` or `/browse`)
**Reference:** `compact.html`
**Purpose:** A grid view of available AI products with filtering and sorting.
- **Hero/Header**: Page title and brief description.
- **Filter Bar**: Horizontal list of pill buttons (All, LLMs, Computer Vision, etc.). Requires `active` state styling.
- **Sort Options**: A dropdown menu to sort the grid (Governance Score, Name A-Z, Newest).
- **Product Grid**: A responsive CSS grid (`grid-template-columns: repeat(auto-fill, minmax(380px, 1fr))`).
- **Compact Card Component**:
  - Highly dense information card.
  - **Top**: Gradient icon, Name, Company, and large Overall Score.
  - **Body**: Truncated purpose text (`-webkit-line-clamp: 2`).
  - **Middle**: Mini Spider Chart on the left, vertical stack of mini `ScoreBar` components on the right.
  - **Footer**: Flex row of `Badge` components for Sector, Capabilities, and Verification status.

### 3.2. Product Detail View (`/product/:id`)
**Reference:** `enhanced.html` and `final_index.html`
**Purpose:** A deep dive into a single AI product's transparency report.
- **Hero Section**: Large header with gradient highlighted text and a main Search Bar.
- **Two-Column Layout** (Stacks on mobile):
  - **Left Column (Transparency Card)**:
    - **Header**: Name, Meta info, Trust Badge.
    - **Sections**: Purpose & Mission, AI Type & Sector (with dot tags), Quick Glance (Inputs/Outputs grouped into columns), Known Limitations (bulleted list).
    - **Deep Dive Accordion**: An HTML `<details>`/`<summary>` equivalent containing data tables for Model Architecture, Training Dataset, and Filter checklists.
  - **Right Column (Spider Panel)** (from `enhanced.html`):
    - Sticky sidebar container.
    - Features the large overall score and the large interactive `SpiderChart`.
    - Below the chart, a list of dimensions that highlight the exact score and description when hovered/clicked.

### 3.3. Comparison View (`/compare`)
**Reference:** `comparison.html`
**Purpose:** Side-by-side comparison of two AI products.
- **Product Selectors**: Two prominent dropdowns to select "Product A" and "Product B". Styled with respective brand colors (`--ai-a` and `--ai-b`).
- **Mini Headers**: Summarized product cards showing the selected items.
- **Comparison Grid**:
  - **Purpose**: Side-by-side text boxes.
  - **Core Specs / Data Tables**: Full-width rows split into three columns (Label, Product A Value, Product B Value).
  - **Checklist/Governance Grid**: Side-by-side verification lists (e.g., PII Removal: Yes vs Partial).
- **Verdict/Winner Highlights**: Cells that conditionally apply a green `.better` or red `.worse` border/background depending on which product scores higher. Includes small `.winner-badge` overlays.

## 4. State Management & Data Model

Use a global store (Zustand/Redux for React, Pinia for Vue) to manage the list of products. 

**Expected Product Object Schema:**
```typescript
interface AIProduct {
  id: string;
  name: string;
  company: string;
  icon: string; // Single letter
  gradient: [string, string]; // Hex codes for icon background
  purpose: string;
  type: string;
  sector: string;
  caps: string[]; // e.g., ["Text", "Images", "Code"]
  cantDo: string[]; // e.g., ["Real-time", "Audio"]
  verified: 'yes' | 'pending' | 'no';
  overall: number; // 0.0 to 10.0
  scores: {
    Transparency: number;
    Ethics: number;
    Fairness: number;
    Accountability: number;
    Safety: number;
    Privacy: number;
    Explainability: number;
    Controllability: number;
  };
  // Data for the Detail/Comparison Views
  details?: {
    architecture: Record<string, string>;
    training: Record<string, string>;
    filtering: Record<string, 'yes' | 'partial' | 'no'>;
    limitations: string[];
  };
}
```

## 5. Development Notes
- **Routing**: Setup standard routing (`react-router-dom` or `vue-router`).
- **Styling Strategy**: The raw CSS provided in the prototypes maps perfectly to CSS Modules, Styled Components, or Tailwind CSS (if you map the custom color palette into `tailwind.config.js`). Ensure hover effects (`transform: translateY(-2px)`, shadow transitions) are preserved as they give the app its premium feel.
- **Responsiveness**: Grid templates should seamlessly collapse to `1fr` on mobile viewports (`max-width: 768px` or `900px`). Ensure side-by-side comparison tables stack or scroll horizontally on narrow screens.
