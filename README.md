World Bank Data in Julia
========================

The World Bank provides access to global development data at
[data.worldbank.org](https://data.worldbank.org).

The primary collection of development indicators is called World
Development Indicators (WDI).

This module provides two functions to access and download the data:
`search_wdi()` and `wdi()`. These functions return
[DataFrames](https://github.com/JuliaData/DataFrames.jl).

It follows roughly the
[R WDI package](https://cran.r-project.org/web/packages/WDI/index.html).

## Build Status

[![GitHub Build Status](https://github.com/4gh/WorldBankData.jl/workflows/WorldBankData.jl%20Continous%20Integration/badge.svg)](https://github.com/4gh/WorldBankData.jl/actions?query=workflow%3A%22WorldBankData.jl+Continous+Integration%22)

## Installation

```julia
using Pkg
Pkg.add("WorldBankData")
```

## Basic Examples

Get a DataFrame of the U.S. population:
```julia
julia> using WorldBankData
julia> df = wdi("SP.POP.TOTL", "US")
60×4 DataFrame
│ Row │ iso2c  │ country       │ year    │ SP_POP_TOTL │
│     │ String │ String        │ Float64 │ Float64?    │
├─────┼────────┼───────────────┼─────────┼─────────────┤
│ 1   │ US     │ United States │ 1960.0  │ 1.80671e8   │
│ 2   │ US     │ United States │ 1961.0  │ 1.83691e8   │
⋮
│ 59  │ US     │ United States │ 2018.0  │ 3.26688e8   │
│ 60  │ US     │ United States │ 2019.0  │ 3.2824e8    │
```

The WDI indicator `SP.POP.TOTL` becomes the column `SP_POP_TOTL` in the
DataFrame, i.e. `.` gets replaced by `_`.

Get a DataFrame of the U.S. population from 1980 until 2012 including region data:
```julia
julia> df = wdi("SP.POP.TOTL", "US", 1980, 2012, extra=true)
33×12 DataFrame. Omitted printing of 5 columns
│ Row │ iso2c  │ country       │ SP_POP_TOTL │ year    │ iso3c  │ name          │ region        │
│     │ String │ String        │ Float64?    │ Float64 │ String │ String        │ String        │
├─────┼────────┼───────────────┼─────────────┼─────────┼────────┼───────────────┼───────────────┤
│ 1   │ US     │ United States │ 2.27225e8   │ 1980.0  │ USA    │ United States │ North America │
│ 2   │ US     │ United States │ 2.29466e8   │ 1981.0  │ USA    │ United States │ North America │
⋮
│ 32  │ US     │ United States │ 3.11557e8   │ 2011.0  │ USA    │ United States │ North America │
│ 33  │ US     │ United States │ 3.13831e8   │ 2012.0  │ USA    │ United States │ North America │
```

ISO 3 letter country codes are also supported:
```julia
df = wdi("SP.POP.TOTL", "USA", 1980, 2012)
```

Multiple indicators and countries can be requested:
```julia
julia> df = wdi(["SP.POP.TOTL", "NY.GDP.MKTP.CD"], ["US","BR"], 1980, 2012)
66×5 DataFrame
│ Row │ iso2c  │ country       │ year    │ NY_GDP_MKTP_CD │ SP_POP_TOTL │
│     │ String │ String        │ Float64 │ Float64?       │ Float64?    │
├─────┼────────┼───────────────┼─────────┼────────────────┼─────────────┤
│ 1   │ BR     │ Brazil        │ 1980.0  │ 2.35025e11     │ 1.20694e8   │
│ 2   │ BR     │ Brazil        │ 1981.0  │ 2.63561e11     │ 1.2357e8    │
⋮
│ 65  │ US     │ United States │ 2011.0  │ 1.55426e13     │ 3.11557e8   │
│ 66  │ US     │ United States │ 2012.0  │ 1.6197e13      │ 3.13831e8   │
```

By default a wide DataFrame is returned (indicators are columns). The data can also be returned in long format which might be more useful if many indicators are requested:
```julia
julia> df = wdi(["SP.POP.TOTL", "NY.GDP.MKTP.CD"], ["US","BR"], 1980, 2012, dflong=true)
132×5 DataFrame
│ Row │ iso2c  │ country       │ year    │ indicator      │ value      │
│     │ String │ String        │ Float64 │ String         │ Float64?   │
├─────┼────────┼───────────────┼─────────┼────────────────┼────────────┤
│ 1   │ BR     │ Brazil        │ 1980.0  │ SP.POP.TOTL    │ 1.20694e8  │
│ 2   │ BR     │ Brazil        │ 1980.0  │ NY.GDP.MKTP.CD │ 2.35025e11 │
⋮
│ 131 │ US     │ United States │ 2012.0  │ SP.POP.TOTL    │ 3.13831e8  │
│ 132 │ US     │ United States │ 2012.0  │ NY.GDP.MKTP.CD │ 1.6197e13  │
```

Get a DataFrame of the total population for all countries in 2000:

```julia
using WorldBankData
df = wdi("SP.POP.TOTL", "all", 1980, 2012)
```

## Arguments

The `wdi` function has the following arguments:

```julia
function wdi(indicators::Union{String,Array{String,1}}, countries::Union{String,Array{String,1}},
             startyear::Integer=-1, endyear::Integer=-1;
             extra::Bool=false, sourceid::Integer=2, dflong::Bool=false, verbose::Bool=false)::DataFrame
```

It needs a minimum of two arguments: the `indicators` (from the WDI
database) and the `countries` (ISO two or three letter country codes or `"all"` for all countries). The rest
are optional arguments.

`startyear`: First year to include.

`endyear`: Last year to include.

`extra`: If `extra=true`, `wdi()` attaches extra country data to the returned DataFrame.

`sourceid`: Database source number, see https://api.worldbank.org/v2/sources

`dflong`: If `dflong=true` return long DataFrame format. Default is wide. If many indicators are requested `long` might be preferable.

`verbose`: If `verbose=true`, `wdi()` will print URL download information.

## Searching

The most convenient way to explore the database is probably through a
web browser at [data.worldbank.org](https://data.worldbank.org).

However, the module does provide a search function: `search_wdi()`.

One can search for "countries" or "indicators" data.

### Example for country search by name

```julia
julia> using WorldBankData
julia> res=search_wdi("countries","name",r"united"i)
julia> res[!, :name]
3-element DataArray{UTF8String,1}:
 "United Arab Emirates"
 "United Kingdom"
 "United States"
julia> res[!, :iso2c]
3-element DataArray{ASCIIString,1}:
 "AE"
 "GB"
 "US"
```

### Example for indicator search by description

```julia
julia> using WorldBankData
julia> res=search_wdi("indicators","description",r"gross national expenditure"i)
6x5 DataFrame
...
julia> res[!, :name]
6-element DataArray{UTF8String,1}:
 "Gross national expenditure deflator (base year varies by country)"
 "Gross national expenditure (current US\$)"
 "Gross national expenditure (current LCU)"
 "Gross national expenditure (constant 2005 US\$)"
 "Gross national expenditure (constant LCU)"
 "Gross national expenditure (% of GDP)"

julia> res[!, :indicator]
6-element DataArray{UTF8String,1}:
 "NE.DAB.DEFL.ZS"
 "NE.DAB.TOTL.CD"
 "NE.DAB.TOTL.CN"
 "NE.DAB.TOTL.KD"
 "NE.DAB.TOTL.KN"
 "NE.DAB.TOTL.ZS"
```

### The search_wdi() function

The `search_wdi()` function has the following arguments
```julia
search_wdi(data::String, entry::String, regx::Regex)::DataFrame
```

`data`: Either `countries` or `indicators`.

`entry`: One of the attributes (like name).

`regex`: Regular expression to search for.

"countries" can be searched for "name", "region", "capital", "iso2c",
"iso3c", "income", and "lending".

"indicators" can be searched for "name", "description", "topics",
"source_database", and "source_organization".

The search function uses two DataFrames `country_cache` and
`indicator_cache` and searches through these. On the first search it
will download the data from the World Bank website. This takes much
longer for the larger indicators data. This only happens once per
session. After the first use the data is cached.

Note that the last argument to `search_wdi()` is a regular expression
denoted by `r"..."` and an `i` at the end means that it is case
insensitive.

### Examples of country searches

```julia
julia> search_wdi("countries","iso2c",r"TZ"i)
1×9 DataFrame. Omitted printing of 2 columns
│ Row │ iso3c  │ iso2c  │ name     │ region              │ capital │ longitude │ latitude │
│     │ String │ String │ String   │ String              │ String  │ Float64?  │ Float64? │
├─────┼────────┼────────┼──────────┼─────────────────────┼─────────┼───────────┼──────────┤
│ 1   │ TZA    │ TZ     │ Tanzania │ Sub-Saharan Africa  │ Dodoma  │ 35.7382   │ -6.17486 │

julia> search_wdi("countries","income",r"upper middle"i)
...

julia> search_wdi("countries","region",r"Latin America"i)
...

julia> search_wdi("countries","capital",r"^Ka"i)
3×9 DataFrame. Omitted printing of 2 columns
│ Row │ iso3c  │ iso2c  │ name        │ region              │ capital   │ longitude │ latitude │
│     │ String │ String │ String      │ String              │ String    │ Float64?  │ Float64? │
├─────┼────────┼────────┼─────────────┼─────────────────────┼───────────┼───────────┼──────────┤
│ 1   │ AFG    │ AF     │ Afghanistan │ South Asia          │ Kabul     │ 69.1761   │ 34.5228  │
│ 2   │ NPL    │ NP     │ Nepal       │ South Asia          │ Kathmandu │ 85.3157   │ 27.6939  │
│ 3   │ UGA    │ UG     │ Uganda      │ Sub-Saharan Africa  │ Kampala   │ 32.5729   │ 0.314269 │

julia> search_wdi("countries","lending",r"IBRD"i)
...

```

### Examples of indicator searches

```julia
julia> search_wdi("indicators","name",r"gross national expenditure"i)
...
julia> search_wdi("indicators","description",r"gross national expenditure"i)
...
julia> search_wdi("indicators","source_database",r"Sustainable"i)
...
julia> search_wdi("indicators","source_organization",r"Global Partnership"i)

```

## Tips and Tricks

### Extracting country data from results

```julia
df = wdi("SP.POP.TOTL", ["US","BR"], 1980, 2012, extra=true)
us_pop = df[df[!, :iso2c] .== "US", :]
```

### Year format

For similarity with the
[R WDI package](https://cran.r-project.org/web/packages/WDI/index.html) the `:year`
column is in Float64 format. WDI data is yearly.

You can easily convert this to a Date series:
```julia
using WorldBankData
using Dates

df = wdi("AG.LND.ARBL.HA.PC", "US", 1900, 2011)
df[!, :year] = map(Date, df[!, :year])
```

### Plotting

Install the [StatsPlots.jl](https://github.com/JuliaPlots/StatsPlots.jl) package with
`import Pkg; Pkg.add("StatsPlots")`.

```julia
using WorldBankData
using StatsPlots
gr(size=(400,300))

df = wdi("SP.POP.TOTL", "US", 1980, 2010)

@df df scatter(:year, :SP_POP_TOTL)
```

### Empty/Missing results

`wdi` will return a DataFrame with missing values if there is no data:
```julia
julia> df = wdi("EN.ATM.CO2E.KT", "AS")
60×4 DataFrames.DataFrame
│ Row │ iso2c  │ country        │ EN_ATM_CO2E_KT │ year    │
│     │ String │ String         │ Missing        │ Float64 │
├─────┼────────┼────────────────┼────────────────┼─────────┤
│ 1   │ AS     │ American Samoa │ missing        │ 1960.0  │
│ 2   │ AS     │ American Samoa │ missing        │ 1961.0  │
│ 3   │ AS     │ American Samoa │ missing        │ 1962.0  │
...

julia> df = wdi("EN.ATM.CO2E.KT", ["AS","US"])
120×4 DataFrames.DataFrame
│ Row │ iso2c  │ country        │ EN_ATM_CO2E_KT │ year    │
│     │ String │ String         │ Float64⍰       │ Float64 │
├─────┼────────┼────────────────┼────────────────┼─────────┤
│ 1   │ AS     │ American Samoa │ missing        │ 1960.0  │
│ 2   │ AS     │ American Samoa │ missing        │ 1961.0  │
...
│ 114 │ US     │ United States  │ 5.15916e6      │ 2013.0  │
│ 115 │ US     │ United States  │ 5.25428e6      │ 2014.0  │
│ 116 │ US     │ United States  │ missing        │ 2015.0  │
│ 120 │ US     │ United States  │ missing        │ 2019.0  │
...
```

### Cache

The data in the World Bank database changes infrequently. Therefore it
makes little sense to download it every time a script is run.

#### Metadata

The `search_wdi()` function internally caches the country and
indicator metadata and therefore downloads the country and indicator
data only once per session. Even that is usually not necessary. This
data can easily be stored on local disk.

Download and store the country and indicator information in csv files:

```julia
using WorldBankData
using DataFrames
using CSV
CSV.write("country_cache.csv",WorldBankData.get_countries())
CSV.write("indicator_cache.csv", WorldBankData.get_indicators())
```

These can be used in the script to set the WorldBankData cache
variables `WorldBankData.country_cache` and
`WorldBankData.indicator_cache` (which are initialized to `false`)
using the `WorldBankData.set_country_cache()` and
`WorldBankData.set_indicator_cache()` functions:

```julia
using WorldBankData
using DataFrames
WorldBankData.set_country_cache(CSV.read("country_cache.csv"))
WorldBankData.set_indicator_cache(CSV.read("indicator_cache.csv"))
```
From then on the `search_wdi()` function will use the data read from
disk.

The caches can be reset with `WorldBankData.reset_country_cache()`
and `WorldBankData.reset_indicator_cache()`.

#### Indicator data

In a similar way the indicator data itself can be cached.

```julia
using WorldBankData
using DataFrames
using CSV

function update_us_pop_totl()
    df = wdi("SP.POP.TOTL", "US")
    CSV.write("us_pop_totl.csv",df)
end
df = CSV.read("us_pop_totl.csv")
```
Occasionally update the data by running the `update_us_pop_totl()` function.

