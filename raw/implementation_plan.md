# Groww — Implementation Plan (Technical)

**Stack:** Python 3.11+  
**Date:** April 6, 2026

---

# TABLE OF CONTENTS

1. [Development Environment Setup](#1-development-environment-setup)
2. [Task 1: Content Engine — Implementation](#2-task-1-content-engine)
   - 2.1 Project Structure
   - 2.2 Component 1: Email Analyzer Pipeline
   - 2.3 Component 2: Block Library
   - 2.4 Component 3: Recipe System
   - 2.5 Component 4: Chart Image Renderer
   - 2.6 Component 5: HTML Email Renderer
   - 2.7 Component 6: MCP Server
   - 2.8 End-to-End Flow
3. [Task 2: Compass MCP Skills — Implementation](#3-task-2-compass-mcp-skills)
   - 3.1 Skill Authoring Toolkit
   - 3.2 Schema Discovery Automation
   - 3.3 Skill Template Generator
   - 3.4 Query Testing Framework
   - 3.5 Skill-by-Skill Implementation Guide
4. [Integration Points](#4-integration-points)
5. [Testing Strategy](#5-testing-strategy)
6. [Deployment & Configuration](#6-deployment--configuration)

---

# 1. Development Environment Setup

## 1.1 System Requirements

```bash
# Python 3.11+
python3 --version

# Node.js 18+ (needed for Puppeteer chart rendering)
node --version

# Install uv (fast Python package manager) — recommended over pip
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## 1.2 Project Initialization

```bash
# Create monorepo structure
mkdir groww-content-engine && cd groww-content-engine
mkdir -p src/{analyzer,blocks,recipes,charts,renderer,mcp,utils}
mkdir -p data/{raw_emails,analyzed,blocks,recipes,brands,charts_output}
mkdir -p output/{html,images}
mkdir -p tests/{analyzer,renderer,charts,mcp}
mkdir -p compass-skills/{templates,tools,generated,tests}

# Initialize Python project
uv init
uv add beautifulsoup4 lxml cssutils Pillow jinja2 pydantic httpx
uv add mcp                          # MCP SDK for Python
uv add playwright                   # For chart screenshot rendering
uv add anthropic                    # For LLM content generation
uv add rich typer                   # CLI tools
uv add pytest pytest-asyncio        # Testing

# Install Playwright browsers
uv run playwright install chromium
```

## 1.3 Full Project Tree

```
groww-content-engine/
│
├── pyproject.toml
├── README.md
│
├── src/
│   ├── __init__.py
│   │
│   ├── analyzer/                    # Component 1: Email Analyzer
│   │   ├── __init__.py
│   │   ├── parser.py                # HTML → structured DOM tree
│   │   ├── block_detector.py        # Detects block types in parsed HTML
│   │   ├── classifier.py            # Classifies email type
│   │   ├── extractor.py             # Extracts content from detected blocks
│   │   └── pipeline.py              # Orchestrates full analysis
│   │
│   ├── blocks/                      # Component 2: Block Library
│   │   ├── __init__.py
│   │   ├── registry.py              # Block schema registry
│   │   ├── schemas.py               # Pydantic models for all 18 blocks
│   │   └── validator.py             # Validates block data against schema
│   │
│   ├── recipes/                     # Component 3: Recipe System
│   │   ├── __init__.py
│   │   ├── registry.py              # Recipe storage and lookup
│   │   ├── recommender.py           # LLM-powered recipe recommendation
│   │   └── schemas.py               # Pydantic models for recipes
│   │
│   ├── charts/                      # Component 4: Chart Renderer
│   │   ├── __init__.py
│   │   ├── renderer.py              # Playwright-based HTML → PNG renderer
│   │   ├── templates/               # HTML/CSS templates for each chart type
│   │   │   ├── price_card.html
│   │   │   ├── listing_card.html
│   │   │   ├── comparison.html
│   │   │   ├── portfolio.html
│   │   │   └── metric.html
│   │   └── schemas.py               # Input data schemas for each chart
│   │
│   ├── renderer/                    # Component 5: HTML Email Renderer
│   │   ├── __init__.py
│   │   ├── engine.py                # Recipe JSON → full HTML email
│   │   ├── templates/               # Jinja2 templates per block type
│   │   │   ├── base.html            # Email boilerplate wrapper
│   │   │   ├── header_logo.html
│   │   │   ├── hero_text.html
│   │   │   ├── article_section.html
│   │   │   ├── chart_image.html
│   │   │   ├── factor_card.html
│   │   │   ├── cta_button.html
│   │   │   ├── footer_standard.html
│   │   │   └── ... (all 18 blocks)
│   │   └── inliner.py               # CSS inlining post-processor
│   │
│   ├── mcp/                         # Component 6: MCP Server
│   │   ├── __init__.py
│   │   ├── server.py                # MCP server entry point
│   │   └── tools.py                 # Tool definitions and handlers
│   │
│   ├── generator/                   # AI content generation
│   │   ├── __init__.py
│   │   ├── copy_writer.py           # LLM prompt for email copy
│   │   └── prompts.py               # Prompt templates
│   │
│   └── utils/
│       ├── __init__.py
│       ├── image_uploader.py        # Upload PNGs to Groww cloud
│       └── config.py                # Brand configs, paths, constants
│
├── data/
│   ├── raw_emails/                  # 450 HTML files from WebEngage
│   ├── analyzed/                    # Analyzer output JSONs
│   ├── blocks/                      # Block schema JSON files
│   ├── recipes/                     # Recipe template JSON files
│   └── brands/
│       ├── groww.json               # Groww brand config
│       ├── 915.json                 # 915 brand config
│       ├── amc.json                 # AMC brand config
│       └── w.json                   # W brand config
│
├── output/                          # Generated outputs
│   ├── html/
│   └── images/
│
├── tests/
│
├── compass-skills/                  # Task 2: Compass MCP Skills
│   ├── templates/
│   │   └── skill_template.md        # Base template for all skills
│   ├── tools/
│   │   ├── schema_discovery.py      # Auto-discover table schemas
│   │   ├── query_tester.py          # Test SQL queries against Compass
│   │   ├── skill_generator.py       # Generate skill scaffolds
│   │   └── validator.py             # Validate skill completeness
│   ├── generated/                   # Generated skill documents
│   │   ├── engagement_comms.md
│   │   ├── user_master.md
│   │   ├── user_events.md
│   │   ├── stocks_trading.md
│   │   └── ...
│   └── tests/
│       └── test_queries/            # SQL test files per skill
│           ├── engagement_comms/
│           ├── user_master/
│           └── ...
│
└── cli.py                           # CLI entry point for all tools
```

---

# 2. Task 1: Content Engine — Implementation

## 2.1 Configuration & Constants

**File: `src/utils/config.py`**

This is the foundation — brand configs, block definitions, and constants that everything else references.

```python
from pathlib import Path
from pydantic import BaseModel

# --- Paths ---
ROOT_DIR = Path(__file__).parent.parent.parent
DATA_DIR = ROOT_DIR / "data"
RAW_EMAILS_DIR = DATA_DIR / "raw_emails"
ANALYZED_DIR = DATA_DIR / "analyzed"
BLOCKS_DIR = DATA_DIR / "blocks"
RECIPES_DIR = DATA_DIR / "recipes"
BRANDS_DIR = DATA_DIR / "brands"
OUTPUT_DIR = ROOT_DIR / "output"
CHARTS_OUTPUT_DIR = OUTPUT_DIR / "images"
HTML_OUTPUT_DIR = OUTPUT_DIR / "html"
TEMPLATES_DIR = ROOT_DIR / "src" / "renderer" / "templates"
CHART_TEMPLATES_DIR = ROOT_DIR / "src" / "charts" / "templates"


class BrandConfig(BaseModel):
    """Brand-specific styling configuration."""
    name: str                       # "groww", "915", "amc", "w"
    display_name: str               # "Groww", "915", "AMC", "W"
    primary_color: str              # Main brand color
    secondary_color: str            # Secondary color
    accent_color: str               # Accent / highlight color
    text_color: str                 # Primary text color
    background_color: str           # Email background
    card_background: str            # Card/section background
    font_family: str                # Primary font
    logo_url: str                   # Hosted logo image URL
    footer_logo_url: str            # Footer logo variant
    sebi_registration: str          # SEBI registration number
    social_links: dict[str, str]    # { "twitter": "url", "instagram": "url", ... }
    app_store_url: str
    play_store_url: str
    unsubscribe_url_template: str   # Template with {{user_id}} placeholder


# Default Groww brand config (extract exact values from real emails)
GROWW_BRAND = BrandConfig(
    name="groww",
    display_name="Groww",
    primary_color="#00D09C",        # Groww green
    secondary_color="#5367FF",      # Groww blue
    accent_color="#00D09C",
    text_color="#44475B",
    background_color="#F5F5F5",
    card_background="#FFFFFF",
    font_family="Inter, Arial, Helvetica, sans-serif",
    logo_url="https://cdn.groww.in/...",           # TODO: Get actual URL
    footer_logo_url="https://cdn.groww.in/...",    # TODO: Get actual URL
    sebi_registration="INZ000169632",              # TODO: Verify
    social_links={
        "twitter": "https://twitter.com/GrowwApp",
        "instagram": "https://instagram.com/groww",
        "youtube": "https://youtube.com/groww",
        "linkedin": "https://linkedin.com/company/groww",
    },
    app_store_url="https://apps.apple.com/app/groww/...",
    play_store_url="https://play.google.com/store/apps/details?id=com.nextbillion.groww",
    unsubscribe_url_template="https://groww.in/unsubscribe?uid={{user_id}}",
)


# Email type classification labels
EMAIL_TYPES = [
    "educational",
    "onboarding",
    "promotional",
    "commodity_update",
    "trading_alert",
    "feature_announcement",
    "weekly_digest",
    "transactional",
    "reactivation",
    "regulatory",
]

# Block type identifiers (all 18)
BLOCK_TYPES = [
    "header_logo",
    "hero_text",
    "hero_image",
    "chart_image",
    "article_section",
    "factor_card",
    "cta_button",
    "social_proof_banner",
    "footer_standard",
    "divider",
    "card_wrapper",
    "contract_row",
    "filter_pills",
    "listing_header",
    "step_instruction",
    "app_screenshot_pair",
    "product_category_grid",
    "social_showcase",
]
```

## 2.2 Component 1: Email Analyzer Pipeline

This is the first thing you build. It takes raw HTML emails and produces structured JSON representations.

### Step 1: HTML Parser

**File: `src/analyzer/parser.py`**

```python
"""
Parse raw HTML email into a normalized tree structure.
Email HTML is table-based, deeply nested. This parser flattens it
into a list of "semantic nodes" — each node represents a visual
element the user sees (heading, paragraph, image, button, etc.)
"""

from dataclasses import dataclass, field
from bs4 import BeautifulSoup, Tag
from enum import Enum


class NodeType(Enum):
    IMAGE = "image"
    HEADING = "heading"
    PARAGRAPH = "paragraph"
    BUTTON = "button"
    DIVIDER = "divider"
    SPACER = "spacer"
    CONTAINER = "container"
    LINK = "link"
    ICON_GROUP = "icon_group"
    UNKNOWN = "unknown"


@dataclass
class SemanticNode:
    """A single visual element extracted from the email HTML."""
    node_type: NodeType
    content: str = ""                    # Text content (for headings, paragraphs)
    src: str = ""                        # Image source URL
    href: str = ""                       # Link URL (for buttons, links)
    alt: str = ""                        # Image alt text
    styles: dict = field(default_factory=dict)  # Relevant inline styles
    children: list = field(default_factory=list)
    raw_html: str = ""                   # Original HTML snippet
    position: int = 0                    # Order in the email (top to bottom)


class EmailParser:
    """
    Parses raw HTML email into a flat list of SemanticNodes.
    
    Email HTML is typically structured as:
    <html>
      <body>
        <table>           ← outer wrapper table
          <tr><td>
            <table>       ← content table (600px centered)
              <tr><td>    ← each row is roughly one "block"
                ...actual content...
              </td></tr>
            </table>
          </td></tr>
        </table>
      </body>
    </html>
    
    Strategy: find the innermost content table, iterate its rows,
    and classify each row's content into SemanticNodes.
    """

    def parse(self, html: str) -> list[SemanticNode]:
        soup = BeautifulSoup(html, "lxml")
        
        # Step 1: Find the main content table
        # Usually the table with width="600" or max-width: 600px
        content_table = self._find_content_table(soup)
        if not content_table:
            # Fallback: use body directly
            content_table = soup.body or soup

        # Step 2: Extract semantic nodes from each row
        nodes = []
        rows = content_table.find_all("tr", recursive=False)
        
        # If no direct <tr> children, look deeper
        if not rows:
            rows = content_table.find_all("tr")
        
        for i, row in enumerate(rows):
            extracted = self._extract_node(row, position=i)
            if extracted:
                nodes.extend(extracted if isinstance(extracted, list) else [extracted])

        return nodes

    def _find_content_table(self, soup: BeautifulSoup) -> Tag | None:
        """Find the main 600px content table."""
        for table in soup.find_all("table"):
            width = table.get("width", "")
            style = table.get("style", "")
            if "600" in str(width) or "600px" in style:
                return table
        return None

    def _extract_node(self, row: Tag, position: int) -> SemanticNode | list[SemanticNode] | None:
        """Classify a table row into one or more SemanticNodes."""
        
        # Skip empty/spacer rows
        text = row.get_text(strip=True)
        if not text and not row.find("img"):
            # Check if it's a divider (hr or thin line)
            if row.find("hr") or self._is_divider_style(row):
                return SemanticNode(node_type=NodeType.DIVIDER, position=position)
            # Spacer row
            return None

        nodes = []

        # Check for images
        images = row.find_all("img")
        for img in images:
            src = img.get("src", "")
            alt = img.get("alt", "")
            # Skip tiny tracking pixels
            width = img.get("width", "999")
            try:
                if int(width) < 10:
                    continue
            except (ValueError, TypeError):
                pass
            
            nodes.append(SemanticNode(
                node_type=NodeType.IMAGE,
                src=src,
                alt=alt,
                styles=self._extract_styles(img),
                raw_html=str(img),
                position=position,
            ))

        # Check for headings (by font size or tag)
        headings = row.find_all(["h1", "h2", "h3"])
        if not headings:
            # Check for large font-size in inline styles
            for el in row.find_all(["td", "p", "span", "div"]):
                style = el.get("style", "")
                if self._get_font_size(style) >= 20:
                    headings.append(el)

        for h in headings:
            nodes.append(SemanticNode(
                node_type=NodeType.HEADING,
                content=h.get_text(strip=True),
                styles=self._extract_styles(h),
                raw_html=str(h),
                position=position,
            ))

        # Check for buttons (styled <a> tags)
        for link in row.find_all("a"):
            style = link.get("style", "")
            if self._is_button_style(style) or link.find("table"):  # buttons often use nested tables
                nodes.append(SemanticNode(
                    node_type=NodeType.BUTTON,
                    content=link.get_text(strip=True),
                    href=link.get("href", ""),
                    styles=self._extract_styles(link),
                    raw_html=str(link),
                    position=position,
                ))

        # Check for paragraphs (remaining text content)
        if not nodes and text:
            nodes.append(SemanticNode(
                node_type=NodeType.PARAGRAPH,
                content=text,
                styles=self._extract_styles(row),
                raw_html=str(row),
                position=position,
            ))

        return nodes if nodes else None

    def _extract_styles(self, tag: Tag) -> dict:
        """Extract relevant inline styles as a dictionary."""
        style = tag.get("style", "")
        result = {}
        for prop in style.split(";"):
            prop = prop.strip()
            if ":" in prop:
                key, val = prop.split(":", 1)
                key = key.strip().lower()
                val = val.strip()
                if key in ("font-size", "color", "background-color", "background",
                           "text-align", "font-weight", "border-radius", "padding",
                           "width", "max-width", "height"):
                    result[key] = val
        return result

    def _get_font_size(self, style: str) -> int:
        """Extract font-size in px from inline style. Returns 0 if not found."""
        for prop in style.split(";"):
            if "font-size" in prop:
                val = prop.split(":")[1].strip()
                try:
                    return int(val.replace("px", "").strip())
                except ValueError:
                    return 0
        return 0

    def _is_button_style(self, style: str) -> bool:
        """Check if an <a> tag is styled as a button."""
        style_lower = style.lower()
        has_bg = "background-color" in style_lower or "background:" in style_lower
        has_radius = "border-radius" in style_lower
        has_padding = "padding" in style_lower
        return has_bg and (has_radius or has_padding)

    def _is_divider_style(self, tag: Tag) -> bool:
        """Check if a row is a visual divider."""
        style = tag.get("style", "")
        # Common divider patterns: thin height, border-top/bottom
        if "border-top" in style or "border-bottom" in style:
            return True
        # Check for very short rows (spacers that act as dividers)
        height = tag.get("height", "")
        if height and int(height) <= 3:
            return True
        return False
```

### Step 2: Block Detector

**File: `src/analyzer/block_detector.py`**

```python
"""
Takes a list of SemanticNodes from the parser and groups them
into email blocks (header_logo, hero_text, article_section, etc.)

Strategy: Use sequential pattern matching. Walk through nodes top-to-bottom
and match against known block patterns.
"""

from dataclasses import dataclass, field
from .parser import SemanticNode, NodeType


@dataclass
class DetectedBlock:
    """A detected email block with its type and content."""
    block_type: str                      # One of the 18 block types
    order: int                           # Position in email (0-indexed)
    content: dict = field(default_factory=dict)  # Extracted content
    confidence: float = 1.0              # Detection confidence (0-1)
    source_nodes: list[SemanticNode] = field(default_factory=list)


class BlockDetector:
    """
    Detects block types from a sequence of SemanticNodes.
    
    Each detect_* method tries to match a pattern at the current
    position in the node list. If it matches, it returns a
    DetectedBlock and the number of nodes consumed.
    """

    def detect(self, nodes: list[SemanticNode]) -> list[DetectedBlock]:
        blocks = []
        i = 0
        block_order = 0

        while i < len(nodes):
            # Try each detector in priority order
            # (Order matters — more specific detectors first)
            result = None

            for detector in [
                self._detect_header_logo,
                self._detect_footer_standard,
                self._detect_cta_button,
                self._detect_divider,
                self._detect_social_proof_banner,
                self._detect_chart_image,
                self._detect_hero_image,
                self._detect_factor_card,
                self._detect_contract_row,
                self._detect_step_instruction,
                self._detect_product_category_grid,
                self._detect_app_screenshot_pair,
                self._detect_hero_text,
                self._detect_article_section,
            ]:
                result = detector(nodes, i)
                if result:
                    block, consumed = result
                    block.order = block_order
                    blocks.append(block)
                    i += consumed
                    block_order += 1
                    break

            if not result:
                # Skip unrecognized node
                i += 1

        return blocks

    def _detect_header_logo(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect brand logo at the top of email."""
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE:
            return None
        # Header logo is typically the first image, contains brand name in URL or alt
        if pos > 3:  # Logo should be near the top
            return None
        src_lower = node.src.lower()
        alt_lower = node.alt.lower()
        if any(kw in src_lower or kw in alt_lower for kw in ["logo", "groww", "915", "header"]):
            brand = "groww"  # Default
            if "915" in src_lower or "915" in alt_lower:
                brand = "915"
            return DetectedBlock(
                block_type="header_logo",
                content={"brand": brand, "image_url": node.src},
                source_nodes=[node],
            ), 1
        return None

    def _detect_hero_text(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect main headline + subtitle block."""
        node = nodes[pos]
        if node.node_type != NodeType.HEADING:
            return None
        # Hero text is a heading near the top, potentially followed by a subtitle paragraph
        if pos > 8:  # Should be in the top portion
            return None
        
        content = {"headline": node.content}
        consumed = 1

        # Check if next node is a subtitle (paragraph right after heading)
        if pos + 1 < len(nodes) and nodes[pos + 1].node_type == NodeType.PARAGRAPH:
            content["subtitle"] = nodes[pos + 1].content
            consumed = 2

        return DetectedBlock(
            block_type="hero_text",
            content=content,
            source_nodes=nodes[pos:pos + consumed],
        ), consumed

    def _detect_chart_image(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect chart/screenshot images (price cards, listings, etc.)."""
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE:
            return None
        src_lower = node.src.lower()
        # Chart images are usually large, with specific URL patterns or dimensions
        width = node.styles.get("width", "")
        is_chart = any(kw in src_lower for kw in ["chart", "price", "listing", "screenshot", "graph"])
        is_large = "100%" in width or "500" in width or "400" in width
        
        if is_chart or is_large:
            # Try to infer chart type from URL or alt text
            chart_type = "unknown"
            if "price" in src_lower:
                chart_type = "price_card"
            elif "listing" in src_lower or "commodit" in src_lower:
                chart_type = "listing_card"
            
            return DetectedBlock(
                block_type="chart_image",
                content={"chart_type": chart_type, "image_url": node.src, "alt": node.alt},
                source_nodes=[node],
            ), 1
        return None

    def _detect_cta_button(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect call-to-action buttons."""
        node = nodes[pos]
        if node.node_type != NodeType.BUTTON:
            return None
        bg = node.styles.get("background-color", node.styles.get("background", ""))
        return DetectedBlock(
            block_type="cta_button",
            content={"text": node.content, "url": node.href, "color": bg},
            source_nodes=[node],
        ), 1

    def _detect_divider(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect horizontal line dividers."""
        node = nodes[pos]
        if node.node_type == NodeType.DIVIDER:
            return DetectedBlock(
                block_type="divider",
                content={},
                source_nodes=[node],
            ), 1
        return None

    def _detect_footer_standard(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect standard footer (SEBI disclaimer, social links, unsubscribe)."""
        node = nodes[pos]
        text_lower = node.content.lower() if node.content else ""
        # Footer detection: look for SEBI, unsubscribe, or social media keywords
        if any(kw in text_lower for kw in ["sebi", "unsubscribe", "securities and exchange"]):
            # Consume all remaining nodes as part of footer
            remaining = len(nodes) - pos
            return DetectedBlock(
                block_type="footer_standard",
                content={"text": node.content},
                source_nodes=nodes[pos:],
            ), remaining
        return None

    def _detect_social_proof_banner(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect trust/credibility banners."""
        node = nodes[pos]
        text_lower = node.content.lower() if node.content else ""
        if any(kw in text_lower for kw in ["crore", "million", "users", "rating", "trusted", "app store"]):
            return DetectedBlock(
                block_type="social_proof_banner",
                content={"text": node.content},
                source_nodes=[node],
            ), 1
        return None

    def _detect_factor_card(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """Detect mint/teal-tinted explainer cards."""
        node = nodes[pos]
        bg = node.styles.get("background-color", node.styles.get("background", "")).lower()
        # Factor cards have a light mint/teal background
        if any(color in bg for color in ["#e8f8f3", "#f0faf7", "rgba(0, 208, 156", "mint", "teal"]):
            # Card typically has a bold title and description
            return DetectedBlock(
                block_type="factor_card",
                content={"text": node.content},
                confidence=0.8,
                source_nodes=[node],
            ), 1
        return None

    def _detect_article_section(self, nodes: list[SemanticNode], pos: int) -> tuple[DetectedBlock, int] | None:
        """
        Detect article sections (heading + paragraphs).
        This is the catch-all for text content blocks.
        """
        node = nodes[pos]
        if node.node_type == NodeType.HEADING:
            content = {"heading": node.content, "paragraphs": []}
            consumed = 1
            # Consume following paragraphs until next heading, image, or button
            while pos + consumed < len(nodes):
                next_node = nodes[pos + consumed]
                if next_node.node_type == NodeType.PARAGRAPH:
                    content["paragraphs"].append(next_node.content)
                    consumed += 1
                else:
                    break
            return DetectedBlock(
                block_type="article_section",
                content=content,
                source_nodes=nodes[pos:pos + consumed],
            ), consumed
        
        # Standalone paragraph (no heading)
        if node.node_type == NodeType.PARAGRAPH and len(node.content) > 30:
            return DetectedBlock(
                block_type="article_section",
                content={"heading": "", "paragraphs": [node.content]},
                source_nodes=[node],
            ), 1
        
        return None

    def _detect_hero_image(self, nodes, pos):
        """Full-width hero image (not a chart)."""
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE:
            return None
        # Hero image is large but not a chart
        if pos <= 5:  # Near top
            src_lower = node.src.lower()
            if not any(kw in src_lower for kw in ["chart", "price", "listing", "logo"]):
                return DetectedBlock(
                    block_type="hero_image",
                    content={"image_url": node.src, "alt": node.alt},
                    source_nodes=[node],
                ), 1
        return None

    def _detect_contract_row(self, nodes, pos):
        """Commodity contract line (has price, change, MCX keyword)."""
        node = nodes[pos]
        text = node.content.lower() if node.content else ""
        if any(kw in text for kw in ["mcx", "lot size", "expiry"]):
            return DetectedBlock(
                block_type="contract_row",
                content={"text": node.content},
                source_nodes=[node],
            ), 1
        return None

    def _detect_step_instruction(self, nodes, pos):
        """Numbered how-to step."""
        node = nodes[pos]
        text = node.content.strip() if node.content else ""
        # Steps typically start with "Step 1", "1.", etc.
        if text and (text[:4].lower().startswith("step") or 
                     (text[0].isdigit() and text[1] in ".)")):
            return DetectedBlock(
                block_type="step_instruction",
                content={"text": text},
                source_nodes=[node],
            ), 1
        return None

    def _detect_product_category_grid(self, nodes, pos):
        """2x2 grid of product categories."""
        # Hard to detect from flat nodes — look for 4 consecutive small images with labels
        if pos + 3 >= len(nodes):
            return None
        # Check if next 4 nodes are images of similar size
        next_four = nodes[pos:pos + 4]
        if all(n.node_type == NodeType.IMAGE for n in next_four):
            return DetectedBlock(
                block_type="product_category_grid",
                content={"categories": [{"image_url": n.src, "alt": n.alt} for n in next_four]},
                source_nodes=next_four,
            ), 4
        return None

    def _detect_app_screenshot_pair(self, nodes, pos):
        """Side-by-side app screenshots."""
        if pos + 1 >= len(nodes):
            return None
        node1, node2 = nodes[pos], nodes[pos + 1]
        if (node1.node_type == NodeType.IMAGE and node2.node_type == NodeType.IMAGE):
            src1, src2 = node1.src.lower(), node2.src.lower()
            if any(kw in src1 + src2 for kw in ["screenshot", "screen", "app"]):
                return DetectedBlock(
                    block_type="app_screenshot_pair",
                    content={"left_image_url": node1.src, "right_image_url": node2.src},
                    source_nodes=[node1, node2],
                ), 2
        return None
```

### Step 3: Email Classifier

**File: `src/analyzer/classifier.py`**

```python
"""
Classifies emails by type based on detected blocks, subject line,
and content keywords.
"""

from dataclasses import dataclass
from .block_detector import DetectedBlock


@dataclass
class Classification:
    email_type: str           # Primary type
    sub_type: str = ""        # Sub-type
    confidence: float = 0.0   # 0-1
    signals: list[str] = None # Why this classification

    def __post_init__(self):
        if self.signals is None:
            self.signals = []


class EmailClassifier:
    """Rule-based email type classifier."""

    def classify(
        self,
        blocks: list[DetectedBlock],
        subject: str = "",
        from_address: str = "",
    ) -> Classification:
        subject_lower = subject.lower()
        block_types = [b.block_type for b in blocks]
        all_text = " ".join(
            b.content.get("text", "") + " " +
            b.content.get("headline", "") + " " +
            " ".join(b.content.get("paragraphs", []))
            for b in blocks
        ).lower()

        scores: dict[str, tuple[float, list[str]]] = {}

        # --- Commodity Update ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["crude", "gold", "silver", "commodity", "mcx", "natural gas"]):
            score += 0.4
            signals.append("commodity keyword in subject")
        if "contract_row" in block_types:
            score += 0.3
            signals.append("has contract_row block")
        if "factor_card" in block_types and any(kw in all_text for kw in ["price", "move", "opec", "supply"]):
            score += 0.2
            signals.append("factor cards about price drivers")
        scores["commodity_update"] = (score, signals)

        # --- Educational ---
        score, signals = 0.0, []
        if block_types.count("article_section") >= 3:
            score += 0.3
            signals.append("3+ article sections")
        if "factor_card" in block_types:
            score += 0.2
            signals.append("has factor cards")
        if "chart_image" in block_types:
            score += 0.2
            signals.append("has chart images")
        if any(kw in subject_lower for kw in ["what", "how", "why", "understand", "learn", "guide"]):
            score += 0.2
            signals.append("educational keyword in subject")
        scores["educational"] = (score, signals)

        # --- Onboarding ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["welcome", "get started", "first step", "hello"]):
            score += 0.4
            signals.append("welcome keyword in subject")
        if "product_category_grid" in block_types:
            score += 0.3
            signals.append("has product category grid")
        if "step_instruction" in block_types:
            score += 0.2
            signals.append("has step instructions")
        scores["onboarding"] = (score, signals)

        # --- Promotional ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["offer", "free", "exclusive", "limited", "deal", "reward"]):
            score += 0.4
            signals.append("promotional keyword in subject")
        cta_count = block_types.count("cta_button")
        if cta_count >= 2:
            score += 0.2
            signals.append(f"{cta_count} CTA buttons")
        if block_types.count("article_section") <= 2:
            score += 0.1
            signals.append("few article sections (action-focused)")
        scores["promotional"] = (score, signals)

        # --- Trading Alert ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["alert", "watchlist", "target", "breakout", "52-week"]):
            score += 0.4
            signals.append("alert keyword in subject")
        if "chart_image" in block_types and block_types.count("article_section") <= 2:
            score += 0.2
            signals.append("chart + minimal text = alert pattern")
        scores["trading_alert"] = (score, signals)

        # --- Feature Announcement ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["new", "introducing", "launch", "now available", "update"]):
            score += 0.3
            signals.append("announcement keyword in subject")
        if "app_screenshot_pair" in block_types or "hero_image" in block_types:
            score += 0.2
            signals.append("has app screenshots or hero image")
        scores["feature_announcement"] = (score, signals)

        # --- Weekly Digest ---
        score, signals = 0.0, []
        if any(kw in subject_lower for kw in ["weekly", "digest", "roundup", "recap", "this week"]):
            score += 0.5
            signals.append("digest keyword in subject")
        scores["weekly_digest"] = (score, signals)

        # Pick the highest scoring type
        best_type = max(scores, key=lambda k: scores[k][0])
        best_score, best_signals = scores[best_type]

        # If score is too low, fall back to generic
        if best_score < 0.3:
            best_type = "educational"  # Default fallback
            best_signals = ["low confidence — defaulting to educational"]
            best_score = 0.3

        return Classification(
            email_type=best_type,
            confidence=best_score,
            signals=best_signals,
        )
```

### Step 4: Analysis Pipeline

**File: `src/analyzer/pipeline.py`**

```python
"""
Orchestrates the full analysis pipeline:
  raw HTML → parse → detect blocks → classify → output JSON
"""

import json
from pathlib import Path
from dataclasses import dataclass, asdict
from .parser import EmailParser
from .block_detector import BlockDetector, DetectedBlock
from .classifier import EmailClassifier, Classification


@dataclass
class AnalyzedEmail:
    source_file: str
    classification: Classification
    blocks: list[DetectedBlock]
    metadata: dict


class AnalysisPipeline:
    def __init__(self):
        self.parser = EmailParser()
        self.detector = BlockDetector()
        self.classifier = EmailClassifier()

    def analyze_file(self, html_path: Path) -> AnalyzedEmail:
        """Analyze a single HTML email file."""
        html = html_path.read_text(encoding="utf-8", errors="replace")
        return self.analyze_html(html, source_file=html_path.name)

    def analyze_html(self, html: str, source_file: str = "unknown") -> AnalyzedEmail:
        """Analyze raw HTML string."""
        # Step 1: Parse HTML into semantic nodes
        nodes = self.parser.parse(html)

        # Step 2: Detect blocks
        blocks = self.detector.detect(nodes)

        # Step 3: Extract subject from HTML <title> or first heading
        subject = self._extract_subject(html, blocks)

        # Step 4: Classify email type
        classification = self.classifier.classify(blocks, subject=subject)

        # Step 5: Compute metadata
        all_text = " ".join(
            b.content.get("text", "") + " " +
            b.content.get("headline", "") + " " +
            " ".join(b.content.get("paragraphs", []))
            for b in blocks
        )
        metadata = {
            "subject": subject,
            "block_count": len(blocks),
            "block_types": [b.block_type for b in blocks],
            "word_count": len(all_text.split()),
            "image_count": sum(1 for b in blocks if b.block_type in ("chart_image", "hero_image")),
            "cta_count": sum(1 for b in blocks if b.block_type == "cta_button"),
        }

        return AnalyzedEmail(
            source_file=source_file,
            classification=classification,
            blocks=blocks,
            metadata=metadata,
        )

    def analyze_directory(self, dir_path: Path, output_dir: Path) -> list[AnalyzedEmail]:
        """Analyze all HTML files in a directory."""
        output_dir.mkdir(parents=True, exist_ok=True)
        results = []

        html_files = sorted(dir_path.glob("*.html"))
        print(f"Found {len(html_files)} HTML files to analyze")

        for i, html_file in enumerate(html_files):
            try:
                result = self.analyze_file(html_file)
                results.append(result)

                # Save individual analysis JSON
                out_file = output_dir / f"{html_file.stem}.json"
                out_file.write_text(json.dumps(asdict(result), indent=2, default=str))

                print(f"  [{i+1}/{len(html_files)}] {html_file.name} → {result.classification.email_type} "
                      f"({result.classification.confidence:.0%}) [{result.metadata['block_count']} blocks]")
            except Exception as e:
                print(f"  [{i+1}/{len(html_files)}] ERROR: {html_file.name} → {e}")

        # Save summary
        summary = {
            "total": len(results),
            "by_type": {},
            "avg_blocks": sum(r.metadata["block_count"] for r in results) / max(len(results), 1),
        }
        for r in results:
            t = r.classification.email_type
            summary["by_type"][t] = summary["by_type"].get(t, 0) + 1

        (output_dir / "_summary.json").write_text(json.dumps(summary, indent=2))
        print(f"\nSummary: {summary}")

        return results

    def _extract_subject(self, html: str, blocks: list[DetectedBlock]) -> str:
        """Try to extract email subject."""
        from bs4 import BeautifulSoup
        soup = BeautifulSoup(html, "lxml")
        title = soup.find("title")
        if title and title.string:
            return title.string.strip()
        # Fallback: use first hero_text headline
        for b in blocks:
            if b.block_type == "hero_text" and b.content.get("headline"):
                return b.content["headline"]
        return ""
```

### Step 5: CLI to run the analyzer

**File: `cli.py`** (top-level)

```python
"""CLI entry point for all Content Engine tools."""

import typer
from pathlib import Path
from rich.console import Console

app = typer.Typer(name="content-engine", help="Groww Content Engine CLI")
console = Console()


@app.command()
def analyze(
    input_dir: Path = typer.Argument(..., help="Directory containing HTML email files"),
    output_dir: Path = typer.Option("data/analyzed", help="Output directory for analysis JSONs"),
):
    """Analyze HTML emails and decompose into blocks."""
    from src.analyzer.pipeline import AnalysisPipeline

    pipeline = AnalysisPipeline()
    results = pipeline.analyze_directory(input_dir, output_dir)
    console.print(f"\n[green]✓ Analyzed {len(results)} emails[/green]")


@app.command()
def generate_chart(
    chart_type: str = typer.Argument(..., help="Chart type: price_card, listing_card, comparison, portfolio, metric"),
    data_file: Path = typer.Argument(..., help="JSON file with chart data"),
    output_path: Path = typer.Option("output/images/chart.png", help="Output PNG path"),
):
    """Generate a chart image from data."""
    import json
    import asyncio
    from src.charts.renderer import ChartRenderer

    data = json.loads(data_file.read_text())
    renderer = ChartRenderer()
    asyncio.run(renderer.render(chart_type, data, output_path))
    console.print(f"[green]✓ Chart saved to {output_path}[/green]")


@app.command()
def render_email(
    recipe_file: Path = typer.Argument(..., help="Recipe JSON file"),
    content_file: Path = typer.Argument(..., help="Content JSON file (filled block data)"),
    output_path: Path = typer.Option("output/html/email.html", help="Output HTML path"),
):
    """Render a recipe + content into HTML email."""
    import json
    from src.renderer.engine import EmailRenderer

    recipe = json.loads(recipe_file.read_text())
    content = json.loads(content_file.read_text())
    renderer = EmailRenderer()
    html = renderer.render(recipe, content)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(html)
    console.print(f"[green]✓ Email HTML saved to {output_path}[/green]")


@app.command()
def serve_mcp():
    """Start the Content Engine MCP server."""
    from src.mcp.server import run_server
    run_server()


@app.command()
def discover_schema(
    catalog: str = typer.Argument(..., help="Catalog path, e.g. platform_iceberg.growth"),
    output_file: Path = typer.Option(None, help="Output markdown file"),
):
    """[Compass] Discover table schemas in a catalog."""
    from compass_skills.tools.schema_discovery import discover
    discover(catalog, output_file)


@app.command()
def scaffold_skill(
    domain: str = typer.Argument(..., help="Skill domain name, e.g. 'stocks_trading'"),
    tables: list[str] = typer.Option([], "--table", "-t", help="Table names to include"),
):
    """[Compass] Generate a skill document scaffold."""
    from compass_skills.tools.skill_generator import generate_scaffold
    generate_scaffold(domain, tables)


@app.command()
def test_queries(
    skill_dir: Path = typer.Argument(..., help="Directory containing SQL test files"),
):
    """[Compass] Test all SQL queries in a skill's test directory."""
    from compass_skills.tools.query_tester import test_all
    test_all(skill_dir)


if __name__ == "__main__":
    app()
```

## 2.3 Component 2: Block Schemas (Pydantic Models)

**File: `src/blocks/schemas.py`**

```python
"""
Pydantic models for all 18 email block types.
These define the data contract — what data each block needs to render.
"""

from pydantic import BaseModel, Field
from enum import Enum
from typing import Optional


class Brand(str, Enum):
    GROWW = "groww"
    NINE_ONE_FIVE = "915"
    AMC = "amc"
    W = "w"


class Alignment(str, Enum):
    CENTER = "center"
    LEFT = "left"


# --- Block 1: Header Logo ---
class HeaderLogoBlock(BaseModel):
    block_type: str = "header_logo"
    brand: Brand = Brand.GROWW


# --- Block 2: Hero Text ---
class HeroTextBlock(BaseModel):
    block_type: str = "hero_text"
    headline: str = Field(..., max_length=100)
    subtitle: str = Field("", max_length=250)
    highlight_word: str = ""      # Word to color in brand accent
    alignment: Alignment = Alignment.CENTER


# --- Block 3: Hero Image ---
class HeroImageBlock(BaseModel):
    block_type: str = "hero_image"
    image_url: str
    alt_text: str = ""
    link_url: str = ""


# --- Block 4: Chart Image ---
class ChartType(str, Enum):
    PRICE_CARD = "price_card"
    LISTING_CARD = "listing_card"
    COMPARISON = "comparison"
    PORTFOLIO = "portfolio"
    METRIC = "metric"


class ChartImageBlock(BaseModel):
    block_type: str = "chart_image"
    chart_type: ChartType
    chart_data: dict                 # Data payload — schema depends on chart_type
    image_url: str = ""              # Populated after rendering
    alt_text: str = ""


# --- Block 5: Article Section ---
class ArticleSectionBlock(BaseModel):
    block_type: str = "article_section"
    heading: str = ""
    paragraphs: list[str] = []
    alignment: Alignment = Alignment.CENTER


# --- Block 6: Factor Card ---
class FactorCardBlock(BaseModel):
    block_type: str = "factor_card"
    title: str = Field(..., max_length=60)
    description: str = Field(..., max_length=200)


# --- Block 7: CTA Button ---
class CTAColor(str, Enum):
    GREEN = "green"
    BLUE = "blue"


class CTAButtonBlock(BaseModel):
    block_type: str = "cta_button"
    text: str = Field(..., max_length=30)
    url: str
    color: CTAColor = CTAColor.GREEN


# --- Block 8: Social Proof Banner ---
class SocialProofBannerBlock(BaseModel):
    block_type: str = "social_proof_banner"
    text: str = ""
    metrics: list[dict] = []   # [{"label": "Users", "value": "10M+"}]


# --- Block 9: Footer Standard ---
class FooterStandardBlock(BaseModel):
    block_type: str = "footer_standard"
    brand: Brand = Brand.GROWW


# --- Block 10: Divider ---
class DividerBlock(BaseModel):
    block_type: str = "divider"


# --- Block 11: Card Wrapper ---
class CardWrapperBlock(BaseModel):
    block_type: str = "card_wrapper"
    children: list[dict] = []     # Nested blocks (rendered recursively)


# --- Block 12: Contract Row ---
class ContractRowBlock(BaseModel):
    block_type: str = "contract_row"
    commodity: str                  # "Crude Oil 20 Apr"
    exchange: str = "MCX"
    price: float
    change: float
    change_pct: float
    lot_size: Optional[int] = None
    expiry: str = ""


# --- Block 13: Filter Pills ---
class FilterPillsBlock(BaseModel):
    block_type: str = "filter_pills"
    pills: list[str]               # ["All", "Crude Oil", "Gold", "Natural Gas"]
    active: str = ""               # Currently active pill


# --- Block 14: Listing Header ---
class ListingHeaderBlock(BaseModel):
    block_type: str = "listing_header"
    title: str


# --- Block 15: Step Instruction ---
class StepInstructionBlock(BaseModel):
    block_type: str = "step_instruction"
    step_number: int
    title: str
    description: str = ""
    image_url: str = ""


# --- Block 16: App Screenshot Pair ---
class AppScreenshotPairBlock(BaseModel):
    block_type: str = "app_screenshot_pair"
    left_image_url: str
    right_image_url: str
    left_caption: str = ""
    right_caption: str = ""


# --- Block 17: Product Category Grid ---
class ProductCategory(BaseModel):
    name: str
    icon_url: str
    link_url: str = ""


class ProductCategoryGridBlock(BaseModel):
    block_type: str = "product_category_grid"
    categories: list[ProductCategory]   # Exactly 4 for 2x2 grid


# --- Block 18: Social Showcase ---
class SocialShowcaseBlock(BaseModel):
    block_type: str = "social_showcase"
    metrics: list[dict]             # [{"label": "App Rating", "value": "4.6 ★"}]


# --- Registry ---
BLOCK_SCHEMA_MAP = {
    "header_logo": HeaderLogoBlock,
    "hero_text": HeroTextBlock,
    "hero_image": HeroImageBlock,
    "chart_image": ChartImageBlock,
    "article_section": ArticleSectionBlock,
    "factor_card": FactorCardBlock,
    "cta_button": CTAButtonBlock,
    "social_proof_banner": SocialProofBannerBlock,
    "footer_standard": FooterStandardBlock,
    "divider": DividerBlock,
    "card_wrapper": CardWrapperBlock,
    "contract_row": ContractRowBlock,
    "filter_pills": FilterPillsBlock,
    "listing_header": ListingHeaderBlock,
    "step_instruction": StepInstructionBlock,
    "app_screenshot_pair": AppScreenshotPairBlock,
    "product_category_grid": ProductCategoryGridBlock,
    "social_showcase": SocialShowcaseBlock,
}
```

## 2.4 Component 4: Chart Renderer

**File: `src/charts/renderer.py`**

```python
"""
Renders chart images using Playwright (HTML → screenshot → PNG).
Each chart type has an HTML/CSS template that gets populated with data
and screenshotted at retina (2x) resolution.
"""

import json
from pathlib import Path
from playwright.async_api import async_playwright

CHART_TEMPLATES_DIR = Path(__file__).parent / "templates"


class ChartRenderer:
    """Render chart data into retina PNG images."""

    async def render(
        self,
        chart_type: str,
        data: dict,
        output_path: Path,
        scale: int = 2,      # 2x for retina
    ) -> Path:
        """
        Render a chart to PNG.
        
        Args:
            chart_type: One of price_card, listing_card, comparison, portfolio, metric
            data: Chart data (schema depends on chart_type)
            output_path: Where to save the PNG
            scale: Pixel scale factor (2 = retina)
        
        Returns:
            Path to the generated PNG
        """
        template_path = CHART_TEMPLATES_DIR / f"{chart_type}.html"
        if not template_path.exists():
            raise ValueError(f"No template found for chart type: {chart_type}")

        # Read template and inject data
        template_html = template_path.read_text()
        rendered_html = template_html.replace(
            "/*__CHART_DATA__*/",
            f"const CHART_DATA = {json.dumps(data)};"
        )

        # Render with Playwright
        async with async_playwright() as p:
            browser = await p.chromium.launch()
            page = await browser.new_page(
                viewport={"width": 600, "height": 800},
                device_scale_factor=scale,
            )
            await page.set_content(rendered_html)
            
            # Wait for any rendering to complete
            await page.wait_for_timeout(500)

            # Screenshot the chart container
            chart_el = await page.query_selector("#chart-container")
            if chart_el:
                await chart_el.screenshot(path=str(output_path))
            else:
                await page.screenshot(path=str(output_path))

            await browser.close()

        return output_path
```

**File: `src/charts/templates/price_card.html`**

```html
<!DOCTYPE html>
<html>
<head>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { 
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif; 
    background: transparent;
  }
  #chart-container {
    width: 520px;
    background: #FFFFFF;
    border-radius: 12px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08);
    border: 1px solid #E8E8E8;
    padding: 20px;
    display: inline-block;
  }
  .header-row {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 4px;
  }
  .exchange { color: #7C7E8C; font-size: 12px; font-weight: 500; }
  .instrument { font-size: 14px; color: #44475B; font-weight: 500; margin-top: 2px; }
  .lot-size { color: #7C7E8C; font-size: 11px; text-align: right; }
  .price { font-size: 24px; font-weight: 700; color: #44475B; margin-top: 12px; }
  .change { font-size: 14px; margin-top: 4px; }
  .change.positive { color: #00D09C; }
  .change.negative { color: #EB5B3C; }
  .sparkline-container { 
    margin-top: 16px; 
    height: 100px; 
    position: relative;
  }
  canvas { width: 100%; height: 100%; }
  .period-selector {
    display: flex;
    gap: 16px;
    justify-content: center;
    margin-top: 12px;
  }
  .period {
    font-size: 12px;
    color: #7C7E8C;
    cursor: pointer;
    padding: 4px 8px;
    border-radius: 12px;
  }
  .period.active {
    background: #F0F0F0;
    color: #44475B;
    font-weight: 600;
  }
  .watchlist-icon {
    width: 20px;
    height: 20px;
    opacity: 0.5;
  }
</style>
</head>
<body>
<div id="chart-container">
  <div class="header-row">
    <div>
      <div class="exchange" id="exchange"></div>
      <div class="instrument" id="instrument"></div>
    </div>
    <div>
      <div class="lot-size" id="lot-size"></div>
    </div>
  </div>
  <div class="price" id="price"></div>
  <div class="change" id="change"></div>
  <div class="sparkline-container">
    <canvas id="sparkline"></canvas>
  </div>
  <div class="period-selector" id="periods"></div>
</div>

<script>
/*__CHART_DATA__*/

// Populate elements
document.getElementById('exchange').textContent = CHART_DATA.exchange || 'MCX';
document.getElementById('instrument').textContent = CHART_DATA.instrument + ' ›';
document.getElementById('price').textContent = '₹' + Number(CHART_DATA.price).toLocaleString('en-IN', {minimumFractionDigits: 2});
document.getElementById('lot-size').textContent = CHART_DATA.lot_size ? `Lot size: ${CHART_DATA.lot_size} qty` : '';

const changeEl = document.getElementById('change');
const isPositive = CHART_DATA.change >= 0;
changeEl.className = `change ${isPositive ? 'positive' : 'negative'}`;
const sign = isPositive ? '+' : '';
changeEl.textContent = `${sign}${Number(CHART_DATA.change).toLocaleString('en-IN', {minimumFractionDigits: 2})} (${CHART_DATA.change_pct}%)  ${CHART_DATA.period || '1M'}`;

// Periods
const periods = ['1D', '1W', '1M', '3M'];
const activePeriod = CHART_DATA.period || '1M';
const periodsEl = document.getElementById('periods');
periods.forEach(p => {
  const span = document.createElement('span');
  span.className = `period ${p === activePeriod ? 'active' : ''}`;
  span.textContent = p;
  periodsEl.appendChild(span);
});

// Draw sparkline
const canvas = document.getElementById('sparkline');
const ctx = canvas.getContext('2d');
const data = CHART_DATA.sparkline_data || [];

if (data.length > 1) {
  canvas.width = canvas.offsetWidth * 2;
  canvas.height = canvas.offsetHeight * 2;
  ctx.scale(2, 2);
  
  const w = canvas.offsetWidth;
  const h = canvas.offsetHeight;
  const min = Math.min(...data);
  const max = Math.max(...data);
  const range = max - min || 1;
  const padding = 4;

  ctx.beginPath();
  ctx.strokeStyle = isPositive ? '#00D09C' : '#EB5B3C';
  ctx.lineWidth = 1.5;
  ctx.lineJoin = 'round';

  data.forEach((val, i) => {
    const x = (i / (data.length - 1)) * w;
    const y = h - padding - ((val - min) / range) * (h - 2 * padding);
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  });
  ctx.stroke();
}
</script>
</body>
</html>
```

## 2.5 Component 5: HTML Email Renderer

**File: `src/renderer/engine.py`**

```python
"""
Renders a recipe (with filled block data) into production HTML email.
Uses Jinja2 templates for each block type.
"""

import json
from pathlib import Path
from jinja2 import Environment, FileSystemLoader
from src.utils.config import TEMPLATES_DIR, BRANDS_DIR, GROWW_BRAND


class EmailRenderer:
    def __init__(self):
        self.env = Environment(
            loader=FileSystemLoader(str(TEMPLATES_DIR)),
            autoescape=False,   # HTML templates, no auto-escaping
        )

    def render(self, recipe: dict, content: dict, brand_name: str = "groww") -> str:
        """
        Render a complete email from recipe + content.
        
        Args:
            recipe: Recipe JSON (defines block order and structure)
            content: Content JSON (filled block data)
            brand_name: Brand config to use
        
        Returns:
            Complete HTML email string
        """
        # Load brand config
        brand = self._load_brand(brand_name)

        # Render each block
        rendered_blocks = []
        for i, block_def in enumerate(recipe["blocks"]):
            block_type = block_def["block"]
            
            # Merge recipe defaults with generated content
            block_data = {**block_def.get("data", {})}
            if str(i) in content.get("blocks", {}):
                block_data.update(content["blocks"][str(i)])
            elif block_type in content.get("blocks_by_type", {}):
                type_blocks = content["blocks_by_type"][block_type]
                if type_blocks:
                    block_data.update(type_blocks.pop(0))

            # Add brand context to every block
            block_data["brand"] = brand

            # Render block template
            try:
                template = self.env.get_template(f"{block_type}.html")
                rendered = template.render(**block_data)
                rendered_blocks.append(rendered)
            except Exception as e:
                rendered_blocks.append(f"<!-- ERROR rendering {block_type}: {e} -->")

        # Wrap in email boilerplate
        base_template = self.env.get_template("base.html")
        full_html = base_template.render(
            blocks="\n".join(rendered_blocks),
            brand=brand,
            subject=content.get("subject", recipe.get("meta", {}).get("subject_template", "")),
            preheader=content.get("preheader", recipe.get("meta", {}).get("preheader_template", "")),
        )

        return full_html

    def _load_brand(self, brand_name: str) -> dict:
        """Load brand config."""
        brand_file = BRANDS_DIR / f"{brand_name}.json"
        if brand_file.exists():
            return json.loads(brand_file.read_text())
        # Default to Groww
        return GROWW_BRAND.model_dump()
```

**File: `src/renderer/templates/base.html`**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{ subject }}</title>
  <!--[if !mso]><!-->
  <style type="text/css">
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
  </style>
  <!--<![endif]-->
</head>
<body style="margin: 0; padding: 0; background-color: {{ brand.background_color }}; font-family: {{ brand.font_family }};">
  <!-- Preheader text (hidden) -->
  <div style="display: none; max-height: 0; overflow: hidden;">
    {{ preheader }}
  </div>
  
  <!-- Outer wrapper table -->
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" border="0" style="background-color: {{ brand.background_color }};">
    <tr>
      <td align="center" style="padding: 20px 0;">
        <!-- Inner content table (600px) -->
        <table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0" style="background-color: {{ brand.card_background }}; border-radius: 8px;">
          {{ blocks }}
        </table>
      </td>
    </tr>
  </table>
</body>
</html>
```

**File: `src/renderer/templates/hero_text.html`**

```html
<tr>
  <td align="{{ alignment | default('center') }}" style="padding: 32px 40px 16px 40px;">
    <h1 style="margin: 0; font-size: 24px; font-weight: 700; color: {{ brand.text_color }}; line-height: 1.3; font-family: {{ brand.font_family }};">
      {% if highlight_word %}
        {{ headline | replace(highlight_word, '<span style="color: ' + brand.primary_color + ';">' + highlight_word + '</span>') }}
      {% else %}
        {{ headline }}
      {% endif %}
    </h1>
    {% if subtitle %}
    <p style="margin: 12px 0 0 0; font-size: 14px; color: #7C7E8C; line-height: 1.5; font-family: {{ brand.font_family }};">
      {{ subtitle }}
    </p>
    {% endif %}
  </td>
</tr>
```

**File: `src/renderer/templates/cta_button.html`**

```html
<tr>
  <td align="center" style="padding: 24px 40px;">
    <table role="presentation" cellpadding="0" cellspacing="0" border="0">
      <tr>
        <td align="center" style="
          background-color: {% if color == 'blue' %}{{ brand.secondary_color }}{% else %}{{ brand.primary_color }}{% endif %};
          border-radius: 8px;
          padding: 14px 48px;
        ">
          <a href="{{ url }}" target="_blank" style="
            color: #FFFFFF;
            font-size: 14px;
            font-weight: 600;
            text-decoration: none;
            font-family: {{ brand.font_family }};
            display: inline-block;
          ">{{ text }}</a>
        </td>
      </tr>
    </table>
  </td>
</tr>
```

You would create similar templates for all 18 block types. Each template uses table-based layout with inline CSS for email compatibility.

## 2.6 Component 6: MCP Server

**File: `src/mcp/server.py`**

```python
"""
Content Engine MCP Server.
Exposes tools for recipe management, email generation, and chart rendering.
"""

import json
import asyncio
from pathlib import Path
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

from src.blocks.schemas import BLOCK_SCHEMA_MAP
from src.recipes.registry import RecipeRegistry
from src.renderer.engine import EmailRenderer
from src.charts.renderer import ChartRenderer
from src.generator.copy_writer import CopyWriter
from src.utils.config import RECIPES_DIR, BLOCKS_DIR, OUTPUT_DIR


# Initialize components
recipe_registry = RecipeRegistry(RECIPES_DIR)
email_renderer = EmailRenderer()
chart_renderer = ChartRenderer()
copy_writer = CopyWriter()

server = Server("content-engine")


@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="list_recipes",
            description="List all available email recipe templates. Optionally filter by brand or email type.",
            inputSchema={
                "type": "object",
                "properties": {
                    "brand": {"type": "string", "enum": ["groww", "915", "amc", "w"], "description": "Filter by brand"},
                    "email_type": {"type": "string", "description": "Filter by email type (educational, onboarding, etc.)"},
                },
            },
        ),
        Tool(
            name="get_recipe",
            description="Get the full details of a specific recipe by ID, including its block structure and generation hints.",
            inputSchema={
                "type": "object",
                "properties": {
                    "recipe_id": {"type": "string", "description": "Recipe identifier"},
                },
                "required": ["recipe_id"],
            },
        ),
        Tool(
            name="recommend_recipe",
            description="Describe what kind of email you want to create, and get recipe recommendations ranked by relevance.",
            inputSchema={
                "type": "object",
                "properties": {
                    "description": {"type": "string", "description": "Natural language description of the email you want to create"},
                },
                "required": ["description"],
            },
        ),
        Tool(
            name="list_blocks",
            description="List all available email building blocks with their schemas.",
            inputSchema={"type": "object", "properties": {}},
        ),
        Tool(
            name="generate_email",
            description="Generate a complete email from a recipe and content brief. This is the main generation tool — it fills recipe blocks with AI-generated content, renders chart images, and produces final HTML.",
            inputSchema={
                "type": "object",
                "properties": {
                    "recipe_id": {"type": "string", "description": "Recipe to use"},
                    "content_brief": {"type": "string", "description": "What the email should be about — topic, key points, tone"},
                    "brand": {"type": "string", "enum": ["groww", "915", "amc", "w"], "default": "groww"},
                    "data": {"type": "object", "description": "Optional structured data (prices, metrics, etc.) to include"},
                },
                "required": ["recipe_id", "content_brief"],
            },
        ),
        Tool(
            name="generate_chart",
            description="Generate a chart image (price_card, listing_card, comparison, portfolio, or metric) from structured data. Returns the image path.",
            inputSchema={
                "type": "object",
                "properties": {
                    "chart_type": {"type": "string", "enum": ["price_card", "listing_card", "comparison", "portfolio", "metric"]},
                    "chart_data": {"type": "object", "description": "Data for the chart (schema depends on chart_type)"},
                },
                "required": ["chart_type", "chart_data"],
            },
        ),
        Tool(
            name="export_email",
            description="Export a generated email as final HTML file with all image URLs. Ready to paste into NARAD.",
            inputSchema={
                "type": "object",
                "properties": {
                    "email_id": {"type": "string", "description": "ID of the generated email to export"},
                },
                "required": ["email_id"],
            },
        ),
    ]


@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    
    if name == "list_recipes":
        recipes = recipe_registry.list(
            brand=arguments.get("brand"),
            email_type=arguments.get("email_type"),
        )
        return [TextContent(type="text", text=json.dumps(recipes, indent=2))]

    elif name == "get_recipe":
        recipe = recipe_registry.get(arguments["recipe_id"])
        return [TextContent(type="text", text=json.dumps(recipe, indent=2))]

    elif name == "recommend_recipe":
        recommendations = recipe_registry.recommend(arguments["description"])
        return [TextContent(type="text", text=json.dumps(recommendations, indent=2))]

    elif name == "list_blocks":
        blocks = {name: schema.model_json_schema() for name, schema in BLOCK_SCHEMA_MAP.items()}
        return [TextContent(type="text", text=json.dumps(blocks, indent=2))]

    elif name == "generate_email":
        # Step 1: Get recipe
        recipe = recipe_registry.get(arguments["recipe_id"])
        
        # Step 2: Generate content via LLM
        content = await copy_writer.generate(
            recipe=recipe,
            brief=arguments["content_brief"],
            brand=arguments.get("brand", "groww"),
            data=arguments.get("data"),
        )
        
        # Step 3: Generate any chart images
        for i, block_def in enumerate(recipe["blocks"]):
            if block_def["block"] == "chart_image":
                chart_data = content["blocks"].get(str(i), {}).get("chart_data")
                if chart_data:
                    chart_type = block_def["data"].get("chart_type", "price_card")
                    img_path = OUTPUT_DIR / "images" / f"chart_{i}_{arguments['recipe_id']}.png"
                    await chart_renderer.render(chart_type, chart_data, img_path)
                    content["blocks"][str(i)]["image_url"] = str(img_path)
                    # TODO: Upload to Groww cloud and replace with hosted URL

        # Step 4: Render HTML
        html = email_renderer.render(recipe, content, brand_name=arguments.get("brand", "groww"))
        
        # Step 5: Save
        email_id = f"{arguments['recipe_id']}_{hash(arguments['content_brief']) % 10000}"
        html_path = OUTPUT_DIR / "html" / f"{email_id}.html"
        html_path.parent.mkdir(parents=True, exist_ok=True)
        html_path.write_text(html)

        return [TextContent(type="text", text=json.dumps({
            "email_id": email_id,
            "html_path": str(html_path),
            "block_count": len(recipe["blocks"]),
            "status": "generated",
            "preview": html[:500] + "...",
        }, indent=2))]

    elif name == "generate_chart":
        chart_type = arguments["chart_type"]
        chart_data = arguments["chart_data"]
        img_path = OUTPUT_DIR / "images" / f"chart_{chart_type}_{hash(json.dumps(chart_data)) % 10000}.png"
        img_path.parent.mkdir(parents=True, exist_ok=True)
        await chart_renderer.render(chart_type, chart_data, img_path)
        return [TextContent(type="text", text=json.dumps({
            "chart_type": chart_type,
            "image_path": str(img_path),
            "status": "rendered",
        }, indent=2))]

    elif name == "export_email":
        email_id = arguments["email_id"]
        html_path = OUTPUT_DIR / "html" / f"{email_id}.html"
        if not html_path.exists():
            return [TextContent(type="text", text=f"Email {email_id} not found")]
        return [TextContent(type="text", text=json.dumps({
            "email_id": email_id,
            "html_path": str(html_path),
            "html_size": html_path.stat().st_size,
            "ready_for_narad": True,
        }, indent=2))]

    return [TextContent(type="text", text=f"Unknown tool: {name}")]


def run_server():
    """Entry point for the MCP server."""
    asyncio.run(stdio_server(server))
```

## 2.7 Content Generator (LLM Integration)

**File: `src/generator/copy_writer.py`**

```python
"""
Uses Anthropic API to generate email copy based on recipe + brief.
"""

import json
from anthropic import AsyncAnthropic
from src.generator.prompts import build_generation_prompt


class CopyWriter:
    def __init__(self):
        self.client = AsyncAnthropic()   # Uses ANTHROPIC_API_KEY env var

    async def generate(
        self,
        recipe: dict,
        brief: str,
        brand: str = "groww",
        data: dict | None = None,
    ) -> dict:
        """
        Generate content for every block in a recipe.
        
        Returns a content dict with filled block data.
        """
        prompt = build_generation_prompt(recipe, brief, brand, data)

        response = await self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            messages=[{"role": "user", "content": prompt}],
        )

        # Parse the JSON response
        text = response.content[0].text
        # Strip markdown fences if present
        text = text.strip()
        if text.startswith("```"):
            text = text.split("\n", 1)[1]
            text = text.rsplit("```", 1)[0]

        return json.loads(text)
```

**File: `src/generator/prompts.py`**

```python
"""Prompt templates for email content generation."""

import json


def build_generation_prompt(
    recipe: dict,
    brief: str,
    brand: str,
    data: dict | None,
) -> str:
    hints = recipe.get("generation_hints", {})
    blocks = recipe.get("blocks", [])
    
    block_specs = []
    for i, b in enumerate(blocks):
        block_type = b["block"]
        existing_data = b.get("data", {})
        block_specs.append(f"  Block {i} ({block_type}): {json.dumps(existing_data)}")

    data_context = ""
    if data:
        data_context = f"\n\nStructured data provided by the user:\n{json.dumps(data, indent=2)}"

    return f"""You are Groww's email copywriter. Generate content for an email.

BRAND: {brand}
BRIEF: {brief}
{data_context}

RECIPE: {recipe.get('name', 'Unknown')}
DESCRIPTION: {recipe.get('description', '')}

TONE: {hints.get('tone', 'professional, clear, confident')}
READING LEVEL: {hints.get('reading_level', 'accessible to retail investors')}
WORD COUNT TARGET: {hints.get('word_count_target', 300)}
AVOID: {', '.join(hints.get('avoid', ['jargon', 'financial advice']))}

BLOCKS TO FILL:
{chr(10).join(block_specs)}

RULES:
- Do NOT give financial advice or recommend specific investments
- Include SEBI-compliant language where needed
- Keep language accessible to Indian retail investors
- Use ₹ symbol for Indian Rupee amounts
- All factual claims must be verifiable
- CTA text should be action-oriented and under 25 characters

RESPONSE FORMAT:
Return ONLY valid JSON (no markdown, no explanation) with this structure:
{{
  "subject": "Email subject line",
  "preheader": "Preview text (50-90 chars)",
  "blocks": {{
    "0": {{}},
    "1": {{ "headline": "...", "subtitle": "..." }},
    "2": {{ "chart_data": {{ ... }} }},
    ...
  }}
}}

For each block index, provide the data fields that block type requires.
Skip blocks that don't need generated content (header_logo, footer_standard, divider).
For chart_image blocks, provide chart_data with the data to render.
For article_section blocks, provide heading and paragraphs array.
For factor_card blocks, provide title and description.
For cta_button blocks, provide text and url.
"""
```

---

# 3. Task 2: Compass MCP Skills — Implementation

## 3.1 Skill Template

**File: `compass-skills/templates/skill_template.md`**

```markdown
# {{DOMAIN_NAME}} Skill

> **Domain**: {{DOMAIN_DESCRIPTION}}
> **Catalogs**: {{CATALOG_PATHS}}
> **Last validated**: {{DATE}}

---

## Tables Overview

| # | Table | Schema | Grain | Freshness | Purpose |
|---|-------|--------|-------|-----------|---------|
{{TABLE_ROWS}}

---

## Join Key Reference

| Identifier | Format | Found In | Notes |
|---|---|---|---|
{{JOIN_KEY_ROWS}}

**Critical rule**: {{CRITICAL_JOIN_RULE}}

---

## Key Columns by Table

{{KEY_COLUMNS_SECTIONS}}

---

## Common Filter Values

{{FILTER_VALUES_SECTIONS}}

---

## Critical Query Best Practices

{{BEST_PRACTICES}}

---

## Example Queries

{{EXAMPLE_QUERIES}}

---

## Query Strategy Reference

| Question | Best Table(s) | Key Filter / Note |
|---|---|---|
{{STRATEGY_ROWS}}

---

## Data Quality Notes

{{DATA_QUALITY_NOTES}}
```

## 3.2 Schema Discovery Tool

**File: `compass-skills/tools/schema_discovery.py`**

```python
"""
Auto-discover table schemas from Compass MCP / database.
Generates the Tables Overview and Key Columns sections automatically.

Usage:
  python cli.py discover-schema platform_iceberg.commodities
"""

import json
from pathlib import Path
from dataclasses import dataclass, field


@dataclass
class ColumnInfo:
    name: str
    data_type: str
    description: str = ""       # Filled manually or from metadata
    is_partition: bool = False
    is_pii: bool = False


@dataclass
class TableInfo:
    name: str
    schema: str
    columns: list[ColumnInfo] = field(default_factory=list)
    row_count: int = 0
    grain: str = ""              # e.g., "1 row per user"
    freshness: str = ""          # e.g., "daily batch"
    purpose: str = ""            # One-line description


def discover(catalog: str, output_file: Path | None = None):
    """
    Discover all tables in a catalog and output a skill scaffold.
    
    This function needs to connect to Compass MCP or the database
    to query table metadata. Below is the template for how it works —
    replace the placeholder queries with actual Compass queries.
    """

    # --- PLACEHOLDER: Replace with actual Compass MCP queries ---
    # 
    # Option A: If you have direct SQL access (Trino/Presto):
    #   SHOW TABLES IN {catalog}
    #   DESCRIBE {catalog}.{table}
    #   SELECT COUNT(*) FROM {catalog}.{table} [with appropriate filters]
    #
    # Option B: If using Compass MCP via Claude Code:
    #   Ask Compass: "List all tables in {catalog} with their column schemas"
    #
    # Option C: If using Datahub API:
    #   Query the Datahub metadata API for dataset schemas
    # ---

    print(f"Discovering schema for catalog: {catalog}")
    print()
    print("=" * 60)
    print("MANUAL STEPS (replace with automation once you have access):")
    print("=" * 60)
    print()
    print(f"1. Run in Compass/Trino:")
    print(f"   SHOW TABLES IN {catalog}")
    print()
    print(f"2. For each table, run:")
    print(f"   DESCRIBE {catalog}.<table_name>")
    print()
    print(f"3. For row counts:")
    print(f"   SELECT COUNT(*) FROM {catalog}.<table_name>")
    print(f"   (Add partition filters for large tables!)")
    print()
    print(f"4. Check Datahub for existing documentation:")
    print(f"   https://prod-dp-ai-datahub.growwinfra.in/")
    print()

    # Generate scaffold markdown
    scaffold = f"""# {catalog.split('.')[-1].replace('_', ' ').title()} Skill

> **Domain**: [FILL IN]
> **Catalogs**: `{catalog}`
> **Last validated**: [TODAY'S DATE]

---

## Tables Overview

| # | Table | Schema | Grain | Freshness | Purpose |
|---|-------|--------|-------|-----------|---------|
| 1 | `[TABLE_NAME]` | `{catalog}` | [FILL] | [FILL] | [FILL] |

<!-- Run SHOW TABLES IN {catalog} and fill this in -->

---

## Join Key Reference

| Identifier | Format | Found In | Notes |
|---|---|---|---|
| `user_account_id` | `ACC<digits>` | [TABLES] | Primary join key — use for cross-table joins |

**Critical rule**: [FILL IN the primary join strategy]

---

## Key Columns by Table

### 1. `{catalog}.[TABLE_NAME]`

<!-- Run DESCRIBE {catalog}.[TABLE_NAME] and document each column -->

| Column | Type | Description |
|---|---|---|
| `column_name` | varchar | [FILL] |

---

## Common Filter Values

<!-- Run SELECT DISTINCT queries to enumerate values -->

---

## Critical Query Best Practices

1. **[MANDATORY FILTER]** — Always filter by [partition column] to avoid full table scans.

---

## Example Queries

### 1. [QUERY TITLE]
```sql
-- [Description of what this query answers]
SELECT ...
FROM {catalog}.[TABLE]
WHERE ...
```

---

## Query Strategy Reference

| Question | Best Table(s) | Key Filter / Note |
|---|---|---|
| "[BUSINESS QUESTION]?" | `[TABLE]` | [FILTER] |

---

## Data Quality Notes

1. **[NOTE]** — [Description]
"""

    if output_file:
        output_file.parent.mkdir(parents=True, exist_ok=True)
        output_file.write_text(scaffold)
        print(f"Scaffold written to: {output_file}")
    else:
        print(scaffold)
```

## 3.3 Query Testing Framework

**File: `compass-skills/tools/query_tester.py`**

```python
"""
Test SQL queries from skill documents against real data.

For each skill, create a directory of .sql test files.
Each file contains a query and expected behavior annotations.

File format:
  -- TITLE: Query title
  -- EXPECTS: rows > 0
  -- EXPECTS: columns contain user_account_id
  -- MAX_RUNTIME: 30s
  SELECT ...

Usage:
  python cli.py test-queries compass-skills/tests/test_queries/engagement_comms/
"""

import re
from pathlib import Path
from dataclasses import dataclass


@dataclass
class QueryTest:
    title: str
    sql: str
    expectations: list[str]
    max_runtime_seconds: int = 60
    file_path: str = ""


@dataclass
class TestResult:
    test: QueryTest
    passed: bool
    message: str
    runtime_seconds: float = 0


def parse_test_file(file_path: Path) -> QueryTest:
    """Parse a .sql test file into a QueryTest."""
    content = file_path.read_text()
    lines = content.split("\n")
    
    title = ""
    expectations = []
    max_runtime = 60
    sql_lines = []

    for line in lines:
        if line.strip().startswith("-- TITLE:"):
            title = line.split("-- TITLE:")[1].strip()
        elif line.strip().startswith("-- EXPECTS:"):
            expectations.append(line.split("-- EXPECTS:")[1].strip())
        elif line.strip().startswith("-- MAX_RUNTIME:"):
            val = line.split("-- MAX_RUNTIME:")[1].strip()
            max_runtime = int(val.replace("s", ""))
        else:
            sql_lines.append(line)

    return QueryTest(
        title=title or file_path.stem,
        sql="\n".join(sql_lines).strip(),
        expectations=expectations,
        max_runtime_seconds=max_runtime,
        file_path=str(file_path),
    )


def test_all(test_dir: Path):
    """
    Run all SQL test files in a directory.
    
    NOTE: This requires Compass MCP or direct database access.
    Replace the placeholder execution with actual query execution.
    """
    sql_files = sorted(test_dir.glob("*.sql"))
    if not sql_files:
        print(f"No .sql files found in {test_dir}")
        return

    print(f"Found {len(sql_files)} test queries in {test_dir}")
    print()

    results = []
    for sql_file in sql_files:
        test = parse_test_file(sql_file)
        print(f"  Testing: {test.title}")
        print(f"    SQL: {test.sql[:100]}...")

        # --- PLACEHOLDER: Replace with actual query execution ---
        # 
        # Option A: Direct SQL via Trino/Presto client
        #   import trino
        #   conn = trino.dbapi.connect(host='...', port=443, ...)
        #   cursor = conn.cursor()
        #   cursor.execute(test.sql)
        #   rows = cursor.fetchall()
        #
        # Option B: Via Compass MCP
        #   Use the MCP client to send the query to Compass
        #
        # For now, we just validate SQL syntax
        # ---

        # Basic SQL validation (syntax only)
        sql_upper = test.sql.upper()
        issues = []
        
        # Check for partition filter on known large tables
        if "EMS_EVENTS" in sql_upper and "PT >=" not in sql_upper and "PT =" not in sql_upper:
            issues.append("MISSING: ems_events requires pt partition filter")
        
        if "GROWTH_EVENTS" in sql_upper:
            if "EVENT_NAME" not in sql_upper and "EVENT_TS" not in sql_upper:
                issues.append("MISSING: growth_events requires event_name or event_ts filter")
        
        if "GROWTH_USER_MASTER" in sql_upper:
            if "XYBY.NET" not in sql_upper:
                issues.append("WARNING: growth_user_master should exclude @xyby.net test accounts")
            if "CREDIT_ONLY_FLAG" not in sql_upper:
                issues.append("WARNING: growth_user_master should filter credit_only_flag = 0")

        # Check for common mistakes
        if "SELECT *" in sql_upper:
            issues.append("WARNING: SELECT * on large tables — specify columns")
        
        if not issues:
            result = TestResult(test=test, passed=True, message="Syntax OK (not executed)")
            print(f"    ✓ Passed (syntax check only)")
        else:
            result = TestResult(test=test, passed=False, message="; ".join(issues))
            for issue in issues:
                print(f"    ✗ {issue}")

        results.append(result)
        print()

    # Summary
    passed = sum(1 for r in results if r.passed)
    total = len(results)
    print(f"Results: {passed}/{total} passed")
    if passed < total:
        print("\nFailed queries need attention before publishing the skill.")
```

## 3.4 Skill Validator

**File: `compass-skills/tools/validator.py`**

```python
"""
Validates a skill document for completeness before publishing.
Checks against the quality checklist from the project plan.
"""

import re
from pathlib import Path
from dataclasses import dataclass


@dataclass
class ValidationResult:
    passed: bool
    check: str
    message: str


def validate_skill(skill_path: Path) -> list[ValidationResult]:
    """Validate a skill document for completeness."""
    content = skill_path.read_text()
    results = []

    # Check 1: Has Tables Overview section with actual table entries
    has_tables_overview = "## Tables Overview" in content
    table_rows = len(re.findall(r"\| \d+ \|", content))
    results.append(ValidationResult(
        passed=has_tables_overview and table_rows >= 1,
        check="Tables Overview",
        message=f"Found {table_rows} table entries" if has_tables_overview else "Section missing",
    ))

    # Check 2: Has Join Key Reference
    has_join_keys = "## Join Key Reference" in content
    results.append(ValidationResult(
        passed=has_join_keys,
        check="Join Key Reference",
        message="Present" if has_join_keys else "Section missing",
    ))

    # Check 3: Has Key Columns section
    has_key_columns = "## Key Columns" in content
    column_tables = len(re.findall(r"\| Column \| Type \|", content))
    results.append(ValidationResult(
        passed=has_key_columns and column_tables >= 1,
        check="Key Columns by Table",
        message=f"Found {column_tables} column tables" if has_key_columns else "Section missing",
    ))

    # Check 4: Has Critical Query Best Practices
    has_best_practices = "## Critical Query Best Practices" in content
    practice_count = len(re.findall(r"^\d+\.", content, re.MULTILINE))
    results.append(ValidationResult(
        passed=has_best_practices and practice_count >= 3,
        check="Query Best Practices",
        message=f"Found {practice_count} practices" if has_best_practices else "Section missing",
    ))

    # Check 5: Has Example Queries (at least 10)
    has_examples = "## Example Queries" in content
    sql_blocks = len(re.findall(r"```sql", content))
    results.append(ValidationResult(
        passed=has_examples and sql_blocks >= 10,
        check="Example Queries (≥10)",
        message=f"Found {sql_blocks} SQL blocks" if has_examples else "Section missing",
    ))

    # Check 6: Has Query Strategy Reference
    has_strategy = "## Query Strategy Reference" in content
    strategy_rows = content.count("| \"") + content.count("| '")
    results.append(ValidationResult(
        passed=has_strategy and strategy_rows >= 10,
        check="Query Strategy Reference (≥15 entries)",
        message=f"Found ~{strategy_rows} entries" if has_strategy else "Section missing",
    ))

    # Check 7: Has Data Quality Notes
    has_quality = "## Data Quality Notes" in content
    results.append(ValidationResult(
        passed=has_quality,
        check="Data Quality Notes",
        message="Present" if has_quality else "Section missing",
    ))

    # Check 8: Partition filter warnings
    has_partition_warnings = "always filter" in content.lower() or "ALWAYS" in content
    results.append(ValidationResult(
        passed=has_partition_warnings,
        check="Partition filter warnings",
        message="Found ALWAYS FILTER warnings" if has_partition_warnings else "No partition filter warnings found",
    ))

    # Check 9: No TODO/FILL placeholders remaining
    todos = len(re.findall(r"TODO|FILL IN|\[FILL\]", content, re.IGNORECASE))
    results.append(ValidationResult(
        passed=todos == 0,
        check="No remaining TODOs",
        message=f"Found {todos} TODO/FILL placeholders" if todos > 0 else "Clean",
    ))

    # Check 10: Has user_account_id in join keys
    has_primary_key = "user_account_id" in content
    results.append(ValidationResult(
        passed=has_primary_key,
        check="Primary join key documented",
        message="user_account_id found" if has_primary_key else "Missing user_account_id reference",
    ))

    # Print results
    print(f"\nValidation: {skill_path.name}")
    print("=" * 50)
    for r in results:
        icon = "✓" if r.passed else "✗"
        print(f"  {icon} {r.check}: {r.message}")
    
    passed = sum(1 for r in results if r.passed)
    total = len(results)
    print(f"\nScore: {passed}/{total}")
    if passed == total:
        print("✓ Ready to publish!")
    else:
        print("✗ Fix the issues above before publishing.")

    return results
```

## 3.5 Skill-by-Skill Implementation Guide

Below is the concrete implementation sequence for each skill. For every skill, you follow the same 4-phase workflow — but the specifics (tables, join keys, gotchas) differ per domain.

### Skill 1: Engagement & Communications Analytics

**Phase 1 — Discovery**

```bash
# Step 1: Generate scaffold
python cli.py scaffold-skill engagement_comms

# Step 2: Identify tables
# Run in Compass/Trino:
SHOW TABLES IN platform_iceberg.ems;
SHOW TABLES IN platform_iceberg.engagement;   -- or whatever the comms schema is
```

Tables you're likely looking for:
- `webengage_event_raw_v2` — already known from EMS (campaign & notification events)
- Campaign metadata tables (campaign ID → name, type, channel)
- Email delivery/open/click event tables
- Push notification delivery tables
- SMS delivery tables
- In-app notification tables
- Template/content tables (which email HTML was used for which campaign)

Key questions this skill must answer:
1. "What's the open rate for commodity emails this month?"
2. "Which campaign had the highest CTR last week?"
3. "How many emails were sent to segment X?"
4. "Compare email vs push notification conversion rates"
5. "Which email template performs best for onboarding?"
6. "Delivery failure rate by channel (email/push/SMS)"
7. "Campaign performance trend over last 90 days"
8. "A/B test results for email subject lines"
9. "Unsubscribe rate by campaign type"
10. "Time-of-day analysis for email opens"

**Phase 2 — Draft (follow the template exactly)**

Key things to document:
- How campaign events flow through WebEngage into `webengage_event_raw_v2`
- Event types: `sent`, `delivered`, `opened`, `clicked`, `bounced`, `unsubscribed`
- How to calculate CTR: clicks / delivered (not sent)
- How to join campaign events to `growth_user_master` for user attributes
- Partition columns and mandatory filters
- How to identify email vs push vs SMS events

**Phase 3 — Write 10+ example queries covering:**
1. Campaign summary (sent/delivered/opened/clicked) by date
2. CTR by campaign type
3. Top campaigns by conversion
4. Delivery failure analysis
5. Channel comparison (email vs push vs SMS)
6. User-level engagement history
7. Unsubscribe trend
8. Template performance comparison
9. Time-of-day optimization
10. Segment-level engagement metrics
11. A/B test comparison query
12. Reactivation campaign effectiveness

**Phase 4 — Validate**
```bash
# Run query tests
python cli.py test-queries compass-skills/tests/test_queries/engagement_comms/

# Validate skill document
python -c "
from compass_skills.tools.validator import validate_skill
from pathlib import Path
validate_skill(Path('compass-skills/generated/engagement_comms.md'))
"
```

### Skills 2–11: Same Pattern

For each subsequent skill, repeat the same 4-phase process. The key differentiators per skill:

**User Master & Growth Analytics (Skill 2):**
- Central table: `growth_user_master` (117M rows) + `growth_user_master_ultimate`
- Key metrics: TTU, NTU, FID, DAU, MAU
- Critical: document all derived flags (`affluent`, `power_trader`, etc.)
- Critical: document the standard exclusions (`@xyby.net`, `credit_only_flag`)

**User Events Analytics v2 (Skill 3):**
- Tables: `ems_events`, `growth_events`, `ems_raw`
- Critical: `ems_events` partition on `pt` — ALWAYS filter
- Critical: `growth_events` dual partition on `event_name` AND `event_ts`
- Document LUID identity resolution flow
- Document `_dp_event_data` JSON parsing for top events

**Stocks & Trading (Skill 4):**
- Order lifecycle: placed → confirmed → executed → settled
- Order types: CNC, MIS, AMO, GTT
- Key metrics: traded value, order count, unique traders
- Gotcha: settle date != trade date (T+1 settlement)

**Mutual Fund (Skill 5):**
- SIP lifecycle: created → active → paused → cancelled
- Fund categorization: equity/debt/hybrid/index
- AUM calculation logic
- Folio-level vs user-level aggregation

**F&O (Skill 6):**
- Contract structure: underlying, strike, expiry, option_type
- Position lifecycle: open → squared off → expired
- Margin calculation references
- Gotcha: expiry dates cause data pattern changes

**Commodities (Skill 7):**
- MCX contract structure: lot sizes, mini vs standard
- Commodity types: crude, gold, silver, natural gas, etc.
- Price data sources
- Reference the crude oil email — what data fields are needed?

**915, Credit, MTF, Onboarding (Skills 8–11):** Follow the same pattern with domain-specific tables and metrics.

---

# 4. Integration Points

## 4.1 Content Engine → Compass MCP (Future)

When Compass MCP integration happens, add a new tool to the Content Engine MCP:

```python
# In src/mcp/tools.py — future addition

Tool(
    name="fetch_live_data",
    description="Fetch live data from Compass MCP for email content (e.g., current prices, user counts).",
    inputSchema={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Natural language data query"},
        },
        "required": ["query"],
    },
)
```

This tool would call Compass MCP's text-to-SQL interface to get live data, then pass it to the chart renderer and copy writer.

## 4.2 Claude Code Configuration

**File: `claude_code_config.json`** (or `~/.claude/mcp.json`)

```json
{
  "mcpServers": {
    "content-engine": {
      "command": "python",
      "args": ["-m", "src.mcp.server"],
      "cwd": "/path/to/groww-content-engine",
      "env": {
        "ANTHROPIC_API_KEY": "sk-ant-..."
      }
    }
  }
}
```

---

# 5. Testing Strategy

## 5.1 Content Engine Tests

```
tests/
├── analyzer/
│   ├── test_parser.py             # Test HTML parsing on sample emails
│   ├── test_block_detector.py     # Test block detection accuracy
│   ├── test_classifier.py         # Test email classification
│   └── fixtures/                  # Sample HTML files for testing
│       ├── commodity_email.html
│       ├── onboarding_email.html
│       └── promotional_email.html
├── renderer/
│   ├── test_engine.py             # Test recipe → HTML rendering
│   └── test_blocks.py             # Test individual block templates
├── charts/
│   └── test_renderer.py           # Test chart image generation
└── mcp/
    └── test_tools.py              # Test MCP tool handlers
```

**Example test: `tests/analyzer/test_parser.py`**

```python
import pytest
from pathlib import Path
from src.analyzer.parser import EmailParser, NodeType


@pytest.fixture
def parser():
    return EmailParser()


@pytest.fixture
def sample_html():
    return """
    <html><body>
    <table width="600" align="center">
      <tr><td><img src="https://cdn.groww.in/logo.png" alt="Groww" width="120"></td></tr>
      <tr><td><h1 style="font-size: 24px;">What moved <span style="color:#00D09C">Crude Oil</span> prices?</h1></td></tr>
      <tr><td><p>Commodities prices don't move without reason.</p></td></tr>
      <tr><td><a href="https://groww.in" style="background-color:#00D09C; border-radius:8px; padding:14px 48px; color:#fff;">EXPLORE NOW</a></td></tr>
    </table>
    </body></html>
    """


def test_parse_finds_nodes(parser, sample_html):
    nodes = parser.parse(sample_html)
    assert len(nodes) >= 3


def test_parse_finds_image(parser, sample_html):
    nodes = parser.parse(sample_html)
    images = [n for n in nodes if n.node_type == NodeType.IMAGE]
    assert len(images) >= 1
    assert "logo" in images[0].src.lower()


def test_parse_finds_heading(parser, sample_html):
    nodes = parser.parse(sample_html)
    headings = [n for n in nodes if n.node_type == NodeType.HEADING]
    assert len(headings) >= 1
    assert "Crude Oil" in headings[0].content


def test_parse_finds_button(parser, sample_html):
    nodes = parser.parse(sample_html)
    buttons = [n for n in nodes if n.node_type == NodeType.BUTTON]
    assert len(buttons) >= 1
    assert "EXPLORE" in buttons[0].content
```

## 5.2 Compass Skills Tests

Each skill gets a directory of `.sql` test files:

**File: `compass-skills/tests/test_queries/engagement_comms/01_campaign_summary.sql`**

```sql
-- TITLE: Campaign summary by date
-- EXPECTS: rows > 0
-- EXPECTS: columns contain campaign_id, sent, delivered, opened
-- MAX_RUNTIME: 30s

SELECT
    DATE(event_ts) AS event_date,
    dp_campaign_id AS campaign_id,
    COUNT(CASE WHEN event_name = 'campaign_sent' THEN 1 END) AS sent,
    COUNT(CASE WHEN event_name = 'campaign_delivered' THEN 1 END) AS delivered,
    COUNT(CASE WHEN event_name = 'campaign_opened' THEN 1 END) AS opened,
    COUNT(CASE WHEN event_name = 'campaign_clicked' THEN 1 END) AS clicked
FROM platform_iceberg.ems.webengage_event_raw_v2
WHERE pt >= TIMESTAMP '2026-03-01 00:00:00 UTC'
  AND pt < TIMESTAMP '2026-04-01 00:00:00 UTC'
  AND event_name IN ('campaign_sent', 'campaign_delivered', 'campaign_opened', 'campaign_clicked')
GROUP BY 1, 2
ORDER BY event_date DESC
LIMIT 100
```

---

# 6. Deployment & Configuration

## 6.1 Running the Content Engine

```bash
# 1. Analyze emails
python cli.py analyze data/raw_emails/ --output-dir data/analyzed/

# 2. Generate a chart (standalone test)
python cli.py generate-chart price_card data/sample_chart_data.json --output-path output/images/test_chart.png

# 3. Render an email (standalone test)
python cli.py render-email data/recipes/commodity_educational.json data/sample_content.json

# 4. Start MCP server (for Claude Code)
python cli.py serve-mcp
```

## 6.2 Compass Skills Workflow

```bash
# 1. Generate scaffold for a new skill
python cli.py scaffold-skill commodities_trading -t commodity_orders -t commodity_contracts

# 2. (Manual) Fill in the scaffold with table details, columns, queries

# 3. Test queries
python cli.py test-queries compass-skills/tests/test_queries/commodities_trading/

# 4. Validate completeness
python -c "
from compass_skills.tools.validator import validate_skill
from pathlib import Path
validate_skill(Path('compass-skills/generated/commodities_trading.md'))
"

# 5. (Manual) Publish to Datahub
```

## 6.3 Environment Variables

```bash
# .env file
ANTHROPIC_API_KEY=sk-ant-...              # For content generation
GROWW_CLOUD_BUCKET=...                     # For image uploads (when available)
COMPASS_MCP_URL=...                        # Compass MCP endpoint (when integrated)
TRINO_HOST=...                             # Direct DB access (if available)
TRINO_PORT=443
```
