..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Affinity Group Instance Live migration Override
==========================================

This spec outlines the prloblem of migrating instances when they are in a server
group with hard affinity(anti-affinity).

Problem description
===================

Migrating instances in server groups with hard affinity(anti-affinity) can 
become impossible when the number of instances in a server group gets large. 

In OpenStack, instances can be added to server groups only at the time of 
instance creation, and you can remove an instance from a server group by 
deleting an instance. 
Users, usually without any information about the number of hypervisors in zone 
or a region, create server groups and add instances. 
Current server group policy can easily block maintenance procedures by 
operators. 
When an administrator tries to decommission a hypervisor for hardware failures, 
a server group's hard anti-affinity or affinity policies can stop the 
evacuation of instances.

Use Cases
---------

Cloud Operator should evacuate a hypervisor for hardware failure. 
Hypervisor hosts instances that are part of a server group with hard 
anti-affinity. Evacuation fails because the number of instances in the server 
group is now more than the number of available hypervisors. Now, the operator 
can delete the instance or the server group.

Proposed change
===============

Change server group policy—the ability to change the server group policy.
When migrations fail, the operator can adjust the server group policy from 
hard to soft. The operator can now migrate the instances and later set the 
server group policy back to hard if desired.

Alternatives
------------

Migration flag to ignore policy; Add a parameter to live migration to ignore 
server group policy. We discussed before in 
https://etherpad.opendev.org/p/nova-bobcat-ptg.
Later, the status on the server group shows that the server group policy 
was ignored.

In either solution, the scheduler will heal the policy over time with subsequent
migrations.

Data model impact
-----------------

Changing server group policy or alternative solution will not change the data 
model.

REST API impact
---------------

A new API method to update ``os-server-groups`` will be added.

* PUT ``/os-server-groups/{server_group_id}``
  * update server group policy with ``anti-affinity``, ``affinity``, 
  ``soft-anti-affinity``, and ``soft-affinity``.

  * Method type PUT

  * Normal response codes: 200

  * Error response codes: badRequest(400), unauthorized(401), forbidden(403), 
  conflict(409)


  * ``/os-server-groups/{server_group_id}``

  * policy field:
    * anti-affinity
    * affinity
    * soft-anti-affinity
    * soft-affinity
  
  * Schema definition for the request body data:
  
  {
    "server_group": {
        "name": "test",
        "policy": "anti-affinity",
    }
  }

Security impact
---------------

N/A

Notifications impact
--------------------

N/A

Other end user impact
---------------------

python-novaclient and openstack client must be updated.

* openstack server group update <server-group> 

Performance Impact
------------------

It does not impact performance.


Other deployer impact
---------------------

N/A

Developer impact
----------------

Discuss things that will affect other developers working on OpenStack,
such as:

* If the blueprint proposes a change to the driver API, discussion of how
  other hypervisors would implement the feature is required.

Upgrade impact
--------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  npustchi

Feature Liaison
---------------

We have previously discussed with the Nova core team at Vancouver Nova PTG[2]_.

Feature liaison:
  Sylvian Bauza
  Balazs Gibizer

Work Items
----------

* API update, add update module to PUT.

* Backend update method.

* DB update method.

* CLI documentation.

Dependencies
============

N/A


Testing
=======

Unit and/or functional testing for the quota limit migrate tool wil be added.


Documentation Impact
====================

Operators and Users are impacted by this change. The following docs will need 
to be updated:

* https://docs.openstack.org/python-openstackclient/latest/cli/
command-objects/server-group.html

References
==========

.. [1] https://etherpad.opendev.org/p/vancouver-june2023-nova
.. [2] https://etherpad.opendev.org/p/nova-bobcat-ptg

History
=======

Optional section intended to be used each time the spec is updated to describe
new design, API or any database schema updated. Useful to let reader understand
what's happened along the time.

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2023.2 Bobcat
     - Introduced
