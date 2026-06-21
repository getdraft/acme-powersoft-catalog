# Onboarding: Tax & Compliance Team

Eleanor -- your repo has two very different systems in it: the modern
Fixed Assets depreciation engine (.NET, SQL Server, fits the schema
cleanly) and the legacy Property Tax PowerBuilder application (on-prem,
manual everything). Model both. For Property Tax specifically: DRAFT
doesn't have a separate object type for "legacy system we're not actively
operating as a scalable service" -- runtime_service is the closest fit.
Use it, answer the architecturalDecisions fields honestly (na/manual is a
valid answer, not a failure), and write up the mismatch as a friction
point rather than silently forcing a better-than-reality answer. That
friction is more useful to the DRAFT maintainers than a clean-looking but
inaccurate record.
