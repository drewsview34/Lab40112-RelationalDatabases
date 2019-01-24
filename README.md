# Lab12-RelationalDBSchemas
Quick repository for today's work on 1-24.

## Relational Databases and Schemas
For this project, we needed to create a schema given the following information:

1. The hotel can hold many guests at any given time, in fact guests can come and go as they please…..asynchronously :)
2. The hotel is named “Async Inn” and has many nationwide locations. Each location will have a name, city, state, address, and phone number.
3. Async Inn prides themselves on their unique layout designs of each hotel room. They advertise as it being your “apartment for the night”. This means they have invested a lot of resources into how each room looks and feels. Some have one bedroom, others have 2 bedrooms, while a few are more of a cozy studio. The team mentioned that they like to label each room with a nickname to better tell the difference between each of the layouts and amenities each room has to offer. (for example, the Seattle location has two 2-bedroom suites, but one is named “Seahawks Snooze” while the other is named “Restful Rainier”, each with their own amenities.)
4. They also take pride in the amenities that each room has to offer. This can consist of features like “air conditioning”, “coffee maker”, “ocean view”, “mini bar”, the list goes on…They requested that they would like the amenities associated with each of the rooms as they do vary.
5. The rooms vary in price, per location, as well as per room number. They also have a few rooms that they want to advertise as pet friendly.
6. The number of rooms for each hotel varies. Some hotels have only a few rooms, while others may have dozens.

Additionally, we were required to implement the following:
- (1) Joint Entity Table with Payload
- (1) Pure Join Table
- (1) Enum

## Schema by Blaise Clarke and Andrew Hinojosa:
![Whiteboard drawing of proposed schema](https://github.com/Dervival/Lab12-RelationalDBSchemas/blob/master/AsyncSchemaDrawing.jpg);

## Schema Component Explanation
Hotel Table:
- Primary Motivation: Several hotels need to be tracked for the Async Inn chain - ergo, we should have a table whose rows each represent a specific instance of a hotel chain.
- Properties:
	- Id: Primary key for the table. Represents a unique numeric (possibly enumerated) value for each entry in the table.
	- Name: Name of the hotel instance. Default is presumably "Async Inn". String type. Does not need to be unique.
	- City: Name of the city of the hotel instance. String type.
	- State: State that the hotel instance resides in. String type. 
	- Address: Address of the hotel instance. String type.
	- Phone: Phone number for the front desk of the hotel instance. String type.
- Relations
	- Reservations - From the first data point, I had thought that it would be good to be able to link customers to hotels in some way. A person can go to several different instances of hotels at different times, and a hotel can contain several guests, so this needs to be represented as a many:many relationship - using a reservation table as either a pure join table or a joint-entity with payload seemed like a reasonable approach. One-to-Many, as a single hotel can have many reservations, but a reservation is specific to a given hotel.
	- HotelConfigs - Additionally, I wanted to represent room configurations as its own table, and a given room configuration can be in several hotels (and vice versa). This is just a direct table linking the RoomConfig table and Hotel tables to allow that relationship. One-to-Many, since a hotel can have several configurations, but each entry should only link to a single combination of hotel and room configuration.
- Navigation
	- Reservations - To traverse between the Hotel table to Customer, CheckIn, and CheckOut tables.
	- HotelConfigs - To traverse between the Hotel table to RoomConfig or Room tables.

Reservation Table:
- Primary Motivation: To link the Customer and Hotel tables as well as provide a route to check in and check out information.
- Properties:
	- Id: Primary key for the table. Represents a unique numeric value for a given reservations. Integer type.
	- HotelId: Part of the composite key for the table, along with the CustomerId and CheckInId. Foreign key for the Hotel table. Integer type.
	- CustomerId: Part of the composite key for the table, along with the HotelId and CheckInId. Foreign key for the Customer Table. Integer type.
	- CheckInId: Part of the composite key for the table; required due to my assumption that while a customer can check into a given hotel more than once, they cannot check in at the same time or using the same check-in id more than once. Corresponds one-to-one with the id of the CheckIn table.
	- CheckOutId: Id representing the id of the checkout object once a customer has left the hotel. Can match up to one id in the CheckOut table.
- Relations
	- Hotel - required for the primary motivation of linking the customer and hotel tables. Many-to-one.
	- Customer - required for the primary motivation of linking the customer and hotel tables. Also many-to-one.
	- CheckIn - Technically not required - all data from CheckIn could be merged into this table. I've separated the two out to separate concerns, however. One-to-one.
	- CheckOut - Links to checkout information. Due to a record in CheckOut not existing until a customer actually checks out, this relation is required to avoid needing nullable values that would be overwritten once a customer checks out. One-to-one.
- Navigation
	- Hotel - to allow Customer to link to Hotel.
	- Customer - to allow Hotel to link to Customer.
	- CheckIn - to allow either Customer or Hotel to link to CheckIn.
	- CheckOut - to allow either Customer or Hotel to link to CheckOut.


Customer Table:
- Primary Motivation: From the first data point, it seems reasonable to track customers of the entire chain.
- Properties:
	- Id - Primary key of the table. Gives a unique integer (possibly enumerable) field for the table. Integer type.
	- FirstName - First name of the customer. Basic customer data.
	- LastName - Last name of the customer. Basic customer data.
	- ContactNumber - Phone number provided by the customer when checking in. Basic customer data.
- Relations
	- Reservations - As mentioned in the Hotel table, it seemed reasonable to link the Hotel and Customer tables in a many-to-many way. We need to use an intermediary table, Reservations, to do this. One-to-many. 
- Navigation
	- Reservations - - To traverse between the Customer table to Hotel, CheckIn, and CheckOut tables.

CheckIn Table:
- Primary Motivation: It seems reasonable to store data about who has checked in to the hotel for a data integrity check and to see which rooms are occupied. Could be integrated into the reservations table, but this allows further expansion of check in information if other info is needed later.
- Properties:
	- Id - Primary key for the table, and foreign key for the Reservation table.
	- CheckInDate
	- CheckInTime
- Relations
	- Reservations
- Navigation
	- Reservations

CheckOut Table:
- Primary Motivation: It also seems reasonable to store data about who has checked out to see which rooms are now unoccupied; additionally, I'd like to be able to avoid having to modify information already in a table, so I went for the approach of separating the check out information from the reservation and check in tables.
- Properties:
	- Id - Primary key for the table, and foreign key for the Reservation table.
	- CheckOutDate
	- CheckOutTime
- Relations
	- Reservations
- Navigation
	- Reservations

HotelConfigs Table:
- Primary Motivation: To link the Hotel and RoomConfig tables. Additionally, fulfills the requirement of using a pure join table with no payload.
- Properties:
	- HotelId
	- ConfigurationId
- Relations
	- Hotel
	- RoomConfig
- Navigation
	- Hotel
	- RoomConfig

RoomConfig Table:
- Primary Motivation: To group the NumberOfRooms, isPetsAllowed, and the enumerable list of amenities for a given room. 
- Properties:
	- ConfigurationName
	- NumberOfRooms
	- IsPetsAllowed
	- Amenities
- Relations
	- Amenities
	- Room
	- HotelConfigs
- Navigation
	- Room
	- HotelConfigs

Room Table:
- Primary Motivation: To describe a specific room with a given configuration - includes a room nickname RoomName as well as a price that is different between rooms and locations.
- Properties:
	- Id
	- RoomName
	- ConfigName
	- Price
- Relations
=	- RoomConfigs
- Navigation
	- RoomConfigs

Amenities Table:
- Primary Motivation: To allow sets of amenities to be set up in a specific room. The implementation of this is a bit strange; I took inspiration from a previous model I used where each amenity is assigned a bit value in a number. The binary representation of the amenity enumerator represents whether or not the amenity is in the room - for instance, a value of 13 where 5 amenities have been coded in would translate to (0)1101, so the first, third, and fourth amenity would be available.
- Properties:
	- Id
	- ListOfAmenites
- Relations
	- RoomConfigs
- Navigation
No navigation to or from amenities.
