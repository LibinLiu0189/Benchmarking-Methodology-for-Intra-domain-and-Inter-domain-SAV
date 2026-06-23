---
stand_alone: true
ipr: trust200902
cat: info # Check
submissiontype: IETF
area: General [REPLACE]
wg: IETF

docname: draft-ietf-bmwg-savnet-sav-benchmarking-01

title: Benchmarking Methodology for Intra-domain and Inter-domain Source Address Validation
abbrev: SAVBench
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
  RFC3704:
  RFC8704:
  RFC2544:
  RFC4061:
 
informative:
  intra-domain-ps:
    target: https://datatracker.ietf.org/doc/draft-ietf-savnet-intra-domain-problem-statement/
    title: Source Address Validation in Intra-domain Networks Gap Analysis, Problem Statement, and Requirements 
    date: 2026
  inter-domain-ps:
    target: https://datatracker.ietf.org/doc/draft-ietf-savnet-inter-domain-problem-statement/
    title: Source Address Validation in Inter-domain Networks Gap Analysis, Problem Statement, and Requirements 
    date: 2026
  intra-domain-arch:
    target: https://datatracker.ietf.org/doc/draft-ietf-savnet-intra-domain-architecture/
    title: Intra-domain Source Address Validation (SAVNET) Architecture 
    date: 2026
  inter-domain-arch:
    target: https://datatracker.ietf.org/doc/draft-wu-savnet-inter-domain-architecture/
    title: Inter-domain Source Address Validation (SAVNET) Architecture 
    date: 2026
  
--- abstract

This document defines methodologies for benchmarking the performance of intra-domain and inter-domain source address validation (SAV) mechanisms. SAV mechanisms are utilized to generate SAV rules to prevent source address spoofing, and have been implemented with many various designs in order to perform SAV in the corresponding scenarios. This document takes the approach of considering a SAV device to be a black box, defining the methodology in a manner that is agnostic to the mechanisms. This document provides a method for measuring the performance of existing and new SAV implementations.

--- middle

# Introduction

Source address validation (SAV) is significantly important to prevent source address spoofing. Operators are suggested to deploy different SAV mechanisms {{RFC3704}} {{RFC8704}} based on their deployment network environments. In addition, existing intra-domain (intra-AS) and inter-domain (inter-AS) SAV mechanisms have problems in operational overhead and SAV accuracy under various scenarios {{intra-domain-ps}} {{inter-domain-ps}}. Intra-domain and inter-domain SAVNET architectures {{intra-domain-arch}} {{inter-domain-arch}} are proposed to guide the design of new intra-domain and inter-domain SAV mechanisms to solve the problems. The benchmarking methodology defined in this document will help operators to get a more accurate idea of the SAV performance when their deployed devices enable SAV and will also help vendors to test the performance of SAV implementation for their devices.

This document provides generic methodologies for benchmarking SAV mechanism performance. To achieve the desired functionality, a SAV device may support multiple SAV mechanisms, allowing operators to enable those most suitable for their specific network environments. This document considers a SAV device to be a black box, regardless of the design and implementation. The tests defined in this document can be used to benchmark a SAV device for SAV accuracy (i.e., false positive and false negative rates), SAV protocol convergence performance, and control plane and data plane forwarding performance. These tests can be performed on a hardware router, a software router, a virtual machine (VM) instance, or a container instance, which runs as a SAV device. This document outlines methodologies for assessing SAV device performance and comparing various SAV mechanisms and implementations.

## Goal and Scope

The benchmarking methodology outlined in this draft focuses on two objectives:

* Assessing &ldquo;which SAV mechanism performs best&rdquo; over a set of well-defined scenarios.

* Measuring the contribution of sub-systems to the overall SAV systems' performance (also known as &ldquo;micro-benchmark&rdquo;).

This benchmark evaluates the SAV performance of individual devices (e.g., hardware/software routers) by comparing different SAV mechanisms under specific network scenarios. The results help determine the appropriate SAV deployment for real-world network scenarios.

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

SAV Control Plane: The SAV control plane consists of processes including gathering and communicating SAV-related information.

SAV Data Plane: The SAV data plane stores the SAV rules within a specific data structure and validates each incoming packet to determine whether to permit or discard it. 

Host-facing Router: An edge router directly connected to a layer-2 host network.

Customer-facing Router: An edge router connected to a non-BGP customer network which includes routers and runs the routing protocol.

AS Border Router: An intra-domain router facing an external AS.


# Test Methodology

## Test Setup

The test setup in general is compliant with {{RFC2544}}. The Device Under Test (DUT) is connected to a Tester and other network devices to construct the network topology introduced in {{testcase-sec}}. The Tester is a traffic generator to generate network traffic with various source and destination addresses in order to emulate the spoofing or legitimate traffic. It is OPTIONAL to choose various proportions of traffic.

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
|         |              |         |
+---------|    Tester    |<--------+
          |              |
          +--------------+
~~~~~~~~~~
{: #testsetup title="Test Setup."}

{{testsetup}} illustrates the test configuration for the Device Under Test (DUT). Within the test network environment, the DUT can be interconnected with other devices to create a variety of test scenarios. The Tester may establish a direct connection with the DUT or link through intermediary devices. The nature of the connection between them is dictated by the benchmarking tests outlined in {{testcase-sec}}. Furthermore, the Tester has the capability to produce both spoofed and legitimate traffic to evaluate the SAV accuracy of the DUT in relevant scenarios, and it can also generate traffic at line rate to assess the data plane forwarding performance of the DUT. Additionally, the DUT is required to support logging functionalities to document all test outcomes.

## Network Topology and Device Configuration 

The positioning of the DUT within the network topology has an impact on SAV performance. Therefore, the benchmarking process MUST include evaluating the DUT at multiple locations across the network to ensure a comprehensive assessment.

The routing configurations of network devices may differ, and the resulting SAV rules depend on these settings. It is essential to clearly document the specific device configurations used during testing.

Furthermore, the role of each device, such as host-facing router, customer-facing router, or AS border router in an intra-domain network, SHOULD be clearly identified. In an inter-domain context, the business relationships between ASes MUST also be specified.

When evaluating data plane forwarding performance, the traffic generated by the Tester must be characterized by defined traffic rates, the ratio of spoofed to legitimate traffic, and the distribution of source addresses, as all of these factors can influence test results.

# SAV Performance Indicators

This section lists key performance indicators (KPIs) of SAV for overall benchmarking tests. All KPIs SHOULD be measured in the benchmarking scenarios described in {{testcase-sec}}. Also, the KPIs SHOULD be measured from the result output of the DUT.
The standard deviation for KPIs' testing results SHOULD be analyzed for each fixed test setup, which can help understand the stability of the DUT's performance. The data plane SAV table refreshing rate and data plane forwarding rate below SHOULD be tested using varying SAV table sizes for each fixed test setup, which can help measure DUT's sensibility to the SAV table size for these two KPIs.

## False Positive Rate

The proportion of legitimate packets that are incorrectly classified as spoofed by the DUT to the total number of legitimate packets sent to the DUT. For the purpose of this document, this metric is computed based on packet counts (i.e., on a per-packet basis).

Note that other computation methods (e.g., based on byte counts) MAY be used for supplementary analysis, but are outside the scope of the normative definitions in this document.

## False Negative Rate

The proportion of spoofed packets that are incorrectly classified as legitimate (i.e., not blocked) by the DUT to the total number of spoofed packets sent to the DUT. For the purpose of this document, this metric is computed based on packet counts (i.e., on a per-packet basis).

Note that other computation methods (e.g., based on byte counts) MAY be used for supplementary analysis, but are outside the scope of the normative definitions in this document.

## Protocol Convergence Time

The control protocol convergence time represents the period during which the SAV control plane protocol converges to update the SAV rules when routing changes happen, and it is the time elapsed from the beginning of routing change to the completion of SAV rule update. This KPI can indicate the convergence performance of the SAV protocol.

## Protocol Message Processing Throughput

The protocol message processing throughput measures the throughput of processing the packets for communicating SAV-related information on the control plane, and it can indicate the SAV control plane performance of the DUT.

## Data Plane SAV Table Refreshing Rate

The data plane SAV table refreshing rate refers to the rate at which a DUT updates its SAV table with new SAV rules, and it can reflect the SAV data plane performance of the DUT.

## Data Plane Forwarding Rate

The data plane forwarding rate measures the SAV data plane forwarding throughput for processing the data plane traffic, and it can indicate the SAV data plane performance of the DUT. It is suggested that measuring the data plane forwarding rate of DUT enabling and disabling SAV to see the proportion of decrease for the data plane forwarding rate. This can help analyze the efficiency for SAV data plane implementation of the DUT.

## Resource Utilization

The resource utilization refers to the CPU and memory usage of the SAV processes within the DUT.

# Benchmarking Tests {#testcase-sec}

## Intra-domain SAV {#intra_domain_sav}

### False Positive and False Negative Rates

**Objective**: Evaluate the false positive rate and false negative rate of the DUT in processing both legitimate and spoofed traffic across various intra-domain network scenarios. These scenarios include SAV implementations for customer/host networks, Internet-facing networks, and aggregation-router-facing networks.

In the following, this document presents the test scenarios for evaluating intra-domain SAV performance on the DUT. Under each scenario, the generated spoofed traffic SHOULD include different types of forged source addresses, such as unused source addresses within the subnetwork, private network source addresses, internal-use-only source addresses of the subnetwork, and external source addresses. The ratios among these different types of forged source addresses SHOULD vary, since different SAV mechanisms may differ in their capability to block packets with forged source addresses of various types. Nevertheless, for all these types of spoofed traffic, the expected result is that the DUT SHOULD block them.

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
                      +--------------------+
                      |Tester (Sub Network)|
                      | ((2001:db8::/55))  |
                      +--------------------+
~~~~~~~~~~
{: #intra-domain-customer-syn title="SAV for customer or host network in intra-domain symmetric routing scenario."}

**SAV for Customer or Host Network**: {{intra-domain-customer-syn}} illustrates an intra-domain symmetric routing scenario in which SAV is deployed for a customer or host network. The DUT performs SAV as a customer/host-facing router and connects to Router 1 for Internet access. A sub network, which resides within the AS and uses the prefix 2001:db8::/55, is connected to the DUT. The Tester emulates a sub network by advertising this prefix in the control plane and generating both spoofed and legitimate traffic in the data plane. In this setup, the Tester is configured so that inbound traffic destined for 2001:db8::/55 arrives via the DUT. The DUT learns the route to 2001:db8::/55 from the Tester, while the Tester sends outbound traffic with source addresses within 2001:db8::/55 to the DUT, simulating a symmetric routing scenario between the two. The IP addresses used in this test case are optional; users may substitute them with other addresses, as applies equally to other test cases.

The **procedure** for testing SAV in this intra-domain symmetric routing scenario is as follows:

1. To verify whether the DUT can generate accurate SAV rules for customer or host network under symmetric routing conditions, construct a testbed as depicted in {{intra-domain-customer-syn}}. The Tester is connected to the DUT and acts as a sub network.

2. Configure the DUT and Router 1 to establish symmetric routing environment.

3. The Tester generates both legitimate traffic (with source addresses in 2001:db8::/55) and spoofed traffic (with source addresses in 2001:db8:0:200::/55) toward the DUT. The prefix 2001:db8:0:200::/55 does not belong to the sub network and thus is not advertised by the Tester. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and allows legitimate traffic originating from the sub network. 

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
                      +---------------------+
                      | Tester (Sub Network)|
                      |   (2001:db8::/55)   |
                      +---------------------+
~~~~~~~~~~
{: #intra-domain-customer-asyn title="SAV for customer or host network in intra-domain asymmetric routing scenario."}

**SAV for Customer or Host Network**: {{intra-domain-customer-asyn}} illustrates an intra-domain asymmetric routing scenario in which SAV is deployed for a customer or host network. The DUT performs SAV as a customer/host-facing router. A sub network, i.e., a customer/host network within the AS, is connected to both the DUT and Router 1, and uses the prefix 2001:db8::/55. The Tester emulates a sub network and handles both its control plane and data plane functions. In this setup, the Tester is configured so that inbound traffic destined for 2001:db8::/56 is received only from the DUT, while inbound traffic for 2001:db8:0:100::/56 is received only from Router 1. The DUT learns the route to prefix 2001:db8::/56 from the Tester, and Router 1 learns the route to 2001:db8:0:100::/56 from the Tester. Both the DUT and Router 1 then advertise their respective learned prefixes to Router 2. Consequently, the DUT learns the route to 2001:db8:0:100::/56 from Router 2, and Router 1 learns the route to 2001:db8::/56 from Router 2. The Tester sends outbound traffic with source addresses in 2001:db8:0:100::/56 to the DUT, simulating an asymmetric routing scenario between the Tester and the DUT.

The **procedure** for testing SAV in this intra-domain asymmetric routing scenario is as follows:

1. To determine whether the DUT can generate accurate SAV rules under asymmetric routing conditions, set up the test environment as shown in {{intra-domain-customer-asyn}}. The Tester is connected to both the DUT and Router 1 and emulates the functions of a sub network. 

2. Configure the DUT, Router 1, and Router 2 to establish the asymmetric routing scenario.

3. The Tester generates both spoofed traffic (using source addresses in 2001:db8::/56) and legitimate traffic (using source addresses in 2001:db8:0:100::/56) toward the DUT. The prefix 2001:db8::/56 does not belong to the sub network and thus is not advertised by the Tester. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic originating from the sub network.

~~~~~~~~~~
                   +---------------------+
                   |  Tester (Internet)  |
                   +---------------------+
                           /\   | Inbound traffic with source 
                           |    | IP address of 2001:db8:0:200::/55
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
| Test Network Environment |    |                          |
|                          |   \/                          |
|                       +----------+                       |
|                       |    DUT   | SAV facing Internet   |
| FIB on DUT            +----------+                       |
| Dest         Next_hop   /\    |                          |
| 2001:db8::/55  Network 1 |    |                          |
|                          |    \/                         |
|                       +~~~~~~~~~~+                       |
|                       | Router 1 |                       |
|                       +~~~~~~~~~~+                       |
|                         /\    |                          |
|             Traffic with |    | Traffic with             |
|      source IP addresses |    | destination IP addresses |
|           of 2001:db8::/55 |    | of 2001:db8::/55       |
|                          |    \/                         |
|                  +--------------------+                  |
|                  |    Sub Network     |                  |
|                  |   (2001:db8::/55)  |                  |
|                  +--------------------+                  |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
~~~~~~~~~~
{: #intra-domain-internet-syn title="SAV for Internet-facing network in intra-domain symmetric routing scenario."}

**SAV for Internet-facing Network**: {{intra-domain-internet-syn}} illustrates the test scenario for SAV in an Internet-facing network under intra-domain symmetric routing conditions. The network topology resembles that of {{intra-domain-customer-syn}}, with the key difference being the positioning of the DUT. In this case, the DUT is connected to Router 1 and the Internet, while the Tester emulates the Internet. The DUT performs SAV from an Internet-facing perspective, as opposed to a customer/host-facing role.

The **procedure** for testing SAV for an Internet-facing network in an intra-domain symmetric routing scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for Internet-facing SAV under symmetric routing, set up the test environment as depicted in {{intra-domain-internet-syn}}. The Tester is connected to the DUT and emulates the Internet.

2. Configure the DUT and Router 1 to establish symmetric routing environment.

3. The Tester generates both spoofed traffic (using source addresses in 2001:db8::/55) and legitimate traffic (using source addresses in 2001:db8:0:200::/55) toward the DUT. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and allows legitimate traffic originating from the Internet.

~~~~~~~~~~
                   +---------------------+
                   |  Tester (Internet)  |
                   +---------------------+
                           /\   | Inbound traffic with source 
                           |    | IP address of 2001:db8:0:200::/55
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
| Test Network Environment |    |                                    |
|                          |   \/                                    |
|                        +----------+                                |
|                        |    DUT   |                                |
| FIB on Router 1        +----------+  FIB on Router 2               |
| Dest           Next_hop  /\     \   Dest                 Next_hop  |
| 2001:db8::/56  Network 1 /       \  2001:db8:0:100::/56 Network 1  |
| 2001:db8:0:100::/56 DUT /        \/ 2001:db8::/56  DUT             |
|               +~~~~~~~~~~+     +~~~~~~~~~~+                        |
|               | Router 1 |     | Router 2 |                        |
|               +~~~~~~~~~~+     +~~~~~~~~~~+                        |
|                     /\            /                                |
|         Traffic with \           / Traffic with                    |
|   source IP addresses \         / destination IP addresses         |
|         of 2001:db8:0:100::/56 / of 2001:db8:0:100::/56            |
|                         \     \/                                   |
|                  +--------------------+                            |
|                  |    Sub Network     |                            |
|                  |   (2001:db8::/55)  |                            |
|                  +--------------------+                            |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
~~~~~~~~~~
{: #intra-domain-internet-asyn title="SAV for Internet-facing network in intra-domain asymmetric routing scenario."}

**SAV for Internet-facing Network**: {{intra-domain-internet-asyn}} illustrates a test case for SAV in an Internet-facing network under intra-domain asymmetric routing conditions. The network topology is identical to that of {{intra-domain-customer-asyn}}, with the key distinction being the placement of the DUT. In this scenario, the DUT is connected to Router 1 and Router 2 within the same AS, as well as to the Internet. The Tester emulates the Internet, and the DUT performs Internet-facing SAV rather than customer/host-network-facing SAV.

The **procedure** for testing SAV in this intra-domain asymmetric routing scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for Internet-facing SAV under asymmetric routing, construct the test environment as shown in {{intra-domain-internet-asyn}}. The Tester is connected to the DUT and emulates the Internet.

2. Configure the DUT, Router 1, and Router 2 to establish the asymmetric routing scenario.

3. The Tester generates both spoofed traffic (using source addresses in 2001:db8::/55) and legitimate traffic (using source addresses in 2001:db8:0:200::/55) toward the DUT. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic originating from the Internet.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                  Test Network Environment                |
|                       +----------+                       |
|                       |    DUT   | SAV facing Router 1   |
| FIB on DUT            +----------+                       |
| Dest           Next_hop  /\    |                         |
| 2001:db8::/55  Network 1 |     |                         |
|                          |    \/                         |
|                       +~~~~~~~~~~+                       |
|                       | Router 1 |                       |
|                       +~~~~~~~~~~+                       |
|                         /\    |                          |
|             Traffic with |    | Traffic with             |
|      source IP addresses |    | destination IP addresses |
|         of 2001:db8::/55 |    | of 2001:db8::/55         |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                           |    \/
                    +--------------------+
                    |Tester (Sub Network)|
                    |   (2001:db8::/55)  |
                    +--------------------+
~~~~~~~~~~
{: #intra-domain-agg-syn title="SAV for aggregation-router-facing network in intra-domain symmetric routing scenario."}

**SAV for Aggregation-router-facing Network**: {{intra-domain-agg-syn}} depicts the test scenario for SAV in an aggregation-router-facing network under intra-domain symmetric routing conditions. The network topology in {{intra-domain-agg-syn}} is identical to that of {{intra-domain-internet-syn}}. The Tester is connected to Router 1 to emulate a sub network, enabling evaluation of the DUT's false positive and false negative rates when facing Router 1.

The **procedure** for testing SAV in this aggregation-router-facing scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for aggregation-router-facing SAV under symmetric routing, construct the test environment as shown in {{intra-domain-agg-syn}}. The Tester is connected to Router 1 and emulates a sub network.

2. Configure the DUT and Router 1 to establish symmetric routing environment.

3. The Tester generates both legitimate traffic (using source addresses in 2001:db8::/56) and spoofed traffic (using source addresses in 2001:db8:0:200::/55) toward Router 1. The prefix 2001:db8:0:200::/55 does not belong to the sub network and thus is not advertised by the Tester. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic originating from the direction of Router 1.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                   Test Network Environment                           |
|                         +----------+                                 |
|                         |    DUT   | SAV facing Router 1 and 2       |
| FIB on Router 1         +----------+   FIB on Router 2               |
| Dest            Next_hop  /\      \    Dest                Next_hop  |
| 2001:db8::/56   Network 1 /        \  2001:db8:0:100::/56  Network 1 |
| 2001:db8:0:100::/56  DUT /         \/ 2001:db8::/56        DUT       |
|                  +~~~~~~~~~~+    +~~~~~~~~~~+                        |
|                  | Router 1 |    | Router 2 |                        |
|                  +~~~~~~~~~~+    +~~~~~~~~~~+                        |
|                       /\           /                                 |
|           Traffic with \          / Traffic with                     |
|     source IP addresses \        / destination IP addresses          |
|         of 2001:db8::/56 \      / of 2001:db8:0:100::/56             |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                            \   \/
                    +--------------------+
                    |Tester (Sub Network)|
                    |  (2001:db8::/55)   |
                    +--------------------+
~~~~~~~~~~
{: #intra-domain-agg-asyn title="SAV for aggregation-router-facing network in intra-domain asymmetric routing scenario."}

**SAV for Aggregation-router-facing Network**: {{intra-domain-agg-asyn}} illustrates the test case for SAV in an aggregation-router-facing network under intra-domain asymmetric routing conditions. The network topology in {{intra-domain-agg-asyn}} is identical to that of {{intra-domain-internet-asyn}}. The Tester is connected to both Router 1 and Router 2 to emulate a sub network, enabling evaluation of the DUT's false positive and false negative rates when facing Router 1 and Router 2.

The **procedure** for testing SAV in this aggregation-router-facing asymmetric routing scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules under asymmetric routing conditions, construct the test environment as shown in {{intra-domain-agg-asyn}}. The Tester is connected to Router 1 and Router 2 and emulates the functions of a sub network.

2. Configure the DUT, Router 1, and Router 2 to establish an asymmetric routing environment.

3. The Tester generates both spoofed traffic (using source addresses in 2001:db8:0:200::/55) and legitimate traffic (using source addresses in 2001:db8::/56) toward Router 1. The prefix 2001:db8:0:200::/55 does not belong to the sub network and thus is not advertised by the Tester. The ratio of spoofed to legitimate traffic may vary, for example, from 1:9 to 9:1.

The **expected results** for this test case are that the DUT blocks spoofed traffic and permits legitimate traffic originating from the direction of Router 1 and Router 2.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|                   Test Network Environment                  |
|     +------------+                     +------------+       |
|     |   Router2  |---------------------|   Router3  |       |
|     +------------+                     +------------+       |
|          /\                                  /\             |
|          |                                   |              |
|          | backup path                       | primary path |
|          |                                   |              |
|     +-----------------------------------------------+       |
|     |                     DUT                       |       |
|     +-----------------------------------------------+       |
|                           /\                                |
|                           | Legitimate and                  |
|                           | Spoofed Traffic                 |
|                           |                                 |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                            |
                  +--------------------+
                  |Tester (Sub Network)|
                  +--------------------+
~~~~~~~~~~
{: #intra-domain-frr-topo title="Intra-domain SAV under Fast Reroute (FRR) scenario."}

**SAV under Fast Reroute (FRR) Scenario**: Fast Reroute (FRR) mechanisms such as Loop-Free Alternates (LFA) or Topology-Independent Loop-Free Alternates (TI-LFA) provide sub-second restoration of traffic forwarding after link or node failures. During FRR activation, temporary forwarding changes may occur before the control plane converges, potentially impacting SAV rule consistency and causing transient
false positives or false negatives.

The **procedure** for testing SAV under FRR scenario is as follows:

1. Configure the DUT and adjacent routers with FRR protection for the primary link (Router3–DUT).  
   
2. The Tester continuously sends legitimate and spoofed traffic toward the protected prefix. 
    
3. Trigger a link failure between Router3 and the DUT, causing FRR switchover to Router2.  
   
4. Measure false positive and false negative rates during the switchover and after reconvergence. 
    
5. Restore the primary link and verify that SAV rules revert correctly.

The **expected results** for this test case are that the DUT should maintain correct SAV behavior throughout FRR activation and recovery. False positive and false negative rates SHOULD remain minimal during FRR events, and SAV rules SHOULD update promptly to reflect restored routing.

~~~~~~~~~~
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
|             Test Network Environment           |
|                 +------------+                 |
|                 |  Router2   |                 |
|                 +------------+                 |
|                       /\                       |
|                        | default path          |
|                        |                       |
|               +----------------+               |
|               |       DUT      |               |
|               +----------------+               |
|                 /\           /\                |
|    policy-based /             \ default path   |
|           path /               \               |
|         +-----------+      +-----------+       |
|         |  Router3  |      |  Router1  |       |
|         +-----------+      +-----------+       |
|              /\                 /\             |
|              |                   |             |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
               |                   |
          +-----------------------------+
          |     Tester (Sub Network)    |
          +-----------------------------+
~~~~~~~~~~
{: #intra-domain-pbr-topo title="Intra-domain SAV under Policy-based Routing (PBR) scenario."}

**SAV under Policy-based Routing (PBR) Scenario**: Policy-based Routing (PBR) enables forwarding decisions based on user-defined match conditions (e.g., source prefix, DSCP, or interface) instead of the standard routing table. Such policies can create asymmetric paths that challenge the SAV mechanism if rules are derived solely from RIB or FIB information.

The **procedure** for testing SAV under PBR scenario is as follows:

1. Configure PBR on the DUT to forward traffic matching a specific source prefix (e.g., 2001:db8::/56) to Router3, while other traffic follows the default path to Router1. 

2. The Tester sends both legitimate and spoofed traffic that matches and does not match the PBR policy.
    
3. Measure the false positive and false negative rates for both traffic types. 
    
4. Dynamically modify or remove the PBR policy and observe SAV rule adaptation.

The **expected results** for this test case are that the DUT SHOULD continue to correctly filter spoofed packets and permit legitimate packets under PBR scenario. SAV rules MUST adapt to policy-based forwarding paths without producing misclassification.

### Control Plane Performance {#intra-control-plane-sec}

**Objective**: Measure the control plane performance of the DUT, including both protocol convergence performance and protocol message processing performance in response to route changes caused by network failures or operator configurations. Protocol convergence performance is quantified by the convergence time, defined as the duration from the onset of a routing change until the completion of the corresponding SAV rule update. Protocol message processing performance is measured by the processing throughput, represented by the total size of protocol messages processed per second.

Note that the tests for control plane performance of the DUT which performs intra-domain SAV are OPTIONAL. Only DUT which implements the SAV mechanism using an explicit control-plane communication protocol, such as SAV-specific information communication mechanism proposed in {{intra-domain-arch}} SHOULD be tested on its control plane performance.

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

**Objective**: Measure the false positive rate and false negative rate of the DUT when processing legitimate and spoofed traffic across multiple inter-domain network scenarios, including SAV implementations for both customer-facing ASes and provider-/peer-facing ASes.

In the following, this document presents the test scenarios for evaluating inter-domain SAV performance on the DUT. Under each scenario, the generated spoofed traffic SHOULD include different types of forged source addresses, such as source addresses belonging to the local AS but not announced to external networks, private network source addresses, source addresses belonging to other ASes, and unallocated (unused) source addresses. The ratios among these different types of forged source addresses SHOULD vary, since different inter-domain SAV mechanisms may differ in their capability to block packets with forged source addresses of various origins. Nevertheless, for all these types of spoofed traffic, the expected result is that the DUT SHOULD block them.

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
|              /        |         \            \           |
|             / (C2P)   |          \ P5[AS 5]   \ P5[AS 5] |
|+~~~~~~~~~~~~~~~~+     |           \            \         |
||    AS 2(P2)    |     | P1[AS 1]   \            \        |
|+~~~~~~~~~~+/\+~~+     | P6[AS 1]    \            \       |
|             \         |              \            \      |
|     P6[AS 1] \        |               \            \     |
|      P1[AS 1] \       |                \            \    |
|          (C2P) \      | (C2P/P2P) (C2P) \      (C2P) \   |
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

**SAV for Customer-facing ASes**: {{inter-customer-syn}} presents a test case for SAV in customer-facing ASes under an inter-domain symmetric routing scenario. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network environment, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which in turn is a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. AS 1 advertises prefixes P1 and P6 to AS 2 and the DUT, respectively. AS 2 then propagates routes for P1 and P6 to the DUT, enabling the DUT to learn these prefixes from both AS 1 and AS 2. In this test, the legitimate path for traffic with source addresses in P1 and destination addresses in P4 is AS 1->AS 2->DUT->AS 4. The Tester is connected to AS 1 to evaluate the DUT's SAV performance for customer-facing ASes.

The **procedure** for testing SAV in this scenario is as follows:

1. To evaluate whether the DUT can generate accurate SAV rules for customer-facing ASes under symmetric inter-domain routing, construct the test environment as shown in {{inter-customer-syn}}. The Tester is connected to AS 1 and generates test traffic toward the DUT.

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
|              /        |         \            \           |
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

SAV for Customer-facing ASes: {{inter-customer-lpp}} presents a test case for SAV in customer-facing ASes under an inter-domain asymmetric routing scenario induced by NO_EXPORT community configuration. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which is itself a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. AS 1 advertises prefix P1 to AS 2 with the NO_EXPORT community attribute, preventing AS 2 from propagating the route for P1 to the DUT. Similarly, AS 1 advertises prefix P6 to the DUT with the NO_EXPORT attribute, preventing the DUT from propagating this route to AS 3. As a result, the DUT learns the route for prefix P1 only from AS 1. The legitimate path for traffic with source addresses in P1 and destination addresses in P4 is AS 1->AS 2->DUT. The Tester is connected to AS 1 to evaluate the DUT's SAV performance for customer-facing ASes.

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
|                         P3[AS 3] /         \ P3[AS 3]           |
|                        P7[AS 3] /           \ P7[AS 3]          |
|                                / (C2P)       \                  |
|                       +----------------+      \                 |
|                       |     DUT(P4)    |       \                |
|                       ++/\+--+/\+--+/\++        \               |
|          P3[AS 4, AS 3] /     |      \           \              |
|         P7[AS 4, AS 3] /      |       \           \             |
|                       /       |        \           \            |
|                      / (C2P)  |         \ P5[AS 5]  \ P5[AS 5]  |
|      +----------------+       |          \           \          |
|User+-+    AS 2(P2)    |       | P1[AS 1]  \           \         |
|      +----------+/\+--+       | P6[AS 1]   \           \        |
|          P6[AS 1] \           |             \           \       |
|           P1[AS 1] \          |              \           \      |
|                     \         |               \           \     |
|                      \ (C2P)  | (C2P)    (C2P) \     (C2P) \    |
|                    +----------------+        +----------------+ |
|                    |AS 1(P1, P6, P7)|        |    AS 5(P5)    | |
|                    +----------------+        +----------------+ |
|                         /\     |                                |
|                          |     |                                |
+~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
                           |    \/
                     +----------------+
                     |     Tester     |
                     | (Edge Server)  |
                     +----------------+
P7 is the anycast prefix and is advertised only by AS 3 via BGP.
Note that, unlike the other AS network figures in this document,
this figure illustrates that AS 3 advertises prefixes P3 and P7 
to AS 2 through AS 4, and does not depict the propagation of 
prefixes P1 and P6 beyond AS 2.
~~~~~~~~~~
{: #inter-customer-dsr title="SAV for customer-facing ASes in the scenario of direct server return (DSR)."}

**SAV for Customer-facing ASes**: {{inter-customer-dsr}} presents a test case for SAV in customer-facing ASes under a Direct Server Return (DSR) scenario. In this setup, AS 1, AS 2, AS 3, the DUT, and AS 5 form the test network, with the DUT performing SAV at the AS level. AS 1 is a customer of both AS 2 and the DUT; AS 2 is a customer of the DUT, which is itself a customer of AS 3; and AS 5 is a customer of both AS 3 and the DUT. When users in AS 2 send requests to an anycast destination IP in P7, the forwarding path is AS 2->DUT->AS 3. Anycast servers in AS 3 receive the requests and tunnel them to edge servers in AS 1. The edge servers then return content to the users with source addresses in prefix P7. If the reverse forwarding path is AS 1->DUT->AS 2, the Tester sends traffic with source addresses in P7 and destination addresses in P2 along the path AS 1->DUT->AS 2. Alternatively, if the reverse forwarding path is AS 1->AS 2, the Tester sends traffic with source addresses in P7 and destination addresses in P2 along the path AS 1->AS 2. In this case, AS 2 may serve as the DUT.

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

**SAV for Customer-facing ASes**: {{inter-customer-reflect}} illustrates a test case for SAV in customer-facing ASes under a reflection attack scenario. In this scenario, a reflection attack using source address spoofing occurs within the DUT's customer cone. The attacker spoofs the victim's IP address (P1) and sends requests to server IP addresses (P5) that are configured to respond to such requests. The Tester emulates the attacker by performing source address spoofing. The arrows in {{inter-customer-reflect}} indicate the business relationships between ASes: AS 3 serves as the provider for both the DUT and AS 5, while the DUT acts as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

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

**SAV for Customer-facing ASes**: {{inter-customer-direct}} presents a test case for SAV in customer-facing ASes under a direct attack scenario. In this scenario, a direct attack using source address spoofing occurs within the DUT's customer cone. The attacker spoofs a source address (P5) and directly targets the victim's IP address (P1), aiming to overwhelm its network resources. The Tester emulates the attacker by performing source address spoofing. The arrows in {{inter-customer-direct}} indicate the business relationships between ASes: AS 3 serves as the provider for both the DUT and AS 5, while the DUT acts as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

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

**SAV for Provider/Peer-facing ASes**: {{reflection-attack-p}} illustrates a test case for SAV in provider/peer-facing ASes under a reflection attack scenario. In this scenario, the attacker spoofs the victim's IP address (P1) and sends requests to server IP addresses (P2) that are configured to respond. The Tester emulates the attacker by performing source address spoofing. The servers then send overwhelming responses to the victim, exhausting its network resources. The arrows in {{reflection-attack-p}} represent the business relationships between ASes: AS 3 acts as either a provider or a lateral peer of the DUT and is the provider for AS 5, while the DUT serves as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

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

{{direct-attack-p}} presents a test case for SAV in provider-facing ASes under a direct attack scenario. In this scenario, the attacker spoofs a source address (P2) and directly targets the victim's IP address (P1), overwhelming its network resources. The arrows in {{direct-attack-p}} represent the business relationships between ASes: AS 3 acts as either a provider or a lateral peer of the DUT and is the provider for AS 5, while the DUT serves as the provider for AS 1, AS 2, and AS 5. Additionally, AS 2 is the provider for AS 1.

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

**SAV under FRR Scenario**: Inter-domain Fast Reroute (FRR) mechanisms, such as BGP Prefix Independent Convergence (PIC) or MPLS-based FRR, allow rapid failover between ASes after a link or node failure. These events may temporarily desynchronize routing information and SAV rules.

The **procedure** for testing SAV under FRR scenario is as follows:

1. Configure FRR or BGP PIC on the DUT for inter-AS links to AS3 (primary) and AS2 (backup). 
    
2. Continuously send legitimate and spoofed traffic from AS1 toward DUT.  
   
3. Trigger a failure on the AS3–DUT link to activate the FRR path via AS2.  
   
4. Measure false positive and false negative rates during and after switchover. 
    
5. Restore the AS3 link and verify SAV table consistency.

The **expected results** for this test case are that the DUT MUST maintain consistent SAV filtering during FRR events. Transient topology changes SHOULD NOT lead to acceptance of spoofed traffic or unnecessary blocking of legitimate packets.

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

**SAV under PBR Scenario**: In inter-domain environments, routing policies such as local preference, route maps, or communities may alter path selection independently of shortest-path routing. Such policy-driven forwarding can affect how the SAV rules are derived and applied.

The **procedure** for testing SAV under PBR scenario is as follows:

1. Configure a routing policy on the DUT (e.g., set local preference) to prefer AS3 for specific prefixes while maintaining AS2 as an alternative path.
   
2. Generate legitimate and spoofed traffic from AS1 matching both policy-affected and unaffected prefixes.

3. Observe SAV filtering behavior before and after policy changes. 
    
4. Modify the routing policy dynamically and measure false positive and false negative rates.

The **expected results** for this test case are that the DUT SHOULD maintain correct SAV filtering regardless of routing policy changes. Legitimate traffic rerouted by policy MUST NOT be dropped, and spoofed traffic MUST NOT be forwarded during or after policy updates.

### Control Plane Performance

The test setup, procedure, and metrics for evaluating protocol convergence performance and protocol message processing performance can refer to {{intra-control-plane-sec}}. Note that the tests for control plane performance of the DUT which performs inter-domain SAV are OPTIONAL. Only DUT which implements the SAV mechanism using an explicit control-plane communication protocol, such as SAV-specific information communication mechanism proposed in {{inter-domain-arch}} SHOULD be tested on its control plane performance.

### Data Plane Performance

The test setup, procedure, and metrics for evaluating data plane SAV table refresh performance and data plane forwarding performance can refer to {{intra-data-plane-sec}}.

## Resource Utilization

When evaluating the DUT for both intra-domain ({{intra_domain_sav}}) and inter-domain SAV ({{inter_domain_sav}}) functionality, CPU utilization (for both control and data planes) and memory utilization (for both control and data planes) MUST be recorded. These metrics SHOULD be recorded continuously and be collected separately per plane to facilitate granular performance analysis.

# Reporting Format

Each test follows a reporting format comprising both global, standardized components and individual elements specific to each test. The following parameters for test configuration and SAV mechanism settings MUST be documented in the test report.

Test Configuration Parameters:

  1. Test device hardware and software versions

  2. Network topology

  3. Test traffic attributes

  4. System configuration (e.g., physical or virtual machine, CPU, memory, caches, operating system, interface capacity)

  5. Device configuration (e.g., symmetric routing, NO_EXPORT)

  6. SAV mechanism


# IANA Considerations {#IANA}

This document has no IANA actions. 

# Security Considerations {#security}

The benchmarking tests outlined in this document are confined to evaluating the performance of SAV devices within a controlled laboratory environment, utilizing isolated networks.

The network topology employed for benchmarking must constitute an independent test setup. It is imperative that this setup remains disconnected from any devices that could potentially relay test traffic into an operational production network.

--- back

# Acknowledgements {#Acknowledgements}
{: numbered="false"}

Many thanks to Aijun Wang, Nan Geng, Susan Hares, Giuseppe Fioccola, Minh-Ngoc Tran, Shengnan Yue, Changwang Lin, Yuanyuan Zhang, and Xueyan Song, Yangfei Guo, Shenglin Jiang, Tian Tong, Meng Li, Ron Bonica, Mohamed Boucadair for their valuable comments and reviews on this document.
Apologies to any others whose names the authors may have missed mentioning.