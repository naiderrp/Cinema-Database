# TABLES

CREATE TABLE Actor (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE Genre (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE ProductionCountry (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE Language (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE HallType (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    Capacity INT NOT NULL
);

CREATE TABLE Movie (
    ID SERIAL PRIMARY KEY,
    Title VARCHAR(255) NOT NULL,
    Description TEXT,
    Duration INT NOT NULL
);

CREATE TABLE MovieActors (
    ID SERIAL PRIMARY KEY,
    MovieID INT NOT NULL REFERENCES Movie(ID),
    ActorID INT NOT NULL REFERENCES Actor(ID)
);

CREATE TABLE MovieGenres (
    ID SERIAL PRIMARY KEY,
    MovieID INT NOT NULL REFERENCES Movie(ID),
    GenreID INT NOT NULL REFERENCES Genre(ID)
);

CREATE TABLE MovieProductionCountries (
    ID SERIAL PRIMARY KEY,
    MovieID INT NOT NULL REFERENCES Movie(ID),
    ProductionCountryID INT NOT NULL REFERENCES ProductionCountry(ID)
);

CREATE TABLE MovieLanguages (
    ID SERIAL PRIMARY KEY,
    MovieID INT NOT NULL REFERENCES Movie(ID),
    LanguageID INT NOT NULL REFERENCES Language(ID)
);

CREATE TABLE Hall (
    ID SERIAL PRIMARY KEY,
    TypeID INT NOT NULL REFERENCES HallType(ID)
);

CREATE TABLE Seat (
    ID SERIAL PRIMARY KEY,
    HallID INT NOT NULL REFERENCES Hall(ID),
    Row INT NOT NULL,
    Col INT NOT NULL
);

CREATE TABLE Showtime (
    ID SERIAL PRIMARY KEY,
    MovieID INT NOT NULL REFERENCES Movie(ID),
    HallID INT NOT NULL REFERENCES Hall(ID),
    StartTime TIMESTAMP NOT NULL,
    EndTime TIMESTAMP NOT NULL,
    Date DATE NOT NULL
);

CREATE TABLE UserRole (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE "User" (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    PasswordHASH VARCHAR(255) NOT NULL,
    Email VARCHAR(255) UNIQUE NOT NULL,
    RoleID INT NOT NULL REFERENCES UserRole(ID)
);

CREATE TABLE Booking (
    ID SERIAL PRIMARY KEY,
    UserID INT NOT NULL REFERENCES "User"(ID),
    ShowtimeID INT NOT NULL REFERENCES Showtime(ID),
    DateBooked TIMESTAMP NOT NULL,
    PaymentID INT REFERENCES Payment(ID)
);

CREATE TABLE Ticket (
    ID SERIAL PRIMARY KEY,
    BookingID INT NOT NULL REFERENCES Booking(ID),
    SeatID INT NOT NULL REFERENCES Seat(ID),
    Price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE PaymentMethod (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);

CREATE TABLE Payment (
    ID SERIAL PRIMARY KEY,
    BookingID INT NOT NULL REFERENCES Booking(ID),
    Date TIMESTAMP NOT NULL
);

CREATE TABLE PaymentMethods (
    ID SERIAL PRIMARY KEY,
    PaymentID INT NOT NULL REFERENCES Payment(ID),
    MethodID INT NOT NULL REFERENCES PaymentMethod(ID)
);

CREATE TABLE GiftCard (
    ID SERIAL PRIMARY KEY,
    Code VARCHAR(255) UNIQUE NOT NULL,
    Balance DECIMAL(10, 2) NOT NULL,
    ExpirationDate DATE NOT NULL
);

CREATE TABLE EquipmentType (
    ID SERIAL PRIMARY KEY,
    Type VARCHAR(255) NOT NULL
);

CREATE TABLE Equipment (
    ID SERIAL PRIMARY KEY,
    EquipmentTypeID INT NOT NULL REFERENCES EquipmentType(ID),
    Name VARCHAR(255) NOT NULL,
    DatePurchased DATE NOT NULL
);

CREATE TABLE JobPosition (
    ID SERIAL PRIMARY KEY,
    Title VARCHAR(255) NOT NULL
);

CREATE TABLE Staff (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    JobPositionID INT NOT NULL REFERENCES JobPosition(ID)
);

CREATE TABLE Comission (
    ID SERIAL PRIMARY KEY,
    Type INT NOT NULL
);

CREATE TABLE Partner (
    ID SERIAL PRIMARY KEY,
    CompanyName VARCHAR(255) NOT NULL,
    ComissionID INT NOT NULL REFERENCES Comission(ID)
);

CREATE TABLE MenuItemType (
    ID SERIAL PRIMARY KEY,
    Type VARCHAR(255) NOT NULL,
    PartnerID INT REFERENCES Partner(ID)
);

CREATE TABLE MenuItem (
    ID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    MenuItemTypeID INT NOT NULL REFERENCES MenuItemType(ID),
    Price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE Menu (
    ID SERIAL PRIMARY KEY,
    MenuItemID INT NOT NULL REFERENCES MenuItem(ID),
    Date DATE NOT NULL
);

# INDICES

CREATE INDEX idx_movie_description ON Movie (Description);

CREATE INDEX idx_genre_name ON Genre (Name);

CREATE INDEX idx_actor_name ON Actor (Name);

CREATE INDEX idx_language_name ON Language (Name);

CREATE INDEX idx_paymentmethod_name ON PaymentMethod (Name);

CREATE INDEX idx_equipmenttype_name ON EquipmentType (Type);

CREATE UNIQUE INDEX idx_giftcard_code ON GiftCard (Code);

CREATE INDEX idx_user_gmail ON "User" USING HASH (Email);

CREATE INDEX idx_user_password_hash ON "User" (PasswordHASH);

CREATE INDEX idx_jobposition_title ON JobPosition (Title);

CREATE INDEX idx_menuitemtype_type ON MenuItemType (Type);

CREATE INDEX idx_movie_title ON Movie (Title);

# TRIGGERS

CREATE OR REPLACE FUNCTION set_showtime_endtime()
RETURNS TRIGGER AS $$
BEGIN
  NEW.endtime := NEW.starttime + (INTERVAL '1 minute' * (SELECT duration FROM Movie WHERE ID = NEW.MovieID));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_set_showtime_endtime
BEFORE INSERT OR UPDATE ON Showtime
FOR EACH ROW
WHEN (NEW.starttime IS NOT NULL AND NEW.MovieID IS NOT NULL)
EXECUTE FUNCTION set_showtime_endtime();


CREATE OR REPLACE FUNCTION check_hall_capacity()
RETURNS TRIGGER AS $$
BEGIN
  IF (SELECT COUNT(*) FROM Seat WHERE HallID = NEW.HallID) >= (SELECT Capacity FROM HallType WHERE ID = (SELECT TypeID FROM Hall WHERE ID = NEW.HallID)) THEN
    RAISE EXCEPTION 'Hall capacity exceeded.';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_check_hall_capacity
BEFORE INSERT ON Seat
FOR EACH ROW
EXECUTE FUNCTION check_hall_capacity();


CREATE OR REPLACE FUNCTION update_partner_comission_on_price_change()
RETURNS TRIGGER AS $$
DECLARE
    current_type INT;
    new_type INT;
BEGIN
    SELECT Comission.Type INTO current_type
    FROM Comission
    JOIN Partner ON Partner.ComissionID = Comission.ID
    WHERE Partner.ID = (SELECT PartnerID FROM MenuItemType WHERE ID = NEW.MenuItemTypeID);

    IF NEW.Price > OLD.Price THEN
        SELECT MIN(Type) INTO new_type
        FROM Comission
        WHERE Type > current_type;

        IF new_type IS NOT NULL THEN
            UPDATE Partner
            SET ComissionID = (SELECT ID FROM Comission WHERE Type = new_type)
            WHERE ID = (SELECT PartnerID FROM MenuItemType WHERE ID = NEW.MenuItemTypeID);
        END IF;

    ELSIF NEW.Price < OLD.Price THEN
        SELECT MAX(Type) INTO new_type
        FROM Comission
        WHERE Type < current_type;

        IF new_type IS NOT NULL THEN
            UPDATE Partner
            SET ComissionID = (SELECT ID FROM Comission WHERE Type = new_type)
            WHERE ID = (SELECT PartnerID FROM MenuItemType WHERE ID = NEW.MenuItemTypeID);
        END IF;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_partner_comission_on_price_change
AFTER UPDATE OF Price ON MenuItem
FOR EACH ROW
WHEN (OLD.Price IS DISTINCT FROM NEW.Price)
EXECUTE FUNCTION update_partner_comission_on_price_change();


CREATE OR REPLACE FUNCTION validate_email()
RETURNS TRIGGER AS $$
BEGIN
    IF NOT (NEW.Email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$') THEN
        RAISE EXCEPTION 'Invalid email format: %', NEW.Email;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_validate_email
BEFORE INSERT OR UPDATE ON "User"
FOR EACH ROW
EXECUTE FUNCTION validate_email();

CREATE OR REPLACE FUNCTION check_seat_availability()
RETURNS TRIGGER AS $$
DECLARE
    hall_capacity INT;
    booked_seats INT;
BEGIN
    SELECT ht.Capacity
    INTO hall_capacity
    FROM Hall h
    JOIN HallType ht ON h.TypeID = ht.ID
    WHERE h.ID = (SELECT HallID FROM Showtime WHERE ID = NEW.ShowtimeID);

    SELECT COUNT(*)
    INTO booked_seats
    FROM Ticket t
    JOIN Booking b ON t.BookingID = b.ID
    WHERE b.ShowtimeID = NEW.ShowtimeID;

    IF booked_seats >= hall_capacity THEN
        RAISE EXCEPTION 'No seats available in the hall for the selected showtime.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER check_seats_before_booking
BEFORE INSERT ON Booking
FOR EACH ROW
EXECUTE FUNCTION check_seat_availability();

# PROCEDURES

CREATE OR REPLACE PROCEDURE get_available_seats(
    IN showtimeID INT
)
LANGUAGE plpgsql AS $$
BEGIN
    SELECT s.ID AS SeatID, s.Row, s.Col
    FROM Seat s
    WHERE s.HallID = (SELECT HallID FROM Showtime WHERE ID = showtimeID)
    AND s.ID NOT IN (
        SELECT t.SeatID
        FROM Ticket t
        JOIN Booking b ON t.BookingID = b.ID
        WHERE b.ShowtimeID = showtimeID
    );
END;
$$;

CREATE OR REPLACE PROCEDURE remove_expired_tickets()
LANGUAGE plpgsql AS $$
BEGIN
    DELETE FROM Ticket
    WHERE BookingID IN (
        SELECT b.ID
        FROM Booking b
        JOIN Showtime s ON b.ShowtimeID = s.ID
        WHERE s.EndTime < NOW()
    );

    DELETE FROM Booking
    WHERE ShowtimeID IN (
        SELECT s.ID
        FROM Showtime s
        WHERE s.EndTime < NOW()
    );
END;
$$;

CREATE OR REPLACE FUNCTION is_gift_card_valid(giftCardCode VARCHAR)
RETURNS BOOLEAN
LANGUAGE plpgsql AS $$
DECLARE
    expirationDate DATE;
    balance DECIMAL(10, 2);
BEGIN
    SELECT ExpirationDate, Balance
    INTO expirationDate, balance
    FROM GiftCard
    WHERE Code = giftCardCode;

    IF NOT FOUND THEN
        RETURN FALSE;
    END IF;

    IF expirationDate >= CURRENT_DATE AND balance > 0 THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
END;
$$;

CREATE PROCEDURE get_total_revenue_for_movies()
LANGUAGE plpgsql AS
$$
BEGIN
    SELECT 
        m.Title, 
        SUM(t.Price) AS TotalRevenue
    FROM Movie m
    JOIN Showtime s ON m.ID = s.MovieID
    JOIN Ticket t ON s.ID = t.ShowtimeID
    GROUP BY m.ID;
END;
$$;

CREATE PROCEDURE get_menu_for_date(menu_date DATE)
LANGUAGE plpgsql AS
$$
BEGIN
    SELECT 
        mi.Name AS MenuItem, 
        mi.Price, 
        mt.Type AS MenuItemType
    FROM Menu m
    JOIN MenuItem mi ON m.MenuItemID = mi.ID
    JOIN MenuItemType mt ON mi.MenuItemTypeID = mt.ID
    WHERE m.Date = menu_date;
END;
$$;

CREATE PROCEDURE check_bookings_for_showtime(showtime_id INT)
LANGUAGE plpgsql AS
$$
DECLARE
    booking_count INT;
BEGIN
    SELECT COUNT(*) INTO booking_count
    FROM Booking
    WHERE ShowtimeID = showtime_id;

    IF booking_count > 0 THEN
        RAISE NOTICE 'There are bookings for this showtime.';
    ELSE
        RAISE NOTICE 'No bookings for this showtime.';
    END IF;
END;
$$;
