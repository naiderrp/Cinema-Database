## Movies by genre

SELECT 
    Movie.Title AS MovieTitle,
    Genre.Name AS GenreName
FROM 
    Movie
JOIN 
    MovieGenres ON Movie.ID = MovieGenres.MovieID
JOIN 
    Genre ON MovieGenres.GenreID = Genre.ID
WHERE 
    Genre.Name = 'Action';

## Movies by actors

SELECT 
    Movie.Title AS MovieTitle,
    Actor.Name AS ActorName
FROM 
    Movie
JOIN 
    MovieActors ON Movie.ID = MovieActors.MovieID
JOIN 
    Actor ON MovieActors.ActorID = Actor.ID
WHERE 
    Actor.Name = 'Leonardo DiCaprio';



## Movies by languages

SELECT 
    Movie.Title AS MovieTitle,
    Language.Name AS LanguageName
FROM 
    Movie
JOIN 
    MovieLanguages ON Movie.ID = MovieLanguages.MovieID
JOIN 
    Language ON MovieLanguages.LanguageID = Language.ID
WHERE 
    Language.Name = 'English';

## Movies by Production Countries

SELECT 
    Movie.Title AS MovieTitle,
    ProductionCountry.Name AS ProductionCountryName
FROM 
    Movie
JOIN 
    MovieProductionCountries ON Movie.ID = MovieProductionCountries.MovieID
JOIN 
    ProductionCountry ON MovieProductionCountries.ProductionCountryID = ProductionCountry.ID
WHERE 
    ProductionCountry.Name = 'USA';


## Movies Watched by a User

SELECT 
    Movie.Title AS MovieTitle,
    Showtime.StartTime AS ShowtimeStart,
    Showtime.EndTime AS ShowtimeEnd,
    Showtime.Date AS ShowtimeDate
FROM 
    "User"
JOIN 
    Booking ON "User".ID = Booking.UserID
JOIN 
    Showtime ON Booking.ShowtimeID = Showtime.ID
JOIN 
    Movie ON Showtime.MovieID = Movie.ID
WHERE 
    "User".ID = 1;

## Methods of Booking Payment by Users

SELECT 
    b.ID AS BookingID,
    b.DateBooked AS DateBooked,
    t.ID AS TicketID,
    t.Price AS TicketPrice,
    p.ID AS PaymentID,
    p.Date AS PaymentDate,
    pm.Name AS PaymentMethod
FROM 
    Booking b
JOIN 
    Ticket t ON b.ID = t.BookingID
JOIN 
    Payment p ON b.ID = p.BookingID
JOIN 
    PaymentMethods pm ON p.ID = pm.PaymentID
JOIN 
    PaymentMethod pm_name ON pm.MethodID = pm_name.ID
WHERE 
    b.UserID = 1; -- ID пользователя

## Viewing Hall Occupancy

SELECT 
    h.ID AS HallID,
    ht.Name AS HallType,
    ht.Capacity AS HallCapacity,
    COUNT(t.ID) AS BookedSeats,
    (COUNT(t.ID) * 100.0 / ht.Capacity) AS OccupancyPercentage
FROM 
    Hall h
JOIN 
    HallType ht ON h.TypeID = ht.ID
JOIN 
    Showtime s ON h.ID = s.HallID
LEFT JOIN 
    Ticket t ON s.ID = t.BookingID
GROUP BY 
    h.ID, ht.Name, ht.Capacity
ORDER BY 
    OccupancyPercentage ASC; -- Сортировка по наименее заполненным залам

