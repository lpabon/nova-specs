..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
FS Attach / Detach API
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/nova/+spec/XXX

Create new share attach/detach APIs in Nova to manage the connectivity and
access to shares created by Manila.

Problem description
===================

Until recently, OpenStack has had only two storage services providing
persistent storage for Nova instances.  The
first is Swift, OpenStack's object storage offering, which requires simple
HTTP to access its storage. The second is Cinder, the block storage service,
which requires the hypervisor to access to the allocated storage.  For
this reason, it is very tightly integrated with Nova, enabling users to
easily attach and detach volumes from their instances.

Now with Manila, the filesystem-as-a-service offering from OpenStack, users
have a third storage model which can provide persistent storage.  This
relatively new model is not very integrated with Nova, and as shuch,
its workflow is not as efficient as Cinder's.  Also, by not having Nova
participate in the workflow, it complicates the addition of future
enhancements in the Manila user model.


Use Cases
---------

User creates a Manila share with the appropriate permissions.  Using the same
workflow as Cinder, they would then *attach* the share to one or more Nova
instances, enabling them to access the network file system.

Proposed change
===============

We propose the following changes to Nova:

* Client needed to communicate with the Manila service.
* State saved in Nova database showing reflecting instance share attach state.
* Additional actions to the Nova action endpoint to attach, detach, and list
  shares associated with specified instances.

Alternatives
------------

Continue using the method used today.

Data model impact
-----------------

The data model will be largely based on the current implementation of how
Cinder volume ids are stored.

REST API impact
---------------

Attach Share
^^^^^^^^^^^^

Enable a Nova instance to access a Manila share.

The following Nova action is proposed to the API::

    POST /v2.1/​{tenant_id}​/servers/​{server_id}​/action

+---------------------+------------+-------------+--------------------------+
| Parameter           | Style      | Type        | Description              |
+=====================+============+=============+==========================+
| share_attach        | plain      | xsd:string  | The action               |
+---------------------+------------+-------------+--------------------------+
| share_id            | plain      | csapi:UUID  | The share ID             |
+---------------------+------------+-------------+--------------------------+
| mount_point(Opt)    | plain      | xsd:string  | The mount point location |
+---------------------+------------+-------------+--------------------------+

.. code-block:: json

  {
      "share_attach": {
          "share_id": "15e59938-07d5-11e1-90e3-e3dffe0c5983",
          "mount_point": "/net/mymount",
      }
  }

This operation does not return a response body.

When this call is executed, Nova will ensure the instance has access to the
share by using Manila *access* endpoint::

    POST /v2/​{tenant_id}​/shares/​{share_id}​/action

Once this call is successful, Nova will then save the attached *share_id*
on the database as it is done for Cinder volumes.

List Shares
^^^^^^^^^^^

List shares accessable by the instance.

A new JSON value is proposed to the following API::

    GET /v2.1/​{tenant_id}​/servers/​{server_id}​

+------------------------+------------+-------------+-----------------------+
| Parameter              | Style      | Type        | Description           |
+========================+============+=============+=======================+
| os-extended-           | plain      | csapi:dict  | Attached shares,      |
| shares:shares_attached |            |             | if any                |
+------------------------+------------+-------------+-----------------------+

Detach Share storage
^^^^^^^^^^^^^^^^^^^^

Remove access to a Manila share.

The following Nova action is proposed to the API::

    POST /v2.1/​{tenant_id}​/servers/​{server_id}​/action

+---------------------+------------+-------------+--------------------------+
| Parameter           | Style      | Type        | Description              |
+=====================+============+=============+==========================+
| share_detach        | plain      | xsd:string  | The action               |
+---------------------+------------+-------------+--------------------------+
| share_id            | plain      | csapi:UUID  | The share ID             |
+---------------------+------------+-------------+--------------------------+
| mount_point(Opt)    | plain      | xsd:string  | The mount point location |
+---------------------+------------+-------------+--------------------------+

.. code-block:: json

  {
      "share_detach": {
          "share_id": "15e59938-07d5-11e1-90e3-e3dffe0c5983",
          "mount_point": "/net/mymount",
      }
  }

This operation does not return a response body.

When this call is executed, Nova will ensure the instance has access to the
share removed by using Manila *access* endpoint::

    POST /v2/​{tenant_id}​/shares/​{share_id}​/action

Once this call is successful, Nova will then save the attached *share_id*
on the database as it is done for Cinder volumes.

Security impact
---------------

None.

Notifications impact
--------------------

Use the same notification service as attaching a Cinder volume.

Other end user impact
---------------------

None.

Performance Impact
------------------

None.

Other deployer impact
---------------------

None.

Developer impact
----------------

* This work is needed for one possible goal of having hypervisor mediated
  access, instead of our current network mediated security model.
* Together with [1] will enable future enhancements which could simplify the
  filesystem access workflow.
* We would like to get feedback not only from the Nova community but also from
  the Manila community.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  lpabon

Work Items
----------

* Create new actions
* Create unit tests for actions

Dependencies
============

None


Testing
=======

* Unit tests

Documentation Impact
====================

API documentation will need to be updated.

References
==========

* [1] Manila support in metadata service
    * https://review.openstack.org/#/c/248301/
* Nova actions API:
    * http://developer.openstack.org/api-ref-compute-v2.1.html#os-server-actions-v2.1
* Manila API:
    * http://developer.openstack.org/api-ref-share-v2.html
* Attach/Detach email:
    * https://www.mail-archive.com/openstack-dev@lists.openstack.org/msg66667.html

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Mitaka
     - Introduced

