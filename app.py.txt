import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="NFHS Dashboard", layout="wide")

# ------------------------------
# Load Data
# ------------------------------
@st.cache_data
def load_data():
    df = pd.read_excel("All India National Family Health Survey.xlsx")
    
    # Remove unwanted header row
    df = df[df["STATE"] != "state"]

    # Convert possible numeric columns
    for col in df.columns:
        df[col] = pd.to_numeric(df[col], errors="ignore")
    
    return df

df = load_data()

# ------------------------------
# Sidebar Filters
# ------------------------------
st.sidebar.title("Filters")

selected_round = st.sidebar.selectbox(
    "Select NFHS Round",
    options=df["nfhs"].dropna().unique()
)

filtered_df = df[df["nfhs"] == selected_round]

numeric_cols = filtered_df.select_dtypes(include=['float64', 'int64']).columns

selected_indicator = st.sidebar.selectbox(
    "Select Indicator",
    options=numeric_cols
)

# ------------------------------
# Dashboard Title
# ------------------------------
st.title("📊 National Family Health Survey Dashboard")
st.markdown(f"### NFHS Round: {selected_round}")

# ------------------------------
# KPI Section
# ------------------------------
col1, col2, col3 = st.columns(3)

avg_value = filtered_df[selected_indicator].mean()
max_state = filtered_df.loc[
    filtered_df[selected_indicator].idxmax()
]["STATE"]
min_state = filtered_df.loc[
    filtered_df[selected_indicator].idxmin()
]["STATE"]

col1.metric("Average Value", round(avg_value, 2))
col2.metric("Highest State", max_state)
col3.metric("Lowest State", min_state)

st.markdown("---")

# ------------------------------
# Top 10 States Chart
# ------------------------------
st.subheader("Top 10 States")

top10 = filtered_df.sort_values(
    by=selected_indicator,
    ascending=False
).head(10)

fig1 = px.bar(
    top10,
    x="STATE",
    y=selected_indicator,
    title="Top 10 States",
)

st.plotly_chart(fig1, use_container_width=True)

# ------------------------------
# Distribution Chart
# ------------------------------
st.subheader("Indicator Distribution")

fig2 = px.histogram(
    filtered_df,
    x=selected_indicator,
    nbins=20,
    title="Distribution Across States",
)

st.plotly_chart(fig2, use_container_width=True)

# ------------------------------
# NFHS Round Comparison
# ------------------------------
st.subheader("Round Comparison")

round_comparison = df.groupby("nfhs")[selected_indicator].mean().reset_index()

fig3 = px.line(
    round_comparison,
    x="nfhs",
    y=selected_indicator,
    markers=True,
    title="Average Indicator by NFHS Round"
)

st.plotly_chart(fig3, use_container_width=True)

# ------------------------------
# State Comparison Tool
# ------------------------------
st.subheader("Compare States")

selected_states = st.multiselect(
    "Select States",
    options=filtered_df["STATE"].unique()
)

if selected_states:
    compare_df = filtered_df[
        filtered_df["STATE"].isin(selected_states)
    ]

    fig4 = px.bar(
        compare_df,
        x="STATE",
        y=selected_indicator,
        title="State Comparison"
    )

    st.plotly_chart(fig4, use_container_width=True)

st.markdown("---")
st.caption("Data Source: National Family Health Survey")
