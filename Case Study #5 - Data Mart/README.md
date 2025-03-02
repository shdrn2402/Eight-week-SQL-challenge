![Project Logo](../images/case5_logo.png)

## Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#case-study-questions)
  - [A. Data Cleansing Steps](#a-data-cleancing-steps)
  - [B. Data Exploration](#b-data-exploration)
  - [C. Before & After Analysis](#c-before--after-analysis)
  - [D. Bonus Question](#d-bonus-question)
- [Conclusion](#conclusion)

## Introduction
...

## Entity Relationship Diagram

<details>
  <summary><em><strong>show database schema*</strong></em></summary>

```SQL
...
```

**\*Note**:
1. Primary keys are not explicitly defined in the tables. This might be intentional due to the educational nature of the project:  
  - The data is artificially generated and static, minimizing the risk of integrity violations.  
  - In real-world scenarios, primary keys are essential to enforce data integrity and uniqueness.  

2. Data type inconsistencies are present in inserted values:
  - For example, the week_date column is defined as VARCHAR(7), while storing date-like values.
  - PostgreSQL implicitly converts values when necessary, but this practice is discouraged in production environments.
  - Explicit type casting should be used to ensure data consistency and prevent unexpected errors.

</details>

![Project Logo](../images/case4_diagram.png)


## Case Study Questions
### A. Data Cleansing Steps

---

### B. Data Exploration

---

### C. Before & After Analysis

---

### D. Bonus Question

---

### Conclusion

---