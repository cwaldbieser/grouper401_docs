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


.. _apdx-401.2.5-future-memberships-query:

--------------------------------
401.2.5 Future Memberships Query
--------------------------------

.. code-block:: sql
   :linenos:

   SELECT 
       ggv.name,
       FROM_UNIXTIME(gmav.IMMEDIATE_MSHIP_ENABLED_TIME / 1000) enabled_time,
       CASE
           WHEN gm.subject_type = 'group' THEN gm.subject_identifier0
           ELSE gm.subject_id
       END member
   FROM `grouper_memberships_all_v` gmav
       INNER JOIN grouper_groups_v ggv
           ON gmav.OWNER_GROUP_ID = ggv.GROUP_ID
       INNER JOIN grouper_members gm
           ON gmav.member_id = gm.id
   WHERE gmav.IMMEDIATE_MSHIP_ENABLED_TIME IS NOT NULL
   ;

.. _apdx-401.3.1-app-skeleton:

-----------------------------------
401.3.1 Application Skeleton Script
-----------------------------------

This script automatically creates an application folder along with
security groups and permission rules.
You must use the Grouper Shell (GSH) to run
a short script.  To run GSH, you must connect to the GTE container
that has the Grouper API installed:

.. code-block:: bash

   root# docker exec -it CONTAINER_NAME /bin/bash 
   bash# cd bin
   bash# gsh

At this point you can paste in the following script:

.. code-block:: groovy
   :emphasize-lines: 3,4
   :linenos:

   // SET THESE
   parent_stem_path = "app";
   app_extension = "boardeffect";
   app_name = "Board Effect";
    
    
   if (!app_name?.trim())
   {
       app_name = app_extension;
   }
    
   def makeStemInheritable(obj, stemName, groupName, priv="admin") {
       baseStem = obj.getStems(stemName)[0];
       aGroup = obj.getGroups(groupName)[0];
       RuleApi.inheritGroupPrivileges(
           SubjectFinder.findRootSubject(),
           baseStem,
           Stem.Scope.SUB,
           aGroup.toSubject(),
           Privilege.getInstances(priv)
       );
       RuleApi.runRulesForOwner(baseStem);
       if(priv == 'admin')
       {
           RuleApi.inheritFolderPrivileges(
               SubjectFinder.findRootSubject(),
               baseStem,
               Stem.Scope.SUB,
               aGroup.toSubject(),
               Privilege.getInstances("stem, create"));
       }
       RuleApi.runRulesForOwner(baseStem);
   }
    
   stem = addStem(parent_stem_path, app_extension, app_name);
   etc_stem = addStem(stem.name, "etc", "etc");
   admin_group_name = "${app_extension}_admins";
   admin_group = addGroup(etc_stem.name, admin_group_name, admin_group_name);
   admin_group.grantPriv(admin_group.toMember().getSubject(), AccessPrivilege.ADMIN);
   mgr_group_name = "${app_extension}_mgr";
   mgr_group = addGroup(etc_stem.name, mgr_group_name, mgr_group_name);
   mgr_group.grantPriv(admin_group.toMember().getSubject(), AccessPrivilege.ADMIN);
   mgr_group.grantPriv(mgr_group.toMember().getSubject(), AccessPrivilege.UPDATE);
   mgr_group.grantPriv(mgr_group.toMember().getSubject(), AccessPrivilege.READ);
   view_group_name = "${app_extension}_viewers";
   view_group = addGroup(etc_stem.name, view_group_name, view_group_name);
   view_group.grantPriv(view_group.toMember().getSubject(), AccessPrivilege.READ);
   view_group.grantPriv(admin_group.toMember().getSubject(), AccessPrivilege.ADMIN);
   view_group.grantPriv(mgr_group.toMember().getSubject(), AccessPrivilege.UPDATE);
   view_group.grantPriv(mgr_group.toMember().getSubject(), AccessPrivilege.READ);
   admin_group.grantPriv(view_group.toMember().getSubject(), AccessPrivilege.READ);
   mgr_group.grantPriv(view_group.toMember().getSubject(), AccessPrivilege.READ);
   // Child objects should also grant perms to these groups.
   makeStemInheritable(this, stem.name, admin_group.name, 'admin');
   makeStemInheritable(this, stem.name, mgr_group.name, 'update');
   makeStemInheritable(this, stem.name, mgr_group.name, 'read');
   makeStemInheritable(this, stem.name, view_group.name, 'read');
   admin_group.revokePriv(mgr_group.toMember().getSubject(), AccessPrivilege.UPDATE);

.. _apdx-401.3.5-temp-access:

-------------------------------
401.3.1 Temporary Access Script 
-------------------------------

This script automatically creates an application folder along with
security groups and permission rules.
You must use the Grouper Shell (GSH) to run
a short script.  To run GSH, you must connect to the GTE container
that has the Grouper API installed:

.. code-block:: bash

   root# docker exec -it CONTAINER_NAME /bin/bash 
   bash# cd bin
   bash# gsh

At this point you can paste in the following script:

.. code-block:: groovy
   :emphasize-lines: 2,3
   :linenos:

   // Script parameters
   group_name = "app:boardeffect:ref:workroom_helpers";
   numDays = 3;
    
   actAs = SubjectFinder.findRootSubject();
   vpn_adhoc = getGroups(group_name)[0];
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

