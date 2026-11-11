<div align="center">

<img width="709" height="443" alt="image" src="https://github.com/user-attachments/assets/ca475e8b-be14-4096-980d-2da64890c06e" />

  
# Enterprise Event Analytics

### Event Data Cleaning & Power BI Dashboard | Python · Pandas · DAX · Power BI

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) 
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black) 
![DAX](https://img.shields.io/badge/DAX-KPI%20Measures-5E5E5E?style=for-the-badge) 
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

> A business intelligence project that prepares inconsistent enterprise event ticketing data for analysis and presents sales, revenue, and attendance metrics in a single-page Power BI dashboard.
>
> **Project progress:** Data cleaning and the Power BI dashboard are completed. The Neo4j graph component listed in the original project requirements is future work.


</div>

## 📑 Table of Contents

- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Business Requirements](#business-requirements)
- [Dataset](#dataset)
- [Data Cleaning](#data-cleaning)
- [KPIs & Calculations](#kpis--calculations)
  - [DAX (Power BI)](#dax-power-bi)
- [Dashboard Layout](#dashboard-layout)
- [Analysis Questions](#analysis-questions)
- [Tech Stack](#tech-stack)
- [Skills Demonstrated](#skills-demonstrated)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Data Maintenance](#data-maintenance)
- [Project Status & Next Steps](#project-status--next-steps)
- [Author](#author)
- [License](#license)

---

## Project Overview

Enterprise Event Analytics is a data preparation and business intelligence project for exploring event ticket orders, revenue, attendee profiles, discounts, and gate attendance. The workflow uses Python and Pandas to clean a messy event ledger, then Power BI to visualize the prepared data.

The current deliverables are the documented data-cleaning pipeline and a single-page Power BI dashboard. The project requirements also describe a Neo4j graph model; graph construction and import have not yet been completed.

## Business Problem

Event transaction data may arrive with inconsistent text labels, mixed timestamp formats, currency text inside numeric fields, and multiple representations of check-in status. These issues can make KPI totals unreliable and make it harder to compare events, attendee types, cities, or sales periods.

The dashboard is designed to help answer questions such as:

- How much net revenue and ticket volume have events generated?
- What share of registrations checked in?
- How do sales and revenue change over time?
- Which attendee profiles and cities contribute to event performance?
- How much discount value was applied, and how does it relate to ticket pricing?

## Business Requirements

| # | Requirement | Project status |
| --- | --- | --- |
| 1 | Standardize and prepare event ledger data for analysis | Completed in the documented Pandas cleaning workflow |
| 2 | Present key event KPIs and business visuals in a single-page report | Completed in Power BI |
| 3 | Support analysis by city, attendee profile, and date range | Included in the dashboard view |
| 4 | Enable event/profile performance comparison | Included as a cross-grid/matrix visual |
| 5 | Build attendee/event graph entities and ticket-purchase relationships in Neo4j | Not completed; future work |

## Dataset

**Input file:** `messy_event_ledger_2026.csv`  
**Clean output:** `clean_event_ledger.csv`  
**Input size documented in the cleaning notebook:** 1,520 rows × 11 columns

The ledger fields documented in the project include:

| Field | Description |
| --- | --- |
| `OrderID` | Ticket/order identifier |
| `OrderTimestamp` | Order date and time |
| `EventName` | Event title |
| `EventType` | Event category, such as Tech or Finance |
| `EventCity` | Event location |
| `AttendeeProfile` | Attendee group |
| `BasePrice_INR` | Listed/base ticket price in INR |
| `DiscountAmount_INR` | Discount amount in INR |
| `TicketSellingPrice_INR` | Ticket selling price in INR |
| `NetRevenue_INR` | Net revenue in INR |
| `CheckInStatus` | Attendance/check-in status |

> The dataset itself is not included in this README. Add the source and cleaned CSV files to the repository only if you are permitted to share them.

## Data Cleaning

The cleaning workflow is documented in the provided notebook export and is intended to turn the raw event ledger into a Power BI-ready CSV.

| Issue in source data | Cleaning treatment documented |
| --- | --- |
| Whitespace around order IDs | Trimmed with string whitespace removal |
| Attendee profile capitalization and variants | Standardized to `Employee`, `Entrepreneur`, `Professional`, and `Social Activity` |
| Inconsistent event-type capitalization | Normalized with title case |
| Currency text or formatting in price fields | Removed before conversion to numeric values |
| Ticket selling price and net revenue stored as text | Converted to numeric values; invalid parsing results become missing, then are filled with `0` in the notebook |
| Multiple check-in labels (`Yes`, `Y`, `1`, `No`, `N`, `0`, etc.) | Mapped to `Attended` or `Not Attended` |
| Mixed timestamp formats and `00-00-2026` values | Parsed using mixed-format parsing; invalid/missing values are assigned the fallback date `2026-01-01` |

**Data-quality note:** Filling invalid monetary values with zero and assigning invalid dates a fallback date can affect KPI totals and time-series charts. Review and document these rules before using the report for operational or financial decisions. See [Data Maintenance](#data-maintenance).

## KPIs & Calculations

| KPI | Business logic |
| --- | --- |
| **Total Net Revenue** | Sum of `NetRevenue_INR` |
| **Total Tickets Sold** | Count of unique `OrderID` values, assuming one unique order ID represents one ticket/order |
| **Attendance Rate %** | Attended unique orders ÷ all unique orders in the current filter context |
| **Total Discounts Burnt** | Sum of `DiscountAmount_INR` |
| **Average Ticket Price** | Average of `TicketSellingPrice_INR` |

### DAX (Power BI)

The examples below assume the cleaned table is named `EventSales`. Rename the table/columns in the formulas if your Power BI model uses different names. Create these as **measures** so they respond to report filters.

```DAX
Total Net Revenue =
SUM(EventSales[NetRevenue_INR])
```

```DAX
Total Tickets Sold =
DISTINCTCOUNT(EventSales[OrderID])
```

If the intended definition is the number of data rows (rather than unique order IDs), use `COUNTROWS(EventSales)` instead. Confirm duplicate-order handling before choosing the business definition.

```DAX
Attended Registrations =
CALCULATE(
    DISTINCTCOUNT(EventSales[OrderID]),
    EventSales[CheckInStatus] = "Attended"
)
```

```DAX
Attendance Rate % =
DIVIDE(
    [Attended Registrations],
    [Total Tickets Sold],
    0
)
```

Format `Attendance Rate %` as a percentage in Power BI.

```DAX
Total Discounts Burnt =
SUM(EventSales[DiscountAmount_INR])
```

```DAX
Average Ticket Price =
AVERAGE(EventSales[TicketSellingPrice_INR])
```

Optional measure for the promotional pricing visual:

```DAX
Average Base Price =
AVERAGE(EventSales[BasePrice_INR])
```

For a proper date slicer, create a calendar table, mark it as the date table, and relate it to the cleaned order date:

```DAX
Date =
CALENDAR(
    MIN(EventSales[OrderTimestamp]),
    MAX(EventSales[OrderTimestamp])
)
```

Create a one-to-many relationship from `Date[Date]` to `EventSales[OrderTimestamp]` and use `Date[Date]` for the date slicer and time-based chart axis.

## Dashboard Layout

The Power BI report is a single-page dashboard. Its visible layout includes:

- **KPI cards:** Total Net Revenue, Total Tickets Sold, Attendance Rate, Total Discounts Burnt, and Average Ticket Price.
- **Revenue Velocity:** Revenue trend over time.
- **Audience Valuation Mix:** Revenue contribution by attendee profile.
- **Geospatial Hotspots:** Ticket/revenue distribution by event city.
- **Gate Turnout Logic:** Attended versus not-attended registrations.
- **Performance Cross-Grid:** Event and attendee profile performance matrix.
- **Promo Optimization Metric:** Ticket price and discount relationship.
- **Report filters:** City, attendee profile, and date range.

![Enterprise Event Analytics Power BI dashboard preview](<img width="709" height="443" alt="image" src="https://github.com/user-attachments/assets/55dbaf47-d34a-4213-881d-c10bca86fe1c" />)

## Analysis Questions

The dashboard supports exploration of the following business questions:

- How are revenue and ticket sales trending over the selected period?
- Which attendee profiles contribute the most revenue?
- How do ticket sales or revenue compare across Mumbai, Bengaluru, and Delhi NCR?
- What is the balance between attendees who checked in and those who did not?
- Which events and attendee profiles have stronger revenue performance?
- How do base ticket prices and discounts vary across the selected groups?

The answers depend on the underlying dataset and selected filters; no fixed business findings are asserted here.

## Tech Stack

- **Data preparation:** Python, Pandas, NumPy
- **Notebook environment:** Jupyter Notebook
- **Business intelligence:** Microsoft Power BI
- **Measure language:** DAX
- **Data exchange:** CSV
- **Version control:** Git / GitHub

## Skills Demonstrated

### Data Analytics

- Data profiling and quality review
- Data cleaning and transformation
- Category and date standardization
- Numeric parsing and validation

### Business Intelligence

- KPI definition and DAX measures
- Interactive dashboard design
- Filter and slicer configuration
- Data visualization and report layout

### Data Maintenance

- Refresh and reconciliation checks
- Schema-change awareness
- Documenting transformation assumptions
- Identifying data-quality risks in KPI reporting

## Repository Structure

Suggested structure for the repository as project files are added:

```text
Enterprise-Event-Analytics-Powerbi-Pandas/
├── README.md
├── LICENSE
├── images/
│   └── powerbi-dashboard.png       # Dashboard preview
├── data/
│   ├── raw/
│   │   └── messy_event_ledger_2026.csv
│   └── processed/
│       └── clean_event_ledger.csv
├── notebooks/
│   └── data_cleaning_pipeline.ipynb
└── Dashboard/
    └── Enterprise_Event_Analytics.pbix
```

Only add files that are present and safe to share. The data, notebook, and `.pbix` paths above describe a recommended structure; make sure the files are actually committed before relying on the run steps below.

## How to Run

### 1. Clone the repository

```bash
git clone https://github.com/Ajitjha3095/Enterprise-Event-Analytics-Powerbi-Pandas.git
cd Enterprise-Event-Analytics-Powerbi-Pandas
```

### 2. Prepare the cleaned dataset

If the notebook and raw CSV have been added to the repository, install the Python dependencies and run the notebook:

```bash
pip install pandas numpy jupyter
jupyter notebook notebooks/data_cleaning_pipeline.ipynb
```

Run the notebook using the raw event ledger as input and export the cleaned file as `clean_event_ledger.csv`. Update file paths in the notebook if your repository layout differs.

### 3. Open the Power BI report

If the `.pbix` report has been added, open it in Power BI Desktop. Confirm the data source points to the cleaned CSV, refresh the model, and check the measures and slicers.

### 4. Explore the report

Use the city, attendee profile, and date filters to review the KPIs and visuals. Check that the selected filters update all relevant visuals consistently.

## Data Maintenance

For each data refresh:

1. Preserve the original raw source file and place the approved new version in the designated input folder.
2. Run the cleaning workflow to regenerate the processed CSV; avoid manually editing the processed output.
3. Review input/output row counts, duplicate or blank order IDs, unparsed timestamps, unrecognized categories, and invalid numeric values.
4. Reconcile ticket count, net revenue, discounts, and attendance between the processed file and Power BI.
5. Refresh the Power BI model and test the city, profile, and date filters.
6. Record the source filename, refresh date, row counts, and any changes to cleaning rules or KPI definitions.
7. If the source schema changes, update and test the notebook, Power BI data types, relationships, and DAX measures together.

For recurring use, add automated validation checks for required columns, approved category values, unique order IDs, valid date ranges, and reasonable monetary ranges. Keep a refresh log so changes in totals can be traced to the source data or transformation logic.

## Project Status & Next Steps

**Completed:**

- Data cleaning workflow documented for the event ledger.
- Single-page Power BI dashboard created with five KPI cards, business visuals, and interactive filters.
- DAX examples and refresh guidance documented in this README.

**Future work:**

- Add the Neo4j attendee/event nodes and ticket-purchase relationships specified in the project requirements.
- Add reproducible data-validation tests and a refresh log.
- Review the fallback-date and zero-fill policies with business stakeholders.
- Add the notebook, permitted dataset files, and Power BI report to the repository if they are intended to be shared.

## Author

**Ajit Jha**
*Data Analytics Portfolio Project*

- **GitHub:** [Ajitjha3095](https://github.com/Ajitjha3095)
- **LinkedIn:** [Ajit Jha](https://www.linkedin.com/in/ajitjha01/)

Passionate about building industry-oriented data analytics projects and transforming data into meaningful business insights. Always interested in learning, collaborating, and exploring new opportunities in Data Analytics and BI.

Feel free to connect or reach out for project discussions, collaboration, or professional opportunities!

---

## License

This project repository includes a [MIT License](LICENSE). Refer to the `LICENSE` file for the complete terms.
