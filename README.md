<p align="center"><a href="https://apify.com/DevilScrapes/breezy-hr-jobs-scraper"><img src="assets/banner.svg" alt="Breezy HR Jobs Scraper — Python example" width="100%"></a></p>

# Breezy HR Scraper in Python: careers board postings into a DataFrame

<p align="center">
<a href="https://apify.com/DevilScrapes/breezy-hr-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/breezy-hr-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/breezy-hr-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

Breezy HR is the applicant tracking system a lot of smaller and mid-sized employers pick, and it is especially common among clinics, care homes, hospitality groups and equipment dealers. Each of those employers gets a public board at `https://{company}.breezy.hr`. This repository is a short, tested Python script that sends a list of those subdomains to the **[Breezy HR Jobs Scraper](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper)** on Apify and saves what comes back: one row per open position, with title, department, employment type, pay text, city, region code, country code, a remote flag and the posting link. The collection runs in Apify's cloud; your side is a `pip install`, a token and a JSON file of employers. The output is a flat table, which is exactly what pandas, DuckDB or a spreadsheet wants.

> **Actor:** [Breezy HR Jobs Scraper](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [breezy-hr-scraper-javascript](https://github.com/devilscrapes/breezy-hr-scraper-javascript)

## 🔥 What you get

- **A whole board per request**: each employer's careers site hands over its complete list of open positions at once, so there is no paging and no half-read board when a run ends early.
- **Subdomain or link, your choice**: `rhynocare` and `https://rhynocare.breezy.hr/` resolve to the same employer. Vanity domains that merely embed a Breezy board are refused with a clear validation message.
- **Dead subdomains are skipped, not fatal**: a name Breezy does not recognise is logged and counted, and every other employer in the list still delivers.
- **A status line you can trust**: the run ends with postings scraped, employers that answered, slugs that were invalid and any that failed after five attempts.
- **Locations as codes**: region comes through as an abbreviation such as `ON` and country as an ISO alpha-2 code, which makes grouping and joining straightforward.
- **Pay left exactly as posted**: the `salary` string is whatever the employer typed, and an empty one becomes `null` so disclosure rates are easy to count.
- **Billing tied to rows**: the per-posting charge only fires for rows written to the dataset; an employer with no openings, or a mistyped one, adds nothing beyond the run's start fee.

## 🎯 Use cases

### Measure hiring demand across a sector's long tail

Large companies publish hiring data in press releases; small ones do not. Collect the Breezy subdomains of a few dozen regional healthcare or hospitality operators, run the script weekly, and count open reqs per employer, per region code and per `job_type`. A notebook with three `groupby` calls is enough to see which segment is hiring hardest.

### Study pay transparency with real postings

Because `salary` stays as the employer wrote it and blank values become `null`, the share of postings with a disclosed wage is a single `notna().mean()`. Split it by `location_state` or `location_country` to compare jurisdictions with and without pay-disclosure rules.

### Score accounts by hiring intent

Sales and business-development analysts can join the rows to a CRM export on company name or domain. An account that went from two open roles to fifteen in a month is a different conversation from one that has been flat all year, and a cron job plus a dated CSV per run is all the history you need.

### Build a regional job dataset for research

Economists and students working on local labour markets get structured rows with country and region codes already separated. Stack weekly files, deduplicate on `posting_id`, and use `published_date` to estimate how long roles stay open.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/breezy-hr-scraper-python.git
cd breezy-hr-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Breezy HR Jobs Scraper](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Strip away the CLI flags and file handling and the integration is the snippet below. `apify-client` starts the run, blocks until it reaches a final state, then iterates the dataset page by page so memory stays flat even for big boards.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/breezy-hr-jobs-scraper").call(run_input={'companySlugs': ['rhynocare', 'barloworldequipment'], 'maxResultsPerCompany': 5, 'proxyConfiguration': {'useApifyProxy': True, 'apifyProxyGroups': ['RESIDENTIAL']}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

The bundled `input.json` names two real Breezy employers, RhynoCare and Barloworld Equipment, and caps each at five postings, so a test run returns at most ten rows. To track your own list, open each employer's careers page, copy the part before `.breezy.hr`, and append it to `companySlugs`. The residential proxy group is the Actor's own default for this site.

```json
{
  "companySlugs": [
    "rhynocare",
    "barloworldequipment"
  ],
  "maxResultsPerCompany": 5,
  "proxyConfiguration": {
    "useApifyProxy": true,
    "apifyProxyGroups": [
      "RESIDENTIAL"
    ]
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `companySlugs` | array | yes | `["rhynocare", "barloworldequipment"]` | — |
| `maxResultsPerCompany` | integer | no | `200` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"]}` | — |

## 📤 Output

A row describes one live posting. The fields that are always filled are listed as such below; department, pay text and the three location fields are nullable because many employers leave them blank. `is_remote` falls back to `false` when Breezy does not say, so treat a `false` as "not marked remote" rather than proof of an on-site role.

| Field | Type | Always present |
|---|---|---|
| `posting_id` | string | yes |
| `company_slug` | string | yes |
| `company_name` | string | yes |
| `title` | string | yes |
| `apply_url` | string | yes |
| `published_date` | string | yes |
| `job_type` | string | yes |
| `department` | string or null | no |
| `salary` | string or null | no |
| `location_city` | string or null | no |
| `location_state` | string or null | no |
| `location_country` | string or null | no |
| `is_remote` | boolean | yes |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.0015 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

This pandas snippet (`pip install pandas`) turns `results.json` into a per-employer summary: open roles, how many are marked remote, the share with a published wage and the newest posting date. It also writes the reqs published in the last two weeks to `fresh_reqs.csv` for a recruiter to work through.

```python
import json

import pandas as pd

jobs = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
jobs["published_date"] = pd.to_datetime(jobs["published_date"], utc=True)
jobs["has_salary"] = jobs["salary"].notna()

summary = jobs.groupby("company_name").agg(
    open_roles=("posting_id", "count"),
    remote_roles=("is_remote", "sum"),
    pay_disclosed=("has_salary", "mean"),
    newest=("published_date", "max"),
)
summary["pay_disclosed"] = (summary["pay_disclosed"] * 100).round().astype(int).astype(str) + "%"
print(summary.sort_values("open_roles", ascending=False).to_string())

cutoff = pd.Timestamp.now(tz="UTC") - pd.Timedelta(days=14)
fresh = jobs[jobs["published_date"] >= cutoff]
fresh[["company_name", "title", "location_city", "salary", "apply_url"]].to_csv("fresh_reqs.csv", index=False)
```

## ❓ FAQ

### Where do I find an employer's Breezy subdomain?

Open their careers page. If the address bar shows something like `acme.breezy.hr`, the slug is `acme`. Some companies link to Breezy from their own site; follow the link until you land on the `breezy.hr` host and copy it from there.

### Can I get the full job description text?

Not from this Actor. Breezy's board listing carries titles, types, departments, pay and locations but no description body, and the Actor does not open each posting page. The `apply_url` column links to the full text if you need it for a subset.

### One of my slugs returned nothing. Did the run fail?

No. Either the employer has zero openings today, which counts as a successful company, or Breezy did not recognise the subdomain, which is counted under invalid slugs in the status message. Check the spelling against the careers URL.

### How do I keep history between runs?

Run the script on a schedule with a dated `--out` folder, then concatenate the CSVs in pandas. `posting_id` is stable, so the first and last snapshot a posting appears in give you a reasonable open-to-close window.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Breezy HR Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/breezy-hr-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Workable Jobs Scraper](https://apify.com/DevilScrapes/workable-jobs-scraper) — the same open-req view for employers hosted on Workable.
- [Teamtailor Jobs Scraper](https://apify.com/DevilScrapes/teamtailor-jobs-scraper) — career sites run on Teamtailor, common with Nordic and European employers.
- [Recruitee Jobs Scraper](https://apify.com/DevilScrapes/recruitee-jobs-scraper) — boards powered by Recruitee, for wider SMB coverage.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/breezy-hr-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Breezy HR.
