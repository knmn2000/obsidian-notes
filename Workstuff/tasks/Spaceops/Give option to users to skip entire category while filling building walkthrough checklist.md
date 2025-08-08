```
Problem Statement: The current Building Walkthrough Checklist is designed to capture detailed information for each floor in a building. However, there are scenarios where certain checklist categories or questions are irrelevant to some floors. This creates redundancy and inefficiencies, as users are required to provide responses even when the data is not applicable.

Key issues:

Categories/questions are replicated across all floors, even if they are irrelevant (e.g., a Gym category is relevant only to the 1st floor but still appears for other floors).

Users need to manually switch between floors to fill out the checklist for each one, which is time-consuming and cumbersome.

Proposed Solution

For a faster solution in Phase 1, focus on minimizing user friction with these steps:

Allow Category/Question Skipping:

Enable a "Skip" option for categories/questions during the Building Walkthrough checklist.

Skipped items can be flagged as "Skipped"  in the report and analytics

Simplify Floor Navigation:

Implement auto-floor selection to reduce the need for manual navigation between floors.
```

checklist category mapping -> add skippable
response entity modify karna hoga -> the user chose to skip this category. boolean. 
controller layer pe 
src/modules/response/response.interface.ts -> skipped categories array lelo. 

when inserting responses into DB, if current resposne ka category is in skipped_categories, toh fir `skipped` = true karna hai. 

checklist ke baad score nikalta hai -> what happens here, need to udnerstand product reqs.

questions mandatory ho toh bhi skipping allowed hona chahiye in UI. 
- pls check if there is any check for mandatory questions. we will need to handle that as well

`skipped` should be false by default in DB.

--- 

check
- checklist category entity x
- response entity x
- questions entity
- options entity
- checklist entity
- question checklist mapping entity
- response asset entity
----

mandatory and critical questions
^ must fill.         ^ more weightage while scoring


scores -> only related to audit, no need to chcek for walkthrough
```typescript
      case ResponseSummaryStatus.Complete:
        if (summary.status !== ResponseSummaryStatus.InProgress) {
          throw new ForbiddenException('This operation is not allowed');
        }
        summary.status = ResponseSummaryStatus.Complete;
        if (summary.checklist.module === checklistModuleEnum.AUDIT) {
          await this.updateScores(summary);
        }
        break;
```
