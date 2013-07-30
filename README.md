gtfs-internationalization
=======

*Introduction*

GTFS offers a powerful framework for defining transit service data feeds. However, at its core is a stop-centric model of transport services which is not appropriate for many of the world's transit networks, particularly those in developing cities. 

The current specification is assumes that passengers board and alight only at defined stops, at precise times (defined by a timetable) or at regular intervals (defined by service headways). 

In actuality, a substantial percentage of the world's population [reference research on total numbers here?] depend on transit systems that do not have formal stops. On these systems -- microbuses, collectivos, matatus, etc. -- passengers are free to board vehicles anywhere along the route. Similarly, many of these systems depend on independent driver/operators that determine their own working schedules or are dispatched by a variety of different mechanisms. As a result route headway and journey times may vary substantially depending on a number of factors including traffic, weather, vehicle availability or demand. 

Based on field data collection efforts in Mexico City and Manila -- both cities that operate substantial non-stop-based transit networks -- our belief is that GTFS as it currently stands is an overly precise framework for defining these systems. However, with the minor modifications proposed here -- modifications that leave the fundamental structure of GTFS intact -- we can accurately represent non-stop based system.

Additionally we recommend changes that improve the way journey times and headway are encoded, as well as localization of route types.

*Proposed Changes*

### Interpolating Stops

We recommend the inclusions of a "interpolate_stops" flag in the trips.txt that allows a feed to define a limited number of stops along a trip while interpolating intermediate boarding/alighting points between these stops at a specified frequency along the trip shape. The interpolate_stops flag when set specifies the average distance in meters between interpolated points. A well defined shape is required for any trip that is to be interpolated.

The implementation details aren't well defined at this stage, however, in our prototyping to-date we have used the following values:

**interpolate_stops**
-1   Do not interpolate stops
0     Allow boarding/alighting at any point along the trip shape (exact implementation will be driven by each GTFS consumer)
>0   Allow boarding/alighting at every x meters (exact placement is "best effort" as determined by GTFS consumer)

When GTFS is consumed by routing engines additional stopping points can be placed along the trip shape at or close to the specified frequency. These points would be treated as non-timepoint stops with journey times interpolated based on distance and travel time from the nearest bracketing time points. As many time point stops as desired can be specified, however, each route/trip variant requires a minimum of two (an origin and destination). 

Our thinking is that this approach minimizes changes to the schema and allows a maximum degree backward compatibility with existing GTFS consumers. However, changes are required by consumers in order to allow interpolated boarding/alighting. 

An argument can also be made for including the interpolated stops in the feed, shifting this responsibility to the feed producer. However, we believe that this is practically in-feasible given the frequency along routes. In our experience, large route networks with a high frequency of stopping points quickly become unmanageable. 

Additionally, by relying on the GTFS consumer to specify the exact boarding and alighting location we believe that it is possible to deliver better and more journey accurate information. For example, it is easier for the GTFS consumer to locate and accurately label a potential boarding location for a given trip. This could be performed when GTFS is combined with base map data at time of consumption. Conversely it is unlikely that a feed producer would be able to create and maintain this information for a large collection of stop points, sometimes reaching well into the tens of thousands for large route networks. 

Finally, our recommended approach avoids an important semantics problem: it's inappropriate to encouraging the explicit specification of something that dose not exist in reality -- in essence "faking" a formalized stop structure to make one type of transit conform to the expectations of another. While this may seem inconsequential from a technical standpoint asking a transit operator or regulatory authority to knowingly misrepresent in a public feed calls into questions the appropriateness of the standard and can undermine the goal of public distribution.


### Localizing Route Types

The current GTFS route type framework presents challenges for international adoption. As currently defined GTFS relies on the USDOT NTD classification schema and includes a very limited set of route types. Modifications have been made to include a more expansive set of types, referred to as a "hierarchical vehicle type" definition, however,  this change is both poorly documented and inadequate as it contains a North American and European-centric view of available modes.

Further, the specification of modes does not allow for use of localized names. This is important both for bridging language barriers and allowing the use of local nomenclature to define service types (as different from operators or brands). Current recommendations for localization, not yet incorporated into the GTFS spec, still do not allow feed producers to extend route types or specify localized names. 

To overcome this limitation we recommend the following extension:

Create a "route_types.txt" definition that overrides values specified int the "route_type" column in the "routes.txt" file. The extension file would include rows with the following columns:

route_type_id: the number used in the routes.txt route_type column to specify a localized type
gtfs_rotue_type: the closest matching route type from the standard GTFS types definition
hvt_route_type: the closest matching HVT type from the proposed HVT types extension, with the option to expand the HVT type categories with a centralized registry
localized_name: the route type name defining this mode of transport using the local language and character set
localized_description: an optional brief description of the mode in the local language


### Reducing Time Precision

**TBD**
