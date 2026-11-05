---
name: dende-sql
description: "Lay out SQL in Dende repositories: one clause per line, aliases with AS, one condition per line, and the why-comment for planner hints. Use when you write or change a query, a view, a function or a migration."
---

# Layout

One clause per line. A space after each comma and around each operator. `AS` for each alias.
Each extra condition on its own line, with the operator at the start of the line.

```sql
-- Bad
SELECT ue.id,ue.status FROM user_examspaces ue
JOIN examspaces e ON e.id=ue.examspace_id AND e.examspace_type='REGULAR'

-- Good
SELECT ue.id, ue.status
FROM user_examspaces AS ue
JOIN examspaces AS e
  ON e.id = ue.examspace_id
 AND e.examspace_type = 'REGULAR'
WHERE ue.deleted_at IS NULL
ORDER BY ue.created_at DESC
LIMIT 20
```

# Rules

- Name the columns. No `SELECT *` outside an `EXISTS` subquery.
- Keywords in upper case, identifiers in lower case.
- Pass values as parameters, never by string formatting.
- A planner hint, such as `OFFSET 0` on a subquery, gets a one-line comment that says why.
- Test a query through its use case and assert the rows, not the SQL text.
