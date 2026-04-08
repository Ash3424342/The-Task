# Groww — Implementation Plan v2 (Technical)

**Stack:** Python 3.11+  
**Date:** April 6, 2026  
**Version:** 2.0 — Updated with Compass MCP documentation, Skills Deep Dive, and Setup Guide

---

# TABLE OF CONTENTS

1. Development Environment Setup
2. Task 1: Content Engine — Implementation
   - 2.1 Project Structure
   - 2.2 Component 1: Email Analyzer Pipeline
   - 2.3 Component 2: Block Library (Pydantic Schemas)
   - 2.4 Component 3: Recipe System
   - 2.5 Component 4: Chart Image Renderer
   - 2.6 Component 5: HTML Email Renderer
   - 2.7 Component 6: MCP Server
   - 2.8 Content Generator (LLM Integration)
   - 2.9 CLI Entry Point
3. Task 2: Compass MCP Skills — Implementation
   - 3.1 Compass Setup & Connection
   - 3.2 Understanding Compass Tools (80+ Available)
   - 3.3 Skill Enhancement Workflow (Using Compass Tools)
   - 3.4 Building Minimal Skills from Scratch
   - 3.5 Skill Authoring Toolkit
   - 3.6 Skill-by-Skill Implementation Guide
4. Testing Strategy
5. Deployment & Configuration

---

# 1. Development Environment Setup

## 1.1 System Requirements

```bash
# Python 3.11+
python3 --version

# Node.js 18+ (needed for Compass MCP proxy + Playwright)
node --version

# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## 1.2 Project Initialization

```bash
# Create project
mkdir groww-content-engine && cd groww-content-engine

# Content Engine directories
mkdir -p src/{analyzer,blocks,recipes,charts,renderer,mcp,generator,utils}
mkdir -p src/charts/templates
mkdir -p src/renderer/templates
mkdir -p data/{raw_emails,analyzed,blocks,recipes,brands}
mkdir -p output/{html,images}
mkdir -p tests/{analyzer,renderer,charts,mcp}

# Compass Skills directories
mkdir -p compass-skills/{tools,enhanced,new,tests}
mkdir -p compass-skills/tests/queries

# Initialize Python project
uv init
uv add beautifulsoup4 lxml Pillow jinja2 pydantic httpx
uv add mcp                          # MCP SDK for Python
uv add playwright                   # Chart screenshot rendering
uv add anthropic                    # LLM content generation
uv add rich typer                   # CLI tools
uv add pytest pytest-asyncio        # Testing

# Install Playwright browsers
uv run playwright install chromium
```

## 1.3 Compass MCP Setup

This is critical — you need Compass access to test skills, discover schemas, and run queries.

```bash
# Option 1: Auto setup (recommended)
# Clone the Compass repo and run:
./scripts/compass-mcp-setup.sh
# This will:
#   1. Check Node.js and VPN
#   2. Ask for Groww email/password
#   3. Install local proxy (auto-starts on login)
#   4. Configure your IDE

# Option 2: Manual setup
# Start proxy (keep terminal open or set up as LaunchAgent):
export MCP_SERVER_URL="https://prod-dp-ai-datahub-gms.growwinfra.in/mcp"
export MCP_AUTH_HEADER="Basic $(echo -n 'your.email@groww.in:yourpassword' | base64)"
export MCP_PROXY_PORT=3100
export NODE_TLS_REJECT_UNAUTHORIZED=0
node ~/.compass-mcp/proxy.mjs

# Add Compass to Claude Code:
claude mcp add compass --transport http http://127.0.0.1:3100/mcp

# Verify connection:
curl http://127.0.0.1:3100/mcp

# Diagnostics if issues:
./scripts/compass-mcp-setup.sh --check
tail -f ~/.compass-mcp/proxy.log
```

**Troubleshooting:**
- "fetch failed" / "Connect Timeout" → Proxy not running. Check `curl http://127.0.0.1:3100/mcp`
- TLS certificate errors → Setup script sets `NODE_TLS_REJECT_UNAUTHORIZED=0` (safe on internal VPN)
- HTTP 401 → Wrong email/password. Re-run setup.
- Port 3100 in use → `lsof -i :3100` and kill conflicting process

## 1.4 Full Project Tree

```
groww-content-engine/
│
├── pyproject.toml
├── cli.py                           # CLI entry point
├── README.md
│
├── src/
│   ├── analyzer/                    # Component 1: Email Analyzer
│   │   ├── parser.py                # HTML → semantic nodes
│   │   ├── block_detector.py        # Detects 18 block types
│   │   ├── classifier.py            # Email type classification
│   │   └── pipeline.py              # Orchestrates full analysis
│   │
│   ├── blocks/
│   │   └── schemas.py               # Pydantic models for all 18 blocks
│   │
│   ├── recipes/
│   │   ├── registry.py              # Recipe storage, lookup, recommendation
│   │   └── schemas.py               # Pydantic models for recipes
│   │
│   ├── charts/
│   │   ├── renderer.py              # Playwright-based chart → PNG
│   │   └── templates/               # HTML/CSS per chart type
│   │       ├── price_card.html
│   │       ├── listing_card.html
│   │       ├── comparison.html
│   │       ├── portfolio.html
│   │       └── metric.html
│   │
│   ├── renderer/
│   │   ├── engine.py                # Recipe JSON → full HTML email
│   │   └── templates/               # Jinja2 templates per block type
│   │       ├── base.html
│   │       ├── header_logo.html
│   │       ├── hero_text.html
│   │       └── ... (all 18)
│   │
│   ├── mcp/
│   │   ├── server.py                # MCP server entry point
│   │   └── tools.py                 # Tool definitions
│   │
│   ├── generator/
│   │   ├── copy_writer.py           # LLM copy generation
│   │   └── prompts.py               # Prompt templates
│   │
│   └── utils/
│       ├── config.py                # Brand configs, paths, constants
│       └── image_uploader.py        # Upload PNGs to Groww cloud
│
├── data/
│   ├── raw_emails/                  # 450 HTML files from WebEngage
│   ├── analyzed/                    # Analyzer output JSONs
│   ├── blocks/                      # Block schema JSONs
│   ├── recipes/                     # Recipe template JSONs
│   └── brands/
│       ├── groww.json
│       ├── 915.json
│       ├── amc.json
│       └── w.json
│
├── output/
│   ├── html/
│   └── images/
│
├── compass-skills/
│   ├── tools/
│   │   ├── discovery.py             # Schema discovery via Compass
│   │   ├── query_tester.py          # Test SQL via Compass run_trino_query
│   │   ├── skill_generator.py       # Generate skill scaffolds
│   │   └── validator.py             # Validate skill completeness
│   ├── enhanced/                    # Enhanced versions of existing skills
│   ├── new/                         # New skills built from scratch
│   └── tests/queries/               # SQL test files per skill
│
└── tests/
```

---

# 2. Task 1: Content Engine — Implementation

## 2.1 Configuration & Constants

**File: `src/utils/config.py`**

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
TEMPLATES_DIR = ROOT_DIR / "src" / "renderer" / "templates"
CHART_TEMPLATES_DIR = ROOT_DIR / "src" / "charts" / "templates"


class BrandConfig(BaseModel):
    name: str                       # "groww", "915", "amc", "w"
    display_name: str
    primary_color: str              # Groww: #00D09C
    secondary_color: str            # Groww: #5367FF
    accent_color: str
    text_color: str                 # Groww: #44475B
    background_color: str           # Groww: #F5F5F5
    card_background: str            # #FFFFFF
    font_family: str                # Inter, Arial, sans-serif
    logo_url: str
    footer_logo_url: str
    sebi_registration: str
    social_links: dict[str, str]
    app_store_url: str
    play_store_url: str
    unsubscribe_url_template: str


# Email type classification labels
EMAIL_TYPES = [
    "educational", "onboarding", "promotional", "commodity_update",
    "trading_alert", "feature_announcement", "weekly_digest",
    "transactional", "reactivation", "regulatory",
]

# All 18 block type identifiers
BLOCK_TYPES = [
    "header_logo", "hero_text", "hero_image", "chart_image",
    "article_section", "factor_card", "cta_button", "social_proof_banner",
    "footer_standard", "divider", "card_wrapper", "contract_row",
    "filter_pills", "listing_header", "step_instruction",
    "app_screenshot_pair", "product_category_grid", "social_showcase",
]
```

## 2.2 Component 1: Email Analyzer Pipeline

### HTML Parser

**File: `src/analyzer/parser.py`**

```python
"""
Parse raw HTML email into a flat list of semantic nodes.

Email HTML is deeply nested tables. Strategy:
1. Find the main content table (width=600)
2. Walk its rows top-to-bottom
3. Classify each row's content into SemanticNodes
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
    UNKNOWN = "unknown"


@dataclass
class SemanticNode:
    node_type: NodeType
    content: str = ""
    src: str = ""
    href: str = ""
    alt: str = ""
    styles: dict = field(default_factory=dict)
    raw_html: str = ""
    position: int = 0


class EmailParser:
    def parse(self, html: str) -> list[SemanticNode]:
        soup = BeautifulSoup(html, "lxml")
        content_table = self._find_content_table(soup)
        if not content_table:
            content_table = soup.body or soup

        nodes = []
        rows = content_table.find_all("tr", recursive=False)
        if not rows:
            rows = content_table.find_all("tr")

        for i, row in enumerate(rows):
            extracted = self._extract_node(row, position=i)
            if extracted:
                nodes.extend(extracted if isinstance(extracted, list) else [extracted])
        return nodes

    def _find_content_table(self, soup: BeautifulSoup) -> Tag | None:
        for table in soup.find_all("table"):
            width = table.get("width", "")
            style = table.get("style", "")
            if "600" in str(width) or "600px" in style:
                return table
        return None

    def _extract_node(self, row: Tag, position: int) -> SemanticNode | list[SemanticNode] | None:
        text = row.get_text(strip=True)
        if not text and not row.find("img"):
            if row.find("hr") or self._is_divider_style(row):
                return SemanticNode(node_type=NodeType.DIVIDER, position=position)
            return None

        nodes = []

        # Images (skip tracking pixels)
        for img in row.find_all("img"):
            src = img.get("src", "")
            width = img.get("width", "999")
            try:
                if int(width) < 10:
                    continue
            except (ValueError, TypeError):
                pass
            nodes.append(SemanticNode(
                node_type=NodeType.IMAGE, src=src, alt=img.get("alt", ""),
                styles=self._extract_styles(img), raw_html=str(img), position=position,
            ))

        # Headings (by tag or font-size >= 20px)
        headings = row.find_all(["h1", "h2", "h3"])
        if not headings:
            for el in row.find_all(["td", "p", "span", "div"]):
                if self._get_font_size(el.get("style", "")) >= 20:
                    headings.append(el)
        for h in headings:
            nodes.append(SemanticNode(
                node_type=NodeType.HEADING, content=h.get_text(strip=True),
                styles=self._extract_styles(h), position=position,
            ))

        # Buttons (styled <a> tags)
        for link in row.find_all("a"):
            style = link.get("style", "")
            if self._is_button_style(style) or link.find("table"):
                nodes.append(SemanticNode(
                    node_type=NodeType.BUTTON, content=link.get_text(strip=True),
                    href=link.get("href", ""), styles=self._extract_styles(link), position=position,
                ))

        # Paragraphs (remaining text)
        if not nodes and text:
            nodes.append(SemanticNode(
                node_type=NodeType.PARAGRAPH, content=text,
                styles=self._extract_styles(row), position=position,
            ))

        return nodes if nodes else None

    def _extract_styles(self, tag: Tag) -> dict:
        style = tag.get("style", "")
        result = {}
        for prop in style.split(";"):
            if ":" in prop:
                key, val = prop.split(":", 1)
                key = key.strip().lower()
                if key in ("font-size", "color", "background-color", "background",
                           "text-align", "font-weight", "border-radius", "padding", "width"):
                    result[key] = val.strip()
        return result

    def _get_font_size(self, style: str) -> int:
        for prop in style.split(";"):
            if "font-size" in prop:
                try:
                    return int(prop.split(":")[1].strip().replace("px", ""))
                except ValueError:
                    return 0
        return 0

    def _is_button_style(self, style: str) -> bool:
        s = style.lower()
        return ("background-color" in s or "background:" in s) and ("border-radius" in s or "padding" in s)

    def _is_divider_style(self, tag: Tag) -> bool:
        style = tag.get("style", "")
        return "border-top" in style or "border-bottom" in style
```

### Block Detector

**File: `src/analyzer/block_detector.py`**

```python
"""
Groups SemanticNodes into email blocks (18 types).
Uses sequential pattern matching — walks nodes top-to-bottom.
"""

from dataclasses import dataclass, field
from .parser import SemanticNode, NodeType


@dataclass
class DetectedBlock:
    block_type: str
    order: int = 0
    content: dict = field(default_factory=dict)
    confidence: float = 1.0
    source_nodes: list[SemanticNode] = field(default_factory=list)


class BlockDetector:
    def detect(self, nodes: list[SemanticNode]) -> list[DetectedBlock]:
        blocks = []
        i = 0
        order = 0
        detectors = [
            self._detect_header_logo, self._detect_footer_standard,
            self._detect_cta_button, self._detect_divider,
            self._detect_social_proof_banner, self._detect_chart_image,
            self._detect_hero_image, self._detect_factor_card,
            self._detect_contract_row, self._detect_step_instruction,
            self._detect_product_category_grid, self._detect_app_screenshot_pair,
            self._detect_hero_text, self._detect_article_section,
        ]

        while i < len(nodes):
            matched = False
            for detector in detectors:
                result = detector(nodes, i)
                if result:
                    block, consumed = result
                    block.order = order
                    blocks.append(block)
                    i += consumed
                    order += 1
                    matched = True
                    break
            if not matched:
                i += 1
        return blocks

    def _detect_header_logo(self, nodes, pos):
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE or pos > 3:
            return None
        src, alt = node.src.lower(), node.alt.lower()
        if any(kw in src or kw in alt for kw in ["logo", "groww", "915", "header"]):
            brand = "915" if "915" in src + alt else "groww"
            return DetectedBlock(block_type="header_logo",
                content={"brand": brand, "image_url": node.src}, source_nodes=[node]), 1
        return None

    def _detect_hero_text(self, nodes, pos):
        node = nodes[pos]
        if node.node_type != NodeType.HEADING or pos > 8:
            return None
        content = {"headline": node.content}
        consumed = 1
        if pos + 1 < len(nodes) and nodes[pos + 1].node_type == NodeType.PARAGRAPH:
            content["subtitle"] = nodes[pos + 1].content
            consumed = 2
        return DetectedBlock(block_type="hero_text", content=content, source_nodes=nodes[pos:pos+consumed]), consumed

    def _detect_chart_image(self, nodes, pos):
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE:
            return None
        src = node.src.lower()
        width = node.styles.get("width", "")
        if any(kw in src for kw in ["chart", "price", "listing", "screenshot", "graph"]) or "100%" in width:
            chart_type = "price_card" if "price" in src else "listing_card" if "listing" in src else "unknown"
            return DetectedBlock(block_type="chart_image",
                content={"chart_type": chart_type, "image_url": node.src}, source_nodes=[node]), 1
        return None

    def _detect_cta_button(self, nodes, pos):
        node = nodes[pos]
        if node.node_type != NodeType.BUTTON:
            return None
        return DetectedBlock(block_type="cta_button",
            content={"text": node.content, "url": node.href}, source_nodes=[node]), 1

    def _detect_divider(self, nodes, pos):
        if nodes[pos].node_type == NodeType.DIVIDER:
            return DetectedBlock(block_type="divider", source_nodes=[nodes[pos]]), 1
        return None

    def _detect_footer_standard(self, nodes, pos):
        text = (nodes[pos].content or "").lower()
        if any(kw in text for kw in ["sebi", "unsubscribe", "securities and exchange"]):
            remaining = len(nodes) - pos
            return DetectedBlock(block_type="footer_standard",
                content={"text": nodes[pos].content}, source_nodes=nodes[pos:]), remaining
        return None

    def _detect_social_proof_banner(self, nodes, pos):
        text = (nodes[pos].content or "").lower()
        if any(kw in text for kw in ["crore", "million", "users", "rating", "trusted"]):
            return DetectedBlock(block_type="social_proof_banner",
                content={"text": nodes[pos].content}, source_nodes=[nodes[pos]]), 1
        return None

    def _detect_factor_card(self, nodes, pos):
        bg = nodes[pos].styles.get("background-color", nodes[pos].styles.get("background", "")).lower()
        if any(c in bg for c in ["#e8f8f3", "#f0faf7", "rgba(0, 208, 156", "mint", "teal"]):
            return DetectedBlock(block_type="factor_card",
                content={"text": nodes[pos].content}, confidence=0.8, source_nodes=[nodes[pos]]), 1
        return None

    def _detect_contract_row(self, nodes, pos):
        text = (nodes[pos].content or "").lower()
        if any(kw in text for kw in ["mcx", "lot size", "expiry"]):
            return DetectedBlock(block_type="contract_row",
                content={"text": nodes[pos].content}, source_nodes=[nodes[pos]]), 1
        return None

    def _detect_step_instruction(self, nodes, pos):
        text = (nodes[pos].content or "").strip()
        if text and (text[:4].lower().startswith("step") or (text[0].isdigit() and len(text) > 1 and text[1] in ".)")):
            return DetectedBlock(block_type="step_instruction",
                content={"text": text}, source_nodes=[nodes[pos]]), 1
        return None

    def _detect_hero_image(self, nodes, pos):
        node = nodes[pos]
        if node.node_type != NodeType.IMAGE or pos > 5:
            return None
        src = node.src.lower()
        if not any(kw in src for kw in ["chart", "price", "listing", "logo"]):
            return DetectedBlock(block_type="hero_image",
                content={"image_url": node.src, "alt": node.alt}, source_nodes=[node]), 1
        return None

    def _detect_product_category_grid(self, nodes, pos):
        if pos + 3 >= len(nodes):
            return None
        next_four = nodes[pos:pos + 4]
        if all(n.node_type == NodeType.IMAGE for n in next_four):
            return DetectedBlock(block_type="product_category_grid",
                content={"categories": [{"image_url": n.src, "alt": n.alt} for n in next_four]},
                source_nodes=next_four), 4
        return None

    def _detect_app_screenshot_pair(self, nodes, pos):
        if pos + 1 >= len(nodes):
            return None
        n1, n2 = nodes[pos], nodes[pos + 1]
        if n1.node_type == NodeType.IMAGE and n2.node_type == NodeType.IMAGE:
            if any(kw in (n1.src + n2.src).lower() for kw in ["screenshot", "screen", "app"]):
                return DetectedBlock(block_type="app_screenshot_pair",
                    content={"left_image_url": n1.src, "right_image_url": n2.src},
                    source_nodes=[n1, n2]), 2
        return None

    def _detect_article_section(self, nodes, pos):
        node = nodes[pos]
        if node.node_type == NodeType.HEADING:
            content = {"heading": node.content, "paragraphs": []}
            consumed = 1
            while pos + consumed < len(nodes) and nodes[pos + consumed].node_type == NodeType.PARAGRAPH:
                content["paragraphs"].append(nodes[pos + consumed].content)
                consumed += 1
            return DetectedBlock(block_type="article_section", content=content,
                source_nodes=nodes[pos:pos+consumed]), consumed
        if node.node_type == NodeType.PARAGRAPH and len(node.content) > 30:
            return DetectedBlock(block_type="article_section",
                content={"heading": "", "paragraphs": [node.content]}, source_nodes=[node]), 1
        return None
```

### Email Classifier

**File: `src/analyzer/classifier.py`**

```python
"""
Rule-based email type classifier.
Uses subject keywords, block patterns, and content analysis.
"""

from dataclasses import dataclass, field
from .block_detector import DetectedBlock


@dataclass
class Classification:
    email_type: str
    confidence: float = 0.0
    signals: list[str] = field(default_factory=list)


class EmailClassifier:
    """Scores each email type and picks the highest."""

    RULES = {
        "commodity_update": {
            "subject_keywords": ["crude", "gold", "silver", "commodity", "mcx", "natural gas"],
            "block_signals": ["contract_row"],
            "content_keywords": ["price", "opec", "supply", "mcx"],
        },
        "educational": {
            "subject_keywords": ["what", "how", "why", "understand", "learn", "guide"],
            "block_signals": ["factor_card"],
            "min_article_sections": 3,
        },
        "onboarding": {
            "subject_keywords": ["welcome", "get started", "first step", "hello"],
            "block_signals": ["product_category_grid", "step_instruction"],
        },
        "promotional": {
            "subject_keywords": ["offer", "free", "exclusive", "limited", "deal", "reward"],
            "min_cta_buttons": 2,
        },
        "trading_alert": {
            "subject_keywords": ["alert", "watchlist", "target", "breakout", "52-week"],
        },
        "feature_announcement": {
            "subject_keywords": ["new", "introducing", "launch", "now available", "update"],
            "block_signals": ["app_screenshot_pair", "hero_image"],
        },
        "weekly_digest": {
            "subject_keywords": ["weekly", "digest", "roundup", "recap", "this week"],
        },
    }

    def classify(self, blocks: list[DetectedBlock], subject: str = "") -> Classification:
        subject_lower = subject.lower()
        block_types = [b.block_type for b in blocks]
        scores: dict[str, tuple[float, list[str]]] = {}

        for email_type, rules in self.RULES.items():
            score, signals = 0.0, []

            # Subject keywords
            for kw in rules.get("subject_keywords", []):
                if kw in subject_lower:
                    score += 0.3
                    signals.append(f"'{kw}' in subject")
                    break

            # Block signals
            for block_sig in rules.get("block_signals", []):
                if block_sig in block_types:
                    score += 0.25
                    signals.append(f"has {block_sig} block")

            # Min article sections
            min_articles = rules.get("min_article_sections", 0)
            if min_articles and block_types.count("article_section") >= min_articles:
                score += 0.2
                signals.append(f"{min_articles}+ article sections")

            # Min CTA buttons
            min_ctas = rules.get("min_cta_buttons", 0)
            if min_ctas and block_types.count("cta_button") >= min_ctas:
                score += 0.2
                signals.append(f"{min_ctas}+ CTA buttons")

            scores[email_type] = (score, signals)

        best_type = max(scores, key=lambda k: scores[k][0])
        best_score, best_signals = scores[best_type]

        if best_score < 0.25:
            return Classification(email_type="educational", confidence=0.25,
                signals=["low confidence — defaulting to educational"])

        return Classification(email_type=best_type, confidence=best_score, signals=best_signals)
```

### Analysis Pipeline

**File: `src/analyzer/pipeline.py`**

```python
"""
Orchestrates: raw HTML → parse → detect blocks → classify → output JSON
"""

import json
from pathlib import Path
from dataclasses import dataclass, asdict
from bs4 import BeautifulSoup
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
        html = html_path.read_text(encoding="utf-8", errors="replace")
        return self.analyze_html(html, source_file=html_path.name)

    def analyze_html(self, html: str, source_file: str = "") -> AnalyzedEmail:
        nodes = self.parser.parse(html)
        blocks = self.detector.detect(nodes)
        subject = self._extract_subject(html, blocks)
        classification = self.classifier.classify(blocks, subject=subject)

        all_text = " ".join(
            b.content.get("text", "") + " " + b.content.get("headline", "") + " " +
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
        return AnalyzedEmail(source_file=source_file, classification=classification,
            blocks=blocks, metadata=metadata)

    def analyze_directory(self, dir_path: Path, output_dir: Path) -> list[AnalyzedEmail]:
        output_dir.mkdir(parents=True, exist_ok=True)
        results = []
        html_files = sorted(dir_path.glob("*.html"))
        print(f"Found {len(html_files)} HTML files")

        for i, f in enumerate(html_files):
            try:
                result = self.analyze_file(f)
                results.append(result)
                (output_dir / f"{f.stem}.json").write_text(
                    json.dumps(asdict(result), indent=2, default=str))
                print(f"  [{i+1}/{len(html_files)}] {f.name} → {result.classification.email_type} "
                      f"({result.classification.confidence:.0%}) [{result.metadata['block_count']} blocks]")
            except Exception as e:
                print(f"  [{i+1}/{len(html_files)}] ERROR: {f.name} → {e}")

        # Summary
        summary = {"total": len(results), "by_type": {}}
        for r in results:
            t = r.classification.email_type
            summary["by_type"][t] = summary["by_type"].get(t, 0) + 1
        (output_dir / "_summary.json").write_text(json.dumps(summary, indent=2))
        print(f"\nSummary: {summary}")
        return results

    def _extract_subject(self, html: str, blocks: list[DetectedBlock]) -> str:
        soup = BeautifulSoup(html, "lxml")
        title = soup.find("title")
        if title and title.string:
            return title.string.strip()
        for b in blocks:
            if b.block_type == "hero_text" and b.content.get("headline"):
                return b.content["headline"]
        return ""
```

## 2.3 Component 2: Block Library

**File: `src/blocks/schemas.py`** — Same as v1 implementation plan (18 Pydantic models). No changes needed. See v1 for the full file with all 18 block schemas.

## 2.4 Component 4: Chart Image Renderer

**File: `src/charts/renderer.py`** — Same as v1 (Playwright-based HTML→PNG). No changes needed.

**File: `src/charts/templates/price_card.html`** — Same as v1 (matches crude oil email). No changes needed.

See v1 implementation plan for the complete code of both files.

## 2.5 Component 5: HTML Email Renderer

**File: `src/renderer/engine.py`** — Same as v1 (Jinja2-based recipe→HTML). No changes needed.

**Template files:** `base.html`, `hero_text.html`, `cta_button.html`, and all 18 block templates — same as v1. See v1 for complete code.

## 2.6 Component 6: MCP Server

**File: `src/mcp/server.py`** — Same structure as v1. One important addition: explicit documentation that this MCP server is **parallel to Compass**, not integrated with it.

```python
"""
Content Engine MCP Server.

IMPORTANT: This server is PARALLEL to Compass MCP — they are separate systems.
- Compass (port 3100): Analytics, querying, dashboards, reports
- Content Engine (stdio): Email generation, recipes, charts, HTML rendering

A Claude Code session can have BOTH connected simultaneously.
The CM uses Compass to understand data, then uses Content Engine to generate emails.
They don't call each other.
"""

# ... rest of server code identical to v1
# See v1 implementation plan for full MCP server code with all 7 tools
```

## 2.7 Content Generator

**File: `src/generator/copy_writer.py`** and **`src/generator/prompts.py`** — Same as v1. No changes needed.

## 2.8 CLI Entry Point

**File: `cli.py`**

```python
"""CLI entry point for Content Engine + Compass Skills tools."""

import typer
from pathlib import Path
from rich.console import Console

app = typer.Typer(name="groww", help="Groww Content Engine & Compass Skills CLI")
console = Console()


# --- Content Engine Commands ---

@app.command()
def analyze(
    input_dir: Path = typer.Argument(..., help="Directory of HTML email files"),
    output_dir: Path = typer.Option("data/analyzed", help="Output directory"),
):
    """Analyze HTML emails → decompose into blocks."""
    from src.analyzer.pipeline import AnalysisPipeline
    pipeline = AnalysisPipeline()
    results = pipeline.analyze_directory(input_dir, output_dir)
    console.print(f"\n[green]✓ Analyzed {len(results)} emails[/green]")


@app.command()
def generate_chart(
    chart_type: str = typer.Argument(..., help="price_card|listing_card|comparison|portfolio|metric"),
    data_file: Path = typer.Argument(..., help="JSON file with chart data"),
    output_path: Path = typer.Option("output/images/chart.png"),
):
    """Generate a chart image from data."""
    import json, asyncio
    from src.charts.renderer import ChartRenderer
    data = json.loads(data_file.read_text())
    asyncio.run(ChartRenderer().render(chart_type, data, output_path))
    console.print(f"[green]✓ Chart saved to {output_path}[/green]")


@app.command()
def render_email(
    recipe_file: Path = typer.Argument(..., help="Recipe JSON"),
    content_file: Path = typer.Argument(..., help="Content JSON"),
    output_path: Path = typer.Option("output/html/email.html"),
):
    """Render recipe + content → HTML email."""
    import json
    from src.renderer.engine import EmailRenderer
    recipe = json.loads(recipe_file.read_text())
    content = json.loads(content_file.read_text())
    html = EmailRenderer().render(recipe, content)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(html)
    console.print(f"[green]✓ Email saved to {output_path}[/green]")


@app.command()
def serve_mcp():
    """Start the Content Engine MCP server."""
    from src.mcp.server import run_server
    run_server()


# --- Compass Skills Commands ---

@app.command()
def validate_skill(
    skill_path: Path = typer.Argument(..., help="Path to skill .md file"),
):
    """Validate a skill document for completeness."""
    from compass_skills.tools.validator import validate
    validate(skill_path)


@app.command()
def test_queries(
    query_dir: Path = typer.Argument(..., help="Directory of .sql test files"),
):
    """Validate SQL queries against best practices."""
    from compass_skills.tools.query_tester import test_all
    test_all(query_dir)


if __name__ == "__main__":
    app()
```

---

# 3. Task 2: Compass MCP Skills — Implementation

## 3.1 Understanding Compass's Actual Tools

This is the key difference from v1: you don't need to build custom schema discovery scripts. Compass already has 80+ tools you can use directly from Claude Code. The relevant ones for skill work:

### Tools for Schema Discovery
| Tool | What It Does | Use For |
|---|---|---|
| `search` | Search datasets, dashboards, charts by keyword | Finding tables in a domain |
| `get_database_tables` | List all tables in a schema | Inventorying tables |
| `get_database_schemas` | List schemas in a database | Finding the right schema |
| `list_schema_fields` | Get all columns with types and descriptions | Column-level documentation |
| `get_entities` | Full metadata for a dataset (ownership, tags, descriptions) | Understanding table purpose |
| `get_dataset_queries` | See saved queries for a dataset | Finding existing SQL patterns |

### Tools for Query Testing
| Tool | What It Does | Use For |
|---|---|---|
| `run_trino_query` | Execute SQL on Trino (read-only, 5000 rows, 600s timeout) | Testing example queries |
| `execute_sql` (Superset) | Execute SQL via Superset connection (10000 rows, 300s) | Alternative query path |

### Tools for Metadata Enhancement
| Tool | What It Does | Use For |
|---|---|---|
| `update_description` | Update dataset/column descriptions | Enriching catalog |
| `add_tags` | Tag datasets | Organizing |
| `add_owners` | Set ownership | Attribution |
| `add_glossary_terms` | Link to business terms | Standardization |

### Tools for Visualization (Useful for Skill Validation)
| Tool | What It Does | Use For |
|---|---|---|
| `generate_explore_link` | Create interactive chart URL | Visualizing query results |
| `generate_chart` | Create and save a chart | Building validation dashboards |

## 3.2 Skill Enhancement Workflow (Using Compass Directly)

The key insight: **you don't write Python scripts to discover schemas — you use Compass's own tools from Claude Code.** The workflow is conversational.

### Phase 1: Discovery (via Claude Code + Compass)

```
You (in Claude Code with Compass connected):

> "Search for all tables related to commodities trading"
  → Compass uses `search` tool, returns dataset URNs and names

> "List all tables in platform_iceberg.core_invest schema"
  → Compass uses `get_database_tables`, returns full table list

> "Show me the columns of commodity_order_master"
  → Compass uses `list_schema_fields`, returns column names, types, descriptions

> "What's the row count and date range of commodity_day_view_seg?"
  → Compass uses `run_trino_query`:
     SELECT COUNT(*) as rows, MIN(order_date) as min_date, MAX(order_date) as max_date
     FROM platform_iceberg.core_invest.commodity_day_view_seg

> "Show me the distinct values of underlying in commodity_order_master (limit 20)"
  → Compass runs the DISTINCT query, returns: GOLD, GOLDM, SILVER, CRUDEOIL, etc.

> "Who owns the commodity_order_master dataset?"
  → Compass uses `get_entities`, returns ownership info
```

You capture all this information and write it into the skill document.

### Phase 2: Writing Tested SQL

```
You (in Claude Code):

> "Run this query and tell me if it works:
   SELECT order_date, COUNT(*) as orders, 
          COUNT(DISTINCT user_account_id) as users
   FROM platform_iceberg.core_invest.commodity_order_master
   WHERE order_date >= DATE '2026-03-01'
   GROUP BY order_date
   ORDER BY order_date DESC
   LIMIT 10"
  → Compass executes via `run_trino_query`, returns results
  → You verify it works, add to skill as tested example query

> "This query timed out. Try with commodity_day_view_seg instead."
  → Compass runs the alternative
  → You document the guardrail: "Use commodity_day_view_seg (684K rows) over 
     commodity_order_master (71.6M, no partition) for aggregated metrics"
```

### Phase 3: Writing the Skill Document

After discovery and testing, you write the skill document as a Markdown file following the standardized format. The document is then published to Datahub.

## 3.3 Building Minimal Skills from Scratch

For the 3 skills that are currently minimal (Context Push, User Financial Health, MF Extended), here's the detailed process:

### Example: Context Push Analytics (Minimal → Full)

**Day 1 — Table Discovery:**

```
Claude Code session:

> "Search DataHub for datasets related to context push notifications"
> "List tables in core_bgv schema that have 'context' or 'push' in the name"
> "Show me columns for each table found"
> "What's the row count and freshness (MAX date) for each table?"
> "What are the distinct values of [key dimension columns]?"
```

Capture all results into a working doc.

**Day 2 — SQL Development:**

Write and test queries for the example questions:
1. "Context push delivery volume by type"
2. "Delivery success rate by trigger context"
3. "User response rates for contextual vs broadcast push"
4. etc.

Test each query via Compass's `run_trino_query`. If a query fails or times out, document why and add as a guardrail.

**Day 3 — Document Assembly:**

Write the full skill document with:
- Tables Overview (from Day 1 discovery)
- Key Dimensions (from DISTINCT queries)
- Example Questions (8+)
- Key Guardrails (from query failures)
- Tested SQL (10+)

**Day 4 — Validation:**

```
Claude Code session:

> "Using the Context Push skill, what was yesterday's contextual push delivery rate?"
  → Verify Compass loads the skill and generates correct SQL

> "Show me context push volume trends for the last 7 days"
  → Verify correct table selection and partition filtering
```

Fix any issues, get domain owner review, publish to Datahub.

## 3.4 Skill Authoring Toolkit

While most discovery happens conversationally through Compass, these Python tools help with quality control:

### Skill Validator

**File: `compass-skills/tools/validator.py`**

```python
"""
Validates a skill document for completeness against the quality checklist.
Run before publishing to Datahub.
"""

import re
from pathlib import Path


def validate(skill_path: Path):
    content = skill_path.read_text()
    checks = []

    # Tables Overview with actual entries
    table_rows = len(re.findall(r"\| \d+ \|", content)) + len(re.findall(r"\| `\w+`", content))
    checks.append(("Tables documented", table_rows >= 1, f"{table_rows} tables found"))

    # Key Dimensions section
    has_dimensions = "dimension" in content.lower() or "key dimension" in content.lower() or "filter value" in content.lower()
    checks.append(("Key dimensions/filter values", has_dimensions, "Found" if has_dimensions else "Missing"))

    # Example Questions
    question_marks = content.count("?")
    checks.append(("Example questions (8+)", question_marks >= 8, f"{question_marks} questions found"))

    # Guardrails
    guardrail_keywords = len(re.findall(r"guardrail|always filter|do not use|stale|warning|critical|partition", content, re.I))
    checks.append(("Guardrails documented", guardrail_keywords >= 3, f"{guardrail_keywords} guardrail references"))

    # SQL queries
    sql_blocks = len(re.findall(r"```sql|SELECT\s", content, re.I))
    checks.append(("SQL queries (10+)", sql_blocks >= 10, f"{sql_blocks} SQL blocks found"))

    # Stale table warnings
    has_stale_warnings = "stale" in content.lower() or "do not use" in content.lower() or "deprecated" in content.lower()
    checks.append(("Stale table warnings", has_stale_warnings, "Found" if has_stale_warnings else "None (OK if no stale tables)"))

    # No TODOs remaining
    todos = len(re.findall(r"TODO|FILL IN|\[FILL\]|\[TBD\]", content, re.I))
    checks.append(("No remaining TODOs", todos == 0, f"{todos} found" if todos else "Clean"))

    # user_account_id mentioned
    has_join_key = "user_account_id" in content
    checks.append(("Join key documented", has_join_key, "Found" if has_join_key else "Missing"))

    # Print results
    print(f"\nValidation: {skill_path.name}")
    print("=" * 55)
    passed = 0
    for name, ok, msg in checks:
        icon = "✓" if ok else "✗"
        print(f"  {icon} {name}: {msg}")
        if ok:
            passed += 1

    print(f"\nScore: {passed}/{len(checks)}")
    print("✓ Ready to publish!" if passed == len(checks) else "✗ Fix issues above.")
```

### Query Tester (SQL Syntax Validator)

**File: `compass-skills/tools/query_tester.py`**

```python
"""
Validates SQL queries from .sql test files against known best practices.
Tests for: missing partition filters, stale table usage, SELECT * on large tables.

Note: Actual query execution is done via Compass's run_trino_query tool
in Claude Code. This script does offline static analysis only.
"""

import re
from pathlib import Path


# Tables that MUST have partition filters
PARTITION_REQUIREMENTS = {
    "core_stocks_order": "order_date",
    "fact_transaction_stocks": "order_date",
    "fno_order_master": "pt",
    "fno_pnl_master": "pt",
    "user_product_txn_master_fact": "order_date",
    "engagement_session_indepth": "session_date",
    "engagement_pn_narad_master": "event_date",
    "onb_usercheckpoints_master": "groww_signup_time",
    "commodity_order_master": None,  # No partition! Warn to use commodity_day_view_seg instead
    "ems_events": "pt",
    "growth_events": "event_ts AND event_name",
}

# Tables that must NOT be used
STALE_TABLES = {
    "stocks_order": "Use core_stocks_order instead",
    "invest_user_fid_info": "Use fact_onboarding_invest FID columns",
    "daily_onb_conversions": "EMPTY — use daily_onboarding_and_fid",
    "hns_freshchat_tickets_master_trino": "BROKEN — do not use",
    "mtf_transactions_v2": "Use aay_mtf_fid_agg instead",
}

# Tables where SELECT * is dangerous
LARGE_TABLES = {
    "engagement_session_indepth": "57B rows — SINGLE DAY queries only!",
    "user_product_txn_master_fact": "8.3B rows",
    "fact_transaction_stocks": "7.1B rows — max 7 days without aggregation",
    "fno_order_master": "3.8B rows — contains future dates to 2090!",
}


def test_all(query_dir: Path):
    sql_files = sorted(query_dir.glob("*.sql"))
    if not sql_files:
        print(f"No .sql files in {query_dir}")
        return

    print(f"Analyzing {len(sql_files)} SQL files in {query_dir}\n")
    total_issues = 0

    for sql_file in sql_files:
        content = sql_file.read_text()
        sql_upper = content.upper()
        issues = []

        # Check stale tables
        for stale, fix in STALE_TABLES.items():
            if stale.upper() in sql_upper:
                issues.append(f"STALE TABLE: {stale} — {fix}")

        # Check partition filters
        for table, partition_col in PARTITION_REQUIREMENTS.items():
            if table.upper() in sql_upper:
                if partition_col is None:
                    issues.append(f"WARNING: {table} has NO partition column — consider using pre-aggregated alternative")
                elif partition_col.upper() not in sql_upper:
                    issues.append(f"MISSING FILTER: {table} requires filter on {partition_col}")

        # Check SELECT * on large tables
        if "SELECT *" in sql_upper or "SELECT\n*" in sql_upper:
            for table, warning in LARGE_TABLES.items():
                if table.upper() in sql_upper:
                    issues.append(f"SELECT * on {table}: {warning}")

        # Check F&O future dates
        if "FNO_ORDER_MASTER" in sql_upper and "2027" not in content and "2090" not in content:
            if "IS_TRADINGDAY" not in sql_upper:
                issues.append("F&O: fno_order_master may include future dates (up to 2090) — add date cap or is_tradingday filter")

        # Check growth_user_master exclusions
        if "GROWTH_USER_MASTER" in sql_upper:
            if "XYBY" not in sql_upper:
                issues.append("WARNING: growth_user_master should exclude @xyby.net test accounts")
            if "CREDIT_ONLY" not in sql_upper:
                issues.append("WARNING: growth_user_master should filter credit_only_flag = 0")

        # Check Pinot sampling
        if "PINOT" in sql_upper and "BOOSTING_FACTOR" not in sql_upper:
            issues.append("PINOT: Raw COUNT(*) returns sampled count — must use SUM(boosting_factor * COUNT(*))")

        # Check engagement_session_indepth date range
        if "ENGAGEMENT_SESSION_INDEPTH" in sql_upper:
            issues.append("CAUTION: 57B total rows — ensure SINGLE DAY filter on session_date")

        # Report
        print(f"  {sql_file.name}:")
        if issues:
            for issue in issues:
                print(f"    ✗ {issue}")
            total_issues += len(issues)
        else:
            print(f"    ✓ No issues detected")
        print()

    print(f"Total: {total_issues} issues across {len(sql_files)} files")
```

## 3.5 Skill-by-Skill Implementation Guide

### Tier 1 — Minimal Skills (Ground-Up Build)

#### Context Push Analytics

**Discovery checklist:**
- [ ] Search DataHub for context push datasets
- [ ] List tables in relevant schemas (`core_bgv`, `core_invest`)
- [ ] Get columns for each table via `list_schema_fields`
- [ ] Run `COUNT(*)` and `MAX(date_column)` for freshness
- [ ] Run `SELECT DISTINCT` on key dimension columns
- [ ] Find the domain data owner via `get_entities`

**Questions this skill must answer:**
1. Context push delivery volume by trigger type
2. Delivery success rate by context
3. User click-through rates for contextual vs broadcast
4. Time-of-day optimization for contextual push
5. Context push coverage (what % of users receive contextual)
6. Trigger frequency distribution
7. Context push → app open → transaction conversion
8. A/B test results for contextual triggers

**Key things to discover:**
- What makes "context push" different from regular push (in `engagement_pn_narad_master`)
- Is there a separate table, or is it a filter on the existing PN table?
- What are the trigger context types?
- How does it join to user_master?

#### User Financial Health Analytics

**Discovery checklist:** Same as above but for financial health domain.

**Questions to answer:**
1. User AUM distribution by tier/cohort
2. Investment diversification score distribution
3. SIP consistency rate (how many months in a row)
4. Portfolio risk profile distribution
5. Revenue per user by engagement level
6. Correlation between financial health score and retention
7. Users at risk of churn based on financial behavior
8. Financial health improvement over time

#### MF Analytics Extended

**Discovery checklist:** Same pattern. This may overlap with existing MF skill (Skill 3) — need to understand what "Extended" covers that the base skill doesn't.

### Tier 2 — Medium Skills Enhancement

#### Engagement & Communications (Skill 8) — Enhancement

This is the most relevant skill for my Content Engine work. The Deep Dive already documents 9 tables and good guardrails. Enhancement focus:

**What to add:**
- [ ] Column-level docs for `engagement_email_backend_master` — every column with type and description (this table directly relates to my Content Engine output)
- [ ] Column-level docs for `engagement_pn_narad_master` — key columns for campaign analysis
- [ ] 10+ tested SQL queries (the Deep Dive lists example questions but not actual SQL)
- [ ] Cross-reference to Content Engine: which campaign metrics should inform recipe design?

**SQL queries to write and test:**

```sql
-- 1. Daily email delivery funnel
SELECT event_date,
       COUNT(*) as total,
       COUNT(*) FILTER (WHERE status = 'SENT') as sent,
       COUNT(*) FILTER (WHERE status = 'DELIVERED') as delivered,
       COUNT(*) FILTER (WHERE status = 'OPENED') as opened,
       COUNT(*) FILTER (WHERE status = 'CLICKED') as clicked
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date >= current_date - interval '7' day
GROUP BY event_date
ORDER BY event_date DESC;

-- 2. Push notification CTR by campaign
-- 3. Email open rate by event_group
-- 4. SMS delivery rate by vendor
-- 5. WhatsApp read receipt rates
-- 6. Cross-channel volume comparison (daily)
-- 7. DND opt-out trend by OS
-- 8. Top campaigns by CTR
-- 9. Session quality (bounce rate, avg duration)
-- 10. App install and push reachability
```

**Note:** The exact column names (status values, event_group, etc.) need to be discovered via Compass's `list_schema_fields` on `engagement_email_backend_master`. The Deep Dive lists the tables and volumes but not column-level detail.

#### Commodities Trading (Skill 10) — Enhancement

**What to add:**
- [ ] Column-level docs for `commodity_day_view_seg` (preferred table) and `commodity_order_master`
- [ ] Document the no-partition issue: `commodity_order_master` has 71.6M rows with NO partition column — always use `commodity_day_view_seg` (684K rows, pre-aggregated) for aggregated metrics
- [ ] Document that timestamps are epoch milliseconds → `from_unixtime(col / 1000)`
- [ ] Document underlying naming: GOLD vs GOLDM (mini) — use `asset` column in turnover tables
- [ ] 10+ tested SQL

#### Mutual Funds (Skill 3) — Add Guardrails

The Deep Dive notes 10+ tables and a table selection matrix but no explicit guardrails. Add:
- [ ] Partition column requirements for large MF tables
- [ ] SIP status values and lifecycle transitions
- [ ] Payment type values for order completion analysis
- [ ] Stale table checks (are any MF tables stale?)

### Tier 3 — Deep Skills (Incremental)

For Stocks, F&O, User Master, User Events, Onboarding — these are already well-documented. Enhancement is:
- [ ] Add 3–5 more tested SQL queries each (especially complex multi-table joins)
- [ ] Add cross-skill join patterns (e.g., "join stock orders to user_master for demographic breakdown")
- [ ] Verify all existing guardrails are still accurate (check stale table dates)
- [ ] Add any new tables created since initial skill writing

---

# 4. Testing Strategy

## 4.1 Content Engine Tests

```
tests/
├── analyzer/
│   ├── test_parser.py          # HTML parsing on sample emails
│   ├── test_block_detector.py  # Block detection accuracy
│   ├── test_classifier.py      # Email classification
│   └── fixtures/               # Sample HTML files
├── renderer/
│   ├── test_engine.py          # Recipe → HTML rendering
│   └── test_blocks.py          # Individual block templates
├── charts/
│   └── test_renderer.py        # Chart image generation
└── mcp/
    └── test_tools.py           # MCP tool handlers
```

## 4.2 Compass Skills Tests

SQL test files per skill, validated by the query tester:

```
compass-skills/tests/queries/
├── engagement_comms/
│   ├── 01_email_delivery_funnel.sql
│   ├── 02_pn_ctr_by_campaign.sql
│   └── ...
├── commodities/
│   ├── 01_daily_order_volume.sql
│   └── ...
└── context_push/
    ├── 01_delivery_by_type.sql
    └── ...
```

Each `.sql` file is also tested interactively via Compass's `run_trino_query` in Claude Code.

---

# 5. Deployment & Configuration

## 5.1 Content Engine

```bash
# Analyze emails
python cli.py analyze data/raw_emails/ --output-dir data/analyzed/

# Generate a chart
python cli.py generate-chart price_card data/sample_chart.json

# Render an email
python cli.py render-email data/recipes/commodity_educational.json data/sample_content.json

# Start MCP server (for Claude Code)
python cli.py serve-mcp
```

## 5.2 Claude Code Configuration (Both MCPs)

A Claude Code session can have **both** Compass and Content Engine connected simultaneously:

```bash
# Add Compass (already configured via setup script)
claude mcp add compass --transport http http://127.0.0.1:3100/mcp

# Add Content Engine
# In .claude.json or via CLI:
claude mcp add content-engine --transport stdio -- python -m src.mcp.server
```

Then in Claude Code, the CM has access to both:
- Compass tools for analytics: "What's the email open rate for commodity campaigns?"
- Content Engine tools for generation: "Generate a commodity email about crude oil"

They're parallel. The human bridges the gap.

## 5.3 Compass Skills Workflow

```bash
# 1. Discover schemas (via Claude Code + Compass — conversational, not scripted)
# 2. Write and test SQL (via Compass run_trino_query)
# 3. Assemble skill document (Markdown)

# 4. Validate completeness
python cli.py validate-skill compass-skills/enhanced/engagement_comms.md

# 5. Test SQL files offline
python cli.py test-queries compass-skills/tests/queries/engagement_comms/

# 6. Publish to Datahub (manual or via Compass metadata tools)
```

## 5.4 Environment Variables

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...              # Content Engine copy generation
GROWW_CLOUD_BUCKET=...                     # Image uploads (when available)

# Compass (handled by proxy, not directly in .env)
# MCP_SERVER_URL, MCP_AUTH_HEADER, MCP_PROXY_PORT
# These are in the LaunchAgent plist, not your app's .env
```
