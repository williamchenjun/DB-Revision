# Database Mock Exam

**Q1**. Given the relation $R(A,B,C,D,E,F,G,H)$ and the set of functional dependencies
```math
\{ A \rightarrow BD,\; DH \rightarrow CF,\; C \rightarrow B,\; BF \rightarrow H,\; H \rightarrow G \}
```
is the functional dependency $AF \rightarrow G$ implied?

**Solution**: The answer is **YES**. We can see that 
```math
\begin{align*}
\because A &\rightarrow BD \implies \begin{cases}A \rightarrow B\\A \rightarrow D\end{cases}\\[1em]
AF &\rightarrow BF \implies AF \rightarrow H \implies AF \rightarrow G.
\end{align*}
```

We are given the following database schema:

- `employee(empid, lname, location, salary, manager)`
- `project(projectID, projectName, projectLeader, budget)`
- `projectemployees(contractID, empID(FK), projectID(FK), contract_length)`

**Q2**. List the last name of the employees based in London who have the lowest salary. Note that more than one employee might have the lowest salary if they have the same salary.

**Solution**: 

```mysql
SELECT lname
FROM employee
WHERE salary = (
  SELECT MIN(salary)
  FROM employee
  WHERE location = "London"
);
```

**Q3**. Which employees are working on the most projects? 

**Solution**:

```mysql
SELECT empID, COUNT(projectID) AS NumberOfProjects
FROM projectemployees
GROUP BY empID
HAVING COUNT(projectID) = (
  SELECT MAX(P) 
  FROM (
    SELECT empID, COUNT(projectID) AS P
    FROM projectemployees 
    GROUP BY empID
  )
);
```

We could also make it better by performing an inner join with the employee table to access more information about the employee:

```mysql
SELECT employees.empID, employees.lname, employees.fname, COUNT(projectemployees.projectID) AS NumberOfProjects
FROM projectemployees INNER JOIN employees ON projectemployees.empID = employees.empID
GROUP BY projectemployees.empID
HAVING COUNT(projectemployees.projectID) = (
  SELECT MAX(P) 
  FROM (
    SELECT empID, COUNT(projectID) AS P
    FROM projectemployees 
    GROUP BY empID
  )
);
```
