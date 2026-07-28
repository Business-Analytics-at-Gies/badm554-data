# badm554-data

Pinned course datasets for **BADM 554 Enterprise Database Management** (Gies College of
Business, University of Illinois Urbana-Champaign).

This repository exists for one reason: **stability**. Course videos and scripts speak
exact numbers out loud. A learner who runs the one-liner in a lesson should get the same
row counts, the same averages, and the same outliers that the instructor got on camera,
for the entire shelf life of the video. Upstream open-data publishers restate, re-issue,
and quietly clean historical files. When that happens, a learner's numbers stop matching
the narration and there is no error message to explain why.

So the exact bytes used to build the course are pinned here as **release assets** and
never change. The data files are **not** committed to git; they are attached to a tagged
release and served over HTTPS.

---

## What is in the pinned release

Release tag: **`nyc-taxi-2024-01`**

| Asset | Size (bytes) | SHA-256 |
|---|---|---|
| `yellow_tripdata_2024-01.parquet` | 49,961,641 | `c4d59da7bbc8abaeeeb1727947ee93d9891a71acb42854bd80db1571b2030510` |
| `taxi_zone_lookup.csv` | 12,331 | `1a99e105092230f8620f301edcca7f80d3080642ff404d28ed957d3fa222c8ed` |

**Verified row counts** (DuckDB 1.5.5):

| File | Rows |
|---|---|
| `yellow_tripdata_2024-01.parquet` | **2,964,624** trips |
| `taxi_zone_lookup.csv` | **265** zones |

---

## Provenance

**Upstream source:** NYC Taxi and Limousine Commission (TLC) Trip Record Data.

| Item | URL |
|---|---|
| Dataset landing page | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |
| Yellow trip file (Jan 2024) | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet |
| Taxi zone lookup table | https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv |
| Trip Record User Guide | https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf |
| Yellow Trips Data Dictionary | https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf |

**Pinned on:** 2026-07-28.

**Upstream `Last-Modified` at pin time:**

- `yellow_tripdata_2024-01.parquet` — Thu, 21 Mar 2024 15:35:44 GMT
- `taxi_zone_lookup.csv` — Thu, 22 Feb 2024 21:33:00 GMT

The assets in this release were verified byte-identical to the upstream files at pin
time by SHA-256 comparison.

### Why January 2024

- It is a **complete calendar month**, so month-boundary and day-of-week effects are
  clean and every worked example has a well-defined denominator.
- At about 3 million rows it is **big enough to be interesting and small enough to be
  free**: a roughly 48 MB download that DuckDB scans in under a second on a laptop, with
  no cloud account, no billing, and no quota.
- It is **old enough to be settled**. TLC publishes on roughly a two-month delay and
  revises recent months; a two-year-old month is far less likely to be restated.
- It **predates the 2025 schema change** that added a `cbd_congestion_fee` column to the
  Yellow, Green, and High Volume FHV datasets, so the column list stays stable.

### The teaching moment that depends on these exact bytes

One course screencast demonstrates a wrong answer that produces no error, no warning,
and no null. Averaging `trip_distance` by pickup borough on the raw, unfiltered table
reports:

| Borough | Trips | Avg miles | Max miles |
|---|---|---|---|
| Brooklyn | 25,258 | **13.11** | **82,015.45** |
| Queens | 273,128 | 12.30 | 10,879.28 |
| Manhattan | 2,646,948 | 2.66 | 312,722.30 |

Brooklyn appears to have the longest rides in New York City. It does not. Brooklyn has a
small sample (25,258 trips) containing a handful of broken-meter rows, the largest at
**82,015.45 miles**. A few bad rows move the mean by about seven miles. Filtered, Brooklyn
comes back to roughly 6 miles.

That lesson only works if those specific rows are still in the file the learner
downloads. Hence the pin.

(For the record, the largest `trip_distance` anywhere in the file is **312,722.30** miles,
on a Manhattan pickup. Nine rows exceed 50,000 miles.)

---

## Usage

### Read the pinned release directly over HTTPS

DuckDB can read the release asset without downloading it first.

```sql
INSTALL httpfs;
LOAD httpfs;

SELECT count(*) AS trips
FROM read_parquet('https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/nyc-taxi-2024-01/yellow_tripdata_2024-01.parquet');
-- 2964624

SELECT count(*) AS zones
FROM read_csv_auto('https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/nyc-taxi-2024-01/taxi_zone_lookup.csv');
-- 265
```

Reading Parquet over HTTPS uses range requests, so a query that touches only a few
columns transfers only those columns rather than the whole 48 MB file.

### Download once, then read locally

Faster and kinder to the network if you are going to run many queries.

```bash
mkdir -p data && cd data
BASE=https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/nyc-taxi-2024-01
curl -L -O "$BASE/yellow_tripdata_2024-01.parquet"
curl -L -O "$BASE/taxi_zone_lookup.csv"

# Optional integrity check
shasum -a 256 yellow_tripdata_2024-01.parquet taxi_zone_lookup.csv
```

```sql
-- from the directory containing the files
SELECT count(*) FROM read_parquet('data/yellow_tripdata_2024-01.parquet');
SELECT count(*) FROM read_csv_auto('data/taxi_zone_lookup.csv');
```

Or persist them into a local DuckDB database:

```sql
CREATE TABLE trip AS
  SELECT * FROM read_parquet('data/yellow_tripdata_2024-01.parquet');
CREATE TABLE zone AS
  SELECT * FROM read_csv_auto('data/taxi_zone_lookup.csv');
```

Join key: `trip.PULocationID` and `trip.DOLocationID` both reference `zone.LocationID`.

---

## Attribution and terms

The data is published by the **New York City Taxi and Limousine Commission (TLC)**.
Please credit the TLC as the source in any work built on it.

Suggested credit line:

> Source: NYC Taxi and Limousine Commission (TLC) Trip Record Data, yellow taxi trips,
> January 2024. https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

**On the license, stated precisely and without embellishment.** The TLC trip-record page
does **not** carry an open-data license such as CC0, CC-BY, or ODbL. It is public data
published on nyc.gov, and the only applicable terms are the site-wide
**[City of New York Terms of Use](https://www.nyc.gov/home/terms-of-use.page)**, whose
intellectual-property section states that materials on nyc.gov "are the property of the
City of New York. All rights are reserved." The AWS Registry of Open Data entry for this
dataset likewise records its license simply as
["NYC terms of use"](https://registry.opendata.aws/nyc-tlc-trip-records-pds/) rather than
naming a standard open license.

**No open license is asserted here.** This repository redistributes an unmodified copy of
a publicly published government file for non-commercial academic instruction, with the
source named and linked. Anyone reusing it for another purpose, and especially for a
commercial one, should read the City terms and, if needed, contact the TLC at
research@tlc.nyc.gov.

**TLC's own accuracy disclaimer**, quoted from the dataset page:

> The data used in the attached datasets were collected and provided to the NYC Taxi and
> Limousine Commission (TLC) by technology providers authorized under the Taxicab &
> Livery Passenger Enhancement Programs (TPEP/LPEP). The trip data was not created by the
> TLC, and TLC makes no representations as to the accuracy of these data.

That disclaimer is not a footnote in this course. It is the point of the Brooklyn example
above.

Neither the TLC nor the City of New York endorses this repository, this course, or the
University of Illinois.

---

## Repository policy

- **No data file is ever committed to git.** `.gitignore` blocks `*.parquet` and friends.
  Data lives in releases only, so the repository stays small and clones stay fast.
- **Releases are immutable.** If a new month or dataset is needed, cut a new tag. Do not
  replace assets on an existing tag, because that silently breaks every video and script
  pinned to it.

Maintained for BADM 554 by Vishal Sachdev (vishal@illinois.edu).
