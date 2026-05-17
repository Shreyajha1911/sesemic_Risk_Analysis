Seismic Risk Intelligence Platform

A production-grade data pipeline built in Apache Spark that answers one business question:

"How should a real estate agency use seismic data to make smarter property acquisition decisions?"

Earthquake exposure is one of the most underestimated risks in real estate — it affects property valuations, insurance costs, and portfolio concentration in ways traditional market analysis overlooks. This platform replaces guesswork with data.

What it does
The pipeline has four stages, each answering a different dimension of the business problem:
StageTechniqueWhat it answersBatch ProcessingRisk Index (40% frequency + 60% severity)Which markets carry the highest seismic risk?Structured Streaming5-min tumbling windows, watermarkingIs seismic activity changing near a property we're evaluating right now?Graph ProcessingPageRank, degree analysis, motif findingAre two properties in our portfolio exposed to the same seismic corridor?Machine LearningK-Means clustering with PCAHow do we group properties into objective risk tiers for insurance and planning?

Key findings

Alaska dominates with a risk index of 0.854 — driven by high-magnitude events, not just frequency
California has ~58% more earthquakes than Alaska but a lower risk index (0.764) — frequency alone is misleading
Alaska's severity score is nearly double California's, meaning per-event damage potential is far higher
K-Means clustering produced 3 distinct risk tiers: Extreme (Tier 1), High (Tier 2), and Moderate (Tier 3) — mapped globally using Plotly
Graph analysis revealed U.S. Virgin Islands has the highest PageRank despite near-zero individual risk — a portfolio concentration insight that batch analysis alone would completely miss


Architecture
USGS API
   │
   ▼
Apache NiFi  ──────────────────────► MinIO (S3-compatible)
(pulls every 5 mins,                  earthquake/ bucket
 JSON Lines format)                        │
                                           ▼
                                    Apache Spark
                                    ├── Batch Processing (Risk Index)
                                    ├── Structured Streaming (Live Windows)
                                    ├── Graph Processing (GraphFrames + PageRank)
                                    └── ML (KMeans + PCA)

Tech stack
Apache Spark Apache NiFi MinIO Python PySpark GraphFrames scikit-learn Plotly Pandas Matplotlib

Setup
1. NiFi flow

Download Earthquake_info_to_MinIO_(S3).json from this repo
Import it into Apache NiFi as a Process Group
Configure AWSCredentialsProviderControllerService with your MinIO credentials

2. MinIO

Create a bucket called earthquake
NiFi will start populating it with JSON files every 5 minutes

3. Run the notebook
bash# Install GraphFrames (first time only, then restart kernel)
pip install graphframes

# Launch Jupyter
jupyter notebook Group_project_final.ipynb

Results & visualisations
The notebook produces 8 visualisations saved as PNG/HTML:

viz1 — Top 20 regions by Risk Index
viz2 — Streaming: live earthquake activity over time
viz3 — Graph: seismic proximity network (node size = PageRank, colour = Risk Index)
viz4 — ML cluster profiles (frequency, magnitude, depth by tier)
viz5 — PageRank vs Risk Index comparison
viz6 — Seismic proximity network diagram
viz7 — Interactive world map coloured by ML risk tier (Plotly HTML)
viz8 — Risk Index composition: frequency vs severity (stacked bar, top 5 regions)


Data source
USGS Earthquake Hazards Program — real-time feed ingested via NiFi every 5 minutes.
No raw data files are committed to this repo. Follow the NiFi setup above to populate your own MinIO bucket.

Repository structure
seismic-risk-pipeline/
├── Group_project_final.ipynb    ← main notebook (all 4 steps)
├── Earthquake_info_to_MinIO.json ← NiFi flow export
├── presentation/
│   └── seismic_risk_deck.pdf    ← project presentation
├── requirements.txt
└── README.md
