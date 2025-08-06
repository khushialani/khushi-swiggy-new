# SWIGGY Project in Snowflake

Discover how food aggregators and quick-commerce platforms like Swiggy leverage Snowflake to power their data pipelines! This project provides a hands-on tutorial, starting from a high-level food order process flow, and walks you through designing, implementing, and visualizing an end-to-end data pipeline.

---

## Project Overview

This project demonstrates how to:
- Transform real-world data requirements into a robust data pipeline.
- Design a 3-layer data warehouse architecture tailored for food aggregator platforms.
- Build fact and dimension tables to enable analytics.
- Develop an interactive dashboard using Streamlit.

Perfect for beginners or those curious about end-to-end data engineering projects in Snowflake.

---

## Features

1. **High-Level Process Flow**: Understand the food order process and map it to a data flow.
2. **ER Diagram**: Analyze an OLTP source for better data insights.
3. **Snowflake Data Pipeline**:
   - Data loading and curation.
   - Transformation and optimization.
4. **Streamlit Dashboard**: Visualize KPIs and insights from the processed data.

---

## Design Considerations 🏗️

Explore the critical decisions made to:
- Craft the data model.
- Structure the project architecture.
- Tailor solutions for a food aggregator’s data pipeline.

---

## End-To-End Data Flow

### Step 1: Data Loading
- Use Snowflake's Snowsight data loading feature to load CSV files without external tools.
- Run `COPY` commands to load files into tables.

### Step 2: Data Transformation
- Use Snowflake's `$` notation to query staged files.
- Transform delta datasets and curate data layers.

### Step 3: Data Modeling
- Design dimension and fact tables:
  - Dimension tables for customer, restaurant, and order details.
  - Fact tables for transactions and metrics.

### Step 4: Data Visualization
- Build a Streamlit dashboard to:
  - Display KPIs.
  - Provide actionable data insights.

---

## Key Questions Addressed 🤔

- How to load data using Snowflake's Snowsight without tool dependency?
- How to process delta datasets through the data pipeline?
- How to query stage files using `$` notation?
- How to create and populate fact and dimension tables?
- How to build a KPI-driven dashboard using Streamlit?

---

## Prerequisites

1. Snowflake account.
2. Basic knowledge of SQL and data modeling.
3. Python environment with Streamlit installed.

---

## How to Run the Project

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/rahulsql/SWIGGY_Project_In_Snowflake.git
   cd SWIGGY_Project_In_Snowflake
streamlit run dashboard.py

Feel free to customize it further based on your project's specifics! Let me know if you want me to add or refine any section.
