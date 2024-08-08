CREATE DATABASE COMP_GRANT_BUTLER
 USE COMP_GRANT_BUTLER
	SELECT current_user(),now(),database();
#REM CREATE DATABASE IF NOT EXISTS company_aa;
#REM USE company_aa;

/*Drop tables if they exist*/
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE if exists equipment;
DROP TABLE if exists dependent;
DROP TABLE if exists assignment;
DROP TABLE if exists employee;
DROP TABLE if exists project;
DROP TABLE if exists dept_location;
DROP TABLE if exists department;

/* Create department table */
CREATE TABLE department (
    dpt_no	INT PRIMARY KEY,
    dpt_name    VARCHAR(20) NOT NULL,
    dpt_mgrssn  CHAR(9),
    dpt_mgr_start_date	DATE);

/* Create dept_locations table */
CREATE TABLE dept_location (
    dpt_no 		INT NOT NULL,
    dpt_location   	VARCHAR(20),
	PRIMARY KEY (dpt_no, dpt_location),
	FOREIGN KEY (dpt_no) REFERENCES department(dpt_no));
  
/* Create project table */
CREATE TABLE project (
    pro_num		INT,
    pro_name		VARCHAR(25) NOT NULL,
    pro_location	VARCHAR(25),
    pro_dept_num     	INT,
	PRIMARY KEY (pro_num),
	FOREIGN KEY (pro_dept_num) REFERENCES department(dpt_no));

/* Create employee table */
CREATE TABLE employee (
    emp_ssn		CHAR(9) PRIMARY KEY,
    emp_last_name	VARCHAR(25) NOT NULL,
    emp_first_name	VARCHAR(25) NOT NULL,
    emp_middle_name	VARCHAR(25),
    emp_address		VARCHAR(50),
    emp_city		VARCHAR(15),
    emp_state		CHAR(2),
    emp_zip		CHAR(9),
    emp_date_of_birth	DATE,
    emp_salary		INT NOT NULL,
	CONSTRAINT ck_emp_salary CHECK (emp_salary <= 85000),
    emp_parking_space	INT UNIQUE,
    emp_gender		CHAR(1),
    emp_dpt_num		INT,
    emp_superssn	CHAR(9),
	CONSTRAINT fk_emp_dpt FOREIGN KEY (emp_dpt_num) REFERENCES department(dpt_no),
	CONSTRAINT fk_emp_superssn FOREIGN KEY (emp_superssn) REFERENCES employee(emp_ssn));
  
/* Create assignment table */
CREATE TABLE assignment (
    work_emp_ssn	CHAR(9),
    work_pro_num	INT,
    work_hours          INT,
    work_hours_planned	INT,
	CONSTRAINT pk_assignment PRIMARY KEY (work_emp_ssn, work_pro_num),
	CONSTRAINT fk_work_emp	FOREIGN KEY (work_emp_ssn) REFERENCES employee(emp_ssn),
	CONSTRAINT fk_work_pro_num FOREIGN KEY (work_pro_num) REFERENCES project(pro_num));
  
/* Create dependent table */
CREATE TABLE dependent (
    dep_emp_ssn		CHAR(9),
    dep_name		VARCHAR(40),
    dep_gender		CHAR(1),
    dep_date_of_birth	DATE,
    dep_relationship	VARCHAR(10) NOT NULL,
	CONSTRAINT pk_dependent PRIMARY KEY (dep_emp_ssn, dep_name),
	CONSTRAINT fk_dep_emp_ssn FOREIGN KEY (dep_emp_ssn) REFERENCES employee(emp_ssn));

/* Create equipment table */
CREATE TABLE equipment (
    eqp_no		CHAR(4) PRIMARY KEY,
    eqp_description	VARCHAR(15),
    eqp_value		INT,
    eqp_qty_on_hand	INT,
    eqp_total_value 	INT generated always as (eqp_value * eqp_qty_on_hand),
    eqp_pro_num		INT,
	CONSTRAINT fk_eqp_pro_num FOREIGN KEY (eqp_pro_num) REFERENCES project(pro_num));

/* POPULATE TABLES */
/* Department rows.  Department manager SSN */
/*  and date_mgr_startdate are null.        */
INSERT INTO department (dpt_no, dpt_name, dpt_mgrssn, dpt_mgr_start_date)
VALUES	(7, 'Production', NULL, NULL),
	(3, 'Admin and Records', NULL, NULL),
	(1, 'Headquarters', NULL, NULL);

/* Dept_locations rows. */  
INSERT INTO dept_location (dpt_no, dpt_location)
VALUES	(1, 'Edwardsville'),
	(3, 'Marina'),
	(7, 'St. Louis'),
	(7, 'Collinsville'),
	(7, 'Edwardsville');

/* Project rows. */
INSERT INTO project (pro_num, pro_name, pro_location, pro_dept_num)
VALUES	(1, 'Order Entry', 'St. Louis', 7),
	(2, 'Payroll', 'Collinsville', 7),
	(3, 'Receivables', 'Edwardsville', 7),
	(10, 'Inventory', 'Marina', 3),
	(20, 'Personnel', 'Edwardsville', 1),
	(30, 'Pay Benefits', 'Marina', 3);

/* Employee rows. */
INSERT INTO employee	(emp_ssn, emp_last_name, emp_first_name, emp_middle_name, 
			 emp_address, emp_city, emp_state, emp_zip, emp_date_of_birth, emp_salary, 
    			 emp_parking_space, emp_gender, emp_dpt_num, emp_superssn)
VALUES	('999666666', 'Bordoloi', 'Zack', NULL, 'South Main #12', 'Edwardsville', 'IL', 62025, ('1954-03-14'), 55000, 1, 'M', 1, NULL ),
	('999555555', 'Joyner', 'Suzanne', 'A', '202 Burns Farm', 'Marina', 'CA', 93941, ('1971-06-20'), 43000, 3, 'F', 3, '999666666'), 
	('999444444', 'Zhu', 'Waiman', 'Z', '303 Lindbergh', 'St. Louis', 'MO', 63121, ('1975-12-08'), 43000, 32, 'M', 7, '999666666'), 
	('999887777', 'Markis', 'Marcia','M', 'High St. #14', 'Monterey', 'CA', 93940, ('1978-07-19'), 25000, 402, 'F', 3, '999555555'), 
	('999222222', 'Amin', 'Hyder', NULL, 'S. Seaside Apt. B', 'Marina', 'CA', 93941, ('1969-03-29'), 25000, 422, 'M', 3, '999555555'), 
	('999111111', 'Bock', 'Douglas', 'B', '#2 Mont Verd Dr.', 'St. Louis', 'MO', 63121, ('1950-12-05'), 30000, 542, 'M', 7, '999444444'), 
	('999333333', 'Joshi', 'Dinesh', NULL, '#10 Oak St.', 'Collinsville', 'IL', 66234, ('1972-09-15'), 38000, 332, 'M', 7, '999444444'), 
	('999888888', 'Prescott', 'Sherri','C', 'Overton Way #4', 'Edwardsville', 'IL', 62025, ('1972-07-31'), 25000, 296, 'F', 7, '999444444'); 

/* Assignment rows. */
INSERT INTO assignment (work_emp_ssn, work_pro_num, work_hours, work_hours_planned)
VALUES ('999111111', 1, 31.4, 35.0),
	('999111111', 2, 8.5, 10.2),
	('999333333', 3, 42.1, 65.0),
	('999888888', 1, 21.0, 20.0),
	('999888888', 2, 22.0, 20.0),
	('999444444', 2, 12.2, 15.5),
	('999444444', 3, 10.5, 30.0),
	('999444444', 1, NULL, NULL),
	('999444444', 10, 10.1, 10.5),
	('999444444', 20, 11.8, 10.2),
	('999887777', 30, 30.8, 25.5),
	('999887777', 10, 10.2, 15.0),
	('999222222', 10, 34.5, 42.3),
	('999222222', 30, 5.1, 11.8),
	('999555555', 30, 19.2, 18.3),
	('999555555', 20, 14.8, NULL),
	('999666666', 20, NULL, 15.5);

/* Dependent rows. */
INSERT INTO dependent (dep_emp_ssn, dep_name, dep_gender, dep_date_of_birth, dep_relationship)
VALUES ('999444444', 'Jo Ellen', 'F', '1996-04-05', 'DAUGHTER'),
	('999444444', 'Andrew',  'M', '1998-10-25', 'SON'),
	('999444444', 'Susan',   'F', '1975-05-03', 'SPOUSE'),
	('999555555', 'Allen',   'M', '1968-02-29', 'SPOUSE'),
	('999111111', 'Jeffery', 'M', '1978-12-31', 'SON'),
	('999111111', 'Deanna',  'F', '1981-05-03', 'DAUGHTER'),
	('999111111', 'Rachael', 'F', '1975-10-04', 'DAUGHTER'),
	('999111111', 'Michelle','F', '1984-03-17', 'DAUGHTER'),
	('999111111', 'Mary Ellen', 'F', '1956-05-28', 'SPOUSE'),
	('999666666', 'Mita',  'F', '1956-06-04', 'SPOUSE'),
	('999666666', 'Anita', 'F', '1984-07-06', 'DAUGHTER'),
	('999666666', 'Monica','F', '1988-12-30', 'DAUGHTER'),
	('999666666', 'Rita',  'F', '1994-05-11', 'DAUGHTER');

/* equipment rows */
INSERT INTO equipment (eqp_no, eqp_description, eqp_value, eqp_qty_on_hand, eqp_pro_num)
VALUES ('4321', 'Computer, PC', 1100.00, 2, 3),
	('2323', 'Table, mobile', 245.50, 3, 2),
	('6987', 'Computer, PC', 849.50, 2, 1),
	('1234', 'Chair, mobile', 78.25, 4, 2),
	('5678', 'Printer', 172.00, 1, 30),
	('9876', 'Computer, Ntpad', 1400.23, 2, 30);

/* Update department rows to add manager ssn and start date. */
UPDATE department
SET dpt_mgrssn = '999444444', 
    dpt_mgr_start_date = '1998-05-22'
WHERE dpt_no = '7';       

UPDATE department
SET dpt_mgrssn = '999555555',
    dpt_mgr_start_date = '2001-1-1'
WHERE dpt_no = '3';       

UPDATE department
SET dpt_mgrssn = '999666666',
    dpt_mgr_start_date = '1981-6-19'
WHERE dpt_no = '1'; 

/* Add FOREIGN KEY constraint between the department */
/* and employee tables.                              */
 ALTER TABLE department ADD CONSTRAINT fk_dept_emp 
    FOREIGN KEY (dpt_mgrssn) REFERENCES employee (emp_ssn); 

/* Count table rows to ensure the script executed properly. */
SELECT COUNT(*) "Department Count should be 3" FROM department;
SELECT COUNT(*) "Dept Location Count s/b 5" FROM dept_location;
SELECT COUNT(*) "Project Count should be 6" FROM project;
SELECT COUNT(*) "Employee Count should be 8" FROM employee;
SELECT COUNT(*) "Assignment Count should be 17" FROM assignment;
SELECT COUNT(*) "Dependent Count should be 13" FROM dependent;
SELECT COUNT(*) "Equipment Count should be 6" FROM equipment;

/*Display records from each table*/
SELECT * FROM department;
SELECT * FROM dept_location;
SELECT * FROM project;
SELECT * FROM employee;
SELECT * FROM assignment;
SELECT * FROM dependent;
SELECT * FROM equipment;

COMMIT;

/* End of Script */

 INSERT INTO EMPLOYEE (emp_ssn, emp_first_name, emp_last_name, emp_dpt_num, emp_superssn, emp_salary)
VALUES ('111111111', 'Grant', 'Butler', '1', 999666666','80000');

 INSERT INTO Dependent (dept_emp_ssn, dep_name, dep_gender, dep_date_of_birth, dep_relationship)
VALUES ('111111111', 'Amya', 'X', '2005-10-11', 'DAUGHTER'); 

 SELECT *
FROM EMPLOYEE;

 SELECT *
FROM DEPENDENT;

 SELECT
    CONCAT(e.emp_first_name, ' ', e.emp_last_name) AS Employee_Name,
    concat(dpt_no, ' ', dpt_name)AS Department,     
    IFNULL(COUNT(a.work_pro_num), 0) AS `# of projects`,
    IFNULL(SUM(a.work_hours), 'NULL') AS 'Actual Hours',
    IFNULL(SUM(a.work_hours_planned), 'NULL') AS 'Planned Hours',
    IFNULL(SUM(a.work_hours_planned), 0) - IFNULL(SUM(a.work_hours), 0) AS ‘Difference’
FROM employee e
LEFT JOIN assignment a ON e.emp_ssn = a.work_emp_ssn
LEFT JOIN department d ON e.emp_dpt_num = d.dpt_no
GROUP BY e.emp_ssn, e.emp_first_name, e.emp_last_name, e.emp_dpt_num, d.dpt_name
ORDER BY e.emp_dpt_num ASC, '# of Projects' ASC;

select concat(substring(e.emp_ssn, 1, 3), '-', substring(e.emp_ssn, 4, 2), '-', substring(e.emp_ssn, 6, 4)) AS 'Employee SSN', concat (e.emp_first_name, ' ', e.emp_last_name) as 'Employee Name', count(distinct a.work_pro_num) as '# of Projects', concat('$',format(sum(eqp.eqp_total_value),0)) as 'Equip. Total'
from employee e
left join assignment a on a.work_emp_ssn = e.emp_ssn
left join equipment eqp on eqp.eqp_pro_num = a.work_pro_num
group by e.emp_ssn
order by sum(eqp.eqp_total_value)  asc;
 
SELECT  CONCAT(p.pro_num, ' - ', p.pro_name) AS Project,
SUM(a.work_hours_planned) AS "Hours Planned",
SUM(a.work_hours) AS "Hours Worked",
SUM(a.work_hours_planned) - SUM(a.work_hours) AS Difference,
COUNT(DISTINCT a.work_emp_ssn) AS "# of Employees" FROM  project p
JOIN  assignment a ON p.pro_num = a.work_pro_num
GROUP BY  p.pro_num, p.pro_name HAVING ABS(SUM(a.work_hours_planned) - SUM(a.work_hours)) > 10
ORDER BY  Difference DESC;

 SELECT
   IF(e.emp_ssn = dpt_mgrssn, NULL, CONCAT(SUBSTRING(m.emp_ssn, 1, 3), '-', SUBSTRING(m.emp_ssn, 4, 2), '-', SUBSTRING(m.emp_ssn, 6, 4))) AS "Manager SSN",
  CONCAT(m.emp_first_name, ' ',m.emp_last_name) AS 'Manager Name',
 concat(dpt_no, ' - ', dpt_name)AS Department,
concat(substring(e.emp_ssn, 1, 3), '-', substring(e.emp_ssn, 4, 2), '-', substring(e.emp_ssn, 6, 4)) AS 'Employee SSN',
  CONCAT(e.emp_first_name, ' ',e.emp_last_name) AS 'Employee Name',
  dep_name AS 'Dependent Name',
  dep_relationship AS 'Relationship',
  DATE_FORMAT(dep_date_of_birth, '%M %d, %Y') AS 'Dependent DOB'
FROM
     employee e
LEFT JOIN
       department d ON e.emp_dpt_num = d.dpt_no
LEFT JOIN 
    employee m ON e.emp_superssn = m.emp_ssn
 JOIN 
     dependent dep ON e.emp_ssn = dep_emp_ssn
ORDER BY 
   d.dpt_no ASC, e.emp_ssn ASC;

 
