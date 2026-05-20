# provider/v1

Logistics provider attributes. Attaches to `beckn:Provider.providerAttributes`.

## What it covers
Identity and licences (NIB, SIUP, ALFI, ASPERINDO), provider model (carrier vs forwarder vs aggregator vs 3PL/4PL vs warehouse), patterns and modes supported, geographic coverage, capabilities (cold chain, hazmat, high-value, customs brokerage), operating hours, fleet type, warehouse locations.

## Mandatory fields (always)
`nibRegistered`, `providerCategory`, `patternsSupported`, `modesSupported`, `coverage[]`, `operatingHours[]`

## Conditional fields
`siupNumber` (freight and warehouse), `beaCukaiLicenceNumber` (cross-border providers), `coldChainTemperatureZones` (if coldChainCertified=true), `hazmatClasses` (if hazmatCertified=true), `warehouseLocations[]` (warehouse providers), `maxDeliveryRadiusKm` (hyperlocal providers).

## Used in patterns
All logistics patterns. Fields applicable per pattern are documented in each pattern's conditional field list.
