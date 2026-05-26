# SQL Login Investigation

## Project Overview

This project demonstrates how SQL filters can be used to investigate suspicious login attempts and employee data.

---

# Technologies Used

| Tool | Purpose |
|---|---|
| SQL | Data filtering |
| SQLite | Database management |
| GitHub | Documentation |
| Cybersecurity Analysis | Threat investigation |

---

# Database Table

## log_in_attempts

| event_id | username | login_date | login_time | country | success |
|---|---|---|---|---|---|
| 101 | jdoe | 2026-05-20 | 09:15 | Australia | TRUE |
| 102 | asmith | 2026-05-20 | 22:45 | Russia | FALSE |

---

# Investigation Query

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00:00'
  AND success = FALSE;
```

---

# Purpose of the Query

This query identifies failed login attempts that occurred after business hours.

---

# Skills Demonstrated

- SQL filtering
- Security investigation
- Data analysis
- AND / OR operators
- Cybersecurity documentation
