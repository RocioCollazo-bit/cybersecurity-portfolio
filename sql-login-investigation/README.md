# 🔐 Rocio's SQL & Cybersecurity Portfolio

Welcome to my SQL & Cybersecurity Portfolio repository!! 🚀

This repository showcases hands-on SQL investigations, cybersecurity-focused projects, and practical data analysis exercises designed to demonstrate my growing experience in SQL, security operations, and threat analysis. 📊

---

# 📝 Table of Contents

- Overview
- SQL Investigation Projects
- Skills Demonstrated
- Technologies Used
- Project Structure
- Future Projects
- Usage Instructions
- Contact

---

# 📊 Overview

This repository contains practical SQL projects focused on cybersecurity investigations and security-related data analysis.

The projects simulate real-world scenarios such as:
- suspicious login investigations
- failed authentication analysis
- employee security audits
- foreign login detection
- threat monitoring

The goal of these projects is to strengthen SQL skills while applying cybersecurity investigation techniques in realistic environments. 🎯

---

# 🔎 SQL Investigation Projects

## 🛡️ SQL Login Investigation

A cybersecurity-focused SQL project designed to investigate suspicious login activity within an organization.

### Key Investigation Areas

- Failed login attempts
- After-hours authentication activity
- Foreign login detection
- Suspicious IP analysis
- Employee security filtering
- Threat pattern identification

### Skills Demonstrated

- SQL filtering
- AND / OR / NOT operators
- LIKE operator
- GROUP BY & HAVING
- DISTINCT queries
- ORDER BY sorting
- Threat analysis
- Security investigation logic
- Database documentation

---

# 📂 Project Structure

```bash
sql-login-investigation/
│
├── README.md
├── queries.sql
├── report.md
└── screenshots/
```

---

# 📸 Investigation Preview

## Example Query

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00:00'
  AND success = FALSE;
```

### Purpose

This query identifies failed login attempts that occurred after business hours.

---

# 🧠 Key Findings

- Multiple failed login attempts were detected outside business hours.
- Several authentication attempts originated from foreign countries.
- Suspicious IP ranges were identified during the investigation.
- Repeated failed attempts may indicate brute-force activity.

---

# 🛠️ Technologies Used

| Tool | Purpose |
|---|---|
| SQL | Data filtering & investigation |
| SQLite | Database management |
| GitHub | Documentation & portfolio |
| Cybersecurity Analysis | Threat investigation |
| Markdown | Technical documentation |

---

# 📚 Future Projects

Upcoming projects planned for this repository:

- 📡 Network Traffic Investigation
- 🔐 Employee Access Audit
- 📊 Security Dashboard Queries
- 🕵️ Phishing Investigation Dataset
- 🌐 VPN Access Monitoring
- ⚠️ Suspicious IP Intelligence Analysis

---

# 👩🏻‍💻 Usage Instructions

To explore this repository:

1. Open the project folders
2. Review the SQL queries
3. Analyze the investigation logic
4. Explore the screenshots and documentation
5. Run the SQL scripts in SQLite or MySQL environments

---

# 📬 Contact

Feel free to connect with me on LinkedIn (https://www.linkedin.com/in/rociocollazo/) or explore my GitHub projects for more cybersecurity and SQL investigations. 🚀

---

# ⭐ Support

If you found these projects useful or interesting, feel free to star the repository ⭐

These projects are part of my continuous learning journey in cybersecurity, SQL analysis, and security operations.
