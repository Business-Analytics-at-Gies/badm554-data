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

There is a second reason, and it is just as load-bearing: **a course video that names a
host is a promise that the host will still be running in three years.** Personal servers,
bare IP addresses and self-hosted APIs do not survive that long, and when one dies it takes
the lesson with it silently. Nothing in this repository is a server. A pinned file with a
published SHA-256 has no administrator, no billing relationship, no firewall rule, and no
port to be blocked. That is why the rental-schema dataset below ships as a file rather than
as a database endpoint.

---

## Datasets

| Release tag | What it is | Used by |
|---|---|---|
| [`nyc-taxi-2024-01`](https://github.com/Business-Analytics-at-Gies/badm554-data/releases/tag/nyc-taxi-2024-01) | NYC TLC yellow taxi trips, January 2024. About 3 million rows of denormalized event data, plus a zone lookup. | Dimensional-modelling worked example: build a star schema, then break its grain. |
| [`pagila-3.0.0`](https://github.com/Business-Analytics-at-Gies/badm554-data/releases/tag/pagila-3.0.0) | Pagila, the PostgreSQL port of the Sakila DVD-rental sample database. 15 normalized tables. | Reading a normalized transactional schema, and the end-to-end ETL worked example that extracts from it and loads a star. |

The two are deliberately different shapes. The taxi extract is one wide table of events; the
rental database is many narrow tables with foreign keys between them. The course uses the
first to teach what a warehouse looks like and the second to teach what you have to extract
*from*.

---

# Dataset 1: NYC yellow taxi, January 2024

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

- `yellow_tripdata_2024-01.parquet`: Thu, 21 Mar 2024 15:35:44 GMT
- `taxi_zone_lookup.csv`: Thu, 22 Feb 2024 21:33:00 GMT

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

# Dataset 2: Pagila (Sakila rental schema)

Release tag: **`pagila-3.0.0`**

Pagila is the PostgreSQL port of Sakila, the DVD-rental sample database originally
developed by Mike Hillyer of the MySQL AB documentation team. It is a textbook teaching
artifact: fifteen normalized tables, real foreign keys, and a schema deliberately built to
make you join.

| Asset | Size (bytes) | SHA-256 |
|---|---|---|
| `pagila.duckdb` | 4,730,880 | `9deae7bf23f5e73aece8d9afa79b3772ac312f0e3b9db528dfdd127bdb5c3453` |
| `pagila.dump` | 866,146 | `3fde6c7c82fb1a026f82feab08b7ebaa562b060de35601f2c421290ce57fb5f0` |
| `pagila.sql` | 3,360,894 | `e3660097aa47a4bb1fb68d004e3cffb141450f8bac57e6f02bf3223a88dda919` |

- **`pagila.duckdb`** is a DuckDB 1.5.5 database holding all fifteen base tables. This is
  the artifact the course uses. Open it and query; there is nothing to install or restore.
- **[`docs/pagila-erd.md`](docs/pagila-erd.md)** is the schema diagram for those fifteen
  tables, rendered by GitHub right in the browser. Read it before you write your first join.
- **`pagila.dump`** is a `pg_dump` custom-format archive (`--no-owner --no-privileges`), for
  anyone who wants the schema in a real PostgreSQL server. Requires `pg_restore`.
- **`pagila.sql`** is the same database as plain SQL, loadable with `psql` alone.

**Verified row counts** (DuckDB 1.5.5, read from `pagila.duckdb`):

| Table | Rows | | Table | Rows |
|---|---|---|---|---|
| `actor` | 200 | | `film_category` | 2,367 |
| `address` | 603 | | `inventory` | 4,581 |
| `category` | 16 | | `language` | 6 |
| `city` | 600 | | `payment` | 16,049 |
| `country` | 109 | | `rental` | 16,044 |
| `customer` | 599 | | `staff` | 1,500 |
| `film` | 1,000 | | `store` | 500 |
| `film_actor` | 5,462 | | | |

`payment.payment_id` is unique across all 16,049 rows, so a one-row-per-payment grain
assertion holds on this data.

Note that Pagila 3.0.0 ships **1,500 staff rows and 500 stores**, not the two and two of the
original MySQL Sakila. That is upstream data, not a build error, and it is worth knowing
before you write a query that assumes a two-store business.

## Provenance

**Upstream source:** <https://github.com/devrimgunduz/pagila> (Pagila 3.0.0).

| Item | Value |
|---|---|
| Branch | `master` |
| Commit pinned | `5ba5a57aeb159f75f02aca2432d3c262186d13d3` (2024-12-01) |
| `pagila-schema.sql` SHA-256 | `8ce358e4c8014087b85296694a0893887bd7a4190e3ce407f2721b86b98e5707` |
| `pagila-data.sql` SHA-256 | `880580fb2cd4daaa99f290ced264988cdd657b3158be63cd281466f796f6dbf2` |
| Pinned on | 2026-07-28 |

**Build path.** The two upstream SQL files were loaded into a local PostgreSQL 17.10
instance. `pagila.dump` and `pagila.sql` are `pg_dump` output from that database.
`pagila.duckdb` was built by attaching that same PostgreSQL database from DuckDB 1.5.5 and
copying each base table across, so the three artifacts describe one build, not three.

**Two differences between the DuckDB file and the PostgreSQL dumps, stated plainly:**

1. Upstream `payment` is a partitioned table with seven monthly partitions
   (`payment_p2022_01` through `payment_p2022_07`). The PostgreSQL dumps preserve the
   partitioning. The DuckDB file collapses it into one `payment` table of 16,049 rows.
   Partitioning is a physical storage detail, not part of what the schema teaches here.
2. The DuckDB file carries **base tables only**. Upstream views, functions, triggers and the
   full-text index are in the PostgreSQL dumps and not in the DuckDB file.

## Why a file and not a server

M6 of the course teaches extract, transform and load. It would be tidier for the extract
stage to open a connection to a real PostgreSQL server, and that option was considered
seriously. It was rejected because the video outlives the server. A hostname spoken on
camera is a three-year commitment to keeping that host alive, reachable, and credentialed,
on someone's personal billing. Outbound port 5432 is also blocked on a meaningful number of
corporate and home networks, which produces a silent hang rather than an error for exactly
the learner who has no instructor to ask at 11pm.

A pinned file gives up one thing, named precisely: learners do not watch someone type a
host, port, user and password. Everything else about the lesson is unchanged, because
extract is reading, and reading a file is still reading.

## Usage

### Query it directly, no download

DuckDB can attach the release asset over HTTPS.

```sql
INSTALL httpfs; LOAD httpfs;
ATTACH 'https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/pagila-3.0.0/pagila.duckdb'
  AS pagila (READ_ONLY);

SELECT count(*) AS rentals FROM pagila.rental;
-- 16044
```

### Download once, then work locally

```bash
curl -L -O https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/pagila-3.0.0/pagila.duckdb
shasum -a 256 pagila.duckdb   # optional integrity check
```

```python
import duckdb
src = duckdb.connect("pagila.duckdb", read_only=True)
rentals = src.sql("SELECT * FROM rental").df()
films   = src.sql("SELECT * FROM film").df()
```

### Load it into PostgreSQL instead

```bash
curl -L -O https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/pagila-3.0.0/pagila.dump
createdb pagila
pg_restore --no-owner --no-privileges -d pagila pagila.dump
```

Or, without `pg_restore`:

```bash
curl -L -O https://github.com/Business-Analytics-at-Gies/badm554-data/releases/download/pagila-3.0.0/pagila.sql
createdb pagila
psql -d pagila -f pagila.sql
```

### The join that the schema exists to teach

A rental does not point at a film. It points at the physical copy that was rented, and the
copy points at the film.

```sql
SELECT r.rental_id, c.first_name, c.last_name, f.title, p.amount
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film      f ON i.film_id      = f.film_id
JOIN customer  c ON r.customer_id  = c.customer_id
LEFT JOIN payment p ON p.rental_id = r.rental_id
LIMIT 5;
```

## Attribution and terms

**Licence, stated as verified rather than as assumed.** The upstream repository's
[`LICENSE.txt`](https://github.com/devrimgunduz/pagila/blob/master/LICENSE.txt) is a
verbatim MIT-style permission grant, copyright Devrim Gunduz, permitting use, copying,
modification, publication and distribution with the notice retained. The upstream
[README](https://github.com/devrimgunduz/pagila) separately says the database "is made
available under PostgreSQL license". Both are permissive and both plainly allow this
redistribution. The discrepancy is upstream's and is recorded here rather than resolved.
GitHub's own licence detector reports `NOASSERTION` for the repository, because the file is
named `LICENSE.txt` and opens with a `# Licence` heading rather than a recognised SPDX
header. No claim is made here beyond what those two upstream files say.

Suggested credit line:

> Source: Pagila (https://github.com/devrimgunduz/pagila), a PostgreSQL port of the MySQL
> Sakila sample database originally developed by Mike Hillyer of the MySQL AB documentation
> team.

Neither Devrim Gunduz, the PostgreSQL project, nor Oracle endorses this repository, this
course, or the University of Illinois.

---

## Repository policy

- **No data file is ever committed to git.** `.gitignore` blocks `*.parquet` and friends.
  Data lives in releases only, so the repository stays small and clones stay fast.
- **Releases are immutable.** If a new month or dataset is needed, cut a new tag. Do not
  replace assets on an existing tag, because that silently breaks every video and script
  pinned to it.

Maintained for BADM 554 by Vishal Sachdev (vishal@illinois.edu).
