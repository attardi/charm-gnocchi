Overview
--------

This charm provides the Gnocchi Metrics as a Service for OpenStack.
Gnocchi provides a REST API service to create and manipulate metric data.
This charm should be deployed with Ceilometer.

Requirements
------------

Gnocchi needs storage for incoming data. The charm currently supports either local file storage or Ceph.
When using Ceph, a Ceph user and a pool must have been created. They can be created for example with:

    ceph osd pool create metrics 8 8 erasure
    ceph auth get-or-create client.gnocchi mon "allow r" osd "allow rwx pool=metrics"

Usage
-----

Gnocchi requires a database for indexing the data, which could be either MySQL or PostgreSQL.
In order to deploy the Gnocchi service:

    juju deploy mysql
    juju deploy gnocchi
    juju add-relation gnocchi mysql

then a Keystone relationship needs to be established:

    juju add-relation gnocchi keystone:identity-service

Gnocchi provides an API service that can be used to retrieve
Openstack metrics.

HA/Clustering
-------------

There are two mutually exclusive high availability options: using virtual
IP(s) or DNS. In both cases, a relationship to hacluster is required which
provides the corosync back end HA functionality.

To use virtual IP(s) the clustered nodes must be on the same subnet such that
the VIP is a valid IP on the subnet for one of the node's interfaces and each
node has an interface in said subnet. The VIP becomes a highly-available API
endpoint.

At a minimum, the config option 'vip' must be set in order to use virtual IP
HA. If multiple networks are being used, a VIP should be provided for each
network, separated by spaces. Optionally, vip_iface or vip_cidr may be
specified.

To use DNS high availability there are several prerequisites. However, DNS HA
does not require the clustered nodes to be on the same subnet.
Currently the DNS HA feature is only available for MAAS 2.0 or greater
environments. MAAS 2.0 requires Juju 2.0 or greater. The clustered nodes must
have static or "reserved" IP addresses registered in MAAS. The DNS hostname(s)
must be pre-registered in MAAS before use with DNS HA.

At a minimum, the config option 'dns-ha' must be set to true and at least one
of 'os-public-hostname', 'os-internal-hostname' or 'os-internal-hostname' must
be set in order to use DNS HA. One or more of the above hostnames may be set.

The charm will throw an exception in the following circumstances:
If neither 'vip' nor 'dns-ha' is set and the charm is related to hacluster
If both 'vip' and 'dns-ha' are set as they are mutually exclusive
If 'dns-ha' is set and none of the os-{admin,internal,public}-hostname(s) are
set

Network Space support
---------------------

This charm supports the use of Juju Network Spaces, allowing the charm to be bound to network space configurations managed directly by Juju.  This is only supported with Juju 2.0 and above.

API endpoints can be bound to distinct network spaces supporting the network separation of public, internal and admin endpoints.

To use this feature, use the --bind option when deploying the charm:

    juju deploy gnocchi --bind "public=public-space internal=internal-space admin=admin-space"

alternatively these can also be provided as part of a juju native bundle configuration:

    gnocchi:
      charm: cs:xenial/ceilometer
      bindings:
        public: public-space
        admin: admin-space
        internal: internal-space

NOTE: Spaces must be configured in the underlying provider prior to attempting to use them.

NOTE: Existing deployments using os-*-network configuration options will continue to function; these options are preferred over any network space binding provided if set.
