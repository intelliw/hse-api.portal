
# Energy API

The `/energy` path provides time-windowed data for four **Energy Types** :`harvest`, `store`, `enjoy`, `grid`. 

The API consolidates data for all four energy types in the requested period (week, month etc.).

API consumers use this data to manage and optimise their energy assets, through graphical views and by scheduling energy use at preferred times (through the `/devices` path).

The following table summarises the four **Energy Types** and examples of their data sources (devices). 

Energy | Assets | Devices
--- | --- | ---
`harvest` | Renewables | PV Modules, Maximum Power Point Trackers (MPPT)
`store` | Storage | Busbar Controllers (BBC), Pack Management Systems (PMS)
`enjoy` | Appliances | Multicore-Cable Current Sensors, Switchboard Clamp Sensors
`grid` | Mains Electricity | Smart Meters, PV Grid-interactive Inverters


### Energy Flows

To restate the Law of Conservation of Energy in this API's terms: Energy flows are based on a "double entry" format where each flow has an equal and opposite flow, expressed with positive and negative values in the data. 

Positive and negative flows in a time period will produce data elements which sum to zero.

The following 6 data elements represent energy flowsin API paths and responses. 

Each data element refers to a subset of the overall energy dataset. 

- `store.in` and `store.out` indicate *charge* and *discharge* flows for batteries.

- `grid.in` and `grid.out` indicate mains use, and feed-in/out flows from the public grid.

- `harvest` refers to renewable energy generation. 

- `enjoy` indicates energy use by end users and appliances. 


# /energy GET
---

The `/energy` path returns a ‘cube’ of energy data for a specified number of periods starting at an epoch. 

It response contains aggregated data for the requested period, and for its child period. For example a request for a *week* period will produce a response with data aggregated for the *week* and for its child period: *day*.

The response will also include links to navigate from the reqested period to adjacent periods (next week, month etc.). 

- /energy/{`energy`}/period/{`period`}/{`epoch`}/{`duration`}?site={`site`}

    e.g. [http://api.sundaya.monitored.equipment/energy/hse/period/week/20150204/1?site=999](http://api.sundaya.monitored.equipment/energy/hse/period/week/20150204/1?site=999 "energy=hse, period=week, duration=1, site=999")

### Path parameters

The following path parameters are required in energy data requests. If a parameter is omitted it will be defaulted as shown. If the paramter was provided it will be validated agains the controlled value lists (enums) specified in the API.     

Parameter | Description | Default
--- | --- | --- 
`energy` | The type of energy flow. | *hse*
`period` | The time window for which total energy is aggregated. The only exception is 'instant' (which is for a single point in time, a millisecond) which is presented without aggregation. | *week*
`epoch` | The starting date and time for the period. | current UTC date-time
`duration` | The number of periods to return starting at epoch. | *1*
`site` | The customer site where energy assets have been installed. | *999*

### period, epoch, duration
- The returned data includes the child and grandchild of the requested `period`. For example if a request is made for a */week* the response will contain total energy for each *day* and a breakdown of energy for each *hour*. 

- The `duration` parameter specifies the third dimension. It returns multiples of the above period data arrays. For example if a request is made for a */week* period with a duration of 3, the response will contain 3 collections of weekly energy data as described above, starting at the requested epoch. 

- `epoch` specifies  the starting date-time of the period. The epoch is displayed in links in the the `href` attribute, after the `period`. 

The following table describes each `period` and formats used for `epoch` in the returned data. 

- The *compressed* format is used in hyperlinks (in the `href` attribute) and the *uncompressed* format is intended for use in display labels.

- An `instant` represents a single point in time (a millisecond). As such there is no data aggregation for `instant`.

- `second` and `minute` periods both include aggregates of any instants in which data was logged.

- `timeofday` refers to 6-hourly blocks of time for Morning, Afternoon, Evening, Night.

Period | Child Period | Duration | Grandchild Period | Duration | Format (*compressed*) | (*uncompressed*)
--- | --- |--- | --- | --- | --- | --- 
`instant` | - | - | - | - | YYYYMMDDTHHmmss.SSS | DD/MM/YY HHmmss.SSSS
`second` | `instant` | *varies* | - | - | YYYYMMDDTHHmmss | DD/MM/YY HHmm:ss
`minute` | `second` | 60 | `instant` | *varies* | YYYYMMDDTHHmm | DD/MM/YY HH:mm
`hour` | `minute` | 60 | `second` | 60 (360) | YYYYMMDDTHHmm | DD/MM/YY HH:mm
`timeofday` | `hour` | 6 | `minute` | 60 (360) | YYYYMMDDTHHmm | DD/MM/YY HH:mm
`day` | `hour` | 4 | `minute` | 60 (240) | YYYYMMDD | DD/MM/YY
`week` | `day` | 7 | `timeofday` | 4 (28) | YYYYMMDD | DD/MM/YY
`month` | `day` | *varies* | `hour` | *varies* | YYYYMMDD | DD/MM/YY
`quarter` | `month` | 3 | `day` | *varies* | YYYYMMDD | DD/MM/YY
`year` | `quarter` | 4 | `month` | *varies* | YYYYMMDD | DD/MM/YY
`fiveyear` | `year` | 5 | `quarter` | 4 (20) | YYYYMMDD | DD/MM/YY

### Query parameters
In all requests the caller must also provide the following query parameters:

Parameter | Description | Default
--- | --- | --- 
`site` | Identifier of the customer site where energy assets have been installed. | *999*

### Body parameter
The `productCatalogItems` optional body parameter specifies a query filter for `/energy` data to be restricted to one or more products. 

The query response will contain data for *any* of the product categories, subcategories, and product types specified in the parameter. 

- If the `productCatalogItems` parameter is missing data will be returned for all products.

- If multiple products are specified data will be returned for *any* of those products.

- If a `productCategory` is specified without a subcategory or product type data will be returned for *all* subcategories and types in that category.

- The same applies if a category and `productSubcategory` is specified without a `productType`.


# /energy GET Response
---

### Link attributes

Response data is returned in a JSON Collection and includes links for the user to navigate to adjacent datasets, for example to the next or previous *week*, or to the previous *month*.

Links have the following attributes:

- **rel**  - The rel property contains the link-to-collection relationship descriptor, which can be one of the following: `self`, `collection`, `up`, `next`, `prev`. These are described below in **Link-relation types** 

- **prompt** - A brief description of the link.

- **title** - The title to display alongside the data for this item, and can be used as a caption or tooltip in the presentation..

- **href** - The URI or URL of the resource.

- **description** - The period or energy descriptor for display by user agents (applications). 

    For "self" links the `description` consists of a list of the resource parameters:
    
    `energy`, `period`, `epoch`, `duration`, `site` 

    e.g:

    -   "rel": "self", "name": "week", "description": "`hse week 20190204 1 999`"

    -   "rel": "collection", "name": "week.day" "description": "`Mon Tue Wed Thu Fri Sat Sun`"

    -   "rel": "collection", "name": "day.timeofday" "description": "`Morning Afternoon Evening Night`"

- **render** - 'image' or 'text' if the link should be retrieved and embedded; or 'link' to display as-is. If the property is missing the href link does not need to be presented.

The following table shows a set of links provided with energy data for a *week* period. 

![Links in energy data](/images/collection-links-table.png)


### Link-relation types

The `rel` attribute of links in `application/vnd.collection+json` responses contain the one of the following registered link-relation types. 

The link-relation typese are based on [RFC8288](https://tools.ietf.org/html/rfc8288#page-6).


- **self**	- Identifies the link's context.

    In `collection.links` it points to the collection as a whole (`name`=*'week'*)            

    e.g. href=[http:/api.sundaya.monitored.equipment/energy/hse/period/week/20190210](http:/api.sundaya.monitored.equipment/energy/hse/period/week/20190210)

    In `collection.items.links` it points to a child item in the collection (`name`=*'day'*).

    e.g. href=[http:/api.sundaya.monitored.equipment/energy/hse/period/day/20190204](http:/api.sundaya.monitored.equipment/energy/hse/period/day/20190204)

- **collection** - in `collection.links` it points to the child items which make up the collection (`name`=*'week.day'*).
    
    e.g. href=[http:/api.sundaya.monitored.equipment/energy/hse/period/day/20190204](http:/api.sundaya.monitored.equipment/energy/hse/period/day/20190204)

- **item** - in `collection.items.links` it points to the subitems of the child item: i.e the grandchild items of the collection (`name`=*'day.hour'*).

    e.g. href=[http:/api.sundaya.monitored.equipment/energy/hse/period/hour/20190205T0600](http:/api.sundaya.monitored.equipment/energy/hse/period/hour/20190205T0600)

- **up** - Identifies the parent of the collection or item (`name`=*'month'* if a link is in collection object for a *'week'*).
    
- **next**, **prev** - Identifies the next or previous sibling of the item series (`name` = *'week'*). The `prompt` and `title` properties signify the next or previous item in the series (`prompt` = *'Week 07 2019'*).


### Accept 'text/html' page format

If the client specifies *text/html* in the request `Accept` header a formatted graph is returned as shown below.

The page displays labels and hyperlink elements from the hypermedia dataset as shown.

![Data element mappings for graph rendering](/images/graph.data-mappings.png)

---

# Visualisations
---
The API is primarily intended to help depict energy flows in graphical views, as time-windowed energy flow from 'sources' at the top, to 'sinks' at the bottom. 

Sources | Sinks    
--- |---
`harvest` `store.out` `grid.out` |`enjoy` `store.in` `grid.in`

For any given time window the net flows from source and sink data elements will sum to zero. 

This is best depicted in a stacked bar graph with up and down bars of the same size, as shown in the following monthly `period` graph.

![Monthly usage example](/images/graph.monthly-usage.png)

**Colours**

The standard colours for each data element type is shown below.

![Colour codes & energy sources](/images/energy.colour-codes.png)

**Top tier**

- _Green_ represents renewable energy generation (`harvest`) and is always shown in the top tier.
- _Black_ in the top tier shows energy drawn from the grid (`grid.out`).
- _Blue_ in the top tier shows energy *discharge* from batteries (`store.out`).

**Bottom tier**

- _Red_ represents energy consumption (`enjoy`) and is always shown in the bottom tier.
- _Black_ in the bottom tier shows a net excess (of `harvest` energy, compared to `store.in` and `enjoy` energy volumes), resulting in feed-in flows to the grid (`grid.in`).
- _Blue_ in the bottom tier shows battery *charge* (`store.in`) again due to a net excess of `harvest` energy.

### Example

A graph with lot of _Black_ in the top tier indicates an opportunity to optimise long term costs.  

In general it indicates a need for more battery capacity and/or `harvest` generation capacity. 

![Stacked bar graph format](/images/graph.stacked-bar-example.png)

The example shows the following behaviour:
- In the 1st hour all `enjoy` energy came from the battery (`store.out`). 
- In the 2nd hour half came from battery and the other half from grid (`grid.out`). 
- In the 3rd hour all came from the grid.
- In the 4th hour the sun starts delivering (`harvest`)
- In the 10th hour harvest data is more than enjoy and the energy flows into store (`store.in`)
 
Note: To query specific assets, API consumers can also filter requests by `category`, `subcategory`, and `productType` (in the request *Body*).
