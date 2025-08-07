# Import python packages
import streamlit as st
import pandas as pd
import altair as alt
from snowflake.snowpark.context import get_active_session

# App Title
st.title("Revenue Dashboard")

# Get the current credentials
session = get_active_session()

def format_revenue(revenue):
    #return f"₹{revenue / 1_000_000:.1f}M"
    return f"₹{revenue:.1f}"

# Function to alternate row colors
def highlight_rows(row):
    color = '#f2f2f2' if row.name % 2 == 0 else 'white'  # Alternate rows
    return ['background-color: {}'.format(color)] * len(row)

# Function to fetch KPI data from Snowflake
def fetch_kpi_data():
    query = """
    SELECT 
        year,
        total_revenue,
        total_orders,
        avg_revenue_per_order,
        avg_revenue_per_item,
        max_order_value
    FROM sandbox.consumption_sch.vw_yearly_revenue_kpis
    ORDER BY year;
    """
    return session.sql(query).collect()

#TO_CHAR(TO_DATE(month::text, 'MM'), 'Mon') AS month_abbr,  -- Converts month number to abbreviated month name
def fetch_monthly_kpi_data(year):
    query = f"""
    SELECT 
        month::number(2) as month,
        total_revenue::NUMBER(10) AS TOTAL_REVENUE
    FROM 
    sandbox.consumption_sch.vw_monthly_revenue_kpis
    WHERE year = {year}
    ORDER BY month;
    """
    return session.sql(query).collect()


def fetch_unique_months(year):
    query = f"""
    SELECT 
        DISTINCT MONTH FROM 
    sandbox.consumption_sch.vw_monthly_revenue_by_restaurant 
    WHERE YEAR = {year} 
    ORDER BY MONTH;
    """
    return session.sql(query).collect()
    
def fetch_top_restaurants(year, month):
    query = f"""
    SELECT
        restaurant_name,
        total_revenue,
        total_orders,
        avg_revenue_per_order,
        avg_revenue_per_item,
        max_order_value
    FROM
        sandbox.consumption_sch.vw_monthly_revenue_by_restaurant
    WHERE
        YEAR = {year}
        AND MONTH = {month}
    ORDER BY
        total_revenue DESC
    LIMIT 10;
    """
    return session.sql(query).collect()
    
# Function to convert Snowpark DataFrame to Pandas DataFrame
def snowpark_to_pandas(snowpark_df):
    return pd.DataFrame(
        snowpark_df,
        columns=[
            'Restaurant Name',
            'Total Revenue (₹)',
            'Total Orders',
            'Avg Revenue per Order (₹)',
            'Avg Revenue per Item (₹)',
            'Max Order Value (₹)'
        ]
    )
# Fetch data
sf_df = fetch_kpi_data()
df = pd.DataFrame(
    sf_df,
    columns=['YEAR','TOTAL_REVENUE','TOTAL_ORDERS','AVG_REVENUE_PER_ORDER','AVG_REVENUE_PER_ITEM','MAX_ORDER_VALUE']
)

# Aggregate Metrics for All Years
#st.subheader("Aggregate KPIs: Overall Performance")
col1, col2, col3 = st.columns(3)

with col1:
    st.metric("Total Revenue (All Years)", format_revenue(df['TOTAL_REVENUE'].sum()))
with col2:
    st.metric("Total Orders (All Years)", f"{df['TOTAL_ORDERS'].sum():,}")
with col3:
    st.metric("Max Order Value (Overall)", f"₹{df['MAX_ORDER_VALUE'].max():,.0f}")

st.divider()

# Year Selection Box
years = df["YEAR"].unique()
default_year = max(years)  # Select the most recent year by default
selected_year = st.selectbox("Select Year", sorted(years), index=list(years).index(default_year))

# Filter data for selected year
year_data = df[df["YEAR"] == selected_year]
total_revenue = year_data["TOTAL_REVENUE"].iloc[0]
total_orders = year_data["TOTAL_ORDERS"].iloc[0]
avg_revenue_per_order = year_data["AVG_REVENUE_PER_ORDER"].iloc[0]
avg_revenue_per_item = year_data["AVG_REVENUE_PER_ITEM"].iloc[0]
max_order_value = year_data["MAX_ORDER_VALUE"].iloc[0]

# Get previous year data
previous_year = selected_year - 1
previous_year_data = df[df["YEAR"] == previous_year]

# If previous year data exists, calculate differences
if not previous_year_data.empty:
    prev_total_revenue = previous_year_data["TOTAL_REVENUE"].iloc[0]
    prev_total_orders = previous_year_data["TOTAL_ORDERS"].iloc[0]
    prev_avg_revenue_per_order = previous_year_data["AVG_REVENUE_PER_ORDER"].iloc[0]
    prev_avg_revenue_per_item = previous_year_data["AVG_REVENUE_PER_ITEM"].iloc[0]
    prev_max_order_value = previous_year_data["MAX_ORDER_VALUE"].iloc[0]

    # Calculate differences
    revenue_diff = total_revenue - prev_total_revenue
    orders_diff = total_orders - prev_total_orders
    avg_rev_order_diff = avg_revenue_per_order - prev_avg_revenue_per_order
    avg_rev_item_diff = avg_revenue_per_item - prev_avg_revenue_per_item
    max_order_diff = max_order_value - prev_max_order_value
else:
    # If previous year data is not found, set differences to None or zero
    revenue_diff = orders_diff = avg_rev_order_diff = avg_rev_item_diff = max_order_diff = None


# Display Metrics for Selected Year with Comparison to Previous Year
# st.subheader(f"KPI Scorecard for {selected_year}")
col1, col2, col3 = st.columns(3)

with col1:
    st.metric(
        "Total Revenue", 
        format_revenue(total_revenue), 
        delta=f"₹{revenue_diff:.1f}" if revenue_diff is not None else None
    )
    st.metric("Total Orders", f"{total_orders:,}", delta=f"{orders_diff:,}" if orders_diff is not None else None)

    #st.metric("Total Revenue", f"₹{total_revenue:,.0f}", delta=f"₹{revenue_diff:,.0f}" if revenue_diff is not None else None)
    #st.metric("Total Orders", f"{total_orders:,}", delta=f"{orders_diff:,}" if orders_diff is not None else None)

with col2:
    st.metric("Avg Revenue per Order", f"₹{avg_revenue_per_order:,.0f}", delta=f"₹{avg_rev_order_diff:,.0f}" if avg_rev_order_diff is not None else None)
    st.metric("Avg Revenue per Item", f"₹{avg_revenue_per_item:,.0f}", delta=f"₹{avg_rev_item_diff:,.0f}" if avg_rev_item_diff is not None else None)

with col3:
    st.metric("Max Order Value", f"₹{max_order_value:,.0f}", delta=f"₹{max_order_diff:,.0f}" if max_order_diff is not None else None)



st.divider()
# -----------------------------------------


# Fetch and prepare data
month_sf_df = fetch_monthly_kpi_data(selected_year)
month_df = pd.DataFrame(
    month_sf_df,
    columns=['Month', 'Total Monthly Revenue']
)

# Map numeric months to abbreviated month names
month_mapping = {
    1: 'Jan', 2: 'Feb', 3: 'Mar', 4: 'Apr',
    5: 'May', 6: 'Jun', 7: 'Jul', 8: 'Aug',
    9: 'Sep', 10: 'Oct', 11: 'Nov', 12: 'Dec'
}
month_df['Month'] = month_df['Month'].map(month_mapping)

# Ensure months are in the correct chronological order
month_df['Month'] = pd.Categorical(
    month_df['Month'],
    categories=['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
    ordered=True
)
month_df = month_df.sort_values('Month')  # Sort by chronological month order

# Convert revenue to millions
month_df['Total Monthly Revenue'] = month_df['Total Monthly Revenue'] 

# Plot Monthly Revenue Trend using Bar Chart
st.subheader(f"{selected_year} - Monthly Revenue Trend")
# Create the Altair Bar Chart with Custom Color
bar_chart = alt.Chart(month_df).mark_bar(color="#ff5200").encode(
    x=alt.X('Month', sort='ascending', title='Month'),
    y=alt.Y('Total Monthly Revenue', title='Revenue (₹)')
).properties(
    width=700,
    height=400
)

# Display the chart in Streamlit
st.altair_chart(bar_chart, use_container_width=True)

# Add a Trending Chart using Altair
st.subheader(f"{selected_year} - Monthly Revenue Trend")

trend_chart = alt.Chart(month_df).mark_line(color="#ff5200", point=alt.OverlayMarkDef(color="#ff5200")).encode(
    x=alt.X('Month', sort='ascending', title='Month'),
    y=alt.Y('Total Monthly Revenue', title='Revenue (₹)', scale=alt.Scale(domain=[0, month_df['Total Monthly Revenue'].max()])),
    tooltip=[
        alt.Tooltip('Month', title='Month'),
        alt.Tooltip('Total Monthly Revenue', title='Revenue (₹M)', format='.2f')  # Format to 2 decimal places
    ]
).properties(
    width=700,
    height=400
).configure_point(
    size=60
)

st.altair_chart(trend_chart, use_container_width=True)

# Month Selection based on the selected year
if selected_year:

    #get the unique months
    month_sf_df = fetch_unique_months(selected_year)
    print(month_sf_df)
    #convert into df
    month_df = pd.DataFrame(
        month_sf_df,
        columns=['MONTH']
    )
    print(month_df)

    # Year Selection Box
    months = month_df["MONTH"].unique()
    default_month = max(months)  # Select the most recent year by default
    selected_month = st.selectbox(f"Select Month For {selected_year}", sorted(months), index=list(months).index(default_month))

    # Fetch and Display Data
    if selected_month:
        st.subheader(f"Top 10 Restaurants for {selected_month}/{selected_year}")
        top_restaurants = fetch_top_restaurants(selected_year, selected_month)
        if top_restaurants:
            top_restaurants_df = snowpark_to_pandas(top_restaurants)
            # Remove index from DataFrame by resetting it and dropping the index column
            #top_restaurants_df_reset = top_restaurants_df.reset_index(drop=True)

            # Display the DataFrame without index
            #st.dataframe(top_restaurants_df_reset)
            #st.dataframe(top_restaurants_df)

            # Apply the alternate color style
            styled_df = top_restaurants_df.style.apply(highlight_rows, axis=1)

            # Display the styled DataFrame
            st.dataframe(styled_df, hide_index= True)
        else:
            st.warning("No data found for the selected year and month.")
