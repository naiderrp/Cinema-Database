INSERT INTO Genre (Name) VALUES 
('Action'),
('Comedy'),
('Drama'),
('Horror'),
('Science Fiction');

INSERT INTO ProductionCountry (Name) VALUES 
('USA'),
('UK'),
('Canada'),
('India'),
('Germany');

INSERT INTO Language (Name) VALUES 
('English'),
('Spanish'),
('French'),
('German'),
('Hindi');

INSERT INTO Movie (Title, Description, Duration) VALUES 
('Inception', 'A mind-bending thriller', 148),
('The Hangover', 'Three friends lose their groom', 100),
('The Godfather', 'The story of a crime family', 175),
('It', 'A horror story about a clown', 135),
('Interstellar', 'Space exploration and survival', 169);

INSERT INTO HallType (Name, Capacity) VALUES 
('Private', 5),
('Public', 10);

INSERT INTO Hall (TypeID) VALUES 
(1),
(2);

INSERT INTO Seat (HallID, Row, Col) VALUES 
(1, 1, 1), (1, 1, 2), (1, 1, 3), (1, 2, 1), (1, 2, 2);

INSERT INTO Seat (HallID, Row, Col) VALUES 
(2, 1, 1), (2, 1, 2), (2, 1, 3), (2, 1, 4), (2, 1, 5),
(2, 2, 1), (2, 2, 2), (2, 2, 3), (2, 2, 4), (2, 2, 5);

INSERT INTO UserRole (Name) VALUES 
('Admin'),
('Client');

INSERT INTO "User" (Name, PasswordHASH, Email, RoleID) VALUES 
('AdminUser', 'hash123', 'admin@example.com', 1),
('ClientUser1', 'hash456', 'client1@example.com', 2),
('ClientUser2', 'hash789', 'client2@example.com', 2);

INSERT INTO PaymentMethod (Name) VALUES 
('Card'),
('GiftCard');

INSERT INTO MovieActors (MovieID, ActorID) VALUES 
(1, 1), (1, 2), -- Inception
(2, 3), (2, 4), -- The Hangover
(3, 5), (3, 6), -- The Godfather
(4, 7), (4, 8), -- It
(5, 9), (5, 10); -- Interstellar

INSERT INTO MovieGenres (MovieID, GenreID) VALUES 
(1, 1), (1, 5), -- Inception: Action, Science Fiction
(2, 2), -- The Hangover: Comedy
(3, 3), -- The Godfather: Drama
(4, 4), -- It: Horror
(5, 1), (5, 5); -- Interstellar: Action, Science Fiction

INSERT INTO MovieProductionCountries (MovieID, ProductionCountryID) VALUES 
(1, 1), -- Inception: USA
(2, 1), -- The Hangover: USA
(3, 2), -- The Godfather: UK
(4, 1), -- It: USA
(5, 1), (5, 3); -- Interstellar: USA, Canada

INSERT INTO MovieLanguages (MovieID, LanguageID) VALUES 
(1, 1), -- Inception: English
(2, 1), -- The Hangover: English
(3, 1), -- The Godfather: English
(4, 1), -- It: English
(5, 1); -- Interstellar: English


INSERT INTO Comission (Type) VALUES
(10),
(30),
(50);

INSERT INTO Partner (CompanyName, ComissionID) VALUES
('Coca-Cola', 2);

INSERT INTO MenuItemType (Type, PartnerID) VALUES
('Drink', 1);

INSERT INTO MenuItem (Name, MenuItemTypeID, Price) VALUES
('Fanta', 1, 5);
