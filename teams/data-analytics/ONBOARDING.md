# Onboarding: Data & Analytics Team

Jamal -- Insight Hub is the one product whose deployment target isn't a
simple AKS pod set: it's an Airflow scheduler on AKS orchestrating DAGs
that move data through the landing zone you already modeled in Phase 4
into Snowflake, plus the churn model training job. Catalog the Airflow
runtime itself, and build the SDP to show the ELT path end-to-end.
