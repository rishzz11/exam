-- #############################
-- College Management System
-- #############################

-- 1. Schema Creation with Foreign Keys & Cascades
CREATE DATABASE IF NOT EXISTS CollegeDB;
USE CollegeDB;

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100) NOT NULL
);

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Age INT,
    Gender VARCHAR(10),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

CREATE TABLE Faculty (
    FacultyID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100) NOT NULL,
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY,
    StudentID INT,
    CourseID INT,
    Grade VARCHAR(2),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE Teaches (
    FacultyID INT,
    CourseID INT,
    Semester VARCHAR(20),
    PRIMARY KEY (FacultyID, CourseID),
    FOREIGN KEY (FacultyID) REFERENCES Faculty(FacultyID)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- 2. Sample Data
INSERT INTO Departments VALUES
(1, 'Computer Science'),
(2, 'Electrical Engineering'),
(3, 'Mechanical Engineering'),
(4, 'Civil Engineering');

INSERT INTO Students VALUES
(101, 'Aarav Sharma', 20, 'Male', 1),
(102, 'Priya Mehta', 21, 'Female', 2),
(103, 'Rohan Gupta', 19, 'Male', 1),
(104, 'Simran Kaur', 22, 'Female', 3),
(105, 'Yash Patel', 20, 'Male', NULL);

INSERT INTO Faculty VALUES
(201, 'Dr. Neha Verma', 1),
(202, 'Dr. Amit Joshi', 2),
(203, 'Dr. Kiran Rao', 3),
(204, 'Dr. Manish Sinha', NULL);

INSERT INTO Courses VALUES
(301, 'DBMS', 1),
(302, 'Data Structures', 1),
(303, 'Digital Circuits', 2),
(304, 'Thermodynamics', 3),
(305, 'Structural Analysis', 4);

INSERT INTO Enrollments VALUES
(1, 101, 301, 'A'),
(2, 101, 302, 'B'),
(3, 102, 303, 'A'),
(4, 103, 301, 'B'),
(5, 104, 304, 'C'),
(6, 105, 305, NULL);

INSERT INTO Teaches VALUES
(201, 301, 'Fall 2023'),
(201, 302, 'Spring 2024'),
(202, 303, 'Fall 2023'),
(203, 304, 'Spring 2024'),
(204, 305, 'Fall 2023');

-- #############################
-- College DB Practice Queries
-- #############################

-- Section 1: JOINS (INNER, LEFT, RIGHT, SELF)
-- Q1. Inner Join: Students with their enrolled Course Names
SELECT S.Name AS StudentName, C.CourseName
FROM Students S
INNER JOIN Enrollments E ON S.StudentID = E.StudentID
INNER JOIN Courses C ON E.CourseID = C.CourseID;

-- Q2. Left Join: All Students and their Courses (if any)
SELECT S.Name AS StudentName, C.CourseName
FROM Students S
LEFT JOIN Enrollments E ON S.StudentID = E.StudentID
LEFT JOIN Courses C ON E.CourseID = C.CourseID;

-- Q3. Courses and their Faculty (if any)
SELECT C.CourseName, F.Name AS FacultyName
FROM Courses C
LEFT JOIN Teaches T ON C.CourseID = T.CourseID
LEFT JOIN Faculty F ON T.FacultyID = F.FacultyID;

-- Q4. Self Join: Faculties in same Department
SELECT F1.Name AS Faculty1, F2.Name AS Faculty2, D.DepartmentName
FROM Faculty F1
JOIN Faculty F2 ON F1.DepartmentID = F2.DepartmentID AND F1.FacultyID <> F2.FacultyID
JOIN Departments D ON F1.DepartmentID = D.DepartmentID;

-- Section 2: IN, NOT IN, EXISTS
-- Q5. Students enrolled in DBMS (CourseID=301)
SELECT Name FROM Students WHERE StudentID IN (
  SELECT StudentID FROM Enrollments WHERE CourseID = 301
);

-- Q6. Students not enrolled in any Course
SELECT Name
FROM Students S
WHERE NOT EXISTS (
  SELECT 1 FROM Enrollments E WHERE E.StudentID = S.StudentID
);

-- Section 3: ORDER BY, GROUP BY, HAVING
-- Q7. List Students Ordered by Age DESC
SELECT Name, Age FROM Students ORDER BY Age DESC;

-- Q8. Count Students per Department
SELECT D.DepartmentName, COUNT(S.StudentID) AS TotalStudents
FROM Departments D
LEFT JOIN Students S ON D.DepartmentID = S.DepartmentID
GROUP BY D.DepartmentName;

-- Q9. Courses with more than 1 Student
SELECT C.CourseName, COUNT(E.StudentID) AS EnrolledCount
FROM Courses C
JOIN Enrollments E ON C.CourseID = E.CourseID
GROUP BY C.CourseName
HAVING COUNT(E.StudentID) > 1;

-- Section 4: Multi-Concept Queries
-- Q10. Students taught by Dr. Neha Verma (JOIN vs IN)
-- Approach 1: JOINs
SELECT DISTINCT S.Name
FROM Students S
JOIN Enrollments E ON S.StudentID = E.StudentID
JOIN Courses C ON E.CourseID = C.CourseID
JOIN Teaches T ON C.CourseID = T.CourseID
JOIN Faculty F ON T.FacultyID = F.FacultyID
WHERE F.Name = 'Dr. Neha Verma';

-- Approach 2: Nested IN
SELECT Name FROM Students
WHERE StudentID IN (
  SELECT E.StudentID FROM Enrollments E
  WHERE E.CourseID IN (
    SELECT CourseID FROM Teaches WHERE FacultyID = (
      SELECT FacultyID FROM Faculty WHERE Name = 'Dr. Neha Verma'
    )
  )
);

-- Q11. Departments without any Faculty
SELECT D.DepartmentName
FROM Departments D
LEFT JOIN Faculty F ON D.DepartmentID = F.DepartmentID
WHERE F.FacultyID IS NULL;

-- Section 5: Aggregate + Complex Filters
-- Q12. Avg Grade Point per Course
SELECT C.CourseName,
  AVG(CASE E.Grade
      WHEN 'A' THEN 10
      WHEN 'B' THEN 8
      WHEN 'C' THEN 6
      ELSE NULL END) AS AvgPoints
FROM Courses C
LEFT JOIN Enrollments E ON C.CourseID = E.CourseID
GROUP BY C.CourseName;

-- Section 6: Misc Practice
-- Q13. Student Names with Department (include NULL)
SELECT S.Name, D.DepartmentName
FROM Students S
LEFT JOIN Departments D ON S.DepartmentID = D.DepartmentID;

-- Q14. Faculty teaching outside their Department
SELECT F.Name AS FacultyName, C.CourseName
FROM Faculty F
JOIN Teaches T ON F.FacultyID = T.FacultyID
JOIN Courses C ON T.CourseID = C.CourseID
WHERE F.DepartmentID IS NOT NULL AND C.DepartmentID <> F.DepartmentID;

-- Q15. Count Faculty per Course
SELECT C.CourseName, COUNT(T.FacultyID) AS FacultyCount
FROM Courses C
LEFT JOIN Teaches T ON C.CourseID = T.CourseID
GROUP BY C.CourseName;

-- #############################
-- Sales Management Assignment
-- #############################

-- Schema & Sample Data
CREATE TABLE Salesman (
    salesman_id INT PRIMARY KEY,
    name VARCHAR(50),
    city VARCHAR(50),
    commission DECIMAL(5,2)
);

CREATE TABLE Customer (
    customer_id INT PRIMARY KEY,
    name VARCHAR(50),
    city VARCHAR(50),
    grade INT,
    salesman_id INT,
    FOREIGN KEY (salesman_id) REFERENCES Salesman(salesman_id)
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    purchase_amt DECIMAL(10,2),
    salesman_id INT,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id),
    FOREIGN KEY (salesman_id) REFERENCES Salesman(salesman_id)
);

INSERT INTO Salesman VALUES
(101, 'John', 'New York', 0.15),
(102, 'Alice', 'Chicago', 0.13),
(103, 'Bob', 'New York', 0.12),
(104, 'Charlie', 'Boston', 0.14);

INSERT INTO Customer VALUES
(201, 'Steve', 'New York', 2, 101),
(202, 'Diana', 'Boston', 1, 104),
(203, 'Sam', 'Chicago', 3, 102),
(204, 'Nina', 'New York', 2, 103);

INSERT INTO Orders VALUES
(301, 201, '2024-04-10', 500.00, 101),
(302, 202, '2024-04-10', 800.00, 104),
(303, 203, '2024-04-11', 900.00, 102),
(304, 204, '2024-04-11', 1200.00, 103);

-- Assignment Views
-- 1a. Salesmen in New York
CREATE VIEW View_NewYork_Salesmen AS
SELECT * FROM Salesman WHERE city = 'New York';

-- 1b. All salesmen basic
CREATE VIEW View_Salesmen_Basic AS
SELECT salesman_id, name, city FROM Salesman;

-- 2a. Customer count per grade
CREATE VIEW View_Customer_Grade_Count AS
SELECT grade, COUNT(*) AS num_customers FROM Customer GROUP BY grade;

-- 3a. Order stats by date
CREATE VIEW View_Order_Stats_By_Date AS
SELECT order_date,
       COUNT(DISTINCT customer_id) AS unique_customers,
       AVG(purchase_amt) AS avg_amount,
       SUM(purchase_amt) AS total_amount
FROM Orders GROUP BY order_date;

-- 3b. Order + Customer + Salesman info
CREATE VIEW View_Order_Customer_Salesperson AS
SELECT O.order_id, O.purchase_amt,
       S.salesman_id, S.name AS salesman_name,
       C.name AS customer_name
FROM Orders O
JOIN Salesman S ON O.salesman_id = S.salesman_id
JOIN Customer C ON O.customer_id = C.customer_id;

-- 3c. Top Salesperson per day
CREATE VIEW View_Top_Salesperson_Per_Day AS
SELECT O.order_date, S.salesman_id, S.name AS salesman_name
FROM Orders O
JOIN Salesman S ON O.salesman_id = S.salesman_id
WHERE (O.purchase_amt, O.order_date) IN (
    SELECT MAX(purchase_amt), order_date FROM Orders GROUP BY order_date
);

-- 3d. Salesmen count per city
CREATE VIEW View_Salesmen_Count_Per_City AS
SELECT city, COUNT(*) AS num_salesmen FROM Salesman GROUP BY city;

-- 3e. Salesman names with city
CREATE VIEW View_Salesmen_Name_City AS
SELECT city, name FROM Salesman;

-- 3f. Drop view from 3e
DROP VIEW View_Salesmen_Name_City;

-- #############################
-- Suppliers-Parts-Catalog Assignment Queries
-- #############################

-- 1. Names of suppliers who supply some red part.
SELECT DISTINCT S.sname
FROM Suppliers S
JOIN Catalog C ON S.sid = C.sid
JOIN Parts P ON C.pid = P.pid
WHERE P.color = 'red';

-- 2. SIDs of suppliers who supply some red or green part.
SELECT DISTINCT S.sid
FROM Suppliers S
JOIN Catalog C ON S.sid = C.sid
JOIN Parts P ON C.pid = P.pid
WHERE P.color IN ('red','green');

-- 3. SIDs of suppliers who supply some red part or are at 221 Packer Street.
SELECT DISTINCT sid FROM (
  SELECT S.sid FROM Suppliers S
  JOIN Catalog C ON S.sid = C.sid
  JOIN Parts P ON C.pid = P.pid
  WHERE P.color = 'red'
  UNION
  SELECT sid FROM Suppliers WHERE address = '221 Packer Street'
) AS T;

-- 4. SIDs of suppliers who supply some red part and some green part.
SELECT R.sid
FROM (
  SELECT DISTINCT sid FROM Catalog C JOIN Parts P ON C.pid=P.pid WHERE P.color='red'
) R
JOIN (
  SELECT DISTINCT sid FROM Catalog C JOIN Parts P ON C.pid=P.pid WHERE P.color='green'
) G USING (sid);

-- 5. SIDs of suppliers who supply every part.
SELECT S.sid
FROM Suppliers S
WHERE NOT EXISTS (
  SELECT * FROM Parts P
  WHERE NOT EXISTS (
    SELECT * FROM Catalog C WHERE C.sid=S.sid AND C.pid=P.pid
  )
);

-- 6. SIDs of suppliers who supply every red part.
SELECT S.sid
FROM Suppliers S
WHERE NOT EXISTS (
  SELECT * FROM Parts P WHERE P.color='red'
    AND NOT EXISTS (
      SELECT * FROM Catalog C WHERE C.sid=S.sid AND C.pid=P.pid
    )
);

-- 7. SIDs of suppliers who supply every red or green part.
SELECT S.sid
FROM Suppliers S
WHERE NOT EXISTS (
  SELECT * FROM Parts P WHERE P.color IN ('red','green')
    AND NOT EXISTS (
      SELECT * FROM Catalog C WHERE C.sid=S.sid AND C.pid=P.pid
    )
);

-- 8. SIDs of suppliers who supply every red part or supply every green part.
SELECT S.sid
FROM Suppliers S
WHERE (
  NOT EXISTS (
    SELECT * FROM Parts P WHERE P.color='red'
      AND NOT EXISTS (
        SELECT * FROM Catalog C WHERE C.sid=S.sid AND C.pid=P.pid
      )
  )
)
OR (
  NOT EXISTS (
    SELECT * FROM Parts P WHERE P.color='green'
      AND NOT EXISTS (
        SELECT * FROM Catalog C WHERE C.sid=S.sid AND C.pid=P.pid
      )
  )
);

-- 9. Pairs of sids such that first supplier charges more for some part than second supplier.
SELECT DISTINCT C1.sid AS sid1, C2.sid AS sid2
FROM Catalog C1
JOIN Catalog C2 ON C1.pid = C2.pid
WHERE C1.cost > C2.cost;

-- 10. PIDs of parts supplied by at least two different suppliers.
SELECT pid
FROM Catalog
GROUP BY pid
HAVING COUNT(DISTINCT sid) >= 2;

-- 11. PIDs of the most expensive parts supplied by suppliers named 'Yosemite Sham'.
SELECT C.pid
FROM Catalog C
JOIN Suppliers S ON C.sid=S.sid
WHERE S.sname='Yosemite Sham'
  AND C.cost = (
    SELECT MAX(C2.cost) FROM Catalog C2
    JOIN Suppliers S2 ON C2.sid=S2.sid
    WHERE S2.sname='Yosemite Sham'
  );

-- 12. PIDs of parts supplied by every supplier at less than $200.
SELECT P.pid
FROM Parts P
WHERE NOT EXISTS (
  SELECT * FROM Suppliers S
  WHERE NOT EXISTS (
    SELECT * FROM Catalog C
    WHERE C.sid = S.sid AND C.pid = P.pid AND C.cost < 200
  )
);


-- 1. Student Table with Age as a virtual/generated column (MySQL)
CREATE TABLE Student (
    Student_ID INT PRIMARY KEY,
    Name VARCHAR(50),
    DOB DATE,
    Age INT GENERATED ALWAYS AS (TIMESTAMPDIFF(YEAR, DOB, CURDATE())) STORED,
    Gender VARCHAR(10)
);

-- 2. Agar table already ban chuka hai, toh ALTER karke column add karna:
ALTER TABLE Student 
ADD COLUMN Age INT GENERATED ALWAYS AS (TIMESTAMPDIFF(YEAR, DOB, CURDATE())) STORED;

-- 3. Agar tumhe ek baar age dekhni ho bina column add kiye:
SELECT Student_ID, Name, DOB, 
       TIMESTAMPDIFF(YEAR, DOB, CURDATE()) AS Age
FROM Student;
