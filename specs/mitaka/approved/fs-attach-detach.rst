..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
FS Attach / Detach API
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/nova/+spec/XXX

Create new Manila share attach/detach APIs in Nova, which will enable future
automatic mounting of shares created by the service.

Problem description
===================

Nova service now has two methods of providing persistent storage.  Cinder, the
block storage service, is very tightly integrated with Nova, enabling users
to easily create, delete, attach, and detach volumes from their virtual
machines.  On the other hand, Manila, the file-as-a-service offering from
OpenStack, does not have the same level of integration with Nova.  Once a share
is created in Manila, it still requires manual intervention to mount and access
the file share.

Use Cases
---------

User creates share with the appropriate permissions.  They would then choose
to attach the share to specified virtual machines.

Proposed change
===============

This change is the first in a number of changes to allow the workflow proposed
in the previous section.  We propose a new API in Nova which would allow the
attachment and detachment of a volume onto a number of virtual machines.  This
specification does not provide the implementation behind the API, but instead
focuses on coming onto an agreement on the proposed API for Nova.

Alternatives
------------

Continue using the manual method used today, which may become unmanigable as
depending on the number of virtual machines accessing the share.

Data model impact
-----------------

TBD.

REST API impact
---------------

The following Nova APIs are proposed:

* Attach Share storage /v2.1/​{tenant_id}​/servers/​{server_id}​/action:

.. code-block:: json

    {
        "share_attach": {
            "share_id": "15e59938-07d5-11e1-90e3-e3dffe0c5983",
            "mount_point": "/net/mymount",
            "mount_options": "rw"
        }
    }

* Detach Share storage `/v2.1/​{tenant_id}​/servers/​{server_id}​/action`:

.. code-block:: json

    {
        "share_detach": {
            "share_id": "15e59938-07d5-11e1-90e3-e3dffe0c5983",
            "mount_point": "/net/mymount",
        }
    }

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

We would like to get feedback not only from the Nova community but also from
the Manila community.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

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

* Nova actions API:
    * http://developer.openstack.org/api-ref-compute-v2.1.html#os-server-actions-v2.1
* Attach/Detach email:
    * https://www.mail-archive.com/openstack-dev@lists.openstack.org/msg66667.html

