# Unit of Work - Story Map

**Version 2** (after the change request): 39 stories mapped to six units. Every story has exactly one **primary** unit, which owns its acceptance criteria; **supporting** units provide components it needs.

| Story | Title | Primary unit | Supporting units |
| --- | --- | --- | --- |
| US-1.1 | Register my résumé | U2 | U1 |
| US-1.2 | Confirm my extracted profile by section | U2 | U1 |
| US-1.3 | Re-run setup without duplicates | U2 | U1 |
| US-1.4 | Register my cover letter | U2 | U1 |
| US-1.5 | Answer the preference questionnaire | U2 | U1 |
| US-2.1 | Fill a page from my confirmed profile | U3 | U1, U2, U6 |
| US-2.2 | Let Claude decide only clear unmatched fields | U3 | U2 |
| US-2.3 | Upload my registered file to a file field | U3 | U2 |
| US-2.4 | Continue a multipage form with my approval | U3 | U6 |
| US-2.5 | Stop at submit, login and CAPTCHA steps | U3 | U6 |
| US-3.1 | Set my fixed sensitive answers once | U2 | U1 |
| US-3.2 | Fill sensitive questions only by exact match | U2 | U3 |
| US-4.1 | Reuse a saved answer in the right scope | U2 | U1 |
| US-4.2 | Draft a new answer within the word limit | U3 | U2 |
| US-4.3 | Save an approved answer with a scope | U2 | U3 |
| US-5.1 | Teach the plugin a fix once | U4 | U1 |
| US-5.2 | Reuse my personal fix on the same control | U4 | U3 |
| US-7.1 | See what changed before I submit | U3 | U1 |
| US-7.2 | Undo the plugin's changes on this page | U3 | - |
| US-8.1 | Delete all my data | U2 | U1, U5 |
| US-9.1 | My résumé file changed or moved | U2 | U3 |
| US-9.2 | A fix stopped working | U4 | U3 |
| US-9.4 | Chrome and the plugin cannot work together | U3 | U1 (spike) |
| US-10.1 | Fixture forms with realistic hard controls | U6 | - |
| US-10.2 | Synthetic applicants with an answer key | U6 | - |
| US-10.3 | CI checks for every change | U4 | U1, U6 |
| US-10.4 | Score the demo against the success criteria | U6 | U3, U4, U5 |
| US-10.5 | Labeled messages and postings | U6 | - |
| US-11.1 | Log my applications in one tracker | U5 | U3 |
| US-11.2 | Confirm that I submitted | U5 | - |
| US-11.3 | Update a status from a reply | U5 | U6 |
| US-11.4 | Review pending updates and undo | U5 | - |
| US-11.5 | See my tracker | U5 | U2 |
| US-11.6 | Export my tracker | U5 | - |
| US-11.7 | Check Gmail for replies (stretch goal) | U5 | - |
| US-12.1 | Save a posting to rank | U5 | - |
| US-12.2 | Get a fit score with evidence | U5 | U2 |
| US-12.3 | See deal-breakers and unknowns | U5 | U2, U6 |
| US-12.4 | Compare saved jobs | U5 | U3 |

## Stories per Unit

| Unit | Primary stories | Count |
| --- | --- | --- |
| U1 Foundation and Contracts | none (enabler; supports 13 stories and runs the spike behind US-9.4) | 0 |
| U2 Profile, Preferences and Answers | US-1.1, 1.2, 1.3, 1.4, 1.5, 3.1, 3.2, 4.1, 4.3, 8.1, 9.1 | 11 |
| U3 Fill Planner and Skills | US-2.1, 2.2, 2.3, 2.4, 2.5, 4.2, 7.1, 7.2, 9.4 | 9 |
| U4 Rule Learning | US-5.1, 5.2, 9.2, 10.3 | 4 |
| U5 Tracking and Ranking | US-11.1, 11.2, 11.3, 11.4, 11.5, 11.6, 11.7, 12.1, 12.2, 12.3, 12.4 | 11 |
| U6 Fixtures, Tests and Demo | US-10.1, 10.2, 10.4, 10.5 | 4 |
| **Total** | | **39** |

All 39 stories are assigned; none is unassigned or assigned twice as primary. Removed stories (US-6.1 to US-6.4, US-9.3) have no unit.

## Success Criteria by Unit

| Success criterion | Units |
| --- | --- |
| SC1 Filling | U3 (fill), U2 (profile), U6 (answer key and scoring) |
| SC2 Learning | U4 (rules), U3 (rule use), U6 (comparison) |
| SC3 Privacy | U4 (sanitizer and scan), U1 (logger allowlist), U6 (scan run) |
| SC4 Safety | U4 (disable), U3 (read-back), U6 (changed-control fixture) |
| SC5 Drafts | U3 (approval flow), U2 (word count) |
| SC6 Tracking | U5 (classification and status rules), U6 (labeled messages and scoring) |
| SC7 Ranking | U5 (scorer), U2 (questionnaire), U6 (postings, top-3 agreement) |

## Extension Compliance Summary (Units Generation, version 2)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-05, 15 | Compliant | Validation and error handling in U1 contracts; every unit's definition of done requires a clean Security Baseline summary |
| SECURITY-10 | Compliant | Lock file, `npm audit` and pinned CI in U1 |
| SECURITY-11 | Compliant | Security-critical logic isolated in named units (C9 in U2; C12 sanitizer and scan in U4; C22 and C23 rules in U5; C14 in U1) |
| SECURITY-13 | Compliant | No downloaded rules or code; rules interpreted as fixed step types (U4) |
| Other SECURITY rules | N/A | As justified in `requirements.md` and `application-design.md` |
| PBT (Partial) | Compliant at this stage | Definition of done includes PBT-02, 03, 07, 08, 09; pure modules C8 (U2), C10 (U3), C12 (U4), C22 and C23 (U5) |
