# Splunk Labs: SOC Analyst Practice

Hands-on Splunk labs built while following the **Cisco Networking Academy Cybersecurity Defense Analyst** career path. The goal is to learn SPL (Search Processing Language) by writing real searches against real application logs, from my own projects and, later, from simulated attack traffic.

## Goals

- Search, filter and summarize security logs in Splunk
- Write SPL queries by hand
- Investigate suspicious activity and explain findings like a junior SOC analyst
- Practice detections on authentication events (failed logins, verification steps, devices, IP addresses)
- Build dashboards and alerts
- Keep a record of what was learned and how confidence improved

## Environment

| Item | Value |
|---|---|
| Splunk edition | Splunk Enterprise (local install) |
| Web address | `http://127.0.0.1:8000` |
| App | Search & Reporting |
| Source type | `csv` (exported from PostgreSQL) |
| Upload path | Settings → Add Data → Upload |

**Tip:** set the time picker to **All time**. Events are dated July 2026 and will not show with the default range.

## Datasets

| Index | Source app | Status | Main practice value |
|---|---|---|---|
| `scholarhub_auditlogs` | ScholarHub | Loaded | Fundamentals, user activity, deletions, login vs logout |
| `jirfux_auditlogs` | Jirflux | To be loaded | Failed password attempts, email verification, device verification |
| Bot / attack logs (index name TBD) | Simulated bots | Future | IP-based analysis, brute force, scanning, rate patterns |


To search several datasets at once:

```
(index="scholarhub_auditlogs" OR index="jirfux_auditlogs") action=login
| stats count by index, username
```

**Practice ideas:**

- Count failed password attempts per user
- Find users with failed attempts followed by a successful login
- Compare email verification and device verification outcomes
- Spot logins from new or unverified devices
- Top source IPs by request or failed-login count
- Many usernames tried from one IP (password spraying)
- Many attempts on one username from many IPs (distributed brute force)
- Bursts over short time windows with `timechart` or `bin`
- IP allow/block list lookups and simple geolocation with `iplocation`

**Fields wanted in the logs:** timestamp, source IP, username, action, success/failure, user agent, device ID.

## General notes

- Field names are case-sensitive (`action` works, `Action` does not). Field values are not (`deleted` matches `Deleted`).
- Results from `table`, `stats` and similar commands appear under the **Statistics** tab, not **Events**.
- Every dataset may use different field names. Check with `| fieldsummary` or the field sidebar before writing searches.


**Golden rule:** filter first, then calculate, then present.

```
index="scholarhub_auditlogs" action=login     <- filter
| stats count by username                     <- calculate
| sort - count                                <- present
```

---

## Lab 1: ScholarHub User Activity Review

**Concepts:** index = database, event = row, field = column, `|` passes output of one command to the next.

**What I learned:** the PostgreSQL table is `audit_log`, but the Splunk index is `scholarhub_auditlogs`.

**Mistakes:** used `index="audit_log"` (the PostgreSQL table name).


### Lesson 2: Searching and filtering (`WHERE`)

```
index="scholarhub_auditlogs" action="login"
index="scholarhub_auditlogs" action="deleted" username="jira"
index="scholarhub_auditlogs" action IN (updated, added)
index="scholarhub_auditlogs" NOT action IN (logout, login)
```

**What I learned:** a space means AND, `IN` works like SQL, filters go before the first pipe, no command word is needed.

**Mistakes:** used `| filter` (not an SPL command); put filters after `stats ... by`.

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

## Planned labs

| Lab | Topic | Dataset | Status |
|---|---|---|---|
| Lab 1 | Splunk fundamentals: filtering, `table`, `stats` | ScholarHub | In progress |
| Lab 2 | Investigations: deletions, login vs logout gaps, user timelines | ScholarHub | Planned |
| Lab 3 | Time analysis with `eval`, `where` and `timechart` | ScholarHub | Planned |
| Lab 4 | Authentication analysis: failed passwords, email and device verification | Jirflux | Planned |
| Lab 5 | IP analysis: top talkers, brute force, spraying, geolocation | Bot logs | Planned |
| Lab 6 | Dashboards (one per lab) | All | Planned |
| Lab 7 | Alerts (for example, failed logins above a threshold per IP) | Jirflux, bot logs | Planned |
| Lab 8 | SOC workflow: write up findings as an analyst | All | Planned |


## How I organize my work

- **Saved reports:** named `Lab#-Lesson#-Number Description`, for example `Lab1-L2-01 Login events`.
- **Dashboards:** one per lab, such as `Lab 1 - Splunk Fundamentals`.
- **Journal:** this file. After each lesson I record queries, what I learned, mistakes, and a confidence score out of 5.
- **New datasets:** add a section under *Datasets* with its index, fields, event counts and known limits before starting labs on it.

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
