`Home <index.html>`_ OpenStack-Ansible Installation Guide

Initial environment configuration
---------------------------------

OpenStack-Ansible depends on various files that are used to build an inventory
for Ansible. Start by getting those files into the correct places:

#. Recursively copy the contents of the
   ``/opt/openstack-ansible/etc/openstack_deploy`` directory to the
   ``/etc/openstack_deploy`` directory.

#. Change to the ``/etc/openstack_deploy`` directory.

#. Copy the ``openstack_user_config.yml.example`` file to
   ``/etc/openstack_deploy/openstack_user_config.yml``.

Deployers can review the ``openstack_user_config.yml`` file and make changes
to how the OpenStack environment is deployed. The file is **heavily** commented
with details about the various options.

There are various types of physical hosts that will host containers that are
deployed by OpenStack-Ansible. For example, hosts listed in the
`shared-infra_hosts` will run containers for many of the shared services
required by OpenStack environments. Some of these services include databases,
memcache, and RabbitMQ.  There are several other host types that contain
other types of containers and all of these are listed in
``openstack_user_config.yml``.

For details about how the inventory is generated from the environment
configuration, please see :ref:`developer-inventory`.

Affinity
^^^^^^^^

OpenStack-Ansible's dynamic inventory generation has a concept called
*affinity*. This determines how many containers of a similar type are deployed
onto a single physical host.

Using `shared-infra_hosts` as an example, let's consider a
``openstack_user_config.yml`` that looks like this:

.. code-block:: yaml

    shared-infra_hosts:
      infra1:
        ip: 172.29.236.101
      infra2:
        ip: 172.29.236.102
      infra3:
        ip: 172.29.236.103

Three hosts are assigned to the `shared-infra_hosts` group, so
OpenStack-Ansible will ensure that each host runs a single database container,
a single memcached container, and a single RabbitMQ container. Each host has
an affinity of 1 by default, and that means each host will run one of each
container type.

Some deployers may want to skip the deployment of RabbitMQ altogether. This is
helpful when deploying a standalone swift environment. For deployers who need
this configuration, their ``openstack_user_config.yml`` would look like this:

.. code-block:: yaml

    shared-infra_hosts:
      infra1:
        affinity:
          rabbit_mq_container: 0
        ip: 172.29.236.101
      infra2:
        affinity:
          rabbit_mq_container: 0
        ip: 172.29.236.102
      infra3:
        affinity:
          rabbit_mq_container: 0
        ip: 172.29.236.103

The configuration above would still deploy a memcached container and a database
container on each host, but there would be no RabbitMQ containers deployed.


.. _security_hardening:

Security Hardening
^^^^^^^^^^^^^^^^^^

OpenStack-Ansible automatically applies host security hardening configurations
using the `openstack-ansible-security`_ role. The role uses a version of the
`Security Technical Implementation Guide (STIG)`_ that has been adapted for
Ubuntu 14.04 and OpenStack.

The role is applicable to physical hosts within an OpenStack-Ansible deployment
that are operating as any type of node -- infrastructure or compute. By
default, the role is enabled. Deployers can disable it by changing a variable
within ``user_variables.yml``:

.. code-block:: yaml

    apply_security_hardening: false

When the variable is set to ``true``, the ``setup-hosts.yml`` playbook applies
the role during deployments.

Deployers can apply security configurations to an existing environment or audit
an environment using a playbook supplied with OpenStack-Ansible:

.. code-block:: bash

    # Perform a quick audit using Ansible's check mode
    openstack-ansible --check security-hardening.yml

    # Apply security hardening configurations
    openstack-ansible security-hardening.yml

For more details on the security configurations that will be applied, refer to
the `openstack-ansible-security`_ documentation. Review the `Configuration`_
section of the openstack-ansible-security documentation to find out how to
fine-tune certain security configurations.

.. _openstack-ansible-security: http://docs.openstack.org/developer/openstack-ansible-security/
.. _Security Technical Implementation Guide (STIG): https://en.wikipedia.org/wiki/Security_Technical_Implementation_Guide
.. _Configuration: http://docs.openstack.org/developer/openstack-ansible-security/configuration.html

--------------

.. include:: navigation.txt
