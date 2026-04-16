# Scrapling — Web Scraping Tool

## Location
`/home/majk/.openclaw/workspace/tools/Scrapling/`

## Installation
```bash
pip install -e /home/majk/.openclaw/workspace/tools/Scrapling/ --break-system-packages
pip install curl-cffi playwright browserforge msgspec --break-system-packages
python3 -m playwright install chromium
```

## Core Concept
Two fetchers, one parser — choose based on site type:

### Fetcher Types

| Fetcher | Engine | Use When |
|---------|--------|----------|
| `Fetcher` | curl_cffi | Static HTML sites, simple pages, fastest |
| `DynamicFetcher` | Playwright (browser) | JS-heavy sites, SPAs, rendered content |

**Rule of thumb:** Try `Fetcher` first. If content is missing or site is JS-driven → use `DynamicFetcher`.

### Parser
The `Response` object IS the parser. Call methods directly on it:
- `response.find_all('p')` — find elements
- `response.get_all_text()` — extract all text
- `response.css('h1')` — CSS selector
- `response.xpath('//p/text()')` — XPath

### Common Pattern
```python
from scrapling import DynamicFetcher

fetcher = DynamicFetcher()
response = fetcher.fetch("https://example.com/article")

# Find article content
article = response.find_all('article')[0]
text = article.get_all_text()

# Or just get all text
text = response.get_all_text()
```

## Common Errors
- `ModuleNotFoundError: 'curl_cffi'` → `pip install curl-cffi --break-system-packages`
- `ModuleNotFoundError: 'playwright'` → `pip install playwright --break-system-packages` + `python3 -m playwright install chromium`
- `ModuleNotFoundError: 'browserforge'` → `pip install browserforge --break-system-packages`
- `ModuleNotFoundError: 'msgspec'` → `pip install msgspec --break-system-packages`
- Timeout on JS sites → `DynamicFetcher` handles this automatically (retries with longer timeout)

## Key Methods

### Fetcher (static)
```python
fetcher = Fetcher()
response = fetcher.get("https://example.com")
```

### DynamicFetcher (JS-heavy)
```python
from scrapling import DynamicFetcher
fetcher = DynamicFetcher()
response = fetcher.fetch("https://example.com")  # synchronous
# or
response = await fetcher.async_fetch("https://example.com")  # async
```

### Response/Parser
```python
response.find_all('div', {'class': 'article-body'})  # find by attrs
response.get_all_text()  # all text
response.css('article h1')  # css selector
response.xpath('//article//p/text()')  # xpath
```

## Pitfalls
- `get_article()` often fails on custom JS-driven layouts — use `find_all('article')` instead
- Basic `Fetcher` returns raw HTML without JS rendering — wrong tool for Slate-like sites
- `DynamicFetcher` is slow (launches browser) but gets the actual rendered content
