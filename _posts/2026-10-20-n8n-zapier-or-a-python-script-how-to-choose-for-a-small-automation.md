---
title: "n8n, Zapier or a Python script? How to choose for a small automation"
tags: [automation, n8n, zapier, python]
---

## The problem

You have a repetitive task: copying form entries into a spreadsheet, sending a message when a row changes, or pulling data from one tool into another. You know it should be automated. Then you hit the first real decision: do you use Zapier, n8n, or write a Python script?

All three can work. The wrong choice usually costs you time later, not on day one. Here is a simple way to pick.

## Start with four questions

**1. How many steps, and how much branching?**
If the flow is "trigger, one or two actions", a hosted no-code tool is hard to beat. Once you need loops, conditions, retries or data reshaping, visual flows get crowded and a script is easier to read and test.

**2. Who will maintain it?**
If the person who owns the process is not a developer, a visual tool lets them open it, see what happens and change a field. If a developer will own it, code in version control is easier to review and roll back.

**3. Where does the data need to live?**
Some data should not pass through a third-party service, or you may want to run things on your own server. n8n can be self-hosted. A Python script runs wherever you put it. Zapier is a hosted service, so your data goes through it.

**4. How will it fail?**
Every automation fails eventually: an API changes, a token expires, a row has an unexpected value. Decide who gets told, and how. Hosted tools give you run history and email alerts. With a script, you build that yourself, but you can make it as precise as you like.

## A rough rule of thumb

- **Zapier**: best for short, linear flows between popular apps, owned by non-developers. You trade flexibility and per-task cost control for speed of setup.
- **n8n**: best when you need branching, custom logic or self-hosting, but still want a visual editor and a built-in library of integrations. It rewards some technical comfort.
- **Python script**: best when the logic is the hard part (parsing messy files, calling an API with odd rules, heavy data cleanup), or when you need tests and version control.

None of these is "the professional choice". The right one is the smallest tool that does the job and that its owner can understand.

## What the script option looks like

For a task like "read new rows from a CSV export, tidy them, and post a summary to a webhook", a script can be short:

```python
import os
import csv
import json
import urllib.request

WEBHOOK_URL = os.environ["SUMMARY_WEBHOOK_URL"]  # set in the environment, not in code

def load_rows(path):
    with open(path, newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            row["email"] = row.get("email", "").strip().lower()
            if row["email"]:
                yield row

def post_summary(count):
    body = json.dumps({"text": f"{count} new rows processed"}).encode()
    req = urllib.request.Request(
        WEBHOOK_URL,
        data=body,
        headers={"Content-Type": "application/json"},
    )
    urllib.request.urlopen(req, timeout=10)

if __name__ == "__main__":
    rows = list(load_rows("export.csv"))
    post_summary(len(rows))
```

Schedule it with cron or a task scheduler and it runs quietly. The same job in n8n would be a handful of nodes; in Zapier it may need a code step or extra tooling for the cleanup.

## Pitfalls to watch for

- **Starting in a no-code tool, then fighting it.** If you find yourself writing code steps inside nearly every node, that is a sign to move the logic into a script.
- **Starting with a script nobody else can read.** If the process owner cannot see what it does, every small change becomes a request to a developer.
- **No error handling.** Whatever you pick, decide where failures are reported. Silent failures are worse than loud ones.
- **Secrets in the wrong place.** Keep keys in environment variables or the tool's credential store, never inside a flow description or a script file.
- **Over-building.** A task that runs a few times a day rarely needs a queue, a database and a dashboard.
- **Ignoring volume.** Hosted tools often bill by task count, so check how often your flow will run before you commit.

## A quick decision path

1. Two or three steps between common apps, owned by a non-developer? Try Zapier.
2. Needs branching, self-hosting or more control, with a visual editor? Try n8n.
3. Logic-heavy, data-heavy or needs tests? Write a Python script.
4. Not sure? Build the smallest version in the tool you can set up fastest, and move it only if you hit a clear limit.

Migrating a small automation is rarely painful, because the hard part is understanding the process, not the tool.

Synzen is a small dev agency using AI-assisted workflows, and every deliverable is reviewed by our team before handover. We build small automations like this.