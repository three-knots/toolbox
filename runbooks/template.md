# runbook-{customer}-{context}

> **Naming:** `{customer}` is the customer's primary name. If they go by others, list them under **Also known as** below. `{context}` matches the alert name exactly, so someone holding the alert can find this page by searching. For tasks with no alert, use a short action phrase (e.g. `rotate-tls-certs`).
>
> Delete every `>` guidance note like this one as you fill in the page.

| | |
| --- | --- |
| **Customer** | {name} (also known as: {other names}) |
| **Alert** | {exact alert name, and where it's defined, e.g. CloudWatch / Datadog / Prometheus} |
| **Default priority** | {Urgent / High / Medium / Low}. See [Priority](link-to-runbooks-landing-page#priority) |
| **Owner** | {person responsible for keeping this page current} |
| **Status** | Draft / Verified |
| **Last verified** | {date this was last confirmed to work, in a real event or dry run} |
| **Quick links** | {dashboard} · {logs} · {console} · {how to get access} |

> The details table exists so a responder can glance at it and go. **Quick links** should open straight to the right view: a dashboard with the time range and filters already set, a log query already filtered.

# Summary

> Two or three sentences: what fired, what it means in plain language, and who or what is affected if it's left alone. A responder who reads only this section should know how worried to be.

# Acknowledge

> What to do in the first few minutes, by priority. Don't repeat the full priority table. Link to it and write only what's specific to this runbook: which channel, who to tell, what to say.
>
> Also say what moves the priority up or down for this alert, e.g. "Raise to Urgent if writes are failing."

**If Urgent or High:**

1. Post in `#<channel>`: "{alert name} fired for {customer}. I'm on it." Include a link to the alert.
2. Go to [Mitigate](#mitigate).

**If Medium or Low:**

1. {e.g. Create a ticket in {queue} and link the alert.}
2. Go to [Diagnose](#diagnose).

**Raise the priority if:** {conditions}

**Lower the priority if:** {conditions}

# Mitigate

> Steps to restore normal service **quickly**. The root cause can wait. This section is required for anything that can be Urgent or High.
>
> Rules for this section:
> - **Each step must be safe to run.** Prefer actions that are harmless if repeated, like a redeploy or a scale-up. If a step can't be repeated or undone, say so in bold before the step.
> - **Give the exact command or click path**, with placeholders marked clearly (e.g. `<instance-id>`).
> - **Say what success looks like and how long to wait**: "within 10 minutes, the X graph should drop below Y."
> - **If access is needed**, link directly to the relevant section of the access doc, not the top of a long write-up.
> - **Say what to do if it doesn't work.** Usually: next step, or [Escalate](#escalate).

1. **{Action}**

   ```
   <command>
   ```

   **Expected:** {what you should see, and within what time}

   **If not:** {next step / escalate}

2. **{Action}**

   ...

# Diagnose

> For finding out **why** it happened. Used after mitigation, or for Medium/Low alerts where there's time to investigate first.
>
> Order the checks from most likely cause to least likely. For each one, say what the result means: "If you see X, the cause is Y. Go to Z."

## {Likely cause 1}

## {Likely cause 2}

# Escalate

> Who to contact, how, and when. The reader should never have to wonder whether they're allowed to ask for help.

Escalate if **any** of these are true:

- Mitigation hasn't worked within {N} minutes
- {condition specific to this alert}
- You're unsure what to do next

| Who | When | How |
| --- | --- | --- |
| {Team / on-call rotation} | {first stop} | {channel / phone / paging tool} |
| {Customer contact} | {e.g. customer-visible impact, or a change needing their approval} | {link to customer contact page} |
| {Vendor support} | {e.g. suspected provider-side issue} | {support portal link, account ID} |

# Background

> The context that helps someone understand the alert rather than just follow the steps. Keep it short and link out for depth.

- **What this alert measures:** {metric, threshold, evaluation window}
- **How the system fits together:** {brief description or diagram link}
- **Known false positives:** {situations where this fires but nothing is wrong}
- **Past incidents:** {links to past incidents and reviews}

# Follow-up

> What happens after the fire is out.

- [ ] Ticket created for the root cause (if not resolved above)
- [ ] Post-incident review, if this was Urgent, or High with customer impact
- [ ] Alert tuning ticket, if the alert fired when nothing was wrong or at the wrong priority
- [ ] This runbook updated with anything that was wrong, missing, or confusing
