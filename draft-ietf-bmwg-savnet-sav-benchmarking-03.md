---
stand_alone: true
ipr: trust200902
cat: info
submissiontype: IETF
area: Operations and Management
wg: BMWG

docname: draft-ietf-bmwg-savnet-sav-benchmarking-03

title: Benchmarking Methodology for Intra-domain and Inter-domain Source Address Validation
abbrev: SAV Benchmarking Methodology
lang: en

author:
- ins:
  name: Li Chen
  org: Zhongguancun Laboratory
  city: Beijing
  country: China
  email: lichen@zgclab.edu.cn
- ins:
  name: Dan Li
  org: Tsinghua University
  city: Beijing
  country: China
  email: tolidan@tsinghua.edu.cn
- ins:
  name: Libin Liu
  org: Zhongguancun Laboratory
  city: Beijing
  country: China
  email: liulb@zgclab.edu.cn
- ins:
  name: Lancheng Qin
  org: Zhongguancun Laboratory
  city: Beijing
  country: China
  email: qinlc@zgclab.edu.cn

normative:
  RFC2827:
  RFC3704:
  RFC4061:
  RFC5210:
  RFC8704:
  RFC2544:
  I-D.ietf-savnet-intra-domain-problem-statement:
  I-D.ietf-savnet-inter-domain-problem-statement:

informative:
  I-D.draft-ietf-savnet-intra-domain-architecture:
  I-D.draft-ietf-savnet-inter-domain-architecture:

--- abstract

This document defines methodologies for benchmarking the performance of intra-domain and inter-domain source address validation (SAV) mechanisms. SAV mechanisms are utilized to generate SAV rules that prevent source address spoofing. The methodology treats a SAV device as a black box and is therefore agnostic to the specific SAV mechanism and implementation used by the device. This document defines test setups, performance indicators, and test cases for SAV accuracy, control-plane and data-plane performance, and resource utilization.

--- middle

# Introduction

Source address validation (SAV) is a fundamental mechanism for mitigating IP source address spoofing {{RFC2827}} {{RFC3704}} {{RFC8704}}. Operators may deploy SAV at different locations, including access networks, intra-domain interfaces, and inter-domain interfaces {{RFC5210}}. Existing intra-domain and inter-domain SAV mechanisms can suffer from improper blocks, improper permits, and operational overhead in several deployment scenarios {{I-D.ietf-savnet-intra-domain-problem-statement}} {{I-D.ietf-savnet-inter-domain-problem-statement}}.

The SAVNET Working Group has analyzed the problem space for both intra-domain and inter-domain SAV. For intra-domain SAV, the relevant deployment point is an external interface of an AS-facing entity that is not a neighboring AS, such as a single host, a set of hosts, or a customer network with no AS. For inter-domain SAV, the relevant deployment point is an external interface directly connected to a neighboring AS, regardless of whether the neighboring AS uses a public or private ASN. This document uses the same conceptual split when defining benchmarking test cases.

This document provides generic methodologies for benchmarking SAV mechanism performance. A SAV device may support one or more SAV mechanisms, and operators may enable different mechanisms depending on their network environments. This document treats the Device Under Test (DUT) as a black box and does not assume a particular implementation. The tests defined in this document can be used to benchmark SAV accuracy, protocol convergence performance, control-plane processing performance, data-plane SAV table refresh performance, data-plane forwarding performance, and resource utilization. These tests can be performed on a hardware router, software router, virtual machine (VM), or container instance that runs as a SAV device.

## Goal and Scope

The benchmarking methodology outlined in this document has two goals:

* Benchmark SAV mechanisms and implementations over a set of well-defined intra-domain and inter-domain scenarios.

* Measure the contribution of control-plane, data-plane, and resource-related sub-systems to the overall performance of a SAV device.

This document focuses on laboratory benchmarking of individual DUTs. It does not define a new SAV mechanism, protocol extension, or operational recommendation. The test cases are intended to evaluate whether a DUT can produce correct SAV behavior and maintain acceptable performance under classic scenarios.

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

This document uses the terminology in {{I-D.ietf-savnet-intra-domain-problem-statement}} and {{I-D.ietf-savnet-inter-domain-problem-statement}}. The following terms are used in this document.

SAV Device: A device that applies SAV to incoming packets. In this document, the SAV device is the DUT.

SAV Control Plane: The processes used to gather, communicate, compute, and update information used for SAV rule generation.

SAV Data Plane: The packet-processing component that validates each incoming packet against the applicable SAV rules and either permits or blocks the packet.

SAV Rule: A rule that indicates the validity of a specific source IP address or source IP prefix on a specific router interface. It is used by a router to make SAV decisions.

Improper Block: The validation result in which packets with legitimate source addresses are blocked improperly due to inaccurate SAV rules or an inaccurate SAV list. The terms "improper block" and "false positive" are used synonymously in this document.

Improper Permit: The validation result in which packets with spoofed source addresses are permitted improperly due to inaccurate SAV rules or an inaccurate SAV list. The terms "improper permit" and "false negative" are used synonymously in this document.

Intra-domain SAV: SAV performed by an AS to validate the source addresses of data traffic that the AS originates directly or indirectly. Intra-domain SAV is applied at external interfaces on routers facing entities that are not neighboring ASes, such as a single host, a set of hosts, or a customer network with no AS.

Inter-domain SAV: SAV performed by an AS to validate the source addresses of data traffic received from a neighboring AS, whether the traffic originated in the neighboring AS or is transited through it. Inter-domain SAV is applied to incoming traffic on external router interfaces directly connected to neighboring ASes.

Customer Network with No AS: A customer network that manages one or more IP prefixes but is not deployed as a neighboring AS of the SAV-performing AS.

Neighboring AS: An AS directly connected to the SAV-performing AS using eBGP. The relationship can be Customer-to-Provider (C2P), Provider-to-Customer (P2C), lateral peering (P2P), or Route Server (RS) to RS-client.

Customer Cone (CC): For a given AS, the set that includes the AS itself, its direct customer ASes, and all indirect customer ASes reachable recursively through provider-to-customer links.

Prefixes in the Customer Cone: IP prefixes permitted by their owners to be originated by, or used as source addresses for data traffic originated from, one or more ASes within the customer cone.

Limited Propagation of a Prefix (LPP): An inter-domain scenario in which a prefix is not propagated to all relevant ASes or interfaces due to mechanisms such as NO_EXPORT, NO_ADVERTISE, or selective export policies, while legitimate traffic using that prefix may still arrive at an interface where the prefix is not visible in BGP.

Hidden Prefix (HP): A scenario in which an entity legitimately originates traffic using source addresses that are not visible to the routing or forwarding information used by the SAV mechanism.

Direct Server Return (DSR): A traffic delivery model commonly used by CDNs that use anycast service addresses while delivering data from edge locations that do not announce those addresses. A request is received by an anycast server or location, but the response is sent directly by another server using the anycast service address as the source address. This can create a legitimate hidden-prefix scenario.

SAV-related information: Routing information (e.g., RIB and FIB) and objects published in the Resource Public Key Infrastructure (RPKI) that were originally proposed for non-SAV purposes but may also be used for SAV. The RPKI objects include existing RPKI object types (e.g., ROAs and ASPAs) as well as any new types that may be proposed.

SAV-specific information: Information dedicated to SAV, which may be defined and exchanged between ASes using potentially new inter-AS communication protocol or an extension of an existing protocol. The information may also take the form of new RPKI object type(s) or management information from operators.

# Test Methodology

## Test Setup

The test setup in general is compliant with {{RFC2544}}. The DUT is connected to a Tester and other network devices to construct the network topology introduced in {{testcase-sec}}. The Tester is a traffic generator that generates network traffic with specified source and destination addresses in order to emulate spoofed or legitimate traffic. The Tester may also emulate routing peers, hosts, customer networks with no AS, or neighboring ASes, depending on the test case.

~~~~~~~~~~
    +~~~~~~~~~~~~~~~~~~~~~~~~~~+
    | Test Network Environment |
    |     +--------------+     |
    |     |              |     |
+-->|     |      DUT     |     |---+
|   |     |              |     |   |
|   |     +--------------+     |   |
|   +~~~~~~~~~~~~~~~~~~~~~~~~~~+   |
|                                  |
|         +--------------+         |
+---------|    Tester    |<--------+
          +--------------+
~~~~~~~~~~
{: #testsetup title="Generic Test Setup."}

{{testsetup}} illustrates the generic test configuration. Within the test network environment, the DUT can be interconnected with other devices to create the specific intra-domain or inter-domain test scenarios described in {{testcase-sec}}. The Tester may connect directly to the DUT or indirectly through other emulated routers or ASes. The Tester generates both spoofed and legitimate traffic for SAV accuracy tests and may generate traffic at line rate for data-plane performance tests. The DUT is expected to provide logs, counters, telemetry, or other observable outputs sufficient to compute the performance indicators defined in this document.

## Network Topology and Device Configuration

The position of the DUT within the test topology has an impact on SAV performance. Therefore, each benchmark report must identify the DUT location and the interface on which SAV is evaluated.

For intra-domain SAV, the report must specify whether the DUT interface faces a single host, a set of hosts, or a customer network with no AS. 
For inter-domain SAV, the report must specify the business relationship between the SAV-performing AS and the neighboring AS on the tested interface. The relationship must be identified as customer, provider, lateral peer, RS, or RS-client when applicable.

The routing, policy, and SAV configurations used in the test must be documented. Examples include IGP configuration, BGP configuration, business relationships, NO_EXPORT or NO_ADVERTISE communities, route-policy configuration, and any SAV-specific configuration. If the DUT uses SAV-related information or SAV-specific information, the sources and update procedures of that information should be documented.

When evaluating data-plane forwarding performance, the traffic generated by the Tester must be characterized by traffic rate, packet size distribution, ratio of spoofed to legitimate traffic, source prefix distribution, destination prefix distribution, and the ingress interface on which the traffic is received.

# SAV Performance Indicators

This section lists key performance indicators (KPIs) for SAV benchmarking tests. All KPIs should be measured in the applicable benchmarking scenarios described in {{testcase-sec}}. The standard deviation of repeated test results should be reported for each fixed test setup. The data-plane SAV table refresh rate and data-plane forwarding rate should be measured using varying SAV table sizes to show the sensitivity of the DUT to the SAV table size.

## False Positive Rate

The proportion of legitimate packets incorrectly classified as spoofed and blocked by the DUT to the total number of legitimate packets sent to the DUT. This metric corresponds to the improper block rate. For the purpose of this document, this metric is computed based on packet counts on a per-packet basis.

Note that other computation methods, such as byte-count-based computation, may be used for supplementary analysis, but are outside the scope of the normative metric definition in this document.

## False Negative Rate

The proportion of spoofed packets incorrectly classified as legitimate and permitted by the DUT to the total number of spoofed packets sent to the DUT. This metric corresponds to the improper permit rate. For the purpose of this document, this metric is computed based on packet counts on a per-packet basis.

Note that other computation methods, such as byte-count-based computation, may be used for supplementary analysis, but are outside the scope of the normative metric definition in this document.

## Protocol Convergence Time

The protocol convergence time represents the elapsed time from a relevant change in SAV-related information or SAV-specific information to the completion of the corresponding SAV table update on the DUT. Relevant changes can include route announcement, route withdrawal, policy change, prefix authorization change, or SAV-specific information update.

## Protocol Message Processing Throughput

The protocol message processing throughput measures the rate at which the DUT processes control-plane messages used to communicate SAV-related or SAV-specific information. It can indicate the SAV control-plane processing performance of the DUT.

## Data Plane SAV Table Refreshing Rate

The data-plane SAV table refreshing rate is the rate at which the DUT updates the data-plane SAV table. It reflects the ability of the DUT to install, update, or remove SAV entries in the data plane.

## Data Plane Forwarding Rate

The data-plane forwarding rate measures the throughput for processing data-plane traffic while SAV is enabled. The same forwarding-rate test should also be performed with SAV disabled, so that the relative performance impact of SAV can be reported.

## Resource Utilization

Resource utilization refers to the CPU, memory, and other relevant resources consumed by SAV control-plane and data-plane processes on the DUT. CPU and memory utilization should be recorded continuously during each test and should be reported separately for control-plane and data-plane components when possible.

# Benchmarking Tests {#testcase-sec}

## Intra-domain SAV {#intra_domain_sav}

### False Positive and False Negative Rates

**Objective**: Evaluate the false positive rate and false negative rate of the DUT when performing intra-domain SAV on external interfaces facing a single host, a set of hosts, or a customer network with no AS.

The intra-domain test cases in this section evaluate the symmetric routing scenario, the asymmetric routing scenario, the hidden prefix scenario, and spoofed traffic that may expose overly permissive validation behavior. The DUT should be evaluated on the external interface where SAV is applied. The generated spoofed traffic should include different types of forged source addresses, such as source addresses not assigned to the connected entity, private-use or special-purpose addresses when applicable, internal-use-only prefixes of the AS, and external prefixes that are routable but not authorized for the tested ingress interface.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                   Test Network Environment                 |
|                         +~~~~~~~~~~+                       |
|                         | Router 1 |                       |
| FIB on DUT              +~~~~~~~~~~+                       |
| Dest           Next_hop   /\    |                          |
| 2001:db8::/55  Network 1   |    |                          |
|                            |    \/                         |
|                         +----------+                       |
|                         |   DUT    |                       |
|                         +----------+                       |
|                           /\    |                          |
|               Traffic with |    | Traffic with             |
|        source IP addresses |    | destination IP addresses |
|           of 2001:db8::/55 |    | of 2001:db8::/55         |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                             |    \/
                      +------------------------+
                      |Tester (Host or customer|
                      |   network with no AS)  |
                      |     (2001:db8::/55)    |
                      +------------------------+
~~~~~~~~~~
{: #intra-baseline title="Intra-domain SAV facing host or customer network with no AS under symmetric routing scenario."}

**Intra-domain Symmetric Routing Scenario**: {{intra-baseline}} shows an intra-domain symmetric routing scenario. The Tester emulates a host, a set of hosts, or a customer network with no AS connected to the DUT. The Tester is authorized to originate traffic using 2001:db8::/55. The DUT applies intra-domain SAV on the interface facing the Tester.

The **procedure** for this test is as follows:

1. Configure the DUT and other routers {{intra-baseline}} so that traffic from the Tester to destinations in the domain or outside the domain is forwarded through the DUT.

2. Configure or advertise the authorized source prefix 2001:db8::/55 according to the SAV mechanism under test.

3. Send legitimate traffic from the Tester using source addresses in 2001:db8::/55.

4. Send spoofed traffic from the Tester using source addresses not authorized for the Tester, for example 2001:db8:0:200::/55.

5. Vary the ratio of legitimate to spoofed traffic, for example from 1:9 to 9:1, and record the DUT counters or logs.

6. Measure the false positive rate and false negative rate.

The **expected result** is that the DUT properly permits legitimate traffic and properly blocks spoofed traffic.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                         Test Network Environment                       |
|                             +~~~~~~~~~~+                               |
|                             | Router 2 |                               |
| FIB on DUT                  +~~~~~~~~~~+   FIB on Router 1             |
| Dest                Next_hop  /\      \    Dest               Next_hop |
| 2001:db8::/56       Network 1 /        \ 2001:db8:0:100::/56  Network 1|
| 2001:db8:0:100::/56 Router 2 /         \/ 2001:db8::/56       Router 2 |
|                    +----------+     +~~~~~~~~~~+                       |
|                    |   DUT    |     | Router 1 |                       |
|                    +----------+     +~~~~~~~~~~+                       |
|                       /\               /                               |
|           Traffic with \              / Traffic with                   |
|     source IP addresses \            / destination IP addresses        |
|   of 2001:db8:0:100::/56 \          / of 2001:db8:0:100::/56           |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                             \      \/
                     +------------------------+
                     |Tester (Host or customer|
                     |   network with no AS)  |
                     |     (2001:db8::/55)    |
                     +------------------------+
~~~~~~~~~~
{: #intra-asymmetric title="Intra-domain SAV facing host or customer network with no AS under asymmetric routing scenario."}

**Intra-domain Asymmetric Routing Scenario**: {{intra-asymmetric}} shows an intra-domain asymmetric routing scenario. The host or customer network with no AS owns 2001:db8::/55 and is connected to both the DUT and Router 2. Inbound traffic for 2001:db8:0::/56 uses the DUT, while inbound traffic for 2001:db8:0:100::/56 uses Router 2. The customer network may nevertheless send outbound traffic with source addresses in 2001:db8:0:100::/56 through the DUT. This creates a legitimate asymmetric path.

The **procedure** for this test is as follows:

1. Configure the topology shown in {{intra-asymmetric}}. The Tester emulates the host or the customer network with no AS and connects to both the DUT and Router 2.

2. Configure the routing system so that the DUT's route to 2001:db8:0:100::/56 points away from the Tester, while the Tester can send traffic with source addresses in 2001:db8:0:100::/56 to the DUT.

3. Send legitimate traffic from the Tester to the DUT using source addresses in 2001:db8:0:100::/56.

4. Send spoofed traffic from the Tester to the DUT using source addresses not authorized for the Tester, for example 2001:db8:0:200::/55.

5. Vary the ratio of legitimate to spoofed traffic, for example from 1:9 to 9:1, and record the DUT counters or logs.

6. Measure the false positive rate and false negative rate.

The **expected result** is that the DUT properly permits legitimate traffic that follows an asymmetric path and properly blocks spoofed traffic.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                   Test Network Environment                 |
|                         +~~~~~~~~~~+                       |
|                         | Router 1 |                       |
| FIB on DUT              +~~~~~~~~~~+                       |
| Dest           Next_hop   /\    |                          |
| 2001:db8::/55  Network 1   |    |                          |
|                            |    \/                         |
|                         +----------+                       |
|                         |   DUT    |                       |
|                         +----------+                       |
|                           /\    |                          |
|               Traffic with |    | Traffic with             |
|        source IP addresses |    | destination IP addresses |
|           of 2001:db8::/55 |    | of 2001:db8::/55         |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                             |    \/
                      +------------------------+
                      |Tester (Host or customer|
                      |   network with no AS)  |
                      +------------------------+
Visible/assigned prefix:  2001:db8::/56
Hidden source prefix:     2001:db8:0:100::/56
~~~~~~~~~~
{: #intra-hidden-prefix title="Intra-domain SAV under a hidden prefix scenario."}

**Intra-domain Hidden Prefix Scenario**: {{intra-hidden-prefix}} shows an intra-domain hidden prefix scenario. The Tester emulates a host or customer network with no AS that legitimately originates traffic from a source prefix not visible to the routing or forwarding information used by the SAV mechanism. Examples include DSR deployments or other cases where the authorized source prefix is not propagated within the operator's intra-domain routing system.

The **procedure** for this test is as follows:

1. Configure the Tester as a host, a set of hosts, or a customer network with no AS connected to the DUT.

2. Configure the test so that 2001:db8::/56 is visible to the DUT as an assigned or routed prefix, while 2001:db8:0:100::/56 is a legitimate source prefix for the Tester but is not visible in the routing or forwarding information normally used by the DUT.

3. Send legitimate traffic from the Tester using source addresses in 2001:db8:0:100::/56.

4. Send spoofed traffic from the Tester using source addresses not authorized for the Tester.

5. Measure the false positive rate and false negative rate.

The **expected result** is that a SAV mechanism capable of handling hidden prefixes properly permits legitimate traffic from the hidden prefix and properly blocks spoofed traffic. If the DUT does not support the information needed to authorize the hidden prefix, the report should record the resulting improper block behavior.

### Control Plane Performance {#intra-control-plane-sec}

**Objective**: Measure the control plane performance of the DUT, including both protocol convergence performance and protocol message processing performance in response to route changes caused by network failures or operator configurations. Protocol convergence performance is quantified by the convergence time, defined as the duration from the onset of a routing change until the completion of the corresponding SAV rule update. Protocol message processing performance is measured by the processing throughput, represented by the total size of protocol messages processed per second.

Note that the tests for control plane performance of the DUT which performs intra-domain SAV are OPTIONAL. Only DUT which implements the SAV mechanism using an explicit control-plane communication protocol, such as SAV-specific information communication mechanism proposed in {{I-D.draft-ietf-savnet-intra-domain-architecture}} should be tested on its control plane performance.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~+      +-------------+          +-----------+
| Emulated Topology |------|   Tester    |<-------->|    DUT    |
+~~~~~~~~~~~~~~~~~~~+      +-------------+          +-----------+
~~~~~~~~~~
{: #intra-convg-perf title="Test setup for protocol convergence performance measurement."}

**Protocol Convergence Performance**: {{intra-convg-perf}} illustrates the test setup for measuring protocol convergence performance. The convergence process of the DUT, during which SAV rules are updated, is triggered by route changes resulting from network failures or operator configurations. In {{intra-convg-perf}}, the Tester is directly connected to the DUT and simulates these route changes by adding or withdrawing prefixes to initiate the DUT's convergence procedure.

The **procedure** for testing protocol convergence performance is as follows:

1. To measure the protocol convergence time of the DUT, set up the test environment as depicted in {{intra-convg-perf}}, with the Tester directly connected to the DUT.

2. The Tester withdraws a specified percentage of the total prefixes supported by the DUT, for example, 10%, 20%, up to 100%.

3. The protocol convergence time is calculated based on DUT logs that record the start and completion times of the convergence process.

Please note that for IGP, proportional prefix withdrawal can be achieved by selectively shutting down interfaces. For instance, if the Tester is connected to ten emulated devices through ten interfaces, each advertising a prefix, withdrawing 10% of prefixes can be accomplished by randomly disabling one interface. Similarly, 20% withdrawal corresponds to shutting down two interfaces, and so forth. This is one suggested method, and other approaches that achieve the same effect should be also acceptable.

The protocol convergence time, defined as the duration required for the DUT to complete the convergence process, should be measured from the moment the last &ldquo;hello&rdquo; message is received from the emulated device on the disabled interface until SAV rule generation is finalized. To ensure accuracy, the DUT should log the timestamp of the last hello message received and the timestamp when SAV rule updates are complete. The convergence time is the difference between these two timestamps.

It is recommended that if the emulated device sends a &ldquo;goodbye hello&rdquo; message during interface shutdown, using the receipt time of this message, rather than the last standard hello, as the starting point will provide a more precise measurement, as advised in {{RFC4061}}.

**Protocol Message Processing Performance**: The test for protocol message processing performance uses the same setup illustrated in {{intra-convg-perf}}. This performance metric evaluates the protocol message processing throughput, the rate at which the DUT processes protocol messages. The Tester varies the sending rate of protocol messages, ranging from 10% to 100% of the total link capacity between the Tester and the DUT. The DUT records both the total size of processed protocol messages and the corresponding processing time.

The **procedure** for testing protocol message processing performance is as follows:

1. To measure the protocol message processing throughput of the DUT, set up the test environment as shown in {{intra-convg-perf}}, with the Tester directly connected to the DUT.

2. The Tester sends protocol messages at varying rates, such as 10%, 20%, up to 100%, of the total link capacity between the Tester and the DUT.

3. The protocol message processing throughput is calculated based on DUT logs that record the total size of processed protocol messages and the total processing time.

To compute the protocol message processing throughput, the DUT logs MUST include the total size of the protocol messages processed and the total time taken for processing. The throughput is then derived by dividing the total message size by the total processing time.

### Data Plane Performance {#intra-data-plane-sec}

**Objective**: Evaluate the data plane performance of the DUT, including both data plane SAV table refresh performance and data plane forwarding performance. Data plane SAV table refresh performance is quantified by the refresh rate, which indicates how quickly the DUT updates its SAV table with new SAV rules. Data plane forwarding performance is measured by the forwarding rate, defined as the total size of packets forwarded by the DUT per second.

**Data Plane SAV Table Refreshing Performance**: The evaluation of data plane SAV table refresh performance uses the same test setup shown in {{intra-convg-perf}}. This metric measures the rate at which the DUT refreshes its SAV table with new SAV rules. The Tester varies the transmission rate of protocol messages, from 10% to 100% of the total link capacity between the Tester and the DUT, to influence the proportion of updated SAV rules and corresponding SAV table entries. The DUT records the total number of updated SAV table entries and the time taken to complete the refresh process.

The **procedure** for testing data plane SAV table refresh performance is as follows:

1. To measure the data plane SAV table refreshing rate of the DUT, set up the test environment as depicted in {{intra-convg-perf}}, with the Tester directly connected to the DUT.

2. The Tester sends protocol messages at varying percentages of the total link capacity, for example, 10%, 20%, up to 100%.

3. The data plane SAV table refreshing rate is calculated based on DUT logs that record the total number of updated SAV table entries and the total refresh time.

To compute the refresh rate, the DUT logs MUST capture the total number of updated SAV table entries and the total time required for refreshing. The refresh rate is then derived by dividing the total number of updated entries by the total refresh time.

**Data Plane Forwarding Performance**: The evaluation of data plane forwarding performance uses the same test setup shown in {{intra-convg-perf}}. The Tester transmits a mixture of spoofed and legitimate traffic at a rate matching the total link capacity between the Tester and the DUT, while the DUT maintains a fully populated SAV table. The ratio of spoofed to legitimate traffic can be varied within a range, for example, from 1:9 to 9:1. The DUT records the total size of forwarded packets and the total duration of the forwarding process.

The procedure for testing data plane forwarding performance is as follows:

1. To measure the data plane forwarding rate of the DUT, set up the test environment as depicted in {{intra-convg-perf}}, with the Tester directly connected to the DUT.

2. The Tester sends a mix of spoofed and legitimate traffic to the DUT at the full link capacity between the Tester and the DUT. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

3. The data plane forwarding rate is calculated based on DUT logs that record the total size of forwarded traffic and the total forwarding time.

To compute the forwarding rate, the DUT logs must include the total size of forwarded traffic and the total time taken for forwarding. The forwarding rate is then derived by dividing the total traffic size by the total forwarding time. 

## Inter-domain SAV {#inter_domain_sav}

### False Positive and False Negative Rates

**Objective**: Evaluate the false positive rate and false negative rate of the DUT when performing inter-domain SAV on an external interface connected to a neighboring AS.

The inter-domain test cases in this section cover customer interfaces, provider interfaces, lateral peer interfaces, and RS/RS-client-related interfaces when applicable. The generated spoofed traffic should include source addresses belonging to prefixes outside the legitimate set for the tested ingress interface, prefixes originated elsewhere in the customer cone, prefixes originated by the SAV-performing AS, special-purpose or unallocated prefixes when applicable, and prefixes associated with other ASes.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                  Test Network Environment                |
|                        +~~~~~~~~~~~~~~~~+                |
|                        |    AS 3(P3)    |                |
|                        +~+/\~~~~~~+/\+~~+                |
|                           /         \                    |
|                          /           \                   |
|                         /             \                  |
|                        / (C2P)         \                 |
|              +------------------+       \                |
|              |      DUT(P4)     |        \               |
|              +-+/\+-+/\+----+/\++         \              |
|                 /     |       \            \             |
|      P2[AS 2]  /      |        \            \            |
|P6[AS 2, AS 1] /       |         \            \           |
|P1[AS 2, AS 1]/ (C2P)  |          \ P5[AS 5]   \ P5[AS 5] |
|+~~~~~~~~~~~~~~~~+     |           \            \         |
||    AS 2(P2)    |     | P1[AS 1]   \            \        |
|+~~~~~~~~~~+/\+~~+     | P6[AS 1]    \            \       |
|             \         |              \            \      |
|     P6[AS 1] \        |               \            \     |
|      P1[AS 1] \       |                \            \    |
|          (C2P) \      | (C2P)     (C2P) \      (C2P) \   |
|             +~~~~~~~~~~~~~~~~+        +~~~~~~~~~~~~~~~~+ |
|             |  AS 1(P1, P6)  |        |    AS 5(P5)    | |
|             +~~~~~~~~~~~~~~~~+        +~~~~~~~~~~~~~~~~+ |
|                  /\     |                                |
|                  |      |                                |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                   |     \/
              +----------------+
              |     Tester     |
              +----------------+
~~~~~~~~~~
{: #inter-customer-syn title="SAV for customer-facing ASes in inter-domain symmetric routing scenario."}

**SAV for Customer-facing ASes under an Inter-domain Symmetric Routing Scenario**: {{inter-customer-syn}} presents a test case for SAV in customer-facing ASes under an inter-domain symmetric routing scenario. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network environment, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which in turn is a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. AS 1 advertises prefixes P1 and P6 to AS 2 and the DUT, respectively. AS 2 then propagates routes for P1 and P6 to the DUT, enabling the DUT to learn these prefixes from both AS 1 and AS 2. In this test, the legitimate path for traffic with source addresses in P1 and destination addresses in P4 is AS 1->AS 2->DUT. The Tester is connected to AS 1 to evaluate the DUT's SAV performance for customer-facing ASes.

The **procedure** for testing SAV in this scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for customer-facing ASes under symmetric inter-domain routing scenario, construct the test environment as shown in {{inter-customer-syn}}. The Tester is connected to AS 1 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to establish symmetric routing environment.

3. The Tester sends both legitimate traffic (with source addresses in P1 and destination addresses in P4) and spoofed traffic (with source addresses in P5 and destination addresses in P4) to the DUT via AS 2. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic received from the direction of AS 2.

Note that the DUT may also be placed at AS 1 or AS 2 in {{inter-customer-syn}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                  Test Network Environment                |
|                        +~~~~~~~~~~~~~~~~+                |
|                        |    AS 3(P3)    |                |
|                        +~+/\~~~~~~+/\+~~+                |
|                           /         \                    |
|                          /           \                   |
|                         /             \                  |
|                        / (C2P)         \                 |
|              +------------------+       \                |
|              |      DUT(P4)     |        \               |
|              ++/\+--+/\+----+/\++         \              |
|                /      |       \            \             |
|      P2[AS 2] /       |        \            \            |
|P6[AS 2, AS 1]/        |         \            \           |
|             / (C2P)   |          \ P5[AS 5]   \ P5[AS 5] |
|+~~~~~~~~~~~~~~~~+     |           \            \         |
||    AS 2(P2)    |     | P1[AS 1]   \            \        |
|+~~~~~~~~~~+/\+~~+     | P6[AS 1]    \            \       |
|    P6[AS 1] \         | NO_EXPORT    \            \      |
|     P1[AS 1] \        |               \            \     |
|     NO_EXPORT \       |                \            \    |
|          (C2P) \      | (C2P)     (C2P) \      (C2P) \   |
|             +~~~~~~~~~~~~~~~~+        +~~~~~~~~~~~~~~~~+ |
|             |  AS 1(P1, P6)  |        |    AS 5(P5)    | |
|             +~~~~~~~~~~~~~~~~+        +~~~~~~~~~~~~~~~~+ |
|                  /\     |                                |
|                  |      |                                |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                   |     \/
              +----------------+
              |     Tester     |
              +----------------+
~~~~~~~~~~
{: #inter-customer-lpp title="SAV for customer-facing ASes in inter-domain asymmetric routing scenario caused by NO_EXPORT."}

SAV for Customer-facing ASes under an Inter-domain Asymmetric Routing Scenario: {{inter-customer-lpp}} presents a test case for SAV in customer-facing ASes under an inter-domain asymmetric routing scenario induced by NO_EXPORT community configuration. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which is itself a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. AS 1 advertises prefix P1 to AS 2 with the NO_EXPORT community attribute, preventing AS 2 from propagating the route for P1 to the DUT. Similarly, AS 1 advertises prefix P6 to the DUT with the NO_EXPORT attribute, preventing the DUT from propagating this route to AS 3. As a result, the DUT learns the route for prefix P1 only from AS 1. The legitimate path for traffic with source addresses in P1 and destination addresses in P4 is AS 1->AS 2->DUT. The Tester is connected to AS 1 to evaluate the DUT's SAV performance for customer-facing ASes.

The **procedure** for testing SAV in this asymmetric routing scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules under NO_EXPORT-induced asymmetric routing, construct the test environment as shown in {{inter-customer-lpp}}. The Tester is connected to AS 1 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to establish the asymmetric routing scenario.

3. The Tester sends both legitimate traffic (with source addresses in P1 and destination addresses in P4) and spoofed traffic (with source addresses in P5 and destination addresses in P4) to the DUT via AS 2. The ratio of spoofed to legitimate traffic may vary—for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic received from the direction of AS 2.

Note that the DUT may also be placed at AS 1 or AS 2 in {{inter-customer-lpp}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                  Test Network Environment                       |
|                                +----------------+               |
|                Anycast Server+-+  AS 3(P3, P7)  |               |
|                                +-+/\----+/\+----+               |
|                                   /       \                     |
|                       / P3[AS 3] /         \ P3[AS 3] \         |
|                      / P7[AS 3] /           \ P7[AS 3] \        |
|                     \/         / (C2P)       \         \/       |
|                       +----------------+      \                 |
|                       |    AS 4(P4)    |       \                |
|                       ++/\+--+/\+--+/\++        \               |
|                         /     |      \           \              |
|       / P3[AS 4, AS 3] /      |       \           \             |
|      / P7[AS 4, AS 3] /       |        \           \            |
|    \/                / (C2P)  |         \ P5[AS 5]  \ P5[AS 5]  |
|      +----------------+       |          \           \          |
|User+-+    AS 2(P2)    |       | P1[AS 1]  \           \         |
|      +----------+/\+--+       | P6[AS 1]   \           \        |
|                   \           |             \           \       |
|           P6[AS 1] \          |              \           \      |
|            P1[AS 1] \         |               \           \     |
|                      \(C2P)   |(C2P)      (C2P)\      (C2P)\    |
|                    +---------------+         +----------------+ |
|       Edge Server+-+  AS 1(P1, P6)  |        |    AS 5(P5)    | |
|                    +----------------+        +----------------+ |
|                         /\     |                                |
|                          |     |                                |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                           |    \/
                     +----------------+
                     |     Tester     |
                     | (Edge Server)  |
                     +----------------+
P7 is the anycast prefix and is originated only by AS 3 via BGP.
Note that the prefix route propagations relevant to the DSR
scenario are depicted; not all prefix propagations are depicted.
~~~~~~~~~~
{: #inter-customer-dsr title="SAV for customer-facing ASes in the scenario of hidden prefix caused by direct server return (DSR)."}

**SAV for Customer-facing ASes under the Scenario of Hidden Prefix**: {{inter-customer-dsr}} presents a test case for SAV in customer-facing ASes under a Direct Server Return (DSR) scenario. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which is itself a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. When users in AS 2 send requests to an anycast destination IP in P7, the forwarding path is AS 2->DUT->AS 3. Anycast servers in AS 3 receive the requests and tunnel them to edge servers in AS 1. The edge servers then return content to the users with source addresses in prefix P7. If the reverse forwarding path is AS 1->DUT->AS 2, the Tester sends traffic with source addresses in P7 and destination addresses in P2 along the path AS 1->DUT->AS 2. Alternatively, if the reverse forwarding path is AS 1->AS 2, the Tester sends traffic with source addresses in P7 and destination addresses in P2 along the path AS 1->AS 2. In this case, AS 2 may serve as the DUT.

The **procedure** for testing SAV in this DSR scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules under DSR conditions, construct the test environment as shown in {{inter-customer-dsr}}. The Tester is connected to AS 1 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to establish the DSR scenario.

3. The Tester sends legitimate traffic (with source addresses in P7 and destination addresses in P2) to AS 2 via the DUT.

The **expected results** for this test case are that the DUT permits legitimate traffic with source addresses in P7 received from the direction of AS 1.

Note that the DUT may also be placed at AS 1 or AS 2 in {{inter-customer-dsr}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
             +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
             |                   Test Network Environment                 |
             |                          +----------------+                |
             |                          |    AS 3(P3)    |                |
             |                          +--+/\+--+/\+----+                |
             |                              /      \                      |
             |                             /        \                     |
             |                            /          \                    |
             |                           / (C2P)      \                   |
             |                  +----------------+     \                  |
             |                  |     DUT(P4)    |      \                 |
             |                  ++/\+--+/\+--+/\++       \                |
             |     P6[AS 1, AS 2] /     |      \          \               |
             |          P2[AS 2] /      |       \          \              |
             |                  /       |        \          \             |
             |                 / (C2P)  |         \ P5[AS 5] \ P5[AS 5]   |
+----------+ |  +----------------+      |          \          \           |
|  Tester  |-|->|                |      |           \          \          |
|(Attacker)| |  |    AS 2(P2)    |      |            \          \         |
|  (P1')   |<|--|                |      | P1[AS 1]    \          \        |
+----------+ |  +---------+/\+---+      | P6[AS 1]     \          \       |
             |     P6[AS 1] \           | NO_EXPORT     \          \      |
             |      P1[AS 1] \          |                \          \     |
             |      NO_EXPORT \         |                 \          \    |
             |                 \ (C2P)  | (C2P)      (C2P) \    (C2P) \   |
             |             +----------------+          +----------------+ |
             |     Victim+-+  AS 1(P1, P6)  |  Server+-+    AS 5(P5)    | |
             |             +----------------+          +----------------+ |
             +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
P1' is the spoofed source prefix P1 by the attacker which is inside of 
AS 2 or connected to AS 2 through other ASes.
~~~~~~~~~~
{: #inter-customer-reflect title="SAV for customer-facing ASes in the scenario of reflection attacks."}

**SAV for Customer-facing ASes under the Reflection Attack Scenario**: {{inter-customer-reflect}} illustrates a test case for SAV in customer-facing ASes under a reflection attack scenario. In this scenario, a reflection attack using source address spoofing occurs within the DUT's customer cone. The attacker spoofs the victim's IP address (P1) and sends requests to server IP addresses (P5) that are configured to respond to such requests. The Tester emulates the attacker by performing source address spoofing. The arrows in {{inter-customer-reflect}} indicate the business relationships between ASes: AS 3 serves as the provider for both the DUT and AS 5, while the DUT acts as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

The **procedure** for testing SAV under reflection attack conditions is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules in a reflection attack scenario, construct the test environment as shown in {{inter-customer-reflect}}. The Tester is connected to AS 2 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to simulate the reflection attack scenario.

3. The Tester sends spoofed traffic (with source addresses in P1 and destination addresses in P5) toward AS 5 via the DUT.

The **expected results** for this test case are that the DUT blocks spoofed traffic with source addresses in P1 received from the direction of AS 2.

Note that the DUT may also be placed at AS 1 or AS 2 in {{inter-customer-reflect}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
             +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
             |                   Test Network Environment                 |
             |                          +----------------+                |
             |                          |    AS 3(P3)    |                |
             |                          +--+/\+--+/\+----+                |
             |                              /      \                      |
             |                             /        \                     |
             |                            /          \                    |
             |                           / (C2P)      \                   |
             |                  +----------------+     \                  |
             |                  |     DUT(P4)    |      \                 |
             |                  ++/\+--+/\+--+/\++       \                |
             |     P6[AS 1, AS 2] /     |      \          \               |
             |          P2[AS 2] /      |       \          \              |
             |                  /       |        \          \             |
             |                 / (C2P)  |         \ P5[AS 5] \ P5[AS 5]   |
+----------+ |  +----------------+      |          \          \           |
|  Tester  |-|->|                |      |           \          \          |
|(Attacker)| |  |    AS 2(P2)    |      |            \          \         |
|  (P5')   |<|--|                |      | P1[AS 1]    \          \        |
+----------+ |  +---------+/\+---+      | P6[AS 1]     \          \       |
             |     P6[AS 1] \           | NO_EXPORT     \          \      |
             |      P1[AS 1] \          |                \          \     |
             |      NO_EXPORT \         |                 \          \    |
             |                 \ (C2P)  | (C2P)      (C2P) \    (C2P) \   |
             |             +----------------+          +----------------+ |
             |     Victim+-+  AS 1(P1, P6)  |          |    AS 5(P5)    | |
             |             +----------------+          +----------------+ |
             +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
P5' is the spoofed source prefix P5 by the attacker which is inside of 
AS 2 or connected to AS 2 through other ASes.
~~~~~~~~~~
{: #inter-customer-direct title="SAV for customer-facing ASes in the scenario of direct attacks."}

**SAV for Customer-facing ASes under the Direct Attack Scenario**: {{inter-customer-direct}} presents a test case for SAV in customer-facing ASes under a direct attack scenario. In this scenario, a direct attack using source address spoofing occurs within the DUT's customer cone. The attacker spoofs a source address (P5) and directly targets the victim's IP address (P1), aiming to overwhelm its network resources. The Tester emulates the attacker by performing source address spoofing. The arrows in {{inter-customer-direct}} indicate the business relationships between ASes: AS 3 serves as the provider for both the DUT and AS 5, while the DUT acts as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

The **procedure** for testing SAV under direct attack conditions is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules in a direct attack scenario, construct the test environment as shown in {{inter-customer-direct}}. The Tester is connected to AS 2 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to simulate the direct attack scenario.

3. The Tester sends spoofed traffic (with source addresses in P5 and destination addresses in P1) toward AS 1 via the DUT.

The **expected results** for this test case are that the DUT blocks spoofed traffic with source addresses in P5 received from the direction of AS 2.

Note that DUT may also be placed at AS 1 or AS 2 in {{inter-customer-direct}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
                                   +----------------+
                                   |     Tester     |
                                   |   (Attacker)   |
                                   |      (P1')     |
                                   +----------------+
                                        |     /\
                                        |      |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
| Test Network Environment              \/     |                    |
|                                  +----------------+               |
|                                  |                |               |
|                                  |    AS 3(P3)    |               |
|                                  |                |               |
|                                  +-+/\----+/\+----+               |
|                                     /       \                     |
|                                    /         \                    |
|                                   /           \                   |
|                                  / (C2P/P2P)   \                  |
|                         +----------------+      \                 |
|                         |     DUT(P4)    |       \                |
|                         ++/\+--+/\+--+/\++        \               |
|            P6[AS 1, AS 2] /     |      \           \              |
|                 P2[AS 2] /      |       \           \             |
|                         /       |        \           \            |
|                        / (C2P)  |         \ P5[AS 5]  \ P5[AS 5]  |
|        +----------------+       |          \           \          |
|Server+-+    AS 2(P2)    |       | P1[AS 1]  \           \         |
|        +----------+/\+--+       | P6[AS 1]   \           \        |
|            P6[AS 1] \           | NO_EXPORT   \           \       |
|             P1[AS 1] \          |              \           \      |
|             NO_EXPORT \         |               \           \     |
|                        \ (C2P)  | (C2P)    (C2P) \     (C2P) \    |
|                      +----------------+        +----------------+ |
|              Victim+-+  AS 1(P1, P6)  |        |    AS 5(P5)    | |
|                      +----------------+        +----------------+ |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
P1' is the spoofed source prefix P1 by the attacker which is inside of 
AS 3 or connected to AS 3 through other ASes.
~~~~~~~~~~
{: #reflection-attack-p title="SAV for provider-facing ASes in the scenario of reflection attacks."}

**SAV for Provider/Peer-facing ASes under the Reflection Attack Scenario**: {{reflection-attack-p}} illustrates a test case for SAV in provider/peer-facing ASes under a reflection attack scenario. In this scenario, the attacker spoofs the victim's IP address (P1) and sends requests to server IP addresses (P2) that are configured to respond. The Tester emulates the attacker by performing source address spoofing. The servers then send overwhelming responses to the victim, exhausting its network resources. The arrows in {{reflection-attack-p}} represent the business relationships between ASes: AS 3 acts as either a provider or a lateral peer of the DUT and is the provider for AS 5, while the DUT serves as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

The **procedure** for testing SAV under reflection attack conditions is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for provider/peer-facing ASes in a reflection attack scenario, construct the test environment as shown in {{reflection-attack-p}}. The Tester is connected to AS 3 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to simulate the reflection attack scenario.

3. The Tester sends spoofed traffic (with source addresses in P1 and destination addresses in P2) toward AS 2 via AS 3 and the DUT.

The **expected results** for this test case are that the DUT blocks spoofed traffic with source addresses in P1 received from the direction of AS 3.

Note that the DUT may also be placed at AS 1 or AS 2 in {{reflection-attack-p}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
                           +----------------+
                           |     Tester     |
                           |   (Attacker)   |
                           |      (P2')     |
                           +----------------+
                                |     /\
                                |      |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
| Test Network Environment      \/     |                    |
|                          +----------------+               |
|                          |    AS 3(P3)    |               |
|                          +-+/\----+/\+----+               |
|                             /       \                     |
|                            /         \                    |
|                           /           \                   |
|                          / (C2P/P2P)   \                  |
|                 +----------------+      \                 |
|                 |     DUT(P4)    |       \                |
|                 ++/\+--+/\+--+/\++        \               |
|    P6[AS 1, AS 2] /     |      \           \              |
|         P2[AS 2] /      |       \           \             |
|                 /       |        \           \            |
|                / (C2P)  |         \ P5[AS 5]  \ P5[AS 5]  |
|+----------------+       |          \           \          |
||    AS 2(P2)    |       | P1[AS 1]  \           \         |
|+----------+/\+--+       | P6[AS 1]   \           \        |
|    P6[AS 1] \           | NO_EXPORT   \           \       |
|     P1[AS 1] \          |              \           \      |
|     NO_EXPORT \         |               \           \     |
|                \ (C2P)  | (C2P)    (C2P) \     (C2P) \    |
|              +----------------+        +----------------+ |
|      Victim+-+  AS 1(P1, P6)  |        |    AS 5(P5)    | |
|              +----------------+        +----------------+ |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
P2' is the spoofed source prefix P2 by the attacker which is inside of 
AS 3 or connected to AS 3 through other ASes.
~~~~~~~~~~
{: #direct-attack-p title="SAV for provider-facing ASes in the scenario of direct attacks."}

**SAV for Provider/Peer-facing ASes under the Direct Attack Scenario**: {{direct-attack-p}} presents a test case for SAV in provider-facing ASes under a direct attack scenario. In this scenario, the attacker spoofs a source address (P2) and directly targets the victim's IP address (P1), overwhelming its network resources. The arrows in {{direct-attack-p}} represent the business relationships between ASes: AS 3 acts as either a provider or a lateral peer of the DUT and is the provider for AS 5, while the DUT serves as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

The procedure for testing SAV under direct attack conditions is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for provider-facing ASes in a direct attack scenario, construct the test environment as shown in {{direct-attack-p}}. The Tester is connected to AS 3 and generates test traffic toward the DUT.

2. Configure AS 1, AS 2, AS 3, the DUT, and AS 5 to simulate the direct attack scenario.

3. The Tester sends spoofed traffic (with source addresses in P2 and destination addresses in P1) toward AS 1 via AS 3 and the DUT.

The **expected results** for this test case are that the DUT blocks spoofed traffic with source addresses in P2 received from the direction of AS 3.

Note that the DUT may also be placed at AS 1 or AS 2 in {{direct-attack-p}} to evaluate its false positive and false negative rates using the same procedure. In these configurations, the DUT is expected to effectively block spoofed traffic.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                   Test Network Environment           |
|          +-----------+            +-----------+      |
|          |   AS3     |------------|   AS2     |      |
|          +-----------+            +-----------+      |
|               /\                       /\            |
|               |                        |             |
| primary link  |            backup link |             |
|               | (C2P)                  | (C2P)       |
|        +-----------------------------------------+   |
|        |                   DUT                   |   |
|        +-----------------------------------------+   |
|                           /\                         |
|                           |                          |
|                           | Legitimate and           |
|                           | Spoofed Traffic          |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                            | (C2P)
                     +-------------+
                     |    Tester   |
                     +-------------+
~~~~~~~~~~
{: #inter-domain-frr-topo title="Inter-domain SAV under FRR scenario."}

**SAV for Customer-facing ASes under FRR Scenario**: Inter-domain Fast Reroute (FRR) mechanisms, such as BGP Prefix Independent Convergence (PIC) or MPLS-based FRR, allow rapid failover between ASes after a link or node failure. These events may temporarily desynchronize routing information and SAV rules.

The **procedure** for testing SAV under FRR scenario is as follows:

1. Configure FRR or BGP PIC on the DUT for inter-AS links to AS3 (primary) and AS2 (backup). 
    
2. Continuously send legitimate and spoofed traffic from AS1 toward DUT.  
   
3. Trigger a failure on the AS3–DUT link to activate the FRR path via AS2.  
   
4. Measure false positive and false negative rates during and after switchover. 
    
5. Restore the AS3 link and verify SAV table consistency.

The **expected results** for this test case are that the DUT must maintain consistent SAV filtering during FRR events. Transient topology changes should not lead to acceptance of spoofed traffic or unnecessary blocking of legitimate packets.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|               Test Network Environment           |
|     +-----------+            +-----------+       |
|     |   AS3     |------------|   AS2     |       |
|     +-----------+            +-----------+       |
|          /\                       /\             |
|           |                        |             |
|           | preferred path         | default path|
|           | (C2P)                  | (C2P)       |
|    +-----------------------------------------+   |
|    |                  DUT                    |   |
|    +-----------------------------------------+   |
|                        /\                        |
|                         | Legitimate and         |
|                         | Spoofed Traffic        |
|                         |                        |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                          | (C2P) 
                   +-------------+
                   |    Tester   |
                   +-------------+
~~~~~~~~~~
{: #inter-domain-pbr-topo title="Inter-domain SAV under PBR scenario."}

**SAV for Customer-facing ASes under PBR Scenario**: In inter-domain environments, routing policies such as local preference, route maps, or communities may alter path selection independently of shortest-path routing. Such policy-driven forwarding can affect how the SAV rules are derived and applied.

The **procedure** for testing SAV under PBR scenario is as follows:

1. Configure a routing policy on the DUT (e.g., set local preference) to prefer AS3 for specific prefixes while maintaining AS2 as an alternative path.
   
2. Generate legitimate and spoofed traffic from AS1 matching both policy-affected and unaffected prefixes.

3. Observe SAV filtering behavior before and after policy changes. 
    
4. Modify the routing policy dynamically and measure false positive and false negative rates.

The **expected results** for this test case are that the DUT should maintain correct SAV filtering regardless of routing policy changes. Legitimate traffic rerouted by policy must not be dropped, and spoofed traffic must not be forwarded during or after policy updates.

### Control Plane Performance

The test setup, procedure, and metrics for evaluating protocol convergence performance and protocol message processing performance can refer to {{intra-control-plane-sec}}. Note that the tests for control plane performance of the DUT which performs inter-domain SAV are OPTIONAL. Only DUT which implements the SAV mechanism using an explicit control-plane communication protocol, such as SAV-specific information communication mechanism proposed in {{I-D.draft-ietf-savnet-inter-domain-architecture}} should be tested on its control plane performance.

### Data Plane Performance

The test setup, procedure, and metrics for evaluating data plane SAV table refresh performance and data plane forwarding performance can refer to {{intra-data-plane-sec}}.

## Resource Utilization

When evaluating the DUT for both intra-domain ({{intra_domain_sav}}) and inter-domain SAV ({{inter_domain_sav}}) functionality, CPU utilization (for both control and data planes) and memory utilization (for both control and data planes) are suggested to record. These metrics should be recorded continuously and be collected separately per plane to facilitate granular performance analysis.

# Reporting Format

Each test report must include both global parameters and test-specific parameters. The following parameters for test configuration and SAV mechanism settings must be documented in the test report.

Test configuration parameters consist of:

1. Test device hardware and software versions.

2. DUT deployment type, such as hardware router, software router, VM, or container.

3. Network topology, including the location of the DUT and the interface on which SAV is evaluated.

4. Intra-domain interface type, if applicable: single host, set of hosts, or customer network with no AS.

5. Inter-domain relationship type, if applicable: customer, provider, lateral peer, RS, or RS-client.

6. Routing configuration, including IGP, BGP, route-policy, NO_EXPORT/NO_ADVERTISE, selective export, and any other policy configuration relevant to the test.

7. SAV mechanism and configuration, including whether the DUT uses SAV-related information, SAV-specific information, or both.

8. SAV table size and update characteristics.

9. Test traffic attributes, including packet size, traffic rate, source prefix distribution, destination prefix distribution, and the ratio of spoofed to legitimate traffic.

10. System configuration, including CPU, memory, caches, operating system, interface capacity, and hardware offload features when applicable.

11. Measurement method, including DUT logs, counters, telemetry, Tester observations, and timestamp sources.

12. Number of repeated runs and statistical treatment of the results.

For each accuracy test, the report must identify which packets are legitimate and which packets are spoofed, and why. For each convergence test, the report must identify the triggering event and the timestamping method. For each performance test, the report must identify whether SAV was enabled or disabled and whether the DUT was operating in steady state or during SAV table update.

# IANA Considerations {#IANA}

This document has no IANA actions.

# Security Considerations {#security}

The benchmarking tests outlined in this document are confined to evaluating the performance of SAV devices within a controlled laboratory environment using isolated networks.

The network topology employed for benchmarking MUST constitute an independent test setup. It MUST remain disconnected from devices that could relay test traffic into an operational production network. Spoofed traffic generated for the benchmarking tests MUST NOT be leaked outside the controlled test environment.

--- back

# Acknowledgements {#Acknowledgements}
{: numbered="false"}

Many thanks to Aijun Wang, Nan Geng, Susan Hares, Giuseppe Fioccola, Minh-Ngoc Tran, Shengnan Yue, Changwang Lin, Yuanyuan Zhang, Xueyan Song, Yangfei Guo, Shenglin Jiang, Tian Tong, Meng Li, Ron Bonica, and Mohamed Boucadair for their valuable comments and reviews on this document.
Apologies to any others whose names the authors may have missed mentioning.

# Appendix A. Summary of Changes (to be removed by RFC Editor before publication) {#appendix}
{: numbered="false"}

## A.1. Changes from Version -02 to Version -03
{: numbered="false"}

Version -03 adds this appendix to summarize the major changes introduced across document revisions and improve revision traceability. 
No technical changes to the benchmarking methodology or benchmark procedures are introduced in this revision.

## A.2. Changes from Version -01 to Version -02
{: numbered="false"}

The major changes from version -01 to version -02 are as follows:

* Clarified the scope of the document as a black-box laboratory benchmarking methodology for individual SAV devices and refined the generic test methodology.  

* Expanded and refined the SAV terminology and deployment scope to align with the intra-domain and inter-domain SAV problem statements.  

* Restructured the intra-domain SAV accuracy tests into symmetric routing, asymmetric routing, and hidden-prefix scenarios, and added an explicit hidden-prefix benchmarking scenario.  

* Refined the inter-domain SAV accuracy tests according to the relationship of the tested interface with the neighboring AS, and clarified the limited prefix propagation and DSR scenarios.  

* Refined the SAV performance indicators and clarified the generation and classification of legitimate and spoofed test traffic.  

* Expanded the reporting requirements and strengthened the security considerations for isolated benchmarking environments.  

## A.3. Changes from Version -00 to Version -01
{: numbered="false"}

The major changes from version -00 to version -01 are as follows:

* Clarified the computation method of false positive rate and false negative rate.  

* Updated the example IP prefixes in the intra-domain test scenarios to use IPv6 documentation prefixes.  

* Revised the DSR test scenario by introducing a dedicated anycast prefix and clarifying the corresponding route propagation and legitimate traffic paths.  

* Clarified the collection of resource utilization metrics, including continuous measurement and separate control-plane and data-plane measurements where possible.  

