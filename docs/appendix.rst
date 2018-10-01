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

