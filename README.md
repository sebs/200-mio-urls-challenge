# The 200 Million URLs Challenge

**Store 200 million URLs. Answer a handful of questions about them, fast. Show everyone how you did it.**

This is a hands-on data storage challenge. Don't write a big architecture document. Build something, measure it, break it, improve it, and then present what you learned. Use any language and any database.

## The problem

A crawler watches 20,000 sources: websites, feeds and sitemaps. Every day it finds about 160,000 URLs. Some of them it has already seen. Over four years it collects 200 million unique URLs.

Your job is to design and build the storage behind it.

That sounds easy until you try it:

- **Size.** A plain table with an index on a text column gets big and slow at this scale.
- **Messy URLs.** Is `HTTP://Example.com/a?b=2&a=1` the same URL as `http://example.com/a?a=1&b=2`? Real URLs break the rules all the time.
- **No test data.** You don't have 200 million real URLs lying around, so you'll have to generate realistic ones.
- **Growth.** Adding new URLs every day has to stay fast in year four, and so do the queries.

There is no single right answer. What's interesting is the trade-offs you make.

## The numbers

| | |
|---|---|
| Unique URLs after four years | **200,000,000** |
| Sources | 20,000 |
| Incoming URLs per day (average) | ~160,000, including URLs already stored |
| New unique URLs per day (average) | ~137,000 |
| Time span | 4 years (~1,460 days) |

## What your system must do

1. **Ingest:** take in the daily batch of URLs and store each unique URL exactly once.
2. **Exists:** is this URL already stored?
   `https://shop.example.com/item?id=42` → yes / no
3. **New on a given day:** list all URLs that were first seen on a given day.
4. **Count by top-level domain:** how many URLs are under `.de`?
5. **Count by domain and subdomain:** how many URLs are under `example.com`, and how many are under `shop.example.com`? Both counts include the subdomains below that name.
6. **Search by GET parameter:** find URLs that have a given parameter, like `utm_source`, or a given parameter with a given value, like `utm_source=newsletter`. Paginated results are fine.

### Optional: the link graph

7. **Outgoing links:** which URLs does a given URL link to?
8. **Incoming links:** which URLs link to a given URL?

The challenge doesn't say how many links a page has. Pick a number, such as 50 outgoing links per URL, and state it in your results.

## Rules

- **Use any tech you like:** Postgres, SQLite, ClickHouse, RocksDB, Parquet files, your own storage engine, a cloud service. Everything is allowed.
- **Run on a single machine or a cluster.** It's your call. Just report what you used.
- **Use synthetic data.** Generate it yourself; the hints below help. If you can, share your generator so others can reuse it.
- **You don't have to load all 200 million.** Testing with 10 or 50 million and extrapolating is fine. Say that you did and show how you calculated it.
- **Report what didn't work.** A well-explained dead end is a valuable result.
- **Keep it fun.** Work iteratively: build a small version, measure it, improve it.

## How to take part

1. **Fork this repository**, or start your own repo and link to it.
2. **Build a first version quickly**, measure it, then improve it.
3. **Write up your results** in a `RESULTS.md` in your repo, using the template below.
4. **Open an issue** in this repository titled `Submission: <your name or team>` and include the link to your repo.
5. **Present your results** (see below).

Teams are welcome. So are unfinished solutions, as long as they come with honest numbers.

## What to report

To make solutions comparable, copy this template into your `RESULTS.md`:

```markdown
# <Name / Team>: 200 Million URLs Challenge

## Approach in one sentence

## Setup
- Hardware (CPU, RAM, disk type) or cloud instance type:
- Software and versions:
- URLs actually loaded:
- How the test data was generated:

## Storage
- Size on disk, including indexes:
- Bytes per URL:
- Projected size at 200 million URLs:

## Ingest
- Time to ingest one day (160,000 URLs, including duplicate check):
- Time for the initial bulk load:
- Does ingest get slower as data grows? (numbers at different sizes if possible)

## Queries
| Query                        | Example used | Median | p99 |
|------------------------------|--------------|--------|-----|
| Exists                       |              |        |     |
| New on a given day           |              |        |     |
| Count by TLD                 |              |        |     |
| Count by domain / subdomain  |              |        |     |
| Search by GET parameter      |              |        |     |
| Outgoing links (optional)    |              |        |     |
| Incoming links (optional)    |              |        |     |

## Design
- Data model and why:
- URL normalization rules (and what you decided NOT to normalize):
- What I tried that didn't work:
- Known weaknesses / what I'd do next:
```

## Presenting your results

Plan for a 10 to 15 minute talk. A structure that works:

1. **The idea:** your approach in one sentence.
2. **The data model:** a diagram is enough.
3. **The numbers:** storage, ingest time and query times.
4. **The surprise:** what broke, what was harder or easier than expected.
5. **Next time:** what you would do differently.

## Hints

**Generating realistic data**
- Real traffic is skewed. A few domains have millions of URLs and most have only a handful, so pick domain sizes from a skewed (e.g. Zipf) distribution instead of a uniform one.
- Include the ugly cases: tracking parameters (`utm_*`, `fbclid`), the same parameters in a different order, mixed-case host names, trailing slashes, `#fragments`, default ports (`:443`), percent-encoding, `www.` vs. no `www.`, internationalized domain names and very long URLs.
- For more realism, get domains from the [Tranco list](https://tranco-list.eu/) or URL samples from [Common Crawl](https://commoncrawl.org/).

**Decisions you'll have to make**
- **When are two URLs the same?** Do you sort parameters? Strip `utm_*`? Lowercase the path, even though paths are case-sensitive? Write your rules down.
- **What is a TLD?** Is it `uk` or `co.uk`? The [Public Suffix List](https://publicsuffix.org/) exists for this. Say which one you count.
- **Do you store the full URL, or only its parts?**

**Back-of-the-envelope**
- If the average URL is 80 bytes, the raw text alone is 16 GB, before any index, metadata or link graph.

**Ideas worth a look**
- Split URLs into parts (scheme, host, path, query) and store them separately.
- Store host names reversed (`com.example.shop`) so a domain and all its subdomains become a single prefix scan.
- Hash URLs to fixed-size keys. How likely are collisions at 200 million?
- Use Bloom filters for a fast "definitely not stored" answer.
- Partition by day or by domain.
- Try columnar formats, LSM-tree stores and dictionary encoding for repeated host names.

---

Have fun, and bring your numbers.
