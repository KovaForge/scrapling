# Scrapling Research Usage

Use this file as the quick agent handoff for using Scrapling as a web-research extraction tool.

## Active source path

```text
/Users/mike/.openclaw/workspace/projects/scrapling
```

The local editable Python install should import Scrapling from this path.

## When to use Scrapling

Use Scrapling when normal web search or basic fetch is insufficient:

- exact source-page content matters more than snippets
- pages are JS-rendered or missing content in static fetch
- sites block simple HTTP clients
- CSS/XPath-targeted extraction is needed
- many related pages need crawling
- clean Markdown/text is needed for LLM research

Default flow:

1. Use normal search to discover candidate URLs.
2. Fetch real source pages with Scrapling.
3. Escalate only as needed: static → browser → stealth → spider.

## CLI quick commands

Always use `--ai-targeted` for LLM-facing research extraction.

```bash
# Static HTML page
scrapling extract get "https://example.com/article" /tmp/page.md --ai-targeted

# Extract a specific region
scrapling extract get "https://example.com/docs" /tmp/docs.md --css-selector "main" --ai-targeted

# JS-rendered page
scrapling extract fetch "https://example.com/app" /tmp/page.md --network-idle --ai-targeted

# Protected / anti-bot page
scrapling extract stealthy-fetch "https://example.com/protected" /tmp/page.md --solve-cloudflare --ai-targeted
```

Read the temp file, extract the needed facts, then clean up temp files.

## Python examples

Static/session fetch:

```python
from scrapling.fetchers import FetcherSession

with FetcherSession(impersonate="chrome") as session:
    page = session.get("https://example.com", stealthy_headers=True)
    title = page.css("h1::text").get()
    links = page.css("a::attr(href)").getall()
```

JS-rendered page:

```python
from scrapling.fetchers import DynamicSession

with DynamicSession(headless=True, network_idle=True) as session:
    page = session.fetch("https://example.com/app", load_dom=False)
    content = page.css("main::text").getall()
```

Protected page:

```python
from scrapling.fetchers import StealthySession

with StealthySession(headless=True, solve_cloudflare=True) as session:
    page = session.fetch("https://example.com/protected", google_search=False)
    content = page.css("main::text").getall()
```

Multi-page crawl:

```python
from scrapling.spiders import Spider, Response

class ResearchSpider(Spider):
    name = "research"
    start_urls = ["https://example.com/"]
    concurrent_requests = 10

    async def parse(self, response: Response):
        for item in response.css("article"):
            yield {
                "title": item.css("h1::text").get(),
                "url": response.url,
                "text": " ".join(item.css("p::text").getall()),
            }

        for href in response.css("a::attr(href)").getall():
            if "/docs/" in href:
                yield response.follow(href)

result = ResearchSpider(crawldir="./crawl_data").start()
result.items.to_jsonl("research.jsonl")
```

## MCP mode

Scrapling can expose an MCP server:

```bash
scrapling mcp
```

Useful tools include `get`, `bulk_get`, `fetch`, `bulk_fetch`, `stealthy_fetch`, `bulk_stealthy_fetch`, `screenshot`, `open_session`, `close_session`, and `list_sessions`.

## Git sync

```bash
cd /Users/mike/.openclaw/workspace/projects/scrapling
git fetch upstream --prune
git pull --ff-only upstream main
git push origin main
git push briarforge main
```

Do not push to upstream `D4Vinci/Scrapling`; upstream push should stay disabled.

## Safety rules

- Respect robots.txt, site terms, rate limits, and privacy/legal constraints.
- Prefer Markdown/text and narrow CSS selectors over raw HTML.
- Use `--ai-targeted` for LLM-facing CLI extraction.
- Use persistent sessions for repeated dynamic/protected-site access.
- Do not log proxy credentials.
