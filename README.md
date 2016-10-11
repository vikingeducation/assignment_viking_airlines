# assignmnent_viking_analytics
*** Deepak

##Queries 1: Warmups

###1. Get a list of all users in California
```sql
User.find_by_sql("
  SELECT users.username
  FROM users JOIN states ON users.state_id = states.id
  WHERE states.name = 'California'
")
```
###2. Get a list of all airports in Minnesota
```sql
Airport.find_by_sql("
  SELECT airports.long_name
  FROM airports JOIN states ON airports.state_id = states.id
  WHERE states.name = 'Minnesota'
")
```
###3. Get a list of all payment methods used on itineraries by the user with email address "heidenreich_kara@kunde.net"
```sql
Itinerary.find_by_sql("
  SELECT itineraries.payment_method
  FROM itineraries JOIN users ON itineraries.user_id = users.id
  WHERE users.email = 'heidenreich_kara@kunde.net'
")
```
###4. Get a list of prices of all flights whose origins are in Kochfurt Probably International Airport.
```sql
Flight.find_by_sql("
  SELECT flights.price
  FROM flights JOIN airports ON flights.origin_id = airports.id
  WHERE airports.long_name='Kochfurt Probably International Airport'
")
```
###5. Find a list of all Airport names and codes which connect to the airport coded LYT.
```sql
Airport.find_by_sql("
  SELECT DISTINCT airports.long_name, airports.code
  FROM airports JOIN flights ON flights.destination_id = airports.id
  WHERE airports.code = 'LYT'
")
```
###6. Get a list of all airports visited by user Dannie D'Amore after January 1, 2012. (Hint, see if you can get a list of all ticket IDs first). Note: Careful how you escape the quote in "D'Amore"... escaping in SQL is different from Ruby.
```sql
User.find_by_sql("
  select airports.long_name, flights.departure_time, flights.arrival_time
  FROM users JOIN itineraries ON users.id = itineraries.user_id
  JOIN tickets ON itineraries.id = tickets.itinerary_id
  JOIN flights ON tickets.flight_id = flights.id
  JOIN airports ON airports.id = flights.origin_id
  WHERE users.first_name = 'Dannie' AND users.last_name = 'D''Amore' AND flights.departure_time > '2012-01-01'
  UNION
  select airports.long_name, flights.departure_time, flights.arrival_time
  FROM users JOIN itineraries ON users.id = itineraries.user_id
  JOIN tickets ON itineraries.id = tickets.itinerary_id
  JOIN flights ON tickets.flight_id = flights.id
  JOIN airports ON airports.id = flights.destination_id
  WHERE users.first_name = 'Dannie' AND users.last_name = 'D''Amore' AND flights.departure_time > '2012-01-01'
")
```

##Queries 2: Adding in Aggregation

###1. Find the top 5 most expensive flights that end in California.
```sql
Flight.find_by_sql("
  SELECT flights.*
  from flights JOIN airports ON flights.destination_id = airports.id
  JOIN states ON airports.state_id = states.id
  WHERE states.name = 'California'
  ORDER BY price DESC
  LIMIT 5
")
```

###2. Find the shortest flight that username "ryann_anderson" took.
```sql
User.find_by_sql("
  SELECT *
  FROM flights JOIN tickets ON flights.id = tickets.flight_id
  JOIN itineraries ON tickets.itinerary_id = itineraries.id
  JOIN users ON users.id = itineraries.user_id
  WHERE users.username = 'ryann[_]anderson'
  ORDER BY distance
  LIMIT 1
  ")
```

###3. Find the average flight distance for flights entering or leaving each city in Florida
```sql
User.find_by_sql("
  SELECT AVG(distance)
  FROM
    (SELECT distance
    FROM flights JOIN airports ON flights.origin_id = airports.id
    JOIN states ON airports.state_id = states.id
    WHERE states.name = 'Florida'
    UNION
    SELECT distance
    FROM flights JOIN airports ON flights.destination_id = airports.id
    JOIN states ON states.id = airports.state_id
    WHERE states.name = 'Florida') AS joint_table
")
```

###4. Find the 3 users who spent the most money on flights in 2013
```sql
User.find_by_sql("
  SELECT users.first_name, SUM(flights.price)
  FROM users JOIN itineraries ON itineraries.user_id = users.id
  JOIN tickets ON tickets.itinerary_id = itineraries.id
  JOIN flights ON tickets.flight_id = flights.id
  WHERE flights.departure_time BETWEEN '2013-01-01' AND '2013-12-31'
  GROUP BY users.id
  ORDER BY SUM(flights.price) DESC
  LIMIT 3
")
```

###5. Count all flights to or from the city of Lake Vivienne that did not land in Florida
```sql
Flight.find_by_sql("
  SELECT SUM(count) AS all
  FROM
  (
    SELECT COUNT(*)
    FROM flights JOIN airports ON flights.origin_id = airports.id
    JOIN states ON airports.state_id = states.id
    JOIN cities ON airports.city_id = cities.id
    WHERE cities.name = 'Lake Vivienne' AND flights.destination_id NOT IN
        (SELECT airports.id
        FROM airports JOIN states ON airports.state_id = states.id
        WHERE states.name = 'Florida')
    UNION
    SELECT COUNT(*)
    FROM flights JOIN airports ON flights.destination_id = airports.id
    JOIN states ON airports.state_id = states.id
    JOIN cities ON airports.city_id = cities.id
    WHERE cities.name = 'Lake Vivienne' AND flights.destination_id NOT IN
        (SELECT airports.id
        FROM airports JOIN states ON airports.state_id = states.id
        WHERE states.name = 'Florida')) AS jt
")
```

###6. Return the range of lengths of flights in the system(the maximum, and the minimum).
```sql
Flight.find_by_sql("
  SELECT MAX(distance), MIN(distance)
  FROM flights
  ")
```

##Queries 3: Advanced

###1. Find the most popular travel destination for users who live in Kansas.
```sql
User.find_by_sql("
  SELECT states.name, COUNT(states.name) AS num
  FROM tickets
  JOIN flights ON tickets.flight_id = flights.id
  JOIN airports ON airports.id = flights.destination_id
  JOIN cities ON cities.id = airports.city_id
  JOIN states ON states.id = airports.state_id
  JOIN itineraries ON itineraries.id = tickets.itinerary_id
  JOIN users ON users.id = itineraries.user_id
  WHERE users.state_id = (
    SELECT states.id
    FROM states
    WHERE states.name = 'Kansas')
  GROUP BY states.name
  ORDER BY num DESC
  ")
```

###2. How many flights have round trips possible? In other words, we want the count of all airports where there exists a flight FROM that airport and a later flight TO that airport.
```sql
Airport.find_by_sql("
  SELECT airports.long_name
  FROM
  (SELECT DISTINCT f1.origin_id, f1.destination_id
    FROM flights f1 JOIN flights f2
    ON f1.origin_id = f2.destination_id
    AND f1.destination_id = f2.origin_id
    WHERE f1.id <> f2.id
    ORDER BY f1.origin_id) AS jt
  JOIN airports ON airports.id = origin_id
")
```


###3. Find the cheapest flight that was taken by a user who only had one itinerary.
```sql
Flight.find_by_sql("
  SELECT flights.id, flights.price, users.first_name
  from flights JOIN tickets ON tickets.flight_id = flights.id
  JOIN itineraries ON itineraries.id = tickets.itinerary_id
  JOIN users ON itineraries.user_id = users.id
  WHERE users.username IN
    (SELECT users.username
    FROM tickets
    JOIN flights ON tickets.flight_id = flights.id
    JOIN itineraries ON itineraries.id = tickets.itinerary_id
    JOIN users ON users.id = itineraries.user_id
    GROUP BY users.username
    HAVING count(itineraries) = 1)
  ORDER BY flights.price
  LIMIT 1
")
```
###4. Find the average cost of a flight itinerary for users in each state in 2012.
```sql
Flight.find_by_sql("
  SELECT AVG(flights.price), states.name
    from flights
    JOIN tickets ON tickets.flight_id = flights.id
    JOIN itineraries ON itineraries.id = tickets.itinerary_id
    JOIN users ON users.id = itineraries.user_id
    JOIN states ON states.id = users.state_id
    WHERE flights.departure_time BETWEEN '2012-01-01' AND '2012-12-31'
    GROUP BY states.name
")
```

