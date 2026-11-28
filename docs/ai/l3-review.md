# Review of docs/ai/l3-draft.md
Reviewers: 6810301003 and 6810301037. Checked against docs/intent/movie-filter.md and SMART.

| บรรทัดในร่าง | ปัญหา | ตัดสิน: แก้ ตัด หรือเก็บ |
|---|---|---|
| 23: "Choosing a value that is not one of the five ratings shall be rejected" (REQ-6) | The parent picks a rating from five choices and never types one, and the customer did not ask for an error case | ตัด |
| 98: "When the parent picks the rating "PG-15"" | A test of a value the parent cannot choose, so the user never sees it | ตัด |
| 104: "Scenario: Parent picks a valid rating that no movie has" (@pending-OQ1) | It answers open question 1 for the customer, who has not decided what the parent should see | แก้: เป็นคำถามถึง customer proxy |
| 124: "a valid rating with no matching movies returns an empty list" (A-6) | Same guess as above, written as an assumption instead of asking the customer | แก้: เป็นคำถามถึง customer proxy |
| 54: "### Story 3: Any moviegoer can filter without signing in" | "No login" is a constraint of the intent, not a new story | แก้: เป็นเงื่อนไขของบัตรใบที่ 1 ไม่ใช่บัตรใหม่ |
| 137: "The PG / PG-13 prefix trap: a naive "starts with" or "contains" comparison would wrongly mix them" (C-5) | A correct concern: PG and PG-13 must not be mixed, and REQ-4 already guards it | เก็บ |

Kept: C-3 (line 135) and C-5 (line 137), because the draft itself marks them as concerns and does not present them as requirements.
Counts: แก้ 3 ตัด 2 เก็บ 1
Customer proxy: Teacher Damrongrits confirmed the three cards on 2026-11-28gi