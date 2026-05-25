# SQL Login Investigation

## Objective
Investigate suspicious login attempts outside business hours.

## Tools Used
- SQL
- Database filtering

## Skills Demonstrated
- SQL queries
- Security investigation
- Log analysis

## Example Query

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00';
