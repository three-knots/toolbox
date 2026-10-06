# Runbooks

Received an alert or request and not sure how to respond? You've come to the right place.

Runbooks are organized by customer. Each one covers a single alert or a single recurring task. It tells you how urgent the situation is, what to do right now, and who to contact if that doesn't work. Where it helps, runbooks link to deeper documentation elsewhere in Confluence so you can learn the underlying concepts and technologies.

# Using a runbook

1. **Open the runbook linked from the alert.** If there's no link, search for the alert name. Runbook titles match alert names.
2. **Confirm the priority.** The runbook suggests one. See [Priority](#priority) below for what each level means.
3. **Follow _Acknowledge_, then _Mitigate_.** Stop at the first step that fixes the problem.
4. **Stuck? Go to _Escalate_.** Asking for help early is always the right call.
5. **Afterward, fix the runbook.** If anything was wrong, missing, or confusing, correct it while it's fresh. You're the best person to do it.

**No runbook for this alert?** Treat it as High, tell the team you're on it, and ask for help early. When it's resolved, raise it as a candidate for a new runbook (see [Runbook lifecycle](#runbook-lifecycle)).

# Priority

Every runbook lists a default priority. Treat it as a starting point. The first responder decides the actual priority based on what they're seeing. **If you're torn between two levels, pick the higher one.** Whether it was the right call can be discussed afterward. Don't debate it during the incident.

| Priority | What it means | During business hours | After hours |
| --- | --- | --- | --- |
| Urgent | Business operations are stopped, or data or security is at risk | Drop everything and address | Drop everything and address |
| High | Multiple workflows are impaired, or it will become Urgent if left alone | Drop everything and address | Gracefully stop what you're doing and address |
| Medium | Something isn't working correctly, but it isn't a substantial blocker | Gracefully stop what you're doing and address | Wait until the next business day |
| Low | Proactive: heading off a problem before it affects anyone | Get to it within a few hours | Wait until the next business day |

## Urgent

This is stopping business operations. We need to resolve it as fast as possible.

- Tell the team right away that you believe there's an urgent issue.
- Go straight to the runbook's **Mitigate** steps. Urgent runbooks always have actions you can take immediately.
- Be quick to ask for help. If you're slowing down, ask someone to join a call. Now is not the time to let a knowledge gap slow down the fix.

## High

This is impairing multiple workflows and slowing work down. It may become Urgent if left alone.

- The runbook's **Mitigate** steps will guide you toward restoring normal operation.
- After hours: if you think it may actually be Urgent, check in with the team lead.

## Medium

Something isn't working correctly, but it isn't a substantial blocker.

- You have time to read the runbook's guidance in full before acting.
- Check with the customer about how soon they need it resolved. Circumstances may raise this to High.
- **If this woke you up after hours, the alert is misconfigured.** Tell the team so the alert routing gets fixed.

## Low

This is proactive support: getting ahead of issues before they affect business operations.

- You have time to read the runbook and the documentation it links to.
- If it's left too long, it can become Medium.
- **If this woke you up after hours, the alert is misconfigured.** Tell the team so the alert routing gets fixed.

# Runbook lifecycle

## Deciding if something needs a runbook

Every alert that notifies a human needs a runbook. Beyond alerts, if there's a task or process you think could benefit from one, bring it up with the team. During that discussion, consider:

- **Should this be automated instead?** If every step is fixed and needs no judgment, a script or automated remediation is better than a runbook.
- **Is this ongoing, temporary, or a one-off?** One-offs usually belong in a ticket, not a runbook.
- **Who will read it?** A runbook for an on-call engineer who's never seen this customer reads differently from one for the customer's lead engineer.

## Writing a draft

1. Create a page under the customer's section, starting from the runbook template. Treat the template as a starting point, not a list of requirements. Drop sections that don't apply.
2. Name it `runbook-{customer}-{context}`, where `{context}` matches the alert name exactly. For tasks with no alert, describe the action (e.g. `runbook-acme-rotate-tls-certs`).
3. Add the `runbook-draft` label.
4. Ask for feedback as you go. You don't have to wait until it's finished.

Once it's published, make sure the alert links to the runbook. Most alerting tools have a runbook URL field or annotation. Put the link in the alert description otherwise.

## Review

Get it reviewed by someone who **isn't** familiar with the system. The goal is to close the knowledge gap. If the runbook only works for people who already know the system, it doesn't reduce reliance on the few people who do.

The reviewer should do a dry run: walk through each step against the real (or a non-production) environment, confirming access works, links go where they claim, and commands run. Anything they had to ask about goes into the runbook.

Once the review is done, publish the page.

## First live use

Treat the first real use with care. As the author, expect the first few runs to be bumpy, and be available to answer questions, especially if the readers vary in experience.

After a real event where the runbook worked:

- Swap the `runbook-draft` label for `runbook-verified`.
- Update the **Last verified** date in the runbook's details table.

## Keeping it current

- **After every use:** the responder fixes anything that was wrong or unclear.
- **Every 6 months, or after significant changes to the system:** the owner re-checks the runbook and updates **Last verified**. If a runbook hasn't been verified in over a year, don't trust it.
- **When the alert or system goes away:** archive the runbook. Don't leave it for someone to find later.
