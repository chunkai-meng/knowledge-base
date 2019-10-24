# postgresql duplicate key violates unique constraint

```sql
SELECT MAX(id) FROM accounting_income;
Select setval(pg_get_serial_sequence('accounting_income', 'id'),(SELECT MAX(id) FROM accounting_income));
```

[source](https://stackoverflow.com/questions/4448340/postgresql-duplicate-key-violates-unique-constraint)