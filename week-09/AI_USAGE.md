# AI Usage Disclosure — Week 09

Required by the course academic policy (Generative AI use level **D — AI-integrated**).
You are responsible for the accuracy, testing and integrity of everything you submit,
including anything an AI tool produced.

| Tool | Exact model / plan | Used for | Which files it touched |
| --- | --- | --- | --- |
| Claude | Sonnet 5.5 | Part 2 — writing the calls for R1, R2, R3 (one fresh chat per request) | calls/ (saved by me, unedited) |
| Claude | Sonnet 5.5 | Step-by-step guidance for the lab; drafting tool/check_room.json; explaining which constraint refuses which S case; drafting lab-report.md sections 2, 5 and 6; preparing the R1-R3 prompt files; help with git and PowerShell commands | tool/check_room.json, lab-report.md (sections 2, 5, 6), AI_USAGE.md |

**I wrote `tool/check_room.json` myself:** no
An AI wrote it: Claude drafted the whole definition (name, description, the five properties and their constraints) and I used it. I checked it by running tests/check_tool.py: D1-D6 and S01-S12 all passed on the first run.

**I used an AI to explain a JSON Schema keyword or a check:** yes
Claude explained which constraint refuses which of the nine malformed calls S03-S12 (for example type integer for S03 and S10, additionalProperties false for S05, required inside items for S08), and why the schema cannot check AC1 (start < end), AC2 and AC4.

**Every reply in `calls/` and in lab-report.md section 4 is exactly what the model returned:** yes
The only change to a file in calls/ is the name: the reply to R3 is a question, so it is saved as r3.txt instead of r3.json. Its content is unchanged.

**Every terminal output in lab-report.md was pasted from a real run, unedited:** yes

**Anything I accepted from the AI without fully understanding it:**
Yes, I understood everything and discussed everything with AI after completing the task

Signed: Tazhibayev Temirlan
Date: 16.11.2026
