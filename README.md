# Potter Airlines Dynamic Pricing System
MMA 2028 Team 6's Python Potter Airlines Project
Team 6: Emma Oh, Kunbo Ma, Rosie Wang, Vaibhav Verma, Yuanyuan Zhong

## 1. Purpose

Potter Airlines is a fictional carrier. This project prices its flights dynamically and reasonably, so two passengers searching for the same flight can see different fares depending on when they book, how full the plane is, and how popular the route is.

The system is a Python application that:

1. Loads fictional flight data from a JSON file and stores it in SQLite.
2. Prices flights with a transparent, rule-based model (Economy and Business).
3. Filters and sort flights with SQLite.
4. Updates and deletes records in the database (either single attribute or object).
5. Validates inputs and pricing results with explicit checks and assertions.

This is a Python programming project, not an optimization model. The pricing coefficients are explainable business assumptions, not values estimated from real sales data.

## 2. Repository structure

```
mma6-potter-airlines/
├── data/
│   └── flights.json            # Generated fictional dataset (1,044 flights)
└── individual-scripts/         # Individual component notebooks
    ├── data_generation.ipynb   # Code that creates flights.json
    ├── connect.ipynb           # SQLite table + parameterized INSERT/SELECT/UPDATE/DELETE
    └── function.ipynb          # Pricing factor functions, validation, and the Flight class
├── .gitignore
├── README.md
├── final_notebook.ipynb        # Main deliverable, the full integrated workflow used for presentation
```

## 3. Setup and how to run

Requirements: Python 3.0, 'pandas', 'sqlite3', 'json', 'datetime‘, 'math', 'interger', 'real'.
No API key is required. The project does not use an LLM or external API.

## 4. Data and database schema

Each flight record has the following fields:

| Field | Meaning |
|---|---|
| `flight_id` | Unique ID (e.g. `PA1169`) |
| `origin`, `destination` | Route (Domestic cities plus international destinations such as Boston, Tokyo)|
| `departure_date` | Departure date in `MM-DD-YYYY` format |
| `days_until_departure` | Days between booking and departure |
| `base_fare` | Route base fare in dollars|
| `seats_remaining`, `capacity` | Seats left and total seats |
| `route_popularity` | Demand signal between 0 and 1 |
| `international` | 1 = international, 0 = domestic |

### Data generation

`individual-scripts/data_generation.ipynb` creates the dataset with Python and `random`, so the data can be recreated from code.

Flight data is synthetically generated from predefined Canadian and international cities grouped by region. 
Each domestic and international route is created in both directions, with two flights per direction. 
Base fare and capacity depend on route distance, and each route has a fixed popularity score. 
Departure dates are random between Nov 1, 2026 and Apr 1, 2027, and the data is exported as JSON.


### Database schema

The SQLite `flights` table enforces the main rules directly in the schema and checks Validity.

SQLite functions includes:

| Operation | Function | Notes |
|---|---|---|
| CREATE | `create_table()` | Drops and recreates the table on each run |
| INSERT | `insert_all_flights()` | `INSERT OR IGNORE` with parameters |
| SELECT | `get_all_flights()`, `get_flight()` | `get_flight` filters by `flight_id = ?` |
| UPDATE | `update_seats()` | Subtracts purchased seats; only succeeds if enough seats remain |
| DELETE | `delete_flight()` | Deletes by `flight_id = ?` |

## 5. Pricing logic

```
price = base_fare × time × demand × capacity × seasonal × class
```

The price is the flight's base fare multiplied by five factors: how soon it departs, how popular the route is, how full the plane is, the season, and the cabin class. Prices go up as departure gets closer, as the route gets more popular, and as the plane fills. The final price is kept between a minimum and maximum set relative to the base fare, so it never gets unreasonable.

The factor values are our own assumptions. The data is synthetic and has no real sales history, so nothing could be estimated from it.

## 6.Design choices

One Flight class holds a flight and calculates its price, with one small method per factor so each can be explained on its own.
Data checks are done both in Python (type, range, and date-format checks) and in the database (CHECK constraints).
All SQL uses ? parameters instead of building strings from input.
Relative price limits are used because base fares vary a lot between short domestic and long international routes.

## 7. Known limitations
The database is rebuilt on every run, so seat updates and deletions do not persist between runs.
days_until_departure is a fixed number in the data. It does not change as time passes.
Prices are recalculated when needed and are not stored in the database.
Price calculation does not account for other factors such as baggage (carry-on/checked-bag), first class, buying with mileage or points, or layovers. 
Step 4 uses input(), so the notebook cannot run fully unattended.
No LLM, GUI, or authentication (all optional).


### Generative AI helped draft the pricing description and generate the city lists used in the data. We tested the code and can explain it.
