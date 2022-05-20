# **Analytix Labs - EXL Service Data Engineering Bootcamp**

## Capstone Project Level 1

### Project Profile:
I have been given HR Employee data from Analytix Labs from 1980s and 1995s. There are a total of 6 ‘csv’ files containing the whole HR Employee data. 

### Business Objective:
The objective of this project is to create a data engineering solution to the HR Employee data. And to create a data model that helps with extraction, transformation and loading of data from one tool to another for fundamental data analysis using the following technology stack. 

### Technology Stack:
    - MySQL (to create database and tables)
    - Linux Commands (to move data)
    - Sqoop (transfer data from MySQL Server to HDFS/Hive)
    - HDFS (to store the data)
    - Hive (to create database)
    - Impala (to perform the EDA)
    - SparkSQL (to perform the EDA)
    - SparkML (to perform model building)

### Data Files:
    - tiles.csv - File with all the job profiles in the organisation
    - departments.sv - File with all the different departments in the organisation
    - employees.csv - File with all employees data like birthdate, joining date, ID, etc
    - dept_emp.csv - File with details about which employee belongs to which department
    - salaries.csv - File with salaries of each employee
    - dept_manager.csv - File with each department's managerial position details

### ERD (Entity Relationship Diagram):
![ERD](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/ERD/ERD_Capstone_Project_Level_1.png)

### Pipeline:
#### Step-1: MySQL data loading.
  The first step for this solution is to create a database and tables in MySQL.
  1) Connect to MySQL through cluster lab 
  ```
  mysql -u username -p
  ```  
  2) Create a database
  ```
  create database csp1_jes;
  
  use csp1_jes;
  ```
  3) Creating required tables
  ```
  create table Titles_jes(
  title_id varchar(10) PRIMARY KEY NOT NULL,
  title varchar(30) NOT NULL);
  
  create table Employees_jes(
  emp_no int PRIMARY KEY NOT NULL,
  emp_title_id varchar(10) NOT NULL,
  birth_date varchar(10) NOT NULL,
  first_name varchar(20) NOT NULL,
  last_name varchar(20) NOT NULL,
  sex varchar(5) NOT NULL,
  hire_date varchar(10) NOT NULL,
  no_of_projects int NOT NULL,
  last_performance_rating varchar(10) NOT NULL,
  left_org int NOT NULL,
  last_date varchar(10),
  FOREIGN KEY (emp_title_id) REFERENCES Titles(title_id));
  
  <!-- make sure not to name any tables with MySQL reserved keywords -->
  
  create table Salaries_jes(
  emp_no int NOT NULL,
  salary bigint NOT NULL, 
  FOREIGN KEY (emp_no) REFERENCES Employees(emp_no));
  
  create table Departments_jes(
  dept_no varchar(20) PRIMARY KEY NOT NULL ,
  dept_name varchar(30) NOT NULL);
  
  CREATE TABLE Department_Employees_jes(
  emp_no int NOT NULL,
  dept_no varchar(20) NOT NULL,
  FOREIGN KEY (emp_no) REFERENCES Employees(emp_no),
  FOREIGN KEY (dept_no) REFERENCES Departments(dept_no)); 
  
  create table Department_Managers_jes(
  dept_no varchar(20) NOT NULL,
  emp_no int NOT NULL,
  FOREIGN KEY (dept_no) REFERENCES Departments(dept_no),
  FOREIGN KEY (emp_no) REFERENCES Employees(emp_no)); 
  ```
  4) Data ingestion
  ```
  load data local infile '/home/anabig114238/titles.csv' into table Titles_jes
  fields terminated by ','
  ignore 1 rows;
  
  load data local infile '/home/anabig114238/employees.csv' into table Employees_jes
  fields terminated by ','
  ignore 1 rows;
  
  load data local infile '/home/anabig114238/salaries.csv' into table Salaries_jes
  fields terminated by ','
  ignore 1 rows;
  
  load data local infile '/home/anabig114238/departments.csv' into table Departments_jes
  fields terminated by ','
  ignore 1 rows;
  
  load data local infile '/home/anabig114238/dept_emp.csv' into table Department_Employees_jes
  fields terminated by ','
  ignore 1 rows;
  
  load data local infile '/home/anabig114238/dept_manager.csv' into table Department_Managers_jes
  fields terminated by ','
  ignore 1 rows;
  ```
  
#### Step-2: Scoop tanfer job to do a Scoop import from local linux machine to the hadoop HDFS in avro format.
```
sqoop import-all-tables  --connect jdbc:mysql://ip-10-1-1-204.ap-south-1.compute.internal:3306/anabig114238 
--username anabig114238 --password Bigdata123 --compression-codec=snappy --as-avrodatafile 
--warehouse-dir=/user/anabig114238/hive/warehouse/jes --m 1 --driver com.mysql.jdbc.Driver
```

#### Step-3: Hive database and table creation and data ingestion.
```
<!-- check if all files were imported successfully from the Scoop tranfer -->
ls -l /home/anabig114238/*.avsc

<!-- create a directory dedicated to the project in HDFS -->
hadoop fs -mkdir /user/anabig114238/csp1_jes

<!-- transfer all the avro schema data files from the Linux to HDFS using the -put command -->
hadoop fs -put /home/anabig114211/Titles_jes.avsc /user/anabig114211/csp1_jes/Titles_jes.avsc

hadoop fs -put /home/anabig114211/Salaries_jes.avsc /user/anabig114211/csp1_jes/Salaries_jes.avsc

hadoop fs -put /home/anabig114211/Departments_jes.avsc /user/anabig114211/csp1_jes/Departments_jes.avsc

hadoop fs -put /home/anabig114211/Department_Employees_jes.avsc /user/anabig114211/csp1_jes/Department_Employees_jes.avsc

hadoop fs -put /home/anabig114211/Department_Managers_jes.avsc /user/anabig114211/csp1_jes/Department_Managers_jes.avsc

hadoop fs -put /home/anabig114211/Employees_jes.avsc /user/anabig114211/csp1_jes/Employees_jes.avsc

hdfs dfs -ls /user/anabig114211/csp1_jes

<!-- Hive table creation using the avro schema files -->
CREATE EXTERNAL TABLE Departments_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Departments_jes.avsc');

CREATE EXTERNAL TABLE Employees_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Employees_jes.avsc');

CREATE EXTERNAL TABLE Salaries_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Salaries_jes.avsc');

CREATE EXTERNAL TABLE Titles_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Titles_jes.avsc');

CREATE EXTERNAL TABLE Dept_emp_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Dept_emp_jes.avsc');

CREATE EXTERNAL TABLE Dept_manager_jes
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
location "/user/anabig114238/hive/warehouse/Departments_jes"
TBLPROPERTIES ('avro.schema.url'='/user/anabig114238/csp1_jes/Dept_manager_jes.avsc');

<!-- check the database for tables -->
show tables;

<!-- check the talbes for data -->
select * from departments_jes;

select * from employees_jes;

select * from titles_jes;

select * from dept_emp_jes;

select * from dept_manager_jes;

select * from salaries_jes;
```

#### Step-4: EDA (Exploratory Data Analysis) in Impala.
![This is snapshot of HUE Impala](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/00%20-%20Impala.png)

##### 1) A list showing employee number, last name, first name, sex, and salary for each employee1. 
```
select e.emp_no, e.last_name, e.first_name, e.sex, s.salary from employees_jes e
JOIN salaries_jes s on e.emp_no = s.emp_no;
```
![Query 1 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/01.png)
![Query 1 output Scatterplot by sex](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/01_1.png)

##### 2) A list showing first name, last name, and hire date for employees who were hired in 1986.
```
select first_name, last_name, hire_date from employees_jes 
where hire_date between '1/1/1986' and '12/31/1986'
order by hire_date;
```

##### 3) A list showing the manager of each department with the following information: department number, department name, the manager's employee number, last name, first name.
```
select d.dept_no, d.dept_name, dm.emp_no, e.last_name, e.first_name from departments_jes d 
join dept_manager_jes dm on d.dept_no = dm.dept_no join employees_jes e
on dm.emp_no = e.emp_no;
```

##### 4) A list showing the department of each employee with the following information: employee number, last name, first name, and department name.
```
select de.emp_no, e.last_name, e.first_name, d.dept_name from dept_emp_jes de
join employees_jes e on de.emp_no = e.emp_no 
join departments_jes d on de.dept_no = d.dept_no;
```

##### 5) A list showing first name, last name, and sex for employees whose first name is "Hercules" and last names begin with "B".
```
select first_name, last_name, sex from employees_jes 
where first_name = 'Hercules' and last_name Like 'B%';
```

##### 6) A list showing all employees in the Sales department, including their employee number, last name, first name, and department name.
```
select e.emp_no, e.last_name, e.first_name, d.dept_name 
from employees_jes e
join dept_emp_jes de
on de.emp_no = e.emp_no
join departments_jes d on de.dept_no = d.dept_no
where d.dept_no like '%d007%';
```

##### 7) A list showing all employees in the Sales and Development departments, including their employee number, last name, first name, and department name.
```
select e.emp_no, e.last_name, e.first_name, d.dept_name 
from employees_jes e
join dept_emp_jes de
on de.emp_no = e.emp_no
join departments_jes d on de.dept_no = d.dept_no
where d.dept_no like '%d007%' or d.dept_no like '%d005%';
```

##### 8) A list showing the frequency count of employee last names, in descending order. ( i.e., how many employees share each last name).
```
select last_name, COUNT(last_name) as lm_frq from employees_jes
group by last_name order by lm_frq DESC;
```

##### 9) Histogram to show the salary distribution among the employees.
```
select salary, count(*) as no_emp from salaries_jes group by salary;

select avg(salary), count(*) as no_emp from salaries_jes group by salary;
```
![Query 9 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/09.png)
![Query 9 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/09_1.png)
![Query 9 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/09_2.png)
![Query 9 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/09_3.png)

##### 10) Bar graph to show the Average salary per title (designation).
```
select t.title, avg(s.salary) as avg_salary  from employees_jes e 
join salaries_jes s on e.emp_no = s.emp_no 
join titles_jes t on t.title_id = e.emp_title_id group by t.title;
```
![Query 10 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/10.png)

##### 11) List of all employees with the designation, joining three tables.
```
elect e.emp_no, e.first_name, e.last_name, s.salary, t.title from employees_jes e 
inner join (select emp_no, MAX(salary) as salary from salaries_jes group by emp_no) s 
on s.emp_no = e.emp_no inner join titles_jes t on t.title_id = e.emp_title_id;
```

##### 13) Second largest salary from all the employees.
```
selectc max(salary) from salaries_jes where salary not in (select max(salary) from salaries_jes) limit 1;
```

##### 14) Max salary by job title.
```
select e.emp_no, e.first_name, e.last_name, s.salary, t.title from employees_jes e
inner join (selet emp_no, max(salary) as salary from salaries_jes group by emp_no) s on s.emp_no = e.emp_no
inner join titles_jes t on t.title_id = e.emp_title_id;
```
![Query 14 output](https://github.com/Mercurial-Unicorn/alabs_capstone_project1/blob/main/EDA_Outputs/14.png)

#### Step-5: EDA (Exploratory Data Analysis) in Pyspark SQL.
please refer to ![Jupyter notebook for PySpark]()

### Challenges Faced:
    - Data type inconsistencies between different versions of tools in the technology stack.
    - Scoop import data failing repeatedly due to lab/internet issues.
    - Version control between all versions of tools that were used.
    
### Future Work:
    - More sophisticated quesries 
    - More number of analysis 
    - Creation of a general purpose dashboard for HR Employee Data for other companies to use
