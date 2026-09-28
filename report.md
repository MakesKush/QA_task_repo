# Facilitators — QA test report

## Executive summary

The main blocker is invitation of an **existing registered account**. All **seven attempted invitations** failed with HTTP 500: four through the account picker and three through the email form. The error page also displayed internal database and server details. Because no invitation to the intended recipients was confirmed, I could not test acceptance, actual facilitator permissions, or permission changes after removal or reinstatement on those accounts.

Inviting a **new email address** did create a pending invitation and deliver a message to the test mail catcher. Navigation, search, filters, pagination, withdrawal, and the list transitions for Remove and Reinstate were exercised. Other findings are duplicate rendered facilitator rows and a pending invitation row with no visible identity when the optional first name is blank. Two pending invitations to one address were also observed; whether that is a defect depends on the product rule.

I checked the controls at approximately **1440, 768, and 390 px** and found them usable. I also applied two filters together: one matching row appeared, and **Clear All** restored the list.

| No. | Classification | Finding | Evidence strength |
| --- | --- | --- | --- |
| 1 | High-impact defect | Inviting an existing account reaches HTTP 500 through both add routes | 7/7 attempts: four picker, three email form |
| 2 | Medium-impact defect | The error page reveals internal database and server information | Five inspected error pages and one additional screenshot |
| 3 | Medium-impact defect | Identical facilitator rows appear twice in the same status list | Multiple rows observed, persisted after reload |
| 4 | Medium-impact usability defect | An invitation without a first name has no visible identity in the list | 1/1 submission |
| 5 | Observation; rule required | Two pending invitations can exist for one email address | 1/1 duplicate attempt |

These classifications describe impact observed in staging. Production scope and the intended rules for pending invitations and status transitions were not available.

## Scope and coverage

| Area | Result |
| --- | --- |
| Access and navigation | All five supplied accounts signed in, selected the designated enterprise, and opened Settings → Facilitators. **Active**, **Sent Invitations**, **Archived**, and **+ Facilitator** were available. |
| Main-list search | Exact-name search narrowed visible rows. Searching a pending invitation by recipient email located its row. **Clear** restored the list. |
| Existing-account picker | Search and **Aliases** filter found candidate accounts; selection and deselection worked. Sorting and late-page pagination loaded results. Submission for registered accounts reached the error in finding 1. |
| Individual picker filters | **Organization**, **Countries**, **Negotiators**, and **Groups** each returned matching candidates. **Clear All** restored the unfiltered list between checks. |
| Combined picker filters | I applied **Aliases** and **Negotiators** together and saw one account matching both selected values; **Clear All** restored the full list. This result does not establish whether different filter categories use AND or OR, because the returned account satisfied both. |
| Email form | Empty Email showed required-field validation, and malformed email was rejected. A new address produced a pending row and a message in the test mail catcher. An optional blank First Name led to finding 4. A registered address reached the error in finding 1. |
| Invitation actions | **Resend** delivered another message, although no clear success confirmation was noticed. **Withdraw** cancellation preserved the row; confirmation removed test invitations. Two simultaneous pending invitations to one address are described in finding 5. |
| Active and Archived actions | Expanding an Active row showed associated classes. **Remove** cancellation preserved the row. Confirmed Remove moved one existing facilitator from **Active** to **Archived**. **Reinstate** moved two previously archived records to **Sent Invitations** with method **Sim Notification**. Actual recipient rights were not checked. |
| Pagination | Changing Active page size from 25 to 10 changed visible rows and controls. Another page loaded different rows; the original page size was restored. |
| Cross-account view | A pending row created by Reinstate under one supplied account was visible under another supplied account in the same enterprise. |
| Responsive controls | I checked search, row `…` menus, **+ Facilitator**, closing the add form, and pagination at approximately 1440, 768, and 390 px. They remained accessible without visible text or button overlap. Exact viewport measurements were not recorded. |

### Two ways to add a facilitator

1. **Existing account:** **+ Facilitator** opens an account picker with search and filters. A candidate can be selected, but **Send Invitation** failed for each directly tested registered candidate.
2. **Email Invitation:** **+ Facilitator → + Email Invitation** accepts an address, optional first name, and optional message. New-address invitations appeared in **Sent Invitations** and in the test mail catcher; attempts with registered addresses failed.

The successful new-address messages contained a registration link. The link was not opened, so its destination and full completion flow remain unverified.

## Detailed findings

### 1. Invitation of an existing account fails with HTTP 500

**Steps to reproduce:**

1. I signed in with a supplied account, selected the designated enterprise, and opened Settings → Facilitators. I used registered candidate accounts as recipients. Searches showed no **Active** or **Sent Invitations** row for `test_user42`; before attempts involving `test_user42` and `test_user43`, neither appeared in **Active**, **Sent Invitations**, or **Archived**.
2. I clicked **+ Facilitator**, searched for a registered candidate, selected the row, confirmed **Selected: 1**, and clicked **Send Invitation** once.
3. In a separate attempt, I chose **+ Facilitator → + Email Invitation**, entered a registered account's email address, and submitted once.
4. I returned to Facilitators, reloaded, and searched for the recipient in **Active** and **Sent Invitations**.

**Expected result:** I expected a clear confirmation and a pending record under **Sent Invitations**. If a registered account was ineligible, I expected the form to explain that without a server error.

**Actual result:** All seven submissions reached an apology page with **HTTP 500**: **four** through the existing-account picker and **three** through the email form. Five inspected responses displayed an unexpected-server-error code and database conflict-handling failure; the screenshot below shows the same details on another response. I received no success confirmation. In checked follow-ups, I found no visible Active or Sent Invitations row for the intended recipient. After an email attempt, no new message to `test_user42` appeared in the test mail catcher; I did not recheck the mail catcher after every picker attempt. I saw no invitation or notification for `test_user42` or `test_user43` after their failed submissions. The picker error also discarded my selection and took me away from the picker. This blocked acceptance and permission checks for those accounts; it does not establish that every registered account fails. The [full technical report](raw-evidence/qa-report-full.md) records the observations.

![HTTP 500 page from a failed invitation, including developer error details](raw-evidence/05-invitation-http-500.png)

The screenshot confirms one HTTP 500 response. By itself, it does not identify which add route produced the error.

### 2. Internal error details appear on the error page

**Steps to reproduce:**

1. I submitted an invitation for an existing registered account through either add route described in finding 1.
2. I inspected the resulting HTTP 500 page.

**Expected result:** I expected a user-facing error with recovery guidance or a reference ID and no internal implementation details.

**Actual result:** I could see database statement details, schema identifiers, a low-level conflict error, and server stack information on five inspected error pages. Another screenshot shows the same class of disclosure. The disclosure is a separate issue from the failed invitation. Exact strings are omitted from this summary but visible in the screenshot below.

![HTTP 500 page exposing database and server details](raw-evidence/02-picker-http-500.jpg)

This recorded picker attempt includes a [selected-account screenshot](raw-evidence/01-account-selected.jpg) before submission. The error screenshot alone does not identify the route.

### 3. Duplicate facilitator rows are rendered

**Steps to reproduce:**

1. I opened Settings → Facilitators before changing any records and inspected **Active**, including an exact-name search for an affected facilitator.
2. I compared two rows showing the same person, job title, metrics, and observed row identifier.
3. I reloaded and repeated the check, then checked **Archived** for the same pattern on another record.

**Expected result:** I expected each facilitator record to appear once per status section, unless the interface distinguished separate memberships or relationships.

**Actual result:** I saw two separate Active facilitators each appear twice, and one Archived facilitator appear twice. Within each pair, the displayed data and observed row identifier matched. The duplicates persisted after reload, and exact-name search still returned both rows. I did not inspect the underlying database. These rows can distort counts and pagination and make action targets ambiguous; whether distinct memberships may share the same display and identifier needs product confirmation.

### 4. Pending invitation has no visible identity when First Name is blank

**Steps to reproduce:**

1. I used a fresh email address with no pending invitation and opened **+ Facilitator → + Email Invitation**.
2. I left **First Name** blank, as the form permits, entered the required email address, and submitted.
3. I found the resulting record under **Sent Invitations** by searching for its email address.

**Expected result:** I expected the list to identify the recipient, for example by showing the email address when First Name was empty. Requiring a name in the form would also prevent an unidentified row.

**Actual result:** The invitation was sent and the mail catcher received it, but I saw no name or email in the pending row's **Facilitator** cell. Searching by address found the row without displaying that address. I could see the recipient address only in the **Withdraw** confirmation, and I withdrew the test invitation afterward. I reproduced this once. I also saw other blank pre-existing rows, but could not establish how they were created. Without a visible label, I could not identify or distinguish these invitations in the list.

### 5. Duplicate pending invitations: observation pending product rule

**Steps to reproduce:**

1. I invited a fresh email address through **+ Email Invitation** and verified one pending row.
2. While it was still pending, I submitted a second invitation to the same address, searched for that address, and checked the test mail catcher.

**Expected result:** The product rule is unconfirmed. If simultaneous invitations are allowed, I would expect the list to distinguish them and indicate which link remains valid. If they are not allowed, I would expect the second submission to be prevented or reconciled.

**Actual result:** Both submissions succeeded. I saw two visually indistinguishable pending rows under **Sent Invitations** and two delivered emails. Each row had its own **Withdraw** action; withdrawing one left the other pending. I withdrew both test rows after verification. This remains an observation, **not a confirmed bug**, until the product team confirms the rule.

## Questions for the product team

1. Is **Reinstate** intended to create a new pending **Sim Notification** invitation, rather than restore Active rights immediately? Should the recipient accept again?
2. Can one person legitimately appear under **Active** and **Sent Invitations** at the same time? An unmodified record was observed in both sections, but its meaning is unclear.
3. Are multiple pending invitations to the same address allowed? If so, how should administrators distinguish them and know which link remains valid?
4. Should the optional **First Name** be required, or should **Sent Invitations** display the email address as a fallback label?
5. Can two visually identical rows with the same observed identifier represent distinct memberships, or should each record appear only once per section?
6. Should **Resend** show a success or failure message? A message was delivered, but no clear page feedback was noticed.
7. Are different picker filter categories intended to combine using AND or OR? The tested account matched both selected conditions, so the observed result cannot decide this.
8. Is **+ Facilitator** the intended entry point for adding facilitators, or should there be a separate tab?

## Blocked checks and next steps

The following checks were **not completed** because invitations to the first and second recipients reached the error in finding 1:

- Recipient sees the invitation through application notification or email and accepts it.
- The sender sees exactly one Active record and no pending record after acceptance.
- The recipient can perform a facilitator-only action after acceptance.
- **Remove** revokes that actual permission, and **Reinstate** restores it or starts a new acceptance flow as specified.

After finding 1 is fixed, first verify both recipients are absent from Active and Sent Invitations. Invite the first recipient through the existing-account picker and the second through the email form, using a single submission for each. Verify the pending records, delivery and recipient acceptance, the sender's resulting status, and a concrete facilitator-only permission before and after Remove and Reinstate. If either route returns 500 again, capture the error and stop that route to avoid ambiguous duplicate submissions.

## Test data and limitations

- I created four successful invitations to fresh addresses: one named, one without a first name, and two duplicate pending invitations. I withdrew all four. I did not open a registration link.
- I reinstated two records from **Archived**; they remained in **Sent Invitations** at the end. One was already Archived before testing; I moved the other there from Active via **Remove**. I did not restore their original states.
- Seven attempts to invite registered accounts failed: four through the picker and three through the email form. A screenshot of an error page does not independently establish which route produced it or whether the server saved a partial invitation. I also checked the responsive controls and combined filters described above.
- I tested the staging environment only. I did not access production. Successful submission of an invitation to a new address does not establish successful acceptance or registration.
