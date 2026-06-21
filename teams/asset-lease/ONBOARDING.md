# Onboarding: Asset & Lease Team

Derek -- your repo (`acme-powersoft-asset-lease-product`) is a clean
.NET + Azure SQL + Service Bus shape, very close to Core Platform's. Reuse
the .NET 8 runtime and SQL Server 2022 TechnologyComponents Marcus and
Eleanor already cataloged rather than re-describing your own runtime
stack -- that's the catalog doing its job. One thing to confirm before you
model the Service Bus topic: is it a dedicated namespace for Asset & Lease
events, or could it become a shared messaging backbone other teams should
also use? Decide and document either way.

## Admin follow-up

Confirming both questions, Derek:

1. Yes -- cross-team TechnologyComponent reuse by uid is the intended
   pattern. A duplicate record per team would make the catalog lie about
   how many distinct runtime stacks the company actually operates.
2. On the Service Bus topic: keep it dedicated to Asset & Lease events for
   now. Don't promote it to a shared messaging backbone speculatively --
   we already have Sofia's rs-messaging-backbone for that, and a second
   "shared" bus with unclear ownership would be worse than two clearly-
   scoped dedicated ones. If a real cross-team need shows up later, route
   it through Sofia's shared instance rather than widening yours.
