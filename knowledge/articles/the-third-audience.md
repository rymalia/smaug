---
title: "The Third Audience"
author: "Dries Buytaert"
source: https://dri.es/the-third-audience
date: "2026-01-14"
archived: "2026-01-14"
tags: [ai, web-development, markdown, seo, ai-optimization]
category: article
---

# The Third Audience

**Author:** Dries Buytaert
**Published:** January 14, 2026
**Source:** https://dri.es/the-third-audience

## Summary

Dries Buytaert experimented with making his website's content available in Markdown format for AI agents and crawlers. Within an hour of implementation, he observed hundreds of requests from ClaudeBot, GPTBot, and OpenAI's SearchBot. This highlights the emergence of AI agents as a "third audience" for web content, alongside humans and traditional search engines.

## Key Points

### The Implementation

- Added content negotiation to serve Markdown when requests include `Accept: text/markdown` headers
- Made URLs appendable with `.md` extension (e.g., `https://dri.es/principles-for-life.md`)
- Implemented "Markdown auto-discovery" using `<link rel="alternate" type="text/markdown">` tags in HTML (similar to RSS auto-discovery)
- The auto-discovery feature was key to rapid adoption by AI crawlers

### Why It Matters

- **Three Audiences:** For two decades, websites optimized for humans and search engines (SEO). AI agents are now the third audience.
- **New Optimization Landscape:** Concepts like Generative Engine Optimization (GEO) and Answer Engine Optimization (AEO) are emerging alongside traditional SEO.
- **Cleaner Content Formats:** AI agents actively seek and consume cleaner, more structured content formats when available.
- **Drupal Made It Easy:** The author's site already stored content as Markdown; Drupal's architecture made serving it in multiple formats straightforward.

### The Dilemma

> "Humans, including me, are teaching machines how to read our sites better, while machines are teaching humans to stop visiting us."

The experiment raises questions about the value exchange between content creators and AI companies:
- Will this lead to more visibility in AI-generated answers?
- Or will it simply make it easier for AI to use content without driving traffic back to creators?
- The broken deal between the web and AI companies remains unsettled

### Technical Approach

1. **Content Negotiation:** Server responds with Markdown when HTTP headers request it
2. **URL Pattern:** Clean `.md` extension approach for direct access
3. **Auto-Discovery:** HTML link tags announce alternate Markdown versions
4. **Immediate Impact:** Crawlers parse HTML, find the alternate link, and immediately request Markdown

## Personal Take

This is a fascinating real-world experiment in the evolving relationship between web publishers and AI systems. The speed of crawler adoption (hundreds of requests within an hour) demonstrates that AI systems are actively looking for cleaner, more structured content formats.

The author acknowledges the tension: helping AI read content better may accelerate "the hollowing out of the web" as AI answers reduce direct site visits. However, he chooses to experiment rather than ignore the shift.

## Implications

- **For Content Creators:** Consider making content available in AI-friendly formats, but be aware of the tradeoff
- **For Web Platforms:** Content management systems should support multiple output formats
- **For the Web Ecosystem:** The relationship between creators and AI companies needs resolution
- **For SEO Strategy:** Optimization now extends beyond Google to AI agent visibility

## Related Concepts

- RSS auto-discovery (the inspiration for Markdown auto-discovery)
- Generative Engine Optimization (GEO)
- Answer Engine Optimization (AEO)
- Content negotiation in HTTP
- The glymphatic system of the web (content clearance and flow)

---

**Archived from:** https://dri.es/the-third-audience
**Archive Date:** January 14, 2026
