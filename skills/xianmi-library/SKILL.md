---
name: xianmi-library
description: Search and read Xianmi Library (显密文库, xianmi.co) — a non-profit digital library of ~250k pages of Chinese Buddhist texts (sutras 佛经, commentaries 注疏, Chinese/Tibetan/Theravada traditions, modern Chinese translations of the canon). Use this skill whenever a task involves finding, quoting, or citing Buddhist texts in Chinese, via the library's MCP server or its plain HTTP/Markdown interfaces.
license: MIT
---

# Xianmi Library (显密文库) access

Xianmi Library is a free, non-profit digital library of Chinese Buddhist literature at https://www.xianmi.co/ (Traditional Chinese edition at /zh-tw/, page-by-page parallel). All interfaces below are public and require no authentication.

## 1. Preferred path: MCP server

Remote MCP server (Streamable HTTP, stateless):

```
{ "mcpServers": { "xianmi": { "type": "http", "url": "https://mcp.xianmi.co/mcp" } }
```

Three tools — pick by query intent:

| Intent | Tool | Notes |
| --- | --- | --- |
| Conceptual / natural-language question ("如何消除烦恼", "the meaning of causality") | `semantic_search` | Embedding search over the **p05 collection only** (现代汉语白话译文 of Buddhist scriptures, Simplified Chinese, ~1.8k texts). Returns passages with relevance scores. |
| Exact quote or keyword lookup ("凡所有相皆是虚妄", an author's name, any term), or anything outside p05 | `search_texts` | Keyword full-text search over the **whole library (~250k pages, both language editions)**. Params: `query`, `lang`, `page` (10/page). |
| Read the full text of one article | `get_passage` | Param: `path` (e.g. `p01/a/109/764`, from a search result; a full page URL also works). Returns complete Markdown with YAML frontmatter. |

Typical flow: `search_texts` or `semantic_search` → pick a hit's `path` → `get_passage(path)` for full text. Every tool response ends with a source/license pointer block — keep it (see §4).

Fair-use caps are enforced with friendly fallback messages: `semantic_search` 800/month, `search_texts` / `get_passage` 5,000/day each.

## 2. Fallback path without an MCP client

Same capabilities over plain HTTP:

- Full-text search API: `GET https://www.xianmi.co/api/search?q=<keywords>&page=1` (Traditional edition: `https://www.xianmi.co/zh-tw/api/search?...`). JSON: `{ results: [{path, title, snippet}], total, page, ... }`.
- Per-page Markdown source: `https://www.xianmi.co/md/<path>.md` (Traditional: `https://www.xianmi.co/zh-tw/md/<path>.md`). Every HTML page also links its Markdown via `rel="alternate"`.
- `https://www.xianmi.co/llms.txt` describes the site structure and section map (部类 p01–p06).

## 3. Language editions (简体/繁體)

Content exists in two parallel editions: Simplified Chinese (简体, `lang: "zh-Hans"`, URLs under `/`) and Traditional Chinese (繁體, `lang: "zh-Hant"`, same paths under `/zh-tw/`). Default is zh-Hans. Choose by the user's reading preference, not by the query's script — the corpus is script-matched per edition, so query in the same script as the chosen edition for best keyword recall. `semantic_search` is zh-Hans-only (its index covers the Simplified p05 collection).

## 4. Citation and licensing

When quoting or republishing library content, attribute it. Every tool/API response already carries the pointer block; preserve it or cite equivalently:

- Source page URL (e.g. `https://www.xianmi.co/p01/a/109/764/`) + site name 显密文库 (Xianmi Library).
- License terms by content class (public domain / CC0 / copyrighted modern works): https://www.xianmi.co/open/ — p05 vernacular translations are NOT public domain (copyright remains with the translators); some modern works (p01/p02) and CBETA-sourced texts carry non-commercial terms. Check before republication.

Suggested citation format: `《标题》· 显密文库（xianmi.co）· <page URL>`.

## 5. Verse anchor comments (偈颂锚记)

Markdown sources may contain HTML comment anchors marking verse passages (偈颂/诗偈, gāthā):

```
<!-- verse:gatha-sutra -->
如是我闻……（verse lines, one line per verse line）
<!-- /verse -->
```

Semantics:

- The block between `<!-- verse:<subclass> -->` and `<!-- /verse -->` is verse, not prose. Subclasses: `gatha-sutra` (偈颂 in sutras), `gatha-treatise` (偈颂 in treatises/论典), `chan` (禅门诗偈), `liturgy` (仪轨唱诵), `poem` (诗), `modern-verse` (现代偈颂), `translated-poem` (翻译诗歌); a bare `<!-- verse -->` is an unclassified verse block.
- Anchors are HTML comments: invisible in rendered pages, safe to strip. They carry no metadata beyond the subclass.
- When quoting such a passage, preserve its line breaks (line = verse line 一句/一行); do not reflow verse into prose paragraphs.
- Verse passages are intentional religious/literary text: quote verbatim (逐字), never paraphrase when citing as scripture.
