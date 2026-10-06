# runbook-acme-prod-rds-free-storage-low

| | |
| --- | --- |
| **Customer** | Acme Corp (also known as: Acme Logistics) |
| **Alert** | `acme-prod-rds-free-storage-low`: CloudWatch alarm in the `acme-prod` AWS account, us-east-1 |
| **Default priority** | High. See [Priority](link-to-runbooks-landing-page#priority) |
| **Owner** | Alex Rivera |
| **Status** | Verified |
| **Last verified** | 2026-08-14 (live incident) |
| **Quick links** | [RDS storage dashboard](link) · [Postgres logs (last 1h)](link) · [RDS console: acme-prod-db](link) · [Getting access to acme-prod](link-to-access-doc#aws-console) |

# Summary

The production database (`acme-prod-db`, RDS PostgreSQL) has less than 10% of its disk free. If the disk fills, Postgres stops accepting writes and Acme's order system goes down. How much time you have depends on how fast free space is falling, so check that first.

# Acknowledge

**If Urgent or High:**

1. Post in `#acme-alerts`: "`acme-prod-rds-free-storage-low` fired. I'm on it." Include a link to the alarm.
2. Go to [Mitigate](#mitigate).

**If Medium:**

1. Create a ticket in the Acme support queue and link the alarm.
2. Go to [Diagnose](#diagnose).

**Raise to Urgent if:** free space is below 5%, the dashboard projects it will be full within 2 hours, or application logs show write errors (`could not extend file`, `No space left on device`).

**Lower to Medium if:** free space is falling slowly (more than 7 days until full at the current rate). This is most likely normal data growth. Plan the storage increase with Acme during business hours.

# Mitigate

1. **Check how fast free space is falling.**

   Open the [RDS storage dashboard](link) and set the range to the last 24 hours. Look at the `FreeStorageSpace` graph.

   - **Steep drop, projected full within a few hours:** this is Urgent. Go to step 2 now.
   - **Sawtooth pattern (drops, then recovers):** probably temporary files from a large query. See [Temporary files from large queries](#temporary-files-from-large-queries). The space usually comes back on its own.
   - **Slow, steady decline:** lower to Medium and go to [Diagnose](#diagnose).

2. **Check whether a storage change is already in progress.**

   ```
   aws rds describe-db-instances \
     --db-instance-identifier acme-prod-db \
     --query 'DBInstances[0].[DBInstanceStatus,AllocatedStorage]' \
     --profile acme-prod
   ```

   **Expected:** `["available", 500]` (status, then size in GiB).

   **If the status is `modifying` or `storage-optimization`:** someone has already increased storage. Don't start another change. Ask in `#acme-alerts` who made it, and keep watching the dashboard.

3. **Increase allocated storage by 25%.**

   > **This cannot be undone.** RDS storage can grow but never shrink. After this change, no further storage changes are possible for 6 hours. Acme has pre-approved increases up to 1,000 GiB. Anything beyond that needs their approval (see [Escalate](#escalate)).

   ```
   aws rds modify-db-instance \
     --db-instance-identifier acme-prod-db \
     --allocated-storage 625 \
     --apply-immediately \
     --profile acme-prod
   ```

   Replace `625` with the current size × 1.25, rounded up. RDS requires an increase of at least 10%.

   **Expected:** Within a few minutes, the status changes to `modifying`, then `storage-optimization`. The database stays online during both. `FreeStorageSpace` on the dashboard should jump up, usually within 30 minutes. The `storage-optimization` status can last for hours. That's normal.

   **If free space hasn't increased after 30 minutes**, or the command returns an error: [Escalate](#escalate).

4. **Post an update in `#acme-alerts`** with the new size and current free space. Then go to [Diagnose](#diagnose) to find out why the disk filled up.

# Diagnose

Connect to the database using the [read-only diagnostic user](link-to-access-doc#acme-prod-db). The checks below are ordered from most to least likely cause.

## Inactive replication slot holding WAL

If a replication consumer (a read replica, a DMS task, a change-data-capture pipeline) stops reading, Postgres keeps every write-ahead log (WAL) file for it, and they pile up without limit. This has caused both past incidents on this database.

In CloudWatch, check `TransactionLogsDiskUsage` for `acme-prod-db`. If it's climbing along with the drop in free space, WAL is the problem. Find the slot:

```sql
SELECT slot_name,
       active,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal
FROM pg_replication_slots
ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) DESC;
```

**If a slot shows `active = false` with a large `retained_wal`:** that slot is the cause. **Don't drop it yourself.** Dropping it breaks whatever depends on it. Escalate to Acme's data team with the slot name. They'll either restart the consumer or approve dropping the slot.

## Table or index growth

```sql
SELECT relname AS table,
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 10;
```

Compare against the sizes recorded in [past incident notes](link). `events` and `order_audit` are the usual growers. If one table has grown much faster than normal, tell Acme's engineering lead. It's often a new feature writing more than expected, or a cleanup job that stopped running.

## Temporary files from large queries

Large sorts and joins write temporary files that are deleted when the query finishes. On the dashboard this looks like a sawtooth: a sharp drop, then recovery. Check for long-running queries:

```sql
SELECT pid, now() - query_start AS runtime, left(query, 100) AS query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY runtime DESC
LIMIT 5;
```

If one query has been running for a long time and space is still falling, ask Acme's engineering lead whether it's safe to cancel it (`SELECT pg_cancel_backend(<pid>);`). Don't cancel it without asking.

# Escalate

Escalate if **any** of these are true:

- Free space hasn't increased within 30 minutes of step 3
- More than 1,000 GiB of storage is needed
- Fixing it requires dropping a replication slot or cancelling a query
- You're unsure what to do next

| Who | When | How |
| --- | --- | --- |
| Three Knots on-call (secondary) | First stop for any of the above | `#oncall` in Slack, or page via PagerDuty |
| Acme engineering lead | Approving storage above 1,000 GiB, cancelling queries, customer-visible impact | See [Acme contacts](link-to-customer-page#contacts) |
| Acme data team | Replication slots and DMS/CDC pipelines | See [Acme contacts](link-to-customer-page#contacts) |
| AWS Support | The storage change fails or gets stuck | [Support Center](https://console.aws.amazon.com/support) in the `acme-prod` account (Business support) |

# Background

- **What this alert measures:** `FreeStorageSpace` for `acme-prod-db` below 50 GiB (10% of 500 GiB) for 15 minutes. **If the database's storage changes, update the alarm threshold to match.**
- **How the system fits together:** Acme's order service (ECS) writes to `acme-prod-db`, PostgreSQL 16 on gp3 storage. A DMS task replicates from it to their data warehouse using a logical replication slot. See the [Acme architecture overview](link).
- **Storage autoscaling is off.** Acme turned it off to control costs, which is why this alert pages a human.
- **Known false positives:** Acme's nightly bulk import (02:00–03:00 UTC) creates a sawtooth dip that can briefly cross the threshold. It recovers without intervention.
- **Past incidents:** [2026-03 DMS task stopped, slot retained 180 GiB of WAL](link) · [2026-08 same cause, runbook verified](link)

# Follow-up

- [ ] Ticket created for the root cause (if not resolved above)
- [ ] Post-incident review, if this was Urgent or Acme saw write failures
- [ ] Alarm threshold updated, if storage was increased
- [ ] Alert tuning ticket, if this fired during the nightly import
- [ ] This runbook updated with anything that was wrong, missing, or confusing
