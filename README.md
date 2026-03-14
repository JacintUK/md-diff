# md-diff

Rendered markdown diff — like GitHub's Rich Diff, but locally.

Takes two markdown files, renders them to HTML via pandoc, and produces a
structural diff of the rendered output. The result is a standalone HTML file
with inline additions and deletions highlighted in context.

## Installation

```
pip install .
```

Requires [pandoc](https://pandoc.org/) to be installed separately.

## Usage

### CLI

```
md-rich-diff old.md new.md [-o output.html]
```

Output defaults to `diff-<old>-vs-<new>.html`.

The ASCII table converter is also available standalone:

```
ascii-table input.md [-o output.md]
```

### As a library

```python
from md_diff import render_markdown, diff_sections, convert_ascii_tables
```

## How it works

1. **ASCII table pre-processing** — `ascii_table` detects code blocks
   containing Unicode box-drawing characters (│┌┐└┘├┤┬┴┼─ etc.), parses
   their grid structure, and converts them to HTML `<table>` elements before
   pandoc sees them.  This prevents pandoc from treating diagrams as plain
   code blocks.

2. **Markdown → HTML** — Both files are rendered through pandoc.

3. **Section-level matching** — The HTML is split by headings and sections
   are matched between the old and new documents using `SequenceMatcher`.
   Unmatched sections appear as whole-block additions or deletions.

4. **Block-level diffing** — Within matched sections, content is split into
   blocks (tables vs. everything else).  Non-table blocks are diffed with
   `lxml.html.diff.htmldiff`.

5. **Table-aware diffing** — Tables are diffed row-by-row and cell-by-cell,
   producing per-cell inline diffs with inserted/deleted/changed row
   styling.

## ASCII table features

Handles complex box-drawing diagrams including:
- Partial separators and spanning (rowspan) columns
- Bullet-list content inside cells (▸, ●, - markers)
- Multi-line cells merged into logical rows
- Sections with varying column counts (colspan)

## Dependencies

- Python 3.10+
- [pandoc](https://pandoc.org/) (external)
- [lxml](https://lxml.de/) (installed automatically via pip)
