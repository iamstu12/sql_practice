
# SQL Lab
### README and Code Solutions
#### by Stuart McColl
<br>

-


### Q1 - Are there any **pay_details** records lacking both a **'local account no'** and **'iban number'**?

`SELECT 
  COUNT(id) AS num_pay_details_without_local_acc_and_iban
FROM pay_details
WHERE local_account_no IS NULL AND iban IS NULL`

<br>

---

### Q2 - Get a table of employees **first_name**, **last_name** and **country**, ordered alphabetically first by **country** and then by **last_name** (put any NULLs last).

`SELECT 
  first_name, 
  last_name, 
  country
FROM employees
ORDER BY country ASC NULLS LAST, last_name ASC NULLS LAST`

<br>

---

### Q3 - Find the details of the top ten highest paid employees in the corporation.

`SELECT *
FROM employees
ORDER BY salary DESC NULLS LAST
LIMIT 10`

<br>

---

### Q4 - Find the **first_name**, **last_name** and salary of the lowest paid employee in Hungary.

`SELECT 
  first_name, 
  last_name, 
  salary
FROM employees
WHERE country = 'Hungary'
ORDER BY salary ASC NULLS LAST
LIMIT 1`

<br>

---

### Q5 - Find all the details of any employees with a ‘yahoo’ email address?

`SELECT *
FROM employees
WHERE email ILIKE '%yahoo%'`

<br>

---

### Q6 - Provide a breakdown of the numbers of employees enrolled, not enrolled, and with unknown enrolment status in the corporation pension scheme.

`SELECT 
  pension_enrol AS pension_enrolled, 
  COUNT(id) AS num_employees
FROM employees
GROUP BY pension_enrol`

<br>

---

### Q7 - What is the maximum salary among those employees in the ‘Engineering’ department who work 1.0 full-time equivalent hours (fte_hours)?

`SELECT 
  MAX(salary) AS max_salary_engineering_1_ftehours
FROM employees
WHERE department = 'Engineering' AND fte_hours = 1.0`

<br>

---

### Q8 - Get a table of country, number of employees in that country, and the average salary of employees in that country for any countries in which more than 30 employees are based. Order the table by average salary descending.

`SELECT 
  country, 
  COUNT(id) AS num_employees, 
  AVG(salary) AS average_salary
FROM employees
GROUP BY country
HAVING COUNT(id) > 30
ORDER BY average_salary DESC`

<br>

---

### Q9 - Return a table containing each employees **first_name**, **last_name**, **full-time equivalent hours** (fte_hours), **salary**, and a new column **effective yearly salary** which should contain **'fte hours'** multiplied by **salary**.

`SELECT 
  first_name,
  last_name,
  fte_hours,
  salary,
  fte_hours * salary AS effective_yearly_salary
FROM employees`

<br>

---

### Q10 - Find the first name and last name of all employees who lack a 'local tax code'.

`SELECT 
  first_name, 
  last_name
FROM employees AS e INNER JOIN pay_details AS pd
ON e.pay_detail_id = pd.id
WHERE pd.local_tax_code IS NULL`

<br>

---

### Q11 - The 'expected profit' of an employee is defined as (48 * 35 * charge cost - salary) * fte hours, where charge cost depends upon the team to which the employee belongs. Get a table showing expected_profit for each employee.

`SELECT 
  e.id,
  e.first_name,
  e.last_name,
  e.fte_hours,
  e.salary,
  t.name,
  t.charge_cost,
  (48 * 35 * t.charge_cost::INT - e.salary) * e.fte_hours
    AS expected_profit
FROM employees AS e LEFT JOIN teams AS t
ON e.team_id = t.id`

<br>

---

### Q12 Get a list of the id, **first_name**, **last_name**, **salary** and **fte_hours** of employees in the largest department. Add two extra columns showing the ratio of each employee’s salary to that department’s average salary, and each employee’s **fte_hours** to that department’s average **fte_hours**.

`WITH biggest_dept(name, avg_salary, avg_fte_hours) AS (
  SELECT 
     department,
     AVG(salary),
     AVG(fte_hours)
  FROM employees
  GROUP BY department
  ORDER BY COUNT(id) DESC NULLS LAST
  LIMIT 1
)
SELECT
  e.id,
  e.first_name,
  e.last_name, 
  e.salary / bd.avg_salary AS salary_over_dept_avg,
  e.fte_hours / bd.avg_fte_hours AS fte_hours_over_dept_avg
FROM employees AS e CROSS JOIN biggest_dept AS bd
WHERE department = bd.name`

<br>

---

### Q13 - Return a table of those employee **first_names** shared by more than one employee, together with a count of the number of times each **first_name** occurs. Omit employees without a stored **first_name** from the table. Order the table descending by **count**, and then alphabetically by **first_name**.

`SELECT 
  first_name, 
  COUNT(*) AS name_count
FROM employees
WHERE first_name IS NOT NULL
GROUP BY first_name 
HAVING COUNT(*) > 1
ORDER BY COUNT(*) DESC, first_name ASC`

<br>

---

### Q14 - Have a look again at your table for core question 6. It will likely contain a blank cell for the row relating to employees with ‘unknown’ pension enrolment status. This is ambiguous: it would be better if this cell contained ‘unknown’ or something similar. Can you find a way to do this, perhaps using a combination of COALESCE() and CAST(), or a CASE statement?

`SELECT 
  COALESCE(CAST(pension_enrol AS VARCHAR), 'unknown') AS pension_enrolled, 
  COUNT(id) AS num_employees
FROM employees
GROUP BY pension_enrol`

<br>

---

### Q15 - Find the first name, last name, email address and start date of all the employees who are members of the ‘Equality and Diversity’ committee. Order the member employees by their length of service in the
### company, longest first.

`SELECT 
  e.first_name, 
  e.last_name, 
  e.email, 
  e.start_date
FROM 
employees AS e INNER JOIN employees_committees AS ec
ON e.id = ec.employee_id
INNER JOIN committees AS c
ON ec.committee_id = c.id
WHERE c.name = 'Equality and Diversity'
ORDER BY e.start_date ASC NULLS LAST`

<br>

---

### Q16 - Use a CASE() operator to group employees who are members of committees into salary_class of 'low' (salary < 40000) or 'high' (salary >= 40000). A NULL salary should lead to 'none' in salary_class. Count the number of committee members in each salary_class.

`SELECT 
  CASE 
    WHEN e.salary < 40000 THEN 'low'
    WHEN e.salary IS NULL THEN 'none'
    ELSE 'high' 
  END AS salary_class,
  COUNT(DISTINCT(e.id)) AS num_committee_members
FROM employees AS e INNER JOIN employees_committees AS ec
ON e.id = ec.employee_id
INNER JOIN committees AS c
ON ec.committee_id = c.id
GROUP BY salary_class`

<br>

-








































