# Unit of Work - Story Map

Every story is assigned to exactly one **primary** unit, which owns its acceptance criteria. **Supporting** units provide components the story needs.

| Story | Title | Primary unit | Supporting units |
| --- | --- | --- | --- |
| US-1.1 | Register my résumé | U2 | U1 |
| US-1.2 | Confirm my extracted profile by section | U2 | U1 |
| US-1.3 | Re-run setup without duplicates | U2 | U1 |
| US-1.4 | Register my cover letter | U2 | U1 |
| US-2.1 | Fill a page from my confirmed profile | U3 | U1, U2, U5 |
| US-2.2 | Let Claude decide only clear unmatched fields | U3 | U2 |
| US-2.3 | Upload my registered file to a file field | U3 | U2 |
| US-2.4 | Continue a multipage form with my approval | U3 | U5 |
| US-2.5 | Stop at submit, login and CAPTCHA steps | U3 | U5 |
| US-3.1 | Set my fixed sensitive answers once | U2 | U1 |
| US-3.2 | Fill sensitive questions only by exact match | U2 | U3 |
| US-4.1 | Reuse a saved answer in the right scope | U2 | U1 |
| US-4.2 | Draft a new answer within the word limit | U3 | U2 |
| US-4.3 | Save an approved answer with a scope | U2 | U3 |
| US-5.1 | Teach the plugin a fix once | U4 | U1 |
| US-5.2 | Reuse my personal fix on the same control | U4 | U3 |
| US-6.1 | Share a proven fix | U4 | U1 |
| US-6.2 | Review a contributed rule with evidence | U4 | U5 |
| US-6.3 | Receive approved fixes automatically | U4 | U3 |
| US-6.4 | Withdraw a bad rule for everyone | U4 | - |
| US-7.1 | See what changed before I submit | U3 | U1 |
| US-7.2 | Undo the plugin's changes on this page | U3 | - |
| US-8.1 | Delete all my data | U2 | U1 |
| US-9.1 | My résumé file changed or moved | U2 | U3 |
| US-9.2 | A fix stopped working | U4 | U3 |
| US-9.3 | GitHub cannot be reached | U4 | U3 |
| US-9.4 | Chrome and the plugin cannot work together | U3 | U1 (spike) |
| US-10.1 | Fixture forms with realistic hard controls | U5 | - |
| US-10.2 | Synthetic applicants with an answer key | U5 | - |
| US-10.3 | CI replay of every shared rule | U4 | U1, U5 |
| US-10.4 | Score the demo against the success criteria | U5 | U3, U4 |

## Stories per Unit

| Unit | Primary stories | Count |
| --- | --- | --- |
| U1 Foundation and Contracts | none (enabler; supports 13 stories and runs the spike behind US-9.4) | 0 |
| U2 Profile and Answer Store | US-1.1, 1.2, 1.3, 1.4, 3.1, 3.2, 4.1, 4.3, 8.1, 9.1 | 10 |
| U3 Fill Planner and Skills | US-2.1, 2.2, 2.3, 2.4, 2.5, 4.2, 7.1, 7.2, 9.4 | 9 |
| U4 Rule Learning and Validation | US-5.1, 5.2, 6.1, 6.2, 6.3, 6.4, 9.2, 9.3, 10.3 | 9 |
| U5 Fixtures, Tests and Demo | US-10.1, 10.2, 10.4 | 3 |
| **Total** | | **31** |

All 31 stories are assigned; none is unassigned or assigned twice as primary.

## Success Criteria by Unit

| Success criterion | Units |
| --- | --- |
| SC1 Field accuracy and flagging | U3 (fill), U2 (profile), U5 (answer key and scoring) |
| SC2 Fewer corrections after teaching | U4 (rules and sync), U3 (rule use), U5 (comparison) |
| SC3 No personal values in shared rules | U4 (sanitizer, scan, CI), U5 (synthetic values for the scan) |
| SC4 Changed control caught | U4 (disable), U3 (read-back), U5 (changed-control fixture) |
| SC5 Draft within limit, approved first | U3 (approval flow), U2 (word count) |

## Extension Compliance Summary (Units Generation)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-05, 13, 15 | Compliant | Validation and error handling sit in U1 contracts; checksum verification in U4; every unit's definition of done requires a clean Security Baseline summary |
| SECURITY-10 | Compliant | Lock file, `npm audit` and pinned CI in U1 |
| SECURITY-11 | Compliant | Security-critical logic isolated in named units (C9 in U2; C12 sanitizer and scan in U4; C14 in U1) |
| Other SECURITY rules | N/A | As justified in `requirements.md` and `application-design.md` |
| PBT (Partial) | Compliant at this stage | Definition of done includes PBT-02, 03, 07, 08, 09; pure modules (C8, C10, C12) are assigned to U2, U3, U4 |
