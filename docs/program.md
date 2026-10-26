# The programme

ThirdUni ran a four-week artificial intelligence programme for people in Algeria
from 1 to 30 September 2026, funded by the U.S. Department of State's Alumni
Engagement Innovation Fund and U.S. Embassy Algiers. The published curriculum moved
from discovery to tools, then to building, then to Demo Day:

| Week | Published theme | What it covered |
| --- | --- | --- |
| 1 | Discovery | AI fundamentals, scam and safety awareness, a first assistant |
| 2 | Tools | Mini-apps written from plain-language prompts, forming teams |
| 3 | Build | Designing a solution and building a working prototype |
| 4 | Share and launch | Pitching on Demo Day |

## What we did with it

We did not build a prototype from zero for a demo. Geniri was already running, so
the cohort worked inside a live product with real customers, a real ledger and a
release already in flight. That is less photogenic than a demo app and it is a
harder place to learn, because every change has to survive the thing it lands in.

Two streams, running together:

- **Production.** Improvements to the platform customers were already using:
  reliability of the generation queue, the paths where a person has to intervene,
  and the smaller defects that never announce themselves but do get mentioned in
  support.
- **v0.4.** The next release, now in internal alpha. Alpha is our own testing;
  beta follows as a protected, sub-public test with a limited set of people
  outside the company. Neither stage has a committed date, and this document will
  not pretend otherwise.

## The constraint that shaped the work

Every change had to survive the product it landed in: a customer's invoice, a
balance, a generation already in flight. That is the difference between this and a
prototype, and it is the part this repository tries to show. The architecture and
the decision records describe the machinery a change has to fit into; the release
state above says honestly where that work stands.

## Still to be added

Before the final submission, this page gets the parts that need real evidence
rather than description: the live demonstration link, the walkthrough video, and
the Demo Day date. The weeks above describe the programme as published; the
week-by-week detail of our own work is being written with those links so that each
claim points at something a judge can open.
