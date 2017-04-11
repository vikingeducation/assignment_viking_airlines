# assignmnent_viking_analytics


David Meza

Nick Sarlo


## Warmups

Here are some queries we'd like you to answer using this ActiveRecord database.

1. Get a list of all users in California

    SELECT users.id, users.first_name, users.last_name, states.name 
    FROM users JOIN states ON users.state_id = states.id 
    WHERE states.name = 'California'

2. Get a list of all airports in Minnesota

    SELECT airports.long_name, states.name
    FROM airports JOIN states ON airports.state_id = states.id
    WHERE states.name='Minnesota'


3. Get a list of all payment methods used on itineraries by the user with email address "heidenreich_kara@kunde.net"

  SELECT users.email, itineraries.payment_method FROM users JOIN itineraries ON users.id=itineraries.user_id WHERE users.email='heidenreich_kara@kunde.net'
  

4  Get a list of prices of all flights whose origins are in Kochfurt Probably International Airport.

  SELECT origin_airport.long_name, dest_airport.long_name, flights.price FROM flights JOIN airports origin_airport ON flights.origin_id=origin_airport.id JOIN airports dest_airport ON flights.destination_id=dest_airport.id WHERE airports.long_name='Kochfurt Probably International Airport'



5.  Find a list of all Airport names and codes which connect to the airport coded LYT.


  SELECT origin_airport.code, dest_airport.long_name FROM flights JOIN airports origin_airport ON flights.origin_id=origin_airport.id JOIN airports dest_airport ON flights.destination_id=dest_airport.id WHERE origin_airport.code='LYT'

6. Get a list of all airports visited by user Dannie D'Amore after January 1, 2015. (Hint, see if you can get a list of all ticket IDs first).

  SELECT CONCAT(users.last_name, ', ', users.first_name), origin_airport.long_name AS origin, dest_airport.long_name AS destination, flights.arrival_time FROM tickets JOIN itineraries ON tickets.itinerary_id=itineraries.id JOIN users ON itineraries.user_id = users.id JOIN flights ON tickets.flight_id=flights.id JOIN airports origin_airport ON flights.origin_id=origin_airport.id JOIN airports dest_airport ON flights.destination_id=dest_airport.id WHERE users.first_name= 'Dannie' AND users.last_name LIKE '%Amore'
