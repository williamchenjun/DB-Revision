# Data Warehouse Design

> **Disclaimer**: The content of this page is highly derivative of the Database module from KCL (reference below).

## Exercise (Designing a STAR Schema)

We are given the following schemas:
- `OWNER(IDOwner, Name, Surname, Address, City, Phone)`
- `ESTATE (IDEstate, IDOwner, Category, Area, City, Province, Rooms, Bedrooms, Garage, Meters)`
- `CUSTOMER (IDCust, Name, Surname, Budget, Address, City, Phone)`
- `AGENT (IDAgent, Name, Surname, Office, Address, City, Phone)`
- `AGENDA (IDAgent, Date, Hour, IDEstate, ClientName)`
- `VISIT (IDEstate,IDAgent, IDCust, Date, Duration)`
- `SALE (IDEstate,IDAgent, IDCust, Date, AgreedPrice, Status)`
- `RENT (IDEstate,IDAgent, IDCust, Date, Price, Status, Time)`

We are asked to build a star schema for a data warehouse for the sales department. First of all, we notice that there are some tables we don't need in order to analyse information about the sales. That is, `AGENDA`, `VISIT` AND `RENT`.

If we use the given schemas, we end up with the following SQL query:

```mysql
CREATE TABLE IF NOT EXISTS OWNER(
IDOwner INT PRIMARY KEY,
Name VARCHAR(255) NOT NULL,
Surname VARCHAR(255) NOT NULL,
Address VARCHAR(255) NOT NULL,
City VARCHAR(255) NOT NULL,
Phone VARCHAR(30) NOT NULL
);

CREATE TABLE IF NOT EXISTS ESTATE(
IDEstate INT PRIMARY KEY, 
IDOwner INT NOT NULL,
Category VARCHAR(100) NOT NULL,
Area VARCHAR(150) NOT NULL, 
City VARCHAR(100) NOT NULL, 
Province VARCHAR(100) NOT NULL,
Rooms INT NOT NULL, 
Bedrooms INT NOT NULL, 
Garage BOOL NOT NULL, 
Meters DECIMAL NOT NULL,
FOREIGN KEY (IDOwner) REFERENCES OWNER (IDOwner)
ON UPDATE CASCADE
ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS CUSTOMER(
IDCust INT PRIMARY KEY, 
Name VARCHAR(255) NOT NULL, 
Surname VARCHAR(255) NOT NULL, 
Budget DOUBLE NOT NULL, 
Address VARCHAR(300) NOT NULL, 
City VARCHAR(100) NOT NULL, 
Phone VARCHAR(30) NOT NULL
);

CREATE TABLE IF NOT EXISTS AGENT (
IDAgent INT PRIMARY KEY, 
Name VARCHAR(255) NOT NULL, 
Surname VARCHAR(255) NOT NULL,
Office VARCHAR(255) NOT NULL, 
Address VARCHAR(255) NOT NULL, 
City VARCHAR(100) NOT NULL, 
Phone VARCHAR(30) NOT NULL
);

CREATE TABLE IF NOT EXISTS SALE (
IDEstate INT NOT NULL,
IDAgent INT NOT NULL, 
IDCust INT NOT NULL, 
SaleDate DATETIME NOT NULL, 
AgreedPrice DECIMAL NOT NULL, 
SaleStatus VARCHAR(100) NOT NULL,
FOREIGN KEY (IDEstate) REFERENCES ESTATE (IDEstate)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDAgent) REFERENCES AGENT (IDAgent)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDCust) REFERENCES CUSTOMER (IDCust)
ON DELETE CASCADE
ON UPDATE CASCADE
);
```
which looks like the following

<p align="center">
  <img src="https://user-images.githubusercontent.com/79821802/209212854-cac2e3b3-01e5-4f91-9541-c5e6715b5104.svg"/>
</p>

This is a snowflake schema for the sales data warehouse. To convert it to a star schema, we sever the `OWNER`-`ESTATE` relation, and we add the `IDOwner` to the `SALES` table. Thus, we remove `IDOwner` from `ESTATE`

```mysql
CREATE TABLE IF NOT EXISTS ESTATE(
IDEstate INT PRIMARY KEY, 
Category VARCHAR(100) NOT NULL,
Area VARCHAR(150) NOT NULL, 
City VARCHAR(100) NOT NULL, 
Province VARCHAR(100) NOT NULL,
Rooms INT NOT NULL, 
Bedrooms INT NOT NULL, 
Garage BOOL NOT NULL, 
Meters DECIMAL NOT NULL,
FOREIGN KEY (IDOwner) REFERENCES OWNER (IDOwner)
ON UPDATE CASCADE
ON DELETE CASCADE
);
```
and we add it to `SALE`

```mysql
CREATE TABLE IF NOT EXISTS SALE (
IDEstate INT NOT NULL,
IDAgent INT NOT NULL, 
IDCust INT NOT NULL, 
IDOwner INT NOT NULL,
SaleDate DATETIME NOT NULL, 
AgreedPrice DECIMAL NOT NULL, 
SaleStatus VARCHAR(100) NOT NULL,
FOREIGN KEY (IDEstate) REFERENCES ESTATE (IDEstate)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDAgent) REFERENCES AGENT (IDAgent)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDCust) REFERENCES CUSTOMER (IDCust)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDOwner) REFERENCES OWNER (IDOwner)
ON DELETE CASCADE
ON UPDATE CASCADE
);
```

Additionally, we can create a new table `TIME` to store datetime information for the sales

```mysql
CREATE TABLE IF NOT EXISTS TIME (
TimeID INT PRIMARY KEY,
DDMMYY DATE NOT NULL
);
```

or alternatively

```mysql
CREATE TABLE IF NOT EXISTS TIME (
TimeID INT PRIMARY KEY,
DAY INT NOT NULL,
MONTH INT NOT NULL,
YEAR INT NOT NULL
);
```

Thus, `SALE` is

```mysql
CREATE TABLE IF NOT EXISTS SALE (
IDEstate INT NOT NULL,
IDAgent INT NOT NULL, 
IDCust INT NOT NULL, 
IDOwner INT NOT NULL,
TimeID INT NOT NULL, 
AgreedPrice DECIMAL NOT NULL,
FOREIGN KEY (IDEstate) REFERENCES ESTATE (IDEstate)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDAgent) REFERENCES AGENT (IDAgent)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDCust) REFERENCES CUSTOMER (IDCust)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (IDOwner) REFERENCES OWNER (IDOwner)
ON DELETE CASCADE
ON UPDATE CASCADE
FOREIGN KEY (TimeID) REFERENCES TIME (TimeID)
ON DELETE CASCADE
ON UPDATE CASCADE
```

We remove the status row since we don't need it. (It is implied that the sale has been completed). Now the data warehouse looks like the following:

<p align="center">
  <img src="https://user-images.githubusercontent.com/79821802/209213968-8834aba4-71d2-4d54-88f4-15937020f9ad.svg"/>
</p>
