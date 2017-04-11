# assignmnent_viking_analytics

Alice & Garrett

1. User.find_by_sql("SELECT * from users JOIN states ON (state_id = states.id) where states.name = 'California'")

2. Airport.find_by_sql("SELECT * FROM airports JOIN states ON (state_id = states.id)WHERE states.name = 'Minnesota'")

3. Get a list of all payment methods used on itineraries by the user with email address "heidenreich_kara@kunde.net"

Itinerary.find_by_sql("SELECT * FROM itineraries JOIN users ON (user_id=users.id) WHERE users.email = 'heidenreich_kara@kunde.net'")

4. Get a list of prices of all flights whose origins are in Kochfurt Probably International Airport.

Flight.find_by_sql("SELECT price FROM flights JOIN airports ON (origin_id = airports.id) WHERE airports.long_name = 'Kochfurt Probably International Airport'")

5. Find a list of all Airport names and codes which connect to the airport coded LYT.

Flight.find_by_sql("SELECT b.code AS o_code, b.long_name AS o_ln,
                    c.code AS d_code, c.long_name AS d_ln
                  FROM flights JOIN airports b ON (origin_id = b.id)
                  JOIN airports c ON (destination_id = c.id)
                  WHERE b.code = 'LYT' OR c.code = 'LYT'")


6. Get a list of all airports visited by user Dannie D'Amore after January 1, 2015. (Hint, see if you can get a list of all ticket IDs first).


Airport.find_by_sql("SELECT DISTINCT long_name \
FROM airports JOIN flights a ON(airports.id = a.destination_id) \
JOIN flights b ON (airports.id = b.origin_id) \
JOIN tickets ON (flight_id = b.id) \
JOIN itineraries ON (tickets.itinerary_id = itineraries.id) \
JOIN users ON (itineraries.user_id = users.id) \
WHERE users.first_name = 'Dannie' AND users.last_name = \"D'Amore\" ")

AGGREGATION
1.Find the top 5 most expensive flights that end in California.

Flight.find_by_sql("SELECT price \
FROM flights JOIN airports \
ON (flights.destination_id = airports.id) \
JOIN states ON (airports.state_id = states.id) \
WHERE states.name = 'California'  \
ORDER BY flights.price DESC \
LIMIT 5")

2. Find the shortest flight that username "ryann_anderson" took.

Flight.find_by_sql("SELECT *
FROM flights JOIN tickets ON (tickets.flight_id = flights.id) \
JOIN itineraries ON (tickets.itinerary_id = itineraries.id) \
JOIN users ON (itineraries.user_id = users.id) \
WHERE users.username = 'ryann_anderson' \
ORDER BY flights.distance")

--







