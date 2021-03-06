Security Segregation
====================

Problem description
-------------------

Problem Definition
++++++++++++++++++

The goal of this use-case is to present the need for a (partly) segregation
of physical resources to support the well-known classic separation of DMZ
and MZ, which is still needed by several applications (VNFs) and requested
by telco security rules.

The main driver therefore is that a vulnerability of a single system must not
affect further critical systems or endanger exposure of sensitive data. On the
one side the benefits of virtualization and automation techniques are mandatory
for telcos but on the other side telecommunication data and involved systems
must be protected by the highest level of security and comply to local
regulatory laws (which are often more strict in comparison with enterprise).

Placement Zones should act as multiple lines of defense against a security
breach. If a security breach happens in a placement zone, all other placement
zones and related VNFs must not be affected. This must be ensured by the design.

This use case affects all of the main OpenStack modules.

Opportunity/Justification
+++++++++++++++++++++++++

Separation of DMZ and MZ is a common requirement of VNFs to meet
communication service provider security requirements.

Use Cases
---------

User Stories
++++++++++++

Current Situation
-----------------
Today the DMZ and MZ concept is an essential part of the security design
of nearly every telco application deployment. Today this separation is
done by a consequent physical segregation including hosts, network and
management systems. This separation leads to high investment and
operational costs.

Enable the following
--------------------
Placement Zones should be used to reduce the risk that the whole cloud platform
is affected by a serious security breach.

If the hypervisor is breached, there is a risk to all of the VNFs on that
hypervisor. To reduce this risk, a physical separation of VMs not assigned to
the same security class is necessary. Placement Zones should be used to ensure
that only VMs following the same security classification will run on the same
group of physical hosts.
This should avoid a mix of VMs from different zones (e.g. DMZ and MZ),
coming up with different security requirements, running on the same group
of hosts. Therefore a host (respectively multiple hosts) must be classified
and assigned to only one placement zone. During the deployment process of a
VM it must be possible to assign it to one placement zone (or use the
default one), which automatically leads to a grouping of VMs.

The security separation within the network can be done on a logical layer
using the same physical elements but adding segregation through VLANs
(Virtual LAN), virtual firewalls and other overlay techniques.

The security separation for virtual machine storage can be done on a logical
layer. It must be ensured that a hypervisor belonging to a specific placement
zone can not access the storage of a different placement zone. Otherwise an
attacker could inject malicious code into the virtual disk of a VM.

Usage Scenarios Examples
++++++++++++++++++++++++

An application presentation layer (e.g. webserver) must be decoupled from
the systems containing sensitive data (e.g. database) through at least one
security enforcement device (e.g. virtual firewall) and a separation of
underlying infrastructure (hypervisor). The intent being to minimize the
likelihood of a breach of a component runing in one zone resulting in a breach
of another component running in a separate zone.

*Potential candidate for Group Based Policy with Service Function Chaining from
a network perspective? GBP would need updates for this concept.*

https://wiki.openstack.org/wiki/File:TelcoWG_Placementzones.png


Related User Stories
++++++++++++++++++++

None.

Requirements
++++++++++++

* One OpenStack installation must be capable to manage different placement
  zones. All resources (compute, network and storage) are assigned to one
  placement zone. By default, all resources are assigned to the "default"
  placement zone of OpenStack
* It must be possible to configure the allowed communication between
  placement zones on the network layer
* SEC is a special placement zone - it provides the glue to connect the
  placement zones on the network layer using VNFs. SEC VNFs may be attached to
  resources of other placement zones
* Placement zone usage requires a permission (in SEC, tenants cannot start VMs,
  this zone supports only the deployment of Xaas services FWaas, LBaas,...])
* If placement zones are required in a cloud, VMs must be assigned to one
  placement zone
* All resources, which are needed to run a VM must belong to the same placement
  zone
* Physical Hosts (compute nodes) must be able to assigned to only one placement
  zone and re-assigning should be possible due to changing utilization
** Several assignments must be restricted by the API
** If a host is reassigned it must evacuate all existing VM
* ...and the whole thing must be optional :-)

Gaps
++++

**Nova issues:**

* Usage of availability zone(AZ)/host aggregates to assign a vm to a placement
  zone is feasable [Ref.1], but:

  * By default a physical host can be assigned to multiple host aggregates
  * It is up to the operator to ensure security using non OpenStack mechanisms
  * Maybe Congress [Ref. 2] (Policy as a Service) could be a solution?

* Cells offer segregation of a compute environment in a manner that is
  transparent to the end user but no not in and of themselves allow the type of
  explicit placement targeting a specific cell that would be required here.

**Neutron issues:**

* AZs or PZs, and Cells are not known to Neutron services

  * It's up to the operator to ensure that the right networks are attached to VMs

**Cinder/Manila/Storage issues:**

* Storage can be segregated with volume-types
* AZs are not known to the storage services

  * Must be ensured from the deployment tool that the right storage is accessible

**OpenStack regions** provide a segregation of all resources. The region concept
can be used to implement placement zones, but:

* Complex and resource consuming installation for the OpenStack management
  systems
* Tenants must deal with additional regions
* No L2 network sharing for VMs in the SEC placement zone required to glue the
  zones together
* No real enforcement
* Complex operations

External References
+++++++++++++++++++

* [1]: http://docs.openstack.org/openstack-ops/content/scaling.html
* [2]: https://wiki.openstack.org/wiki/Congress

Glossary
--------

**AZ**
  Availability Zone (OpenStack terminology)

**DMZ**
  Demilitarized Zone provides access to the public network,
  but adds an additional security layer (e.g. virtual firewall). Designed for
  security critical customer facing services (e.g. customer control center).

**EHD**
  Exposed Host Domain provides direct access from the public network (e.g.
  Internet).
  Designed for services which require a high traffic volume (e.g. CDN) and are
  not security critical.

**MZ**
  Militarized Zone is a logical network without any access from the public
  network. Designed for systems without direct customer connectivity (e.g.
  databases containing sensitive data) and high security demands.

**PZ**
  Placement Zone is a concept to classify different securiy areas based on
  different security requirements. PZ are separated on a per host basis.

**SEC**
  Secure Network Zone for all devices providing a security function including
  devices providing connectivity between Placement Zones (e.g. virtual firewall
  for DMZ-MZ traffic).

**VNF**
  Virtual Network Function is an implementation of an functional building block
  within a network infrastructure that can be deployed on a virtualization
  infrastructure or rather an OpenStack based cloud platform (a non virtualized
  network function is today often a physical appliance).
