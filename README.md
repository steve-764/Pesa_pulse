# Customer Data Analysis

A small Python program that fetches customer records from a public API, flattens them, counts customers by city and by email domain, cross-checks the counts with pandas, and saves CSV files and charts.

Data source: <https://jsonplaceholder.typicode.com/users> (10 sample customers).

## Approach

The program is a pipeline. Each step is one function and uses the output of the one before it.

| Step | Function | What it does |
|------|----------|--------------|
| 1 | `get_customers()` | Downloads the customer list with `requests`. Handles timeouts, connection errors, bad HTTP status and invalid JSON by printing a clear message and returning an empty list instead of crashing. |
| 2 | `explore(record)` | Recursively prints every key in a nested record with its type, indented by depth, to understand the structure. |
| 3 | `extract_attributes(customers)` | Turns each nested record into one flat dict with: id, name, username, email, email_domain, phone, city, zipcode, latitude, longitude, company, website. Latitude and longitude are converted to floats. |
| 4 | `unique_cities(customers)` | Uses a set comprehension to find the distinct cities. |
| 5 | `city_counts()` / `email_domain_counts()` | Count customers per city (plain loop, no `Counter`) and per email top-level domain (dict comprehension). Printed from most to least common. |
| 6 | `to_dataframe(rows)` | Builds a pandas DataFrame, prints its shape and dtypes, and checks that `value_counts()` gives the same numbers as the hand-written counts. |
| 7 | `export_results()` | Writes the two CSV files, without the pandas index. |
| 8 | `plot_distribution()` | Saves bar charts of the city and email domain distributions. |
| 9 | `main()` | Runs everything in order and prints a short findings summary. |

## How to run

**Requirements:** Python 3.10 or newer, and an internet connection (the data is downloaded each time).

1. Open a terminal in the project folder.

2. (Optional) Create and activate a virtual environment:


3. Install the dependencies based on the requirements.txt file:

   ```bash
   pip install -r requirements.txt   
   ```

4. Run the script (replace `analysis.ipynb` with the name of your file):

   ```bash
   python analysis.ipynb
   ```

The program prints its results to the terminal and creates an `output/` folder next to where you run it (it is created automatically if it doesn't exist).

### Output files

| File | Contents |
|------|----------|
| `output/customers.csv` | One row per customer with the 12 extracted attributes. |
| `output/city_counts.csv` | Columns `city`, `count`, sorted from most to least common. |
| `output/customer_distribution.png` | Bar chart of customers per city. |
| `output/email_domain_distribution.png` | Bar chart of customers per email top-level domain. |

## Findings

1. **Customers are spread evenly across cities.** There are 10 customers and 10 different cities, so no city has more than one customer. City-based campaigns would each reach a single person, so a regional or national approach makes more sense.

2. **Email domains show more variation.** Counting by top-level domain (the part after the last dot), `.biz` is the most common with 3 customers, and the other 7 customers are spread across `.org`, `.tv`, `.net`, `.ca`, `.info`, `.me` and `.io`, one each. This is the only grouping in the data where one group stands out.

3. **The hand-written counts and pandas agree.** The city and email-domain counts from the loops and comprehensions match `value_counts()` exactly veryfying that the extraction and counting logic is correct.


## Assumptions

- **Source data:** the API returns a JSON list of customer records in the structure of the JSONPlaceholder users endpoint (nested `address`, `address.geo` and `company` objects).
- **"Company"** means the company's `name`, not its catch phrase or `bs` text.
- **"Email domain"** is everything after the `@` (for example `april.biz`). For the counts, "domain" means the **top-level domain** only (`biz`), taken after the last dot, because every full domain in this data is unique.
- **Missing data:** if a field is missing in a record, the extracted value is `None` rather than the program crashing. Rows with no city are skipped when counting cities.
- **Failed download:** if the API can't be reached, the program prints an error and stops without writing any files.
- **Coordinates:** latitude and longitude arrive as strings and are converted to floats. A value that can't be converted becomes `None`.
- **Case and spacing:** city names are compared exactly as given. This data is considered clean, so no normalizing is done.
- **CSV columns:** the column names and order in the CSV files follow the attribute list in the task description 
- **File and folder names:** the script is run from the project folder, and `output/` is relative to wherever you run it.
