I want to create a data hub and dashboarding tool. It is meant for my personal use first and foremost, but will also be published on a public git repo, more as a portfolio thing than as something that will expect external contributions.

What I want to be able to do:
* Collect data from sources
* Create visualizations on this data easily, and pin those visuals to dashboards
	* I'm envisioning using Claude Design to create the visuals templates, before using a coding agent to hook those up to the actual data
==> Those are the 2 main functions I want. There's a 3rd one I'd like:
* A "data lab", in which I can ask natural language questions on some of the data, and an AI agent finds the answers in the data

I want to use the dashboards to get information quickly, at a glance, on certain topics. Some examples:
* Economic indicators, about France / other important countries
* Demographic indicators
* Climate/carbon indicators
* Personal finances indicators

The data will probably come from 2 main types of sources:
* public databases, like the World Bank Databank or EDGAR ==> the collection from these should be automatic
* private sources, like my bank ==> collection from these will be manual = I extract a CSV file or similar into an input folder
Ingestion should occur regularly, probably monthly.

It is important that the following are easy and straightforward:
* Executing ingestion
* Opening dashboards
* Adding a new ingestion source
* Creating a new visual & adding it to a dashboard

I have some ideas about the technical aspects, though everything can be challenged:
* I'd like the data to be stored in DuckDB.
* I'd like dashboards to use duckdb-wasm as their cache, so that dynamic dashboards can slice & dice data near-instantanously
* I don't want to use an existing dashboarding engine (like superset or dash). I'd like dashboards to be html/svg/css/javascript, so that they can be fully created and customized with claude design. 
* I believe that the previous point should also make it possible to serve these dashboards from static storage. This isn't a hard requirement but would be highly preferred, as it would make it quite easy to quickly open them locally, or host them somewhere cheaply.
* I'm curious if there are cheap options to trigger ingestion automatically, as it should be simple pipelines, executed only once a month. Like, would it be doable with github actions?