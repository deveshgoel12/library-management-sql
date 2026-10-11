# library-management-sql
SQL Project
# 📚 Library Management System - SQL Portfolio Project

## 🔍 Project Overview
A relational database system designed and implemented from scratch to manage library operations, including book inventory, member memberships, staff management, book borrowing transactions, and fine calculations. 

## 🛠️ Tech Stack
* **Database Management System:** PostgreSQL / MySQL
* **Key Concepts:** Relational Modeling, Primary & Foreign Keys, Constraints (`NOT NULL`, `CHECK`), Joins, Subqueries, Stored Procedures, Views, and Aggregate Functions.

---

## 🗄️ Database Schema & Entity-Relationship (ER)
The database consists of 5 core interconnected tables:
1. **Authors:** Stores author details.
2. **Books:** Stores book titles, ISBN, published year, and foreign keys to authors and publishers.
3. **Members:** Stores library patron contact information and registration dates.
4. **Staff:** Stores library employee details and branch assignments.
5. **Transactions:** Tracks book borrowing, return dates, and fine amounts.

---

## 💻 SQL Implementation Code

### 1. Schema Creation & Table Constraints
```sql
CREATE DATABASE LibraryDB;
USE LibraryDB;

-- Authors Table
CREATE TABLE Authors (
    AuthorID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Country VARCHAR(50)
);

-- Books Table
CREATE TABLE Books (
    BookID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(255) NOT NULL,
    AuthorID INT,
    PublishedYear INT,
    AvailableCopies INT CHECK (AvailableCopies >= 0),
    FOREIGN KEY (AuthorID) REFERENCES Authors(AuthorID) ON DELETE CASCADE
);

-- Members Table
CREATE TABLE Members (
    MemberID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Email VARCHAR(150) UNIQUE NOT NULL,
    JoinDate DATE NOT NULL
);

-- Staff Table
CREATE TABLE Staff (
    StaffID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Position VARCHAR(50),
    HireDate DATE
);

-- Transactions Table (Borrowing & Returns)
CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY AUTO_INCREMENT,
    BookID INT,
    MemberID INT,
    StaffID INT,
    BorrowDate DATE NOT NULL,
    ReturnDate DATE,
    FineAmount DECIMAL(5,2) DEFAULT 0.00,
    FOREIGN KEY (BookID) REFERENCES Books(BookID),
    FOREIGN KEY (MemberID) REFERENCES Members(MemberID),
    FOREIGN KEY (StaffID) REFERENCES Staff(StaffID)
);


INSERT INTO Authors (FirstName, LastName, Country) VALUES 
('J.K.', 'Rowling', 'UK'),
('George', 'Orwell', 'UK'),
('Dan', 'Brown', 'USA');

INSERT INTO Books (Title, AuthorID, PublishedYear, AvailableCopies) VALUES 
('Harry Potter', 1, 1997, 5),
('1984', 2, 1949, 3),
('The Da Vinci Code', 3, 2003, 4);

INSERT INTO Members (FirstName, LastName, Email, JoinDate) VALUES 
('Alice', 'Smith', 'alice@email.com', '2025-01-10'),
('Bob', 'Jones', 'bob@email.com', '2025-02-15');

INSERT INTO Staff (FirstName, LastName, Position, HireDate) VALUES 
('Charlie', 'Brown', 'Librarian', '2023-06-01');

SELECT 
    m.FirstName, m.LastName, 
    b.Title, 
    t.BorrowDate 
FROM Transactions t
JOIN Members m ON t.MemberID = m.MemberID
JOIN Books b ON t.BookID = b.BookID
WHERE t.ReturnDate IS NULL;

SELECT 
    m.MemberID, 
    CONCAT(m.FirstName, ' ', m.LastName) AS FullName, 
    SUM(t.FineAmount) AS TotalFinesPaid
FROM Transactions t
JOIN Members m ON t.MemberID = m.MemberID
GROUP BY m.MemberID, m.FirstName, m.LastName
HAVING SUM(t.FineAmount) > 0;

SELECT Title 
FROM Books 
WHERE BookID IN (
    SELECT BookID 
    FROM Transactions 
    GROUP BY BookID 
    ORDER BY COUNT(*) DESC 
    LIMIT 3
);
