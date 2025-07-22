# Officer Action Requests: Managing HR Changes for Law Enforcement
This Power Platform solution streamlines the "Officer Action Requests" workflowm, an essential HR-driven process designed for law enforcement agencies. An "Officer Action Request" refers to any HR-related update concerning a sworn officer that requires formal documentation and leadership approval. For example, these requests may include role changes, promotions, salary adjustments, address updates, reassignment to a new precinct or division, badge number revisions, and even status changes due to retirement or extended leave. Each of these actions has operational impact and must be carefully tracked and authorized.

By centralizing and automating this process, the solution ensures that every personnel update—no matter how routine or complex—is handled consistently, transparently, and securely. For state and local law enforcement organizations, this system offers more than just workflow automation: it strengthens institutional accountability, supports informed decision-making, and lightens the administrative load on HR and command staff. Built on Microsoft’s low-code Power Platform, it provides a scalable and responsive framework that adapts to evolving agency needs.

## Demo
You can find a demo video of this solution's capabilities by clicking the image below:

[![demo thumbnail](https://i.imgur.com/K5hf5Tg.png)](https://youtu.be/58DGS8trTBs)

Timestamps:
- **0:00** - an admin submits a new Officer Action Request
- **1:17** - the officer involved can review the OAR in the "My Officer Action Requests" app
- **1:36** - each OAR approver (leadership) gets notified via email that a new OAR is awaiting their approval
- **1:45** - *approval scenario:* each OAR approver reviews and approves the OAR in the "OAR Review and Approval" app
- **3:04** - *approval scenario:* admin gets a notification that all approvers have approved and the completed OAR is available
- **4:02** - *approval scenario:* staff responsible for enacting the OAR change get notified of a newly assigned task via email
- **4:48** - *approval scenario:* the officer can see their OAR has now been marked as complete
- **5:05** - *rejection scenario:* OAR is reviewed and rejected due to incorrect data
- **5:47** - *rejection scenario:* admin is notified of the rejection, with comments
- **6:10** - *rejection scenario:* admin makes necessary changes to OAR and re-routes for approval

## Here's how it works!
![solution](https://i.imgur.com/weezvHx.png)

As depicted above, there are **four** unique personas, or roles, that are involved in the Officer Action Request process:
- **The Admin**: kicks off the process by initiating the officer action request and routing for approval.
- **The Officer**: the officer that the OAR pertains to. They will have read-only access to the OAR process, observing their HR-related update as it proceeds through approval.
- **OAR Approver**: approvers that act in a leadership capacity, reviewing and approving/rejecting the OARs.
- **OAR Implementer**: staff that, once each OAR is approved, *enact* the change outlined in the OAR in the related systems (i.e. an OAR that involves a role change might also bring about a necessary change in that officer's working hours and pay level).

The overall process is described below:
1. **The admin** uses the *OAR Submission App*, a Power Apps Canvas App, to submit a new Officer Action Request (OAR). In this app, they fill in officer details as well as details about the change that is being proposed in this OAR (i.e. promotion, preccint change, etc). At the very end, they assign the appropriate approvers of this OAR (or this can be auto-assigned, it is up to you).
2. **The officer**, via the *OAR Review App*, a Power Apps Canvas App, has read-only access to their OARs. The officer can open the app and see the documented changes the OAR that was just filed is proposing.
3. **The OAR Approvers** can review the filed OAR, its proposed change, and any comments that were provided. After review, they can choose to approve or reject an OAR and provide their signature via a pen input control or via "auto-sign" (fills in a cursive version of their name). This step involves three-way approval from the officer's **Commander**, **Colonel**, and **Director**, but the specific approving roles can be changed.
4. Upon complete approval from all three approvers, a Power Automate workflow automatically merges all relevant OAR data into an OAR PDF, merges the approver's signatures, and saves it to OneDrive for archive purposes. If desired, this can then be physically printed if storing hard copies (actual paper) is a requirement. A notification email to the admin is automatically routed via that Power Automate flow as well, notifying them the merged PDF is now available for printing.
5. After approval of the OAR, the **OAR Implementers** are then assigned *tasks* to complete their applicable duties to carry out the proposed change in the OAR. For example, if an officer receives a promotion to a new role in a new preccint, staff will need to make necessary changes in the agency's payroll system. These implementers are notified of their new task via email and can access a model-driven Power App for reviewing these tasks and marking them complete.

## Solution
*(solution coming)*

As described further below, the *Produce OAR PDF* workflow fills OAR data into a Word Document Template before exporting that document to a PDF. You can download a copy of that Word doc template: [OAR_Template.docx](https://github.com/microsoft/SLG-Business-Applications/releases/download/42/OAR_Template.docx)

## All Power Apps Apps, Described
This solution package comes with **four** Power Apps apps:

![apps](https://i.imgur.com/jGa7SPY.png)

- **Officer Action Request Submission**: used by the admin to submit new Officer Action Requests.
- **Review my OARs**: used by any officer to view a list of their OARs (that are making changes to them), in read-only mode.
- **OAR Review and Approval**: used by the OAR Approvers to review and approve/reject OARs that they have been assigned to approve.
- **Officer Action Request Management**: model-driven app used to manage the overall process. Can be used by an admin to update OARs and the OAR implementers to view their assigned enactment tasks and mark them as complete.

## All Power Automate Flows, Described
This solution package comes with **five** Power Automate flows.

![flows](https://i.imgur.com/tTGe1rV.png)

- **When OAR is Submitted, Assign Approvers**: this is the first flow that kicks off an begins the process. Upon an admin submitting an *Officer Action Request* record, this flow triggers, creates "Approval Tasks" (an *Approval Task* record) for each of the three assigned approvers, and then emails each of the three assigned approvers to inform them they have a new OAR outstanding, awaiting their review and approval.
- **Route OAR for Rejection**: the moment an approver decides to *reject* an OAR, this flow kicks off. It is trigger directly via the Power App approver app they use. This flow handles the responsibility of canceling (deleting) all *remaining* approval tasks on that OAR (no need for other approvers to review it now that it is rejected) and then drafts an email to the admin with the approver's comments on why they rejected it and delivers this to the admin. The admin then can read those comments, make the necessary changes, and then "re-route" the OAR for approval.
- **Produce OAR PDF**: upon all three approvers approving the OAR, this workflow kicks off and performs the merging of all OAR data, and its approvals, into a PDF document that represents the OAR. It uses the Microsoft Word connector to populate a word document, convert it to a PDF, save it to OneDrive, and then save a link to that PDF back to the OAR Dataverse record where anyone can later access it.
- **Create OAR Enactment Tasks**: upon all three approvers approving the OAR, this flow then fires off to assign an *OAR Enactment Task* to each of the three staff that need to update the systems they are accountable for - payroll, time, and GIR (insurance). The staff (referred to as "OAR Implementers" in this) can then review their assigned enactment tasks in the model-driven app.
- **Notify of new Enactment Tasks**: triggers upon the new *OAR Enactment Task* record being made. Delivers an email to the person the OAR Enactment Task is assigned to to notify them of their newly assigned task.

## Known Limitations
Please keep in mind the provided solution is not intended as a *complete*, OOTB-ready solution. Instead, you can look at this as "an accelerator" that is approximately 80% of the way there as a working solution, but needs final configuration/adaptation to your particular business processes.

If you choose to implement this accelerator, these are the areas you will need to spend time building out and configuring further:
- "Once fully approved by all approvers" flow trigger logic not built:
    - The *Produce OAR PDF* workflow is supposed to trigger upon all three approvers approving the OAR. This logic is not configured. For demo purposes, this workflow is simply configured with a "Manually trigger a flow" trigger. If implementing this, you will need to build the logic that checks and validates all three approvers have approved before continuing.
    - The *Create OAR Enactment Tasks* workflow's trigger is not configured to automatically run upon all three approvers approving. For demonstration of capability purposes, the trigger is just set to "Manually trigger a flow". If implementing this solution, you'll need to build in the logic that continuously checks for all three approvers having approved and triggers from there.
- The *Produce OAR PDF* workflow that performs the merging of content into the Word Document template only uses the signature image thumbnail, not the full resolution image. If you are actually deploying this, it is probably best to use the Dataverse API call to pull down the full resolution image instead of using the low-quality thumbnail it provides.
- The *Product OAR PDF* workflow does not select what type of OAR it is (i.e. put an "X" near "promotion", etc). That logic would have to be built.
- In the model-driven app, the "Re-Route for Approval" button that the admin uses to kick-off the OAR approval process again (assign approval tasks) does not actually work. If you are implementing this solution, you will need to build that logic that triggers a similar process to the "When OAR is Submitted, Assign Approvers" workflow; it may even be best to extrapolate that logic to a single workflow that can be called both at the beginning of the OAR being submitted *and* in the event of a re-route.

## Credit
Created by [Tim Hanewich](https://github.com/TimHanewich).