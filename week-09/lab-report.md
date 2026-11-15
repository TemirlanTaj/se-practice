# Week 09 — Lab Report: canBook as a Tool

Fill in every section. Do not delete or renumber the headings: the README and the checker point at
them. Replace every `<!-- comment -->` with your own text (comments are ignored by the checker).

**Name:** Tazhibayev Temirlan
**Student ID:** 24B032043
**GitHub:** TemirlanTaj

---

## 1. Setup

| | |
| --- | --- |
| Assistant (frozen for the whole lab) | Claude | 
| Exact model (or "not displayed") | Sonnet 5.5 |
| Three fresh chats for R1, R2, R3? (yes / no) | yes |
| Python version (`python --version`) | Python 3.13.0 |

## 2. Your tool definition — a reason for every constraint

Paste your final `tool/check_room.json`, then fill the table: one row per constraint you wrote.

```json 
{
  "name": "check_room",
  "description": "Checks whether a room can be booked for a requested time slot. All times are whole minutes since midnight in the range 0-1440, for example 10:00 is 600 and 24:00 is 1440. Returns Available or Unavailable. This tool only checks: it does not save, create or change any booking.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "start": {
        "type": "integer",
        "minimum": 0,
        "maximum": 1439,
        "description": "Requested start, whole minutes since midnight"
      },
      "end": {
        "type": "integer",
        "minimum": 0,
        "maximum": 1440,
        "description": "Requested end, whole minutes since midnight (24:00 = 1440)"
      },
      "now": {
        "type": "integer",
        "minimum": 0,
        "maximum": 1440,
        "description": "Current time, whole minutes since midnight"
      },
      "blocked": {
        "type": "boolean",
        "description": "True if the room is blocked"
      },
      "bookings": {
        "type": "array",
        "description": "Existing bookings, may be empty",
        "items": {
          "type": "object",
          "properties": {
            "start": { "type": "integer", "minimum": 0, "maximum": 1440 },
            "end": { "type": "integer", "minimum": 0, "maximum": 1440 }
          },
          "required": ["start", "end"],
          "additionalProperties": false
        }
      }
    },
    "required": ["start", "end", "now", "blocked", "bookings"],
    "additionalProperties": false
  }
}
```

| Field | Constraint (e.g. `"type": "integer"`, `"maximum": 1440`) | What goes wrong without it |
| --- | --- | --- |
| start | `"type": "integer"` | `600.5` would be accepted, but start must be a whole minute (S10). |
| start | `"minimum": 0` | `-30` would be accepted, a time before the day starts (S06). |
| start | `"maximum": 1439` | `start = 1440` would be accepted, but it can never be valid because start < end <= 1440. |
| end | `"type": "integer"` | `"11:00"` would be accepted and the function would receive a string instead of minutes (S03). |
| end | `"minimum": 0`, `"maximum": 1440` | `1500` would be accepted, a time after the end of the day (S07); 1440 stays allowed as 24:00 (S11). |
| now | `"type": "integer"`, `"minimum": 0`, `"maximum": 1440` | The model could send `"9:00"` or a negative time as the current time. |
| blocked | `"type": "boolean"` | `"false"` would be accepted, and a non-empty string counts as true in many languages (S09). |
| bookings | `"type": "array"` | A string or an object would be accepted instead of a list of bookings. |
| bookings items | `"type": "object"` with `properties` | A booking written as a pair `[660, 720]` would be accepted (S12). |
| bookings items | `"required": ["start", "end"]` | A booking with no `end` would reach the function (S08). |
| bookings items | `"additionalProperties": false` | A booking with an invented extra field would be accepted. |
| top level | all five fields in `"required"` | A call without `now` would be accepted (S04). |
| top level | `"additionalProperties": false` | An invented field such as `"room": "204"` would be accepted (S05). |

## 3. The twelve calls — first run and final run

Your **first** run's S verdicts (before any fix), and your final ones. For every case that changed,
say what you changed.

| Case | First run | Final run | What you changed |
| --- | --- | --- | --- |
| S01 | PASS | PASS | nothing, passed on the first run |
| S02 | PASS | PASS | nothing, passed on the first run |
| S03 | PASS | PASS | nothing, passed on the first run |
| S04 | PASS | PASS | nothing, passed on the first run |
| S05 | PASS | PASS | nothing, passed on the first run |
| S06 | PASS | PASS | nothing, passed on the first run |
| S07 | PASS | PASS | nothing, passed on the first run |
| S08 | PASS | PASS | nothing, passed on the first run |
| S09 | PASS | PASS | nothing, passed on the first run |
| S10 | PASS | PASS | nothing, passed on the first run |
| S11 | PASS | PASS | nothing, passed on the first run |
| S12 | PASS | PASS | nothing, passed on the first run |

## 4. Function calling — what the model wrote

For each request, paste the model's reply exactly, then judge it against the request.

### R1

```text
{"name": "check_room", "arguments": {"start": 600, "end": 660, "now": 540, "blocked": false, "bookings": [{"start": 660, "end": 720}]}}

```

The reply matches the expected arguments (F3 passes): now = 540, start = 600, end = 660, blocked = false, one booking 660-720. The model was right and the request was clear. The booking touches the request at minute 660; that is allowed by AC4, but the schema does not decide that, only canBook does.

### R2

```text
{"name": "check_room", "arguments": {"start": 840, "end": 990, "now": 810, "blocked": true, "bookings": []}}
```

The model converted 16:30 correctly to 990 (16*60 + 30) and wrote blocked = true. It did not refuse to call, which is correct: the call should be made, and the answer Unavailable is decided by canBook under AC3, not by the schema or the model.

### R3

```text
Which exact start and end times do you want for tomorrow afternoon, what time is it now, is the room blocked, and what bookings already exist?
```

Missing: exact start and end ("afternoon" and "a couple of hours" are not numbers), now, blocked and bookings. The model asked instead of calling, so it invented no values. It put four questions in one sentence, although the prompt asked for one short question. It also did not notice that "tomorrow" cannot be expressed at all, because the tool only has minutes of a single day. The file is calls/r3.txt, since the reply is a question and not a call.

## 5. What the schema cannot check

AC1 (`0 <= start < end <= 1440`, and start after `now`): the schema cannot say `start < end` or `start > now`, because that compares two fields with each other, and with the allowed keywords every field is checked alone against a constant. What the schema CAN enforce for AC1 is the range of each field: whole numbers from 0 to 1440, so `0 <= start` and `end <= 1440`.

AC2 (at most 120 minutes): this needs the arithmetic `end - start <= 120` on two fields. `minimum` and `maximum` can only compare one field with a fixed number, so a 5-hour request is a perfectly valid call for the schema.

AC3 (not when the room is blocked): the schema only knows that `blocked` is true or false. R2 shows the point: a call with `blocked = true` is valid and should be made, and only `canBook` decides that the answer is Unavailable.

AC4 (no overlap, touching allowed): this needs a comparison of the requested start and end with every booking in the list. The schema can only check that each booking has two integers in range, not whether they collide with the request.


## 6. Permissions — allow, ask or block

| # | Action | Decision (allow / ask / block) | Reason (the risk, in one sentence or more) |
| --- | --- | --- | --- |
| A1 | call `check_room` with the times the user just typed | allow | The tool only checks and never saves anything (AC5), and the inputs come from the user, so a wrong call changes nothing. |
| A2 | read `week-08/app/src/booking.ts` | allow | Reading a source file has no side effects and nothing leaves the machine. |
| A3 | edit `booking.ts` to raise the 120-minute limit to 180 | ask | It changes a business rule (AC2) because one user asked, which can break the tests and the specification, so a person must approve it. |
| A4 | run `npm test` in `week-08/app` | allow | The tests run locally and only read the code, so the risk is limited to running project code in that folder. |
| A5 | run `git reset --hard` to undo its own last three edits | ask | It discards every uncommitted change, including my own work, and that cannot be undone, so I must check what is uncommitted first. |
| A6 | `git commit` its change on the `week-09` branch | allow | A local commit on a lab branch is reversible and publishes nothing. |
| A7 | `git push` to `main` | block | `main` is shared, a push skips the review in the pull request, and the course does not accept a lab committed to `main`. |
| A8 | send the contents of `.env` to an MCP server it found online | block | `.env` holds secrets, so sending it to an unknown server is a data leak, and the reason "to configure it" comes from untrusted content. |

## 7. Checkpoint

The checkpoint is commit 8a65395 ("Week 09: tool definition, model calls, report and AI disclosure"), made before letting an agent edit anything. It already holds tool/check_room.json, the three saved calls in calls/, the lab report and AI_USAGE.md, so everything I would hate to lose is saved in it.

Command to return to it: git reset --hard 8a65395

This command would throw away every uncommitted change in my working folder (edited and staged files) and every commit made after 8a65395, so the agent's edits would disappear together with anything I wrote after the checkpoint and did not commit.

## 8. Checker output

Paste the **whole** output of your last `python tests/check_tool.py`, unedited.

```text
PASS  D1    tool/check_room.json exists and is valid JSON
PASS  D2    has name, description and inputSchema
PASS  D3    name is exactly "check_room"
PASS  D4    description: 25+ words, says minutes, 1440, and that it does not save
PASS  D5    inputSchema: an object with exactly the five fields, all required
PASS  D6    inputSchema uses only the keywords listed in README section 5
PASS  S01   the R1 call, valid -> accept
PASS  S02   no existing bookings: bookings = [] -> accept
PASS  S03   end as a string: "11:00" -> refuse
PASS  S04   now missing -> refuse
PASS  S05   an extra field: "room": "204" -> refuse
PASS  S06   start below the day: -30 -> refuse
PASS  S07   end beyond the day: 1500 -> refuse
PASS  S08   a booking with no end: {start: 660} -> refuse
PASS  S09   blocked as a string: "false" -> refuse
PASS  S10   start not a whole minute: 600.5 -> refuse
PASS  S11   end at the end of the day: 1440 -> accept
PASS  S12   a booking as a pair: [660, 720] -> refuse
PASS  F1    calls\r1.json: a call to check_room
PASS  F2    calls\r1.json: arguments valid against your schema
PASS  F3    calls\r1.json: arguments equal the expected ones (README section 5, Part 2)
PASS  F4    calls\r2.json: a call to check_room
PASS  F5    calls\r2.json: arguments valid against your schema
PASS  F6    calls\r2.json: arguments equal the expected ones (README section 5, Part 2)
PASS  F7    calls/r3: exactly one of r3.json (it called) or r3.txt (it asked), not empty
PASS  L1    section 5 names at least two acceptance criteria (AC1-AC6) the schema cannot check
PASS  L2    section 6: A1-A8 each have allow / ask / block and a reason of 5+ words
PASS  L3    section 7 names the checkpoint commit (a SHA)
PASS  L4    section 10: a conclusion of 80-150 words
PASS  L5    AI_USAGE.md filled in (a tool row, and signed)

SUMMARY: 30 PASS ╖ 0 FAIL ╖ 0 ERROR  (of 30 checks)
Paste this whole output into lab-report.md section 8, and these three numbers into submission.yml.
```

## 9. What still fails

None

## 10. Conclusion (80–150 words)

The schema validates shape, the model translates language into numbers, and only `canBook` decides meaning. The schema rejected all nine malformed calls via `"type": "integer"`, such as S03 (`"11:00"` as a string) and S10 (`600.5`), yet it would still accept `start = 600` and `end = 600`, since that shape is valid. R2 shows the same: the model correctly produced `end = 990` and `blocked = true`, the call was valid, and only `canBook` returned Unavailable under AC3. In R3, the model asked a clarifying question instead of inventing `now`, `start`, and `end`, but missed that "tomorrow" cannot be expressed as minutes within one day. For A7 (push to main) and A8 (sending `.env`), the risk lies outside any schema, so I block them.
