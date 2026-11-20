# Splunk Labs: SOC Analyst Practice 

Hands-on Splunk labs built while following the **Cisco Networking Academy Cybersecurity Defense Analyst** career path. The goal is to learn SPL (Search Processing Language) by writing real searches against real application logs

## Goals

- Search, filter and summarize security logs in Splunk
- Write SPL queries by hands
- Investigate suspicious activity and explain findings like a junior SOC analyst
- Build dashboards and alerts
- Keep a record of what was learned and how confidence improved

## Environment

| Item | Value |
|---|---|
| Splunk edition | Splunk Enterprise (local install) |
| Web address | `http://127.0.0.1:8000` |
| App | Search & Reporting |
| Index | `scholarhub_auditlogs, jirfux_auditlogs` |
| Source file | `audit_log.csv` (exported from PostgreSQL) |
| Source type | `csv` |

## Dataset: ScholarHub audit log & Jirflux Audit Logs

Exported from the PostgreSQL `audit_log` table and uploaded through **Settings → Add Data → Upload**. Splunk combined the separate `date` and `time` columns into `_time` automatically.

| Field | Meaning | Example |
|---|---|---|
| `_time` | Event time built by Splunk from `date` + `time` | 7/19/26 1:25:38 PM |
| `id` | Row number from the PostgreSQL table | 1 |
| `username` | User who performed the action | jira |
| `action` | What the user did | login, logout, register, Added, Updated, Deleted |
| `entity` | What was affected | credential |
| `user_id` | User's database ID | 1 |

**Dataset facts (verified in Splunk):**

- 387 events in total
- 11 distinct usernames
- Time picker must be set to **All time**, because the events are dated July 2026

| Action | Count |
|---|---|
| Added | 214 |
| login | 94 |
| logout | 51 |
| Updated | 16 |
| register | 10 |
| Deleted | 2 |

**Known limits of this dataset:** there are no IP addresses, device details, or login success/failure flags. Investigations that need those require extra logging in ScholarHub.

**Notes:**

- Field names are case-sensitive (`action` works, `Action` does not). Field values are not (`deleted` matches `Deleted`).
- Results from `table`, `stats` and similar commands appear under the **Statistics** tab, not **Events**.

## SQL to SPL cheat sheet

| SQL | SPL |
|---|---|
| `FROM audit_log` | `index="scholarhub_auditlogs"` |
| `WHERE action = 'login'` | `action=login` |
| `WHERE a = 'x' AND b = 'y'` | `a=x b=y` (a space means AND) |
| `WHERE action IN ('Added','Updated')` | `action IN (Added, Updated)` |
| `WHERE action != 'login'` | `action!=login` or `NOT action=login` |
| `WHERE username LIKE 'j%'` | `username=j*` |
| `SELECT username, action` | `\| table username action` |
| `SELECT username AS user` | `\| rename username AS user` |
| `SELECT DISTINCT username` | `\| dedup username \| table username` |
| `COUNT(*) ... GROUP BY username` | `\| stats count by username` |
| `COUNT(DISTINCT username)` | `\| stats dc(username)` |
| `ORDER BY COUNT(*) DESC` | `\| sort - count` |
| `LIMIT 3` | `\| head 3` |

**Golden rule:** filter first, then calculate, then present.

```
index="scholarhub_auditlogs" action=login     <- filter
| stats count by username                     <- calculate
| sort - count                                <- present
```

---

## Lab 1: Splunk Fundamentals

### Lesson 1: SQL vs SPL

**Concepts:** index = database, event = row, field = column, `|` passes output of one command to the next.

**What I learned:** the PostgreSQL table is `audit_log`, but the Splunk index is `scholarhub_auditlogs`.

**Mistakes:** used `index="audit_log"` (the PostgreSQL table name).

**Confidence:** _/5

### Lesson 2: Searching and filtering (`WHERE`)

```
index="scholarhub_auditlogs" action="login"
index="scholarhub_auditlogs" action="deleted" username="jira"
index="scholarhub_auditlogs" action IN (updated, added)
index="scholarhub_auditlogs" NOT action IN (logout, login)
```

**What I learned:** a space means AND, `IN` works like SQL, filters go before the first pipe, no command word is needed.

**Mistakes:** used `| filter` (not an SPL command); put filters after `stats ... by`.

**Confidence:** _/5

### Lesson 3: Choosing columns (`SELECT`)

```
index="scholarhub_auditlogs" action="login"
| table _time username action entity
| rename username AS user

index="scholarhub_auditlogs"
| dedup username
| table username

index="scholarhub_auditlogs" action="deleted"
| fields - user_id id
```

**What I learned:** `table` controls which columns show, `dedup` works like `DISTINCT`, `rename ... AS` works like SQL. Result: **11 distinct usernames**.

**Mistakes:** repeated one query instead of adding `rename`; forgot that `table` moves results to the Statistics tab.

**Confidence:** _/5

### Lesson 4: Counting and grouping (`GROUP BY`)

```
index="scholarhub_auditlogs"
| stats count by action

index="scholarhub_auditlogs"
| stats count by username
| sort - count
| head 3

index="scholarhub_auditlogs"
| stats dc(username)
```

**What I learned:** _(fill in after completing the exercises)_

**Mistakes:** _(fill in)_

**Confidence:** _/5

---

## Findings so far

- `login` (94) exceeds `logout` (51) by 43. Likely sessions that expired or tabs that were closed. Low concern, but a good example of an unexpected-but-probably-harmless pattern.
- Only 2 `Deleted` events exist, so each can be reviewed by hand.
- Most active user: _(fill in after Lesson 4)_

## Planned labs

| Lab | Topic | Status |
|---|---|---|
| Lab 1 | Splunk fundamentals: filtering, `table`, `stats` | In progress |
| Lab 2 | Investigations: deletions, login vs logout gaps, user timelines | Planned |
| Lab 3 | Time analysis with `eval`, `where` and `timechart` | Planned |
| Lab 4 | Dashboards (one per lab) | Planned |
| Lab 5 | Alerts | Planned |
| Lab 6 | SOC workflow: write up findings as an analyst | Planned |

## Progress tracker

- [x] Install Splunk Enterprise
- [x] Export PostgreSQL `audit_log` to CSV
- [x] Import CSV into index `scholarhub_auditlogs`
- [x] Verify timestamps and fields
- [x] First searches and filters
- [x] Lessons 1 to 3
- [ ] Lesson 4: `stats` and `GROUP BY`
- [ ] First dashboard panel
- [ ] Investigate the two deletions in full
- [ ] Add IP address and login status logging to ScholarHub

## How I organize my work

- **Saved reports:** named `Lab#-Lesson#-Number Description`, for example `Lab1-L2-01 Login events`.
- **Dashboards:** one per lab, such as `Lab 1 - Splunk Fundamentals`.
- **Journal:** this file. After each lesson I record queries, what I learned, mistakes, and a confidence score out of 5.

## Cisco learning path

| No. | Course | Status |
|---|---|---|
| 1 | The Cybersecurity Landscape | Completed |
| 2 | Understanding Threats and Attacks | Completed |
| 3 | Security Operations and the Defense Analyst | In progress |
| 4 | Introduction to Splunk | Completed |
| 5 | Data and Tools for Defense Analysts | Upcoming |
| 6 | The Art of Investigation | Upcoming |
| 7 | SOC Essentials: Investigating with Splunk | Upcoming |
| 8 | SOC Essentials: Introduction to Threat Hunting | Upcoming |
