# Onboarding: Asset & Lease Team

Derek -- your repo (`acme-powersoft-asset-lease-product`) is a clean
.NET + Azure SQL + Service Bus shape, very close to Core Platform's. Reuse
the .NET 8 runtime and SQL Server 2022 TechnologyComponents Marcus and
Eleanor already cataloged rather than re-describing your own runtime
stack -- that's the catalog doing its job. One thing to confirm before you
model the Service Bus topic: is it a dedicated namespace for Asset & Lease
events, or could it become a shared messaging backbone other teams should
also use? Decide and document either way.
