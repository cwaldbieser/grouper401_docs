========
Appendix
========

.. _apdx-401.1.4-auto-end-date:

-------------------------------
401.1.4 Automatic End Date Rule
-------------------------------

To configure the automatic rule end date on the access control list,
`app:vpn:ref:vpn_adhoc` you must use the Grouper Shell (GSH) to run
a short script.  To run GSH, you must connect to the GTE container
that has the Grouper API installed:

.. code-block:: bash

   root# docker exec -it CONTAINER_NAME /bin/bash 
   bash# cd bin
   bash# gsh

At this point you can paste in the following script:

.. code-block:: groovy
   :emphasize-lines: 1,3
   :linenos:

   numDays = 180;
   actAs = SubjectFinder.findRootSubject();
   vpn_adhoc = getGroups("app:vpn:ref:vpn_adhoc")[0];
   attribAssign = vpn_adhoc.getAttributeDelegate().addAttribute(RuleUtils.ruleAttributeDefName()).getAttributeAssign();
   attribValueDelegate = attribAssign.getAttributeValueDelegate();
   attribValueDelegate.assignValue(RuleUtils.ruleActAsSubjectSourceIdName(), actAs.getSourceId());
   attribValueDelegate.assignValue(RuleUtils.ruleRunDaemonName(), "F");
   attribValueDelegate.assignValue(RuleUtils.ruleActAsSubjectIdName(), actAs.getId());
   attribValueDelegate.assignValue(RuleUtils.ruleCheckTypeName(), RuleCheckType.membershipAdd.name());
   attribValueDelegate.assignValue(RuleUtils.ruleIfConditionEnumName(), RuleIfConditionEnum.thisGroupHasImmediateEnabledNoEndDateMembership.name());
   attribValueDelegate.assignValue(RuleUtils.ruleThenEnumName(), RuleThenEnum.assignMembershipDisabledDaysForOwnerGroupId.name());
   attribValueDelegate.assignValue(RuleUtils.ruleThenEnumArg0Name(), numDays.toString());
   attribValueDelegate.assignValue(RuleUtils.ruleThenEnumArg1Name(), "T");

.. _apdx-401.1.5-pit-query:

--------------------------------------
401.1.5 Point-in-Time Membership Query
--------------------------------------

.. code-block:: sql
   :linenos:

   SELECT 
       gpm.SUBJECT_ID, 
       gpg.NAME, 
       FROM_UNIXTIME(gpmav.MEMBERSHIP_START_TIME / 1000000) start_time, 
       FROM_UNIXTIME(gpmav.MEMBERSHIP_END_TIME / 1000000) end_time 
   FROM grouper_pit_memberships_all_v gpmav 
       INNER JOIN grouper_pit_groups gpg 
           ON gpmav.owner_group_id = gpg.id 
       INNER JOIN grouper_pit_members gpm 
           ON gpmav.MEMBER_ID = gpm.id 
       INNER JOIN grouper_pit_fields gpf 
           ON gpmav.field_id = gpf.id
   WHERE gpg.name = 'app:vpn:vpn_authorized' 
   AND gpm.subject_type = 'person'
   AND gpf.name = 'members'
   ORDER BY gpmav.MEMBERSHIP_START_TIME DESC 
   ;


