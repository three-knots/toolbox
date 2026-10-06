# Runbooks

A starting kit for writing runbooks: the approach, a landing page, a template, and a worked example. Written in markdown so it can be copied into Confluence (or any similar documentation tool) with minimal cleanup.

## What's in here

| File | What it is | What to do with it |
| --- | --- | --- |
| [documentation.md](documentation.md) | Landing page for the runbooks section: how to use a runbook, priority levels, and how runbooks are created and maintained. | Copy it in as the parent page of the runbooks section. Every runbook lives underneath it. |
| [template.md](template.md) | Blank runbook with guidance under each heading. | Copy it for each new runbook. Delete the guidance notes (the `>` quotes) as you fill it in. |
| [example.md](example.md) | A filled-in runbook for a database storage alert. | Read it before writing your first runbook. Don't copy it. |

## Getting started

1. Read [example.md](example.md) to see what a finished runbook looks like.
2. Copy [documentation.md](documentation.md) into your wiki as the parent page for runbooks. Adjust the priority levels and response times to fit your team.
3. For each alert that pages a human, copy [template.md](template.md) into a child page. Title it with the alert's exact name, and link the alert to the page.
4. Have someone unfamiliar with the system review it and dry-run it before you publish.

## The approach in brief

1. **One alert, one runbook.** Every alert that notifies a human links directly to its runbook. If an alert has nothing for a human to do, fix or delete the alert instead of writing a runbook for it.
2. **Written for someone who's never seen the system.** The reader is tired, stressed, and possibly new. Spell out exact commands, the result you expect from each, and how long to wait. Link directly to the part of a doc that matters, not the top of a long page.
3. **Most urgent content first.** The page goes in order: what's happening → do this now → dig deeper → who to call → background. The reader should be able to stop reading as soon as the problem is fixed.
4. **Stop the damage before hunting for the cause.** The Mitigate steps restore service and are safe to run. Root-cause work comes after, in Diagnose.
5. **Always offer a way out.** Every step says what to do if it doesn't work. Escalating early is always acceptable.
6. **Priority is a default, not a verdict.** The runbook suggests a priority. The person responding decides. When unsure, pick the higher level.
7. **A runbook isn't done until someone else has used it.** It's reviewed by someone unfamiliar with the system and dry-run before publishing. It's marked verified only after it works in a real event. It gets re-checked periodically and retired when its alert goes away.
8. **If no step needs judgment, automate it.** A runbook that is just a fixed list of commands should become a script or automated remediation.

## Further reading

- [Google SRE Book – Being On-Call](https://sre.google/sre-book/being-on-call/): the argument for alerts a human can act on, with a playbook entry for each
- [PagerDuty Incident Response – Severity Levels](https://response.pagerduty.com/before/severity_levels/): "if unsure, pick the higher severity" and defining levels by impact
- [GitLab Runbooks](https://runbooks.gitlab.com/): a large public collection of runbooks, linked from their alerts
