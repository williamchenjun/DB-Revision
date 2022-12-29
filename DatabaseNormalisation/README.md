# Database Normalisation

> **Disclaimer** : These are notes on a KCL Database course I'm taking (module ID below). The example database is taken from the lectures. This page was created only for the purpose of revising in preparation for the winter exams.<br>
> **KCL Module Ref.**: 7CCSMDDW.
## Introduction
We will look at database normalisation and how to fix bad table designs. We will go through 1NF up to 3NF (including Boyce-Codd normal form).

The following is the relational table we will be working on.
> __Note__: Assume that a project (at least one) is always assigned to an employee.

**Schema (MySQL v5.7)**
```mysql
CREATE TABLE IF NOT EXISTS EMPLOYEE(
empid INT NOT NULL,
name VARCHAR(255) NOT NULL,
job VARCHAR(150) NOT NULL,
salary INT NOT NULL,
project VARCHAR(255) NOT NULL,
manager VARCHAR(255) NOT NULL,
prlength INT NOT NULL,
prcost INT NOT NULL
);

INSERT INTO EMPLOYEE VALUES(1, "Jones", "engineer", 35000, "Gherkin", "Davis", 5, 500);
INSERT INTO EMPLOYEE VALUES(2, "Wilson", "sales", 28000, "Tunnel", "Fulton", 6, 250);
INSERT INTO EMPLOYEE VALUES(1, "Jones", "engineer", 35000, "Tunnel", "Fulton", 6, 250);
INSERT INTO EMPLOYEE VALUES(3, "Peters", "tech", 24000, "Gherkin", "Davis", 5, 500);
INSERT INTO EMPLOYEE VALUES(3, "Peters", "tech", 24000, "Dome", "Mandelson", 2, 2000);
INSERT INTO EMPLOYEE VALUES(4, "Price", "runner", 17500, "Dome", "Mandelson", 2, 2000);
INSERT INTO EMPLOYEE VALUES(5, "Dollis", "designer", 45000, "Airport", "Craig", 5, 500);
```

**Query #1**
```mysql
SELECT * 
FROM EMPLOYEE
ORDER BY empid, project;
```

| empid | name   | job      | salary | project | manager   | prlength | prcost |
| ----- | ------ | -------- | ------ | ------- | --------- | -------- | ------ |
| 1     | Jones  | engineer | 35000  | Gherkin | Davis     | 5        | 500    |
| 1     | Jones  | engineer | 35000  | Tunnel  | Fulton    | 6        | 250    |
| 2     | Wilson | sales    | 28000  | Tunnel  | Fulton    | 6        | 250    |
| 3     | Peters | tech     | 24000  | Dome    | Mandelson | 2        | 2000   |
| 3     | Peters | tech     | 24000  | Gherkin | Davis     | 5        | 500    |
| 4     | Price  | runner   | 17500  | Dome    | Mandelson | 2        | 2000   |
| 5     | Dollis | designer | 45000  | Airport | Craig     | 5        | 500    |

### Example of an anomaly

**Query #2**
```mysql
UPDATE EMPLOYEE
SET project = "Dome"
WHERE empid = 1;
```

**Query #3**
```mysql
SELECT * 
FROM EMPLOYEE
ORDER BY empid, project;
```

| empid | name   | job      | salary | project | manager   | prlength | prcost |
| ----- | ------ | -------- | ------ | ------- | --------- | -------- | ------ |
| 1     | Jones  | engineer | 35000  | Dome    | Davis     | 5        | 500    |
| 1     | Jones  | engineer | 35000  | Dome    | Fulton    | 6        | 250    |
| 2     | Wilson | sales    | 28000  | Tunnel  | Fulton    | 6        | 250    |
| 3     | Peters | tech     | 24000  | Dome    | Mandelson | 2        | 2000   |
| 3     | Peters | tech     | 24000  | Gherkin | Davis     | 5        | 500    |
| 4     | Price  | runner   | 17500  | Dome    | Mandelson | 2        | 2000   |
| 5     | Dollis | designer | 45000  | Airport | Craig     | 5        | 500    |

We can observe that a (theoretical) update anomaly occurred. We also take notice of the redundancy of some columns in this table. Each time a project is assigned to an employee, their name, job and salary is reinserted into the table again. 

## First Normal Form (1NF)

The table above is already in first normal form (1NF). The requirement to be in 1NF is that each row of a non-key column must be atomic. That is, we only allow one entry per row. 

## Second Normal Form (2NF)

We check if the table above is in second normal form (2NF). That is, there can be no non-prime attributes (attributes that are not part of any of the candidate keys) that only **partially** depend on the candidate key. In otherwords, all the non-prime columns must depend fully on the candidate key.

We start by finding the candidate keys of our relational table. Remember that the candidate key must be unique and define a single row.

Some of the functional dependencies that are present in our table are:
- empid &xrarr; {name, job}
- project &xrarr; {manager, prlength, prcost}
- prlength &xrarr; prcost
- job &xrarr; salary

We use these functional dependencies to find the closure of each set of attributes. The **minimal superkey**, that is, the set with the least number of attributes on the LHS, is going to be our candidate key:
- {empid}<sup>+</sup> = {empid, name, job}
- {name}<sup>+</sup> = {name}
- {job}<sup>+</sup> = {job, salary}
- {salary}<sup>+</sup> = {salary}
- {project}<sup>+</sup> = {project, manager, prlength, prcost}
- {manager}<sup>+</sup> = {manager}
- {prlength}<sup>+</sup> = {prcost}
- {prcost}<sup>+</sup> = {prcost}
- {empid, job}<sup>+</sup> = {empid, name, job, salary}
- {empid, project}<sup>+</sup> = {empid, name, project, manager, prlength, prcost}
- {empid, job, project}<sup>+</sup> = {empid, name, job, salary, project, manager, prlength, prcost}

I skipped a few combinations of attributes, but we can see that {empid, job, project} is a candidate key. Their union defines all the attributes in the table. However, this candidate key does not make our table 2NF since we can see that there are partial dependencies.

To turn this table to 2NF, we need to decompose it. We will do this when we convert it to BCNF.

## Third Normal Form (3NF) and Boyce-Codd Normal Form (BCNF)

We want to check that it's in 3NF. To be in the 3NF we have two requirements:
1. The table must be in 2NF.
2. There cannot be any dependencies between non-prime and any other attributes other than the entire candidate key. 

Additionally, for a table to be in BCNF we require:
1. The table must be in 3NF.
2. The determinant of every functional dependency is a candidate key.

The table is clearly not in 3NF as it is not even in 2NF. What we will do is a process called "**decomposition**". We will be separating this table into different tables in order to make them all BCNF.

To start the decomposition process, we select a functional dependency and create a table using the determinant as the key and the rest as non-prime attributes.

We start with {job} &xrarr; {salary}. We create a new relation 

<p align="center">E<sub>1</sub>(:key: job, salary).</p> 

We then remove all the non-key attributes of E<sub>1</sub> from the employee table and create a new relation

<p align="center">E<sub>2</sub>(empid, name, job, project, manager, prlength, prcost).</p>

To make sure that this decomposition did not make us lose any information, we need to check that the union of the attributes of both tables make up the original table, and the natural join also returns the original table.

In fact,

<p align="center">
{salary} &cup; {empid, name, job, project, manager, prlength, prcost} = Employee
</p>

and 

<p align="center">
  E<sub>1</sub> &bowtie; E<sub>2</sub> = Employee
</p>

(**Reminder**: The natural join merges two tables by the common columns and common rows)

We check if any of the new tables are in BCNF. We know that E<sub>1</sub> is in 1NF since the rows are atomic, it's in 2NF since there are no partial dependencies. It's also in 3NF since the salary depends fully on the key and there is no transitive dependency. It's also in BCNF because there is only one possible FD and the dependant is the candidate key. However, we notice that E<sub>2</sub> is not even in 2NF. We still have partial dependencies.

We repeat the same procedure and create two new tables E<sub>3</sub> and E<sub>4</sub>. We pick empid &xrarr; {name, job}. Thus, 

<p align="center">
  E<sub>3</sub>(:key: empid, job, name),<br>
  E<sub>4</sub>(empid, project, manager, prlength, prcost).
</p>

So now we have three tables: E<sub>1</sub>, E<sub>3</sub> and E<sub>4</sub>. We see that E<sub>3</sub> is in 1NF and 2NF since every non-key attribute depends fully on empid. It's also in 3NF since every non-key attribute depends fully on the candidate key and there is no FD between name and job. It's also in BCNF as the dependant is always the candidate key. E<sub>4</sub> is still not in 2NF. We repeat the process again.

We now consider the FD {project} &xrarr; {manager, prlength, prcost}:
<p align="center">
  E<sub>5</sub>(:key: project, manager, prlength, prcost),<br>
  E<sub>6</sub>(empid, project).
</p>
Now E<sub>6</sub> is definitely in 3NF and BCNF, since each project depends on the empid and there are are no other FDs. However, E<sub>5</sub> is not even in 2NF because we know that prlength &xrarr; prcost. Thus, we decompose E<sub>5</sub> into E<sub>7</sub> and E<sub>8</sub>:
<p align="center">
  E<sub>7</sub>(:key: prlength, prcost),<br>
  E<sub>8</sub>(:key: project, manager, prlength).
</p>

So, to recap we now have the following decomposed tables: E<sub>1</sub>, E<sub>3</sub>, E<sub>6</sub>, E<sub>7</sub> and E<sub>8</sub>. We can see that E<sub>8</sub> is finally in 3NF since there are no partial dependencies and every non-key attributes depend fully on the candidate key. It's also in BCNF since the dependant of every FD is the candidate key.  E<sub>7</sub> is obviously also in BCNF.

Now we check if we lost any information while decomposing the tables:

1. The union must equal the original table <p align="center">{job, salary} &cup; {empid, job, name} &cup; {empid, project} &cup; {prlength, prcost} &cup; {project, manager, prlength} = {empid, name, job, salary, project, manager, prlength, prcost}.</p>
2. The natural join must equal the original table (I won't be showing this but it is true).

Therefore, we have successfully decomposed our original table, which was not even in 2NF, to five new tables which are all in BCNF.
<p align="center">
<img src="https://user-images.githubusercontent.com/79821802/208998358-f9314e43-f5ba-4899-8211-ba99a938fa5c.svg"/>
</p>
