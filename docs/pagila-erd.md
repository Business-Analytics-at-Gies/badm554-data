<!-- audience: external -->
# Pagila schema diagram (ERD)

This is the entity-relationship diagram for the film-rental source used in Module 5,
generated from the pinned `pagila.duckdb` release asset itself
(release [`pagila-3.0.0`](https://github.com/Business-Analytics-at-Gies/badm554-data/releases/tag/pagila-3.0.0),
SHA-256 `9deae7bf23f5e73aece8d9afa79b3772ac312f0e3b9db528dfdd127bdb5c3453`), so every
table and column below exists in the file you download. All fifteen tables are shown.
`PK` marks a primary key, `FK` a foreign key. Crow's-foot notation: the `||` side is
the "one", the `{` side is the "many".

## How to view it

- **Right here on GitHub.** GitHub renders the diagram below natively. No install.
- **In VS Code.** Clone this repository and open this file, then install the
  *Markdown Preview Mermaid Support* extension (`bierner.markdown-mermaid`) and press
  `Cmd+Shift+V` (macOS) or `Ctrl+Shift+V` (Windows) for the rendered preview:

  ```bash
  git clone https://github.com/Business-Analytics-at-Gies/badm554-data.git
  cd badm554-data
  code docs/pagila-erd.md
  ```

Reading tip: start at `rental`, in the middle of the action. A rental does not point at
a film. It points at the physical copy (`inventory`), and the copy points at the film.
That two-hop path is the join the schema exists to teach.

One data note repeated from the main README: this build ships 1,500 staff rows and 500
stores, not the two and two of the original Sakila. Write queries accordingly.

```mermaid
erDiagram
    actor {
        int actor_id PK
        varchar first_name
        varchar last_name
        timestamptz last_update
    }
    address {
        int address_id PK
        varchar address
        varchar address2
        varchar district
        int city_id FK
        varchar postal_code
        varchar phone
        timestamptz last_update
    }
    category {
        int category_id PK
        varchar name
        timestamptz last_update
    }
    city {
        int city_id PK
        varchar city
        int country_id FK
        timestamptz last_update
    }
    country {
        int country_id PK
        varchar country
        timestamptz last_update
    }
    customer {
        int customer_id PK
        int store_id FK
        varchar first_name
        varchar last_name
        varchar email
        int address_id FK
        boolean activebool
        date create_date
        timestamptz last_update
        int active
    }
    film {
        int film_id PK
        varchar title
        varchar description
        varchar release_year
        int language_id FK
        int original_language_id FK
        smallint rental_duration
        decimal rental_rate
        smallint length
        decimal replacement_cost
        enum rating
        timestamptz last_update
        varchar_array special_features
        varchar fulltext
    }
    film_actor {
        int actor_id FK
        int film_id FK
        timestamptz last_update
    }
    film_category {
        int film_id FK
        int category_id FK
        timestamptz last_update
    }
    inventory {
        int inventory_id PK
        int film_id FK
        int store_id FK
        timestamptz last_update
    }
    language {
        int language_id PK
        varchar name
        timestamptz last_update
    }
    payment {
        int payment_id PK
        int customer_id FK
        int staff_id FK
        int rental_id FK
        decimal amount
        timestamptz payment_date
    }
    rental {
        int rental_id PK
        timestamptz rental_date
        int inventory_id FK
        int customer_id FK
        timestamptz return_date
        int staff_id FK
        timestamptz last_update
    }
    staff {
        int staff_id PK
        varchar first_name
        varchar last_name
        int address_id FK
        varchar email
        int store_id FK
        boolean active
        varchar username
        varchar password
        timestamptz last_update
        blob picture
    }
    store {
        int store_id PK
        int manager_staff_id FK
        int address_id FK
        timestamptz last_update
    }
    city ||--|{ address : "is in"
    country ||--|{ city : "is in"
    address ||--|{ customer : "lives at"
    store ||--|{ customer : "signed up at"
    language ||--|{ film : "spoken in"
    language ||--o{ film : "originally in"
    film ||--|{ film_actor : "casts"
    actor ||--|{ film_actor : "features"
    film ||--|{ film_category : "classifies"
    category ||--|{ film_category : "labels"
    film ||--|{ inventory : "is a copy of"
    store ||--|{ inventory : "sits at"
    inventory ||--|{ rental : "checks out"
    customer ||--|{ rental : "rented by"
    staff ||--|{ rental : "handled by"
    rental ||--|{ payment : "pays for"
    customer ||--|{ payment : "paid by"
    staff ||--|{ payment : "taken by"
    address ||--|{ staff : "lives at"
    store ||--|{ staff : "works at"
    staff ||--|{ store : "managed by"
    address ||--|{ store : "located at"
```
