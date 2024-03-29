###  Requirements
* Rider selects pickup point on Map
* can view ETA(Estimated Time of Arrival) & Price.
* Rider pays to request ride.
* Drivers & Riders are matched/ driver accepts ride.
* Rider receives information about the driver(name, car details, review), location updates in real-time.
### High-Level  Overview (Key Components)

![[Pasted image 20240329105232.png]]

### Flow of Information
![[Pasted image 20240329111622.png]]
****
### Drivers & Riders API
* **load balancers**: should be able to handle large number of users.(*horizontal scaling*)
* **web-sockets**: Lots of real-time data. (*not http polling, obviously*)
### Map Infra (Mapbox/Google Maps)
* Serve Map Images & Location Details
* Convert *Street Address* to *latitude/longitude* (Geocoding).
* Get Directions
### Payment Infrastructure (Stripe)
* Rider needs to pay for the service
* Driver needs to get paid
* Ride Confirmation API needs to know when payment finishes processing successfully.

![[Pasted image 20240329124144.png]]
### Pricing (dynamic)
* Pricing varies by demand (surge pricing)
* Kafka, (Spark - Distributed Data Analytics Engine, Price API), Redis - In-memory Storage
* Spark: Structured Streaming
### Rides & ETA
* Needs to efficiently find drivers in an area.
* Needs to calculate ETA.
* API - Driver Locations & ETA Service
*  Geo-spatial Indexing Algorithm - **Uber H3**: Shard by cell (hexagonal/bee hive) ID. Cell ID can be calculated from longitude/latitude. Adjacent cell IDs can be derived from one cell ID.
### Matching
* When a rider requests ride -
	* Find closest (optimal) driver using rides service.
	* Ask Driver to accept or decline ride:
		* Accepted? : Notify Rider & Server driver direction to pickup location
		* Declined?: Repeat with next optimal driver.
***
### Database
#### Schemas
1. **Driver**
	* ID (*PRIMARY_KEY*)
	* Name
	* Ride Type
	* Car Info
	* Phone no.
	* Authentication Details (Email & Password hash)
	* Location
2. **User**/**Rider**:
	* ID (*PRIMARY_KEY)
	* Name
	* Authentication Details (Email & Password hash)
	* Contact Details
	* Payment Info
3. **Trip History**:
	* Trip ID (*PRIMARY_KEY*)
	* Driver ID (*FOREIGN_KEY*)
	* Rider ID (*FOREIGN_KEY*)
	*  Price
	* Pickup Location
	* Destination
	* Date
	* *Status*
	* Ride Type
#### Architecture
* **Shard Key**: ***High Cardinality & Low Frequency***
* Driver & Rider: Shard by ID
* Trip History: Shard by Trip Id Primarily. Other Sharding Patterns can be *Driver ID* or *Rider ID*, sorted by *Status*.

*** 
### Additional Optimization for Efficiency

* Redundancy
* Algorithms for ETA Service
* Data Analytics 
* Algorithms for Sharding driver location
* Math - scaling, algorithms
* ....