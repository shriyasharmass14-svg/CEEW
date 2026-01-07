# ============================
# IMPORT LIBRARIES
# ============================
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

# ============================
# LOAD DATA
# ============================
# Replace with your actual file name
df = pd.read_excel("opportunity_matrix.xlsx")

# ============================
# CALCULATE AGGREGATES
# ============================
df["Avg_Awareness"] = df["Awareness"]

df["Avg_Intent"] = df[["Assurance", "Accessibility"]].mean(axis=1)

df["Avg_Constraint"] = df[["Affordability", "After_sales"]].mean(axis=1)

df["Opportunity_Index"] = (df["Avg_Intent"] * df["Avg_Awareness"]) / df["Avg_Constraint"]

# ============================
# GRAPH 1: OPPORTUNITY INDEX HEATMAP
# ============================
fig1 = px.density_heatmap(
    df,
    x="Technology",
    y="Location",
    z="Opportunity_Index",
    color_continuous_scale="RdYlGn",
    title="Opportunity Index by Technology and Location"
)
fig1.show()

# ============================
# GRAPH 2: BIGGEST DETERRENTS (PILLAR SCORES)
# ============================
pillar_avg = df[[
    "Awareness",
    "Assurance",
    "Accessibility",
    "Affordability",
    "After_sales"
]].mean().reset_index()

pillar_avg.columns = ["Pillar", "Average_Score"]

fig2 = px.bar(
    pillar_avg.sort_values("Average_Score"),
    x="Average_Score",
    y="Pillar",
    orientation="h",
    color="Average_Score",
    color_continuous_scale="RdYlGn",
    title="Biggest Deterrents to Adoption (Lower Score = Bigger Problem)"
)
fig2.show()

# ============================
# GRAPH 3: STAKEHOLDER-WISE GAP ANALYSIS
# ============================
stakeholder_summary = (
    df.groupby("Stakeholder")[[
        "Awareness",
        "Assurance",
        "Accessibility",
        "Affordability",
        "After_sales"
    ]]
    .mean()
)

fig3 = px.imshow(
    stakeholder_summary,
    color_continuous_scale="RdYlGn",
    aspect="auto",
    title="Stakeholder-wise Adoption Gaps"
)
fig3.show()

# ============================
# GRAPH 4: FUNNEL DROP-OFF
# ============================
funnel_df = pd.DataFrame({
    "Stage": ["Awareness", "Intent", "Constraint"],
    "Score": [
        df["Avg_Awareness"].mean(),
        df["Avg_Intent"].mean(),
        df["Avg_Constraint"].mean()
    ]
})

fig4 = px.bar(
    funnel_df,
    x="Stage",
    y="Score",
    color="Score",
    color_continuous_scale="RdYlGn",
    title="Adoption Funnel: Where Drop-offs Occur"
)
fig4.show()

# ============================
# GRAPH 5: WHAT IS WORKING VS WHAT IS BLOCKING (RADAR)
# ============================
radar_labels = [
    "Awareness",
    "Assurance",
    "Accessibility",
    "Affordability",
    "After-sales"
]

radar_values = [
    df["Awareness"].mean(),
    df["Assurance"].mean(),
    df["Accessibility"].mean(),
    df["Affordability"].mean(),
    df["After_sales"].mean()
]

fig5 = go.Figure()

fig5.add_trace(go.Scatterpolar(
    r=radar_values,
    theta=radar_labels,
    fill="toself",
    name="Current State"
))

fig5.update_layout(
    polar=dict(radialaxis=dict(visible=True, range=[0,5])),
    title="What Is Working vs What Needs Intervention"
)

fig5.show()
