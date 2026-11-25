= CARES — CAN Availability Recovery and Escalation System
Ullas
:doctype: article
:imagesdir: CAN_DOS_Attack_CARES_Solution_media
:toc: left
:toclevels: 3
:sectnums:
:sectanchors:
:figure-caption: Figure

CARES: A Resilient Architecture for Arbitration-Based Denial-of-Service Attacks on Automotive CAN

|===
|**Paper scope**

This draft explains the CAN arbitration DoS problem, its operational and safety consequences, currently available industry controls, the remaining capability gap, and CARES as the proposed system-level solution. The concept section intentionally avoids using named academic defense frameworks as explanatory dependencies.
|===

= Abstract

Controller Area Network (CAN) uses priority-based, non-destructive bitwise arbitration to provide deterministic bus access. The same mechanism can be abused by a malicious transmitter that repeatedly sends a low numerical identifier and continuously wins arbitration. Legitimate electronic control units then defer transmission, causing safety-relevant messages to exceed their latency, freshness, or end-to-end deadline requirements even when the malicious frames are syntactically valid. Existing industry controls provide useful but incomplete protection: CAN intrusion detection and prevention systems identify identifier, timing, diagnostic, signal, and network-load anomalies; secure transceivers constrain flooding and unauthorized identifiers from their local host; gateways filter and rate-limit traffic crossing managed boundaries; and higher-layer authentication protects authenticity and freshness of application data. These mechanisms cannot selectively remove an independently connected, noncompliant transmitter already driving a shared protected CAN segment. This paper proposes CARES, a CAN Availability Recovery and Escalation System. CARES combines deadline-aware attack confirmation, attacker-path classification, protocol-compliance assessment, zonal source localization, evidence-driven quarantine, an independent shadow availability control plane, minimum-service operation, and staged recovery. The architecture prioritizes restoration of bounded safety-service availability rather than detection alone.

**Keywords: **Controller Area Network, denial of service, arbitration, availability, automotive cybersecurity, zonal containment, resilience, recovery.

= 1. Introduction

CAN remains widely used for time-sensitive communication among automotive electronic control units because it is deterministic, efficient, robust, and well supported by automotive hardware. CAN is a multi-master broadcast network: every node observes the same bus, and simultaneous transmitters resolve access through identifier arbitration. The current ISO 11898-1 standard defines CAN data-link behavior, bus access, error detection, error signaling, fault confinement, Error Active, Error Passive and Bus-Off behavior [1].

The arbitration mechanism provides priority but not source authentication. A frame with a lower numerical identifier has more dominant bits earlier in the identifier field and therefore wins against a competing frame with a higher numerical identifier. A compromised ECU or a physically connected rogue device can exploit this rule to monopolize the bus. Because losing legitimate nodes simply stop transmitting and retry later, an attack composed of valid frames may not create the error pattern expected from a wiring fault or malformed frame.

The research problem is therefore not only whether the attack can be detected. The central problem is how the vehicle restores bounded availability of safety-critical services when the malicious transmitter is already on the protected CAN segment and may not obey normal fault-confinement behavior.

|===
|**Research objective**

Design an architecture that uses the least disruptive effective control first, recognizes when that control cannot work, escalates to physical source-zone containment, maintains minimum safety service, and avoids immediate reinfection during recovery.
|===

= 2. CAN Denial-of-Service Attack by Arbitration

*Figure 1. Arbitration-based denial of service: a repeated low identifier wins access and causes legitimate message deadline misses.*

image::image1.png[Arbitration-based denial of service,width=85%,align=center]

== 2.1 Normal CAN Arbitration

During the arbitration field, all transmitting nodes monitor the bus while sending the identifier. CAN uses dominant and recessive bus states. A dominant bit overwrites a recessive bit. If a node transmits recessive but observes dominant, the node recognizes that a higher-priority frame is present, stops transmitting without reporting an error, and retries when the bus becomes available. The rules for CAN bus access and arbitration are standardized in ISO 11898-1; CAN in Automation describes arbitration mode as the operating mode in which dominant bits may overwrite recessive bits [1], [2].

For an 11-bit identifier, lower numerical values contain dominant zero bits earlier than higher values. Therefore, identifier 0x000 has the highest possible identifier priority. Identifier priority is a scheduling property, not a statement of message trustworthiness.

[cols="1,1,1,3"]
|===
|**Transmitter** | **Example ID** | **Relative priority** | **Arbitration outcome during attack**

|Rogue node | 0x000 | Highest | Wins arbitration whenever it begins with legitimate nodes
|Brake ECU | 0x120 | High | Loses when the rogue node transmits concurrently
|Steering ECU | 0x180 | Medium-high | Loses and retries later
|Powertrain ECU | 0x200 | Medium | Can be repeatedly delayed
|Body ECU | 0x500 | Lower | May experience severe starvation
|===

== 2.2 Attack Execution

* The attacker obtains a transmit path to a CAN segment through a compromised ECU, an exposed diagnostic or gateway path, or direct wiring access.

* The attacker selects a low numerical identifier, commonly an unassigned identifier or one positioned above the targeted legitimate traffic.

* The attacker begins transmission whenever the bus becomes idle and repeats frames at a high rate.

* When a legitimate ECU begins at the same time, the attacker’s dominant identifier bits override the legitimate recessive bits.

* The legitimate ECU loses arbitration, does not treat the event as a protocol error, and retries after the current frame.

* Repeated attacker frames keep winning. Protected messages accumulate delay, exceed freshness limits, or miss deadlines.

* The vehicle experiences a service-availability violation even though each attacker frame may have valid format, bit stuffing, CRC, ACK, and end-of-frame fields.

== 2.3 Variants of Arbitration DoS

[cols="1,1,1,3"]
|===
|**Variant** | **Behavior** | **Why conventional detection may struggle**

|Full-bus domination | Attacker repeatedly transmits 0x000 or another extremely high-priority ID | Valid arbitration losses may produce few protocol errors
|Targeted starvation | Attacker selects an ID above victim traffic but below more critical traffic | Total bus load may remain below a simple overload threshold
|Rotating-ID flooding | Attacker changes among several low IDs | Single-ID frequency thresholds can be bypassed
|Burst attack | Short aggressive bursts are timed near protected-message release times | Average-rate monitoring may hide short availability violations
|Compromised-ECU flooding | Legitimate ECU transmits unauthorized IDs or excessive rates | Physical source appears to be an installed node
|External rogue node | Separate controller is attached directly to CANH/CANL | Bypasses host-side policy inside legitimate ECUs
|Noncompliant physical attacker | Custom transmitter ignores standard error confinement | Protocol-level shutdown assumptions may fail
|Pause-and-resume attack | Attacker becomes silent during mitigation and returns after reconnection | Immediate recovery creates repeated outages
|===

= 3. Consequences of the Attack

== 3.1 Loss of Timely Message Availability

The primary consequence is not necessarily permanent loss of every frame. The critical consequence is that information arrives too late for the receiving function. Automotive control depends on message age, periodicity, and end-to-end deadlines. Once those timing assumptions are violated, a syntactically valid message may be operationally unusable.

== 3.2 Functional Consequences

[cols="1,1,1,3"]
|===
|**Affected function** | **Example unavailable information** | **Potential system consequence**

|Braking and stability | Wheel speed, yaw state, brake request, actuator status | Loss or degradation of coordinated braking, stability control, or brake blending
|Steering | Steering angle, torque request, vehicle state | Reduced assistance, disabled coordinated function, or transition to local safe behavior
|Powertrain | Torque command, speed, transmission state, inverter status | Torque limiting, propulsion shutdown, or degraded drivability
|Battery and energy management | Voltage, temperature, isolation, current information | Conservative power limitation or local protection action
|ADAS and automated functions | Object, motion, chassis, and actuator availability | Feature disengagement and safe-state request
|Diagnostics and network management | Heartbeat, mode, diagnostic session, fault status | Loss of observability, cascading timeouts, and difficult recovery
|===

== 3.3 System-Level Consequences

* Critical message deadlines are missed and age-of-information grows beyond control assumptions.

* Multiple ECUs may record communication faults and enter local degraded or limp-home behavior.

* Diagnostic trouble codes and network-management transitions may cascade across the vehicle.

* Repeated retries and timeouts can increase processor load and complicate fault isolation.

* A safety function may become unavailable even though the electrical CAN bus has no physical short.

* Attack recovery can become cyclic if the attacker resumes immediately after communication is restored.

* Forensic attribution is difficult because classical CAN identifiers identify message meaning and priority, not a cryptographically verified physical sender.

|===
|**Safety interpretation**
The paper avoids claiming that every arbitration DoS directly causes a specific hazardous vehicle action. The consequence depends on vehicle architecture, function allocation, timeout handling, local fallback, redundancy, operating state, and functional-safety design. The defensible claim is that the attack violates communication-availability assumptions and can force safety-related functions into degraded or unavailable states.
|===

= 4. Current Industry Solutions

Current industry controls are valuable, but each protects a particular boundary or threat path. The proposed paper presents the controls accurately and identify the residual gap rather than characterizing current products as ineffective.

[cols="1,1,1,3"]
|===
|**Industry control** | **What it provides** | **Strength** | **Residual limitation**

|CAN IDPS / traffic monitoring | Identifier, DLC, timing, diagnostic, signal, payload, network-load, and injection anomaly detection; security-event reporting | Detects abnormal behavior and supports reaction logic | Does not by itself selectively silence a rogue transmitter already driving the shared wires
|Secure CAN transceiver | Hardware pass/block policy, local flooding protection, spoofing/tamper detection, dominant timeout | Detects compromised software in the protected local ECU | Protection applies to the local host path; a separate external device bypasses it
|Gateway filtering and rate limiting | Allowlist, denylist, burst control, diagnostic-state policy, interface shutdown | Highly effective when traffic crosses the managed gateway | Cannot filter a frame injected directly on the downstream protected segment
|Network segmentation / zonal gateway | Separates communication domains and limits propagation | Reduces attack blast radius and enables branch isolation | Coarse isolation also removes legitimate ECUs in that zone
|Secure onboard communication | Message authentication and freshness verification at the application/PDU layer | Protects authenticity, integrity, and replay freshness | Authenticated messages may still be delayed by arbitration monopolization
|Secure boot and software isolation | Prevents unauthorized ECU software and restricts access to communication peripherals | Reduces probability that an installed ECU becomes the attacker | Does not address an external physical node on the wires
|Local degraded operation | Timeout handling, local sensors, torque limitation, limp-home or safe-state logic | Maintains a subset of vehicle function during communication failure | Does not identify, remove, or contain the attacker
|Redundant communication path | Second CAN, Ethernet, point-to-point safety link, or discrete fallback | Preserves selected critical information independently | Adds hardware, wiring, integration, and validation cost
|===

== 4.1 CAN IDPS

Commercial CAN protection products monitor traffic, detect threats and anomalies, and provide prevention mechanisms for attacks including denial of service. Industry offerings support baseline and specification enforcement for message identifiers, diagnostics, signals, and network load, as well as timing and data correlation [3]. These capabilities are suitable for detecting prohibited low identifiers, excessive rates, target-message deadline loss, and unexpected diagnostic behavior.

== 4.2 Secure Transceivers

Secure transceivers add enforcement close to the ECU transmit path. NXP describes the TJA1153A as detecting and containing local-node flooding, transmission of identifiers not assigned to the local node, remote reuse of a uniquely assigned local identifier, and tampered messages [4]. The broader TJA115x family uses integrated filtering for hardware-based local CAN authorization and flooding protection from the local host [5].

This boundary is crucial: the secure transceiver controls the MCU connected through that transceiver. An independently powered rogue node attached elsewhere on CANH/CANL does not pass through another ECU’s secure transceiver.

== 4.3 Gateway and Zonal Controls

A gateway can filter, rate-limit, terminate diagnostics, or disable an ingress path when malicious traffic crosses that gateway. This is a strong and production-compatible control. Modern zonal architectures also provide useful hardware boundaries. The residual problem appears when the malicious transmitter is already on the protected local segment or attached beyond the filtering boundary.

== 4.4 Secure Message Authentication

AUTOSAR Secure Onboard Communication specifies authenticity and freshness protection for secured protocol data units and interaction with security-event reporting [6]. Authentication helps reject spoofed or replayed application data. It does not reserve bus time or prevent an electrical transmitter from repeatedly winning arbitration. Therefore, authentication complements but does not replace availability protection.

|===
|**Industry capability gap****
**Current controls can detect the attack, prevent a protected local ECU from becoming the attacker, block traffic at managed boundaries, and preserve selected functions through fallback. The remaining gap is coordinated recovery when a physically connected transmitter is already on a protected CAN segment and cannot be selectively disabled through existing local-host controls.
|===

= 5. Candidate and Alternative Mitigation Concepts

Several additional countermeasures can reduce the impact of arbitration-based denial of service or preserve selected services. They are useful design options, but each depends on assumptions about attacker capability, physical-layer compatibility, synchronization, or network topology. This section evaluates these methods before introducing CARES, because CARES is designed to coordinate their strengths while avoiding reliance on any one method as a universal solution.

image::Candidate_mitigations_considered_for_arbitration_DoS.png[width=800, align="center"]

*Figure 2. Candidate mitigations considered for arbitration DoS and the principal limitation of each standalone approach.*

image::image2.png[Candidate mitigations,width=85%,align=center]

[cols="1,1,1,3"]
|===
|**Candidate method** | **Operating principle** | **Problems it can handle** | **Why it is not a complete standalone solution**

|Dynamic baud-rate switching | Legitimate ECUs switch to a preconfigured alternate bit rate after confirmed attack. A simple attacker configured only for the original rate may lose synchronization. | Unadaptive external adapter or fixed-rate attack tool. Can create a short emergency communication window. | Every legitimate ECU on the fallback network must switch at the same time and use compatible bit timing. Nodes at different rates interpret each other as errors, so transition itself may disrupt communication. A monitoring FPGA can estimate and follow the new rate. The method therefore delays rather than guarantees containment.
|Alternate voltage or signaling levels | Legitimate nodes enter a second electrical signaling mode or use a separate transceiver characteristic to distinguish trusted traffic. | Conceptually separates emergency traffic from ordinary high-speed CAN signaling. | Standard high-speed CAN uses defined differential voltage and receiver thresholds. Changing levels on the same bus requires compatible custom transceivers and full EMC/electrical validation. An attacker connected to the same conductors can measure or clamp the new signaling. Out-of-range signaling risks component stress and is unacceptable.
|Reserved emergency high-priority IDs | Critical ECUs temporarily transmit selected safety messages using reserved low numerical identifiers. | Useful if the attacker uses an identifier with lower priority than the reserved range and is prevented from using that range. | No identifier has a higher priority than 0x000. If the attacker already uses 0x000, or learns the reserved range, priority raising cannot restore access. Changing IDs also requires synchronized receiver configuration and may affect diagnostics, gateways, and safety validation.
|Independent alternate communication channel | Critical ECUs exchange health, safe-state and minimum-service data over a separate CAN, Ethernet, serial, point-to-point, or discrete link. | Preserves essential coordination even when the primary CAN data plane is unavailable. | Adds wiring, connectors, transceivers, key management, scheduling, software and validation. If it duplicates all vehicle traffic, the cost becomes high. The fallback path must be physically and logically independent, or the same attacker can affect both paths.
|Single-wire communication fallback | Use a dedicated single-wire CAN physical layer or another single-ended emergency link for low-rate safety communication. | Can provide a low-cost, physically distinct minimum-service channel if designed into every required ECU. | A standard high-speed CAN transceiver cannot become SAE J2411 single-wire CAN through software alone. SWCAN needs a compatible transceiver and wiring, operates at much lower rates, and has lower noise immunity. Reusing CANH or CANL does not isolate an attacker already connected to that conductor.
|Low-speed fault-tolerant CAN fallback | Use a network designed around the ISO 11898-3 low-speed fault-tolerant physical layer, which supports continued communication under defined wiring faults. | Useful for wiring-fault tolerance and selected low-rate functions. | It is a different physical-layer design, not an automatic emergency mode of ordinary high-speed CAN. Its lower rate and hardware requirements make it unsuitable as a transparent retrofit. Fault tolerance does not authenticate or silence a malicious transmitter.
|CAN switches, relays, or zonal couplers | Place controllable hardware boundaries between CAN branch and disconnect the branch associated with malicious traffic. | Provides real physical containment and protects other zones, including against a transmitter that ignores CAN error confinement. | Source zones must be identified correctly. Isolation removes legitimate ECUs together with the attacker and can change termination. Added switching hardware introduces cost, failure modes, and timing implications. This is effective only when the topology provides controllable boundaries.
|Physical-layer fingerprinting | Classify a transmitter using voltage, edge shape, clock, timing, common-mode behavior, or other analog features. | Can indicate whether a frame originates from an expected ECU and can support source-zone confidence. | Measurements vary with temperature, supply, harness, aging, transceiver type, and loading. Similar hardware can produce overlapping signatures, while an adaptive transmitter may imitate features. Fingerprinting attributes a source but does not itself prevent transmission.
|Multi-point physical localization | Use synchronized monitors to compare edge arrival, amplitude, settling, and direction at multiple locations. | Can estimate the branch or zone from which a rogue transmitter drives the network. | Requires additional measurement points, calibration, and stable topology. Accuracy may fall in short networks or under environmental variation. Localization still requires a switch or coupler to perform quarantine.
|Local degraded operation | Critical ECUs use local sensors, cached state, bounded estimation, torque limits, or safe-state logic when CAN information is stale. | Maintains selected vehicle functions while communication is unavailable. | Does not detect, identify, or remove the attacker. The safe-service set is limited by locally available sensors and functional-safety assumptions.
|Redundant message paths | Transmit selected critical information over a second CAN, Ethernet, direct sensor connection, or discrete safety path. | Improves functional availability and allows cross-checking or failover. | Adds cost and architectural complexity. Common power, harness, gateway, software, or physical-access dependencies can defeat both paths unless independence is explicitly designed.
|Secure transceiver or local bus guardian | Restrict IDs, rates, and timing before a legitimate ECU reaches the shared bus. | Highly effective against compromised application software in the protected ECU. | Controls only the local host path. A separately powered device tapped elsewhere on CANH/CANL bypasses this enforcement.
|Selective traffic suppression and emergency schedule | Silence nonessential legitimate traffic and permit only a minimum critical schedule during attack response. | Reduces self-generated congestion, clarifies attribution, and simplifies degraded operation. | A rogue node is not required to obey the schedule. The method cannot overcome 0x000 monopolization without independent containment.
|Message authentication and freshness | Receivers authenticate critical application data and reject spoofed or replayed content. | Protects integrity, authenticity, and freshness of accepted data. | Does not reserve bus time. Authenticated legitimate messages can still be delayed or prevented from transmission by arbitration flooding.
|===

== 5.1 Dynamic Baud-Rate Switching

Changing the bit rate can temporarily separate legitimate ECUs from an attacker that is fixed to one configuration. However, CAN communication requires all nodes on a segment to share compatible nominal bit timing and sample points. High-speed CAN supports configured rates over a broad range, but the rate is a network design parameter rather than an authenticated secret [9], [10]. A controlled fallback would therefore require every participating ECU, gateway, coupler and test interface to enter the new timing profile at a common epoch. During an unsynchronized transition, nodes interpret traffic incorrectly and can generate errors. A capable FPGA can measure edge intervals and adapt quickly. For these reasons, CARES treats baud-rate switching only as an optional temporary recovery aid on an isolated zone, not as the primary security mechanism.

== 5.2 Alternate Voltage Levels or Signaling Modes

High-speed CAN transceivers are designed around standardized dominant and recessive differential behavior. Typical high-speed signaling biases both lines near 2.5 V in recessive state and produces a differential dominant state by raising CANH and lowering CANL within defined ranges [9], [10]. A custom emergency mode using new voltage levels would require compatible transceivers, receiver thresholds, bus loading and termination across all trusted ECUs. It must also remain inside component ratings and electromagnetic compatibility limits. Because a rogue device is physically attached to the same conductors, the attacker can observe or electrically disturb the alternate levels. CARES therefore do not rely on nonstandard voltages; electrically distinct emergency communication should use a separately engineered channel.

== 5.3 Reserved Emergency Message Identifiers

A vehicle can reserve a small set of low numerical identifiers for brake availability, steering availability, speed estimation, torque limitation and safe-state commands. During a partial or targeted attack, critical ECUs may switch to those identifiers under a predefined emergency profile. This is effective only if the attacker is restricted below the reserved priority and cannot transmit those identifiers. It is ineffective against 0x000 because no standard 11-bit identifier has higher priority. CARES can use emergency IDs inside unaffected zones or after a coupler has contained the attack but does not treat priority raising as a universal defense.

== 5.4 Independent Alternate Communication Channel

An independent channel is the strongest method for preserving coordination when the primary CAN data plane is unavailable. The channel may be a second CAN, a protected Ethernet path, a low-rate serial link, point-to-point safety wiring, or discrete safe-state signals. To control cost, CARES uses this principle as a low-bandwidth Shadow Availability Control Plane rather than duplicating all normal vehicle traffic. The channel carries attack votes, health, quarantine commands, minimum-service state and recovery authorization. Independence must include physical routing, power, interfaces, access control and software to trust boundaries.

== 5.5 Single-Wire and Fault-Tolerant CAN Options

SAE J2411 defines a single-wire automotive CAN physical layer that uses one single-ended data wire. National Instruments documents normal operation at approximately 33.333 kbit/s and a higher-rate service mode at approximately 83.333 kbit/s, with lower noise immunity than differential CAN [11], [12]. ISO 11898-3 defines low-speed fault-tolerant CAN above 40 kbit/s up to 125 kbit/s [13]. These are separate physical-layer implementations, not software-selectable fallback modes of an ordinary high-speed transceiver. They require compatible transceivers, wiring, signaling, and validation. A dedicated SWCAN safety link can be useful, but merely reusing CANH or CANL leaves the emergency signal exposed to the same attached attacker.

== 5.6 CAN Switching and Zonal Isolation

Controllable CAN switches, relays, bridges or zonal couplers provide the most direct containment mechanism for a noncompliant physical transmitter. When a branch is isolated, the attacker remains active only inside that branch and cannot consume arbitration time on the protected backbone. The tradeoff is the loss of every legitimate ECU on the branch and the need to maintain correct electrical termination after switching. CARES adopts smart zonal couplers because they provide a physical enforcement point but requires multi-evidence source confidence and minimum-service planning before quarantine.

== 5.7 PHY Fingerprinting and Physical Localization

Physical-layer fingerprinting can use voltage levels, edge timing, clock characteristics, rise/fall time, overshoot, common-mode behavior and other analog features to associate traffic with an ECU or source region. It is attractive because CAN identifiers do not identify the physical sender. The limitations are calibration effort, environmental drift, transceiver similarity, harness dependence, and attacker adaptation. CARES therefore treats PHY evidence as corroboration rather than the only quarantine trigger. Coupler direction is used first; fingerprinting, multi-point measurements and optional safe topology-response profiles increase the source-zone confidence.

== 5.8 Message Authentication and Freshness Protection

Application-layer authentication and freshness mechanisms allow receivers to reject forged or replayed protected data and provide useful security-event evidence [6]. They are essential when an attacker attempts to impersonate a critical message or reuse old valid content. Authentication does not reserve bus time and is evaluated only after a frame has reached the receiver. A malicious electrical transmitter can therefore prevent an authenticated legitimate frame from being sent by repeatedly winning arbitration. CARES treats authentication status as evidence and protects availability through enforcement boundaries, independent coordination, and deadline monitoring.

== 5.9 Selective Traffic Suppression and Emergency Scheduling

During an availability incident, legitimate ECUs can suppress diagnostics, comfort traffic, software-update communication, and other nonessential transmissions. Critical messages can be admitted through a predefined emergency schedule with conservative burst limits and reserved forwarding capacity. This reduces self-generated load, makes source attribution clearer, and helps unaffected zones maintain deterministic service. The method cannot compel a rogue device to follow the emergency schedule and cannot defeat continuous 0x000 transmission on the same arbitration domain. CARES uses traffic suppression after containment or within unaffected zones, where enforcement can be trusted.

== 5.10 Secure Transceivers and Local Bus Guardians

A secure transceiver or hardware bus guardian constrains the transmission behavior of the ECU connected through it. Typical controls include identifier allowlists, timing and rate enforcement, dominant-timeout protection, local silent mode, and reporting of unauthorized host requests [4], [5]. This is highly effective when application software inside an installed ECU is compromised because every local transmission traverses the guardian. The protection boundary ends at that local host path. An independently powered controller attached elsewhere to CANH/CANL bypasses the guardian, so CARES uses local enforcement first for compromised ECUs and escalates to zonal or port isolation for direct physical attachments.

== 5.11 Redundant Message and Communication Paths

Selected safety information can be carried over a second CAN network, switched Automotive Ethernet, a direct point-to-point link, or a discrete safety signal. Redundancy can preserve function when the primary CAN is unavailable and can provide cross-checking between independent sources. The benefit depends on genuine independence: common power supplies, gateways, software components, harness routing, connectors, keys, or physical access can create a common-cause failure. Duplicating all traffic is expensive and increases integration and validation effort. CARES therefore uses redundancy selectively for minimum safety coordination and the shadow control plane rather than duplicating the complete vehicle network.

== 5.12 Local Degraded Operation

Local degraded operation preserves a restricted function when remote CAN information is stale or unavailable. A brake controller may continue with local wheel-speed and actuator feedback; a steering controller may provide limited local assistance, and a powertrain controller may enforce a conservative torque limit. This approach is effective when the safety concept already provides local sensing, bounded estimation, timeout handling, and a validated safe state. It does not identify or remove the attacker, and the supported duration is limited by the quality and independence of local information. CARES incorporates degraded operation as the minimum-service layer used during containment and recovery rather than presenting it as a network defense by itself.

== 5.13 Multi-Point Physical Localization

Multi-point localization places synchronized monitors at two or more electrically separated locations and compares first-edge arrival, dominant-state amplitude, rise and fall time, settling, and direction of propagation. Its main advantage over a single fingerprint sensor is that the measurements are interpreted against known topology: a frame observed first on the local side of a coupler and later on the backbone provides direct source-region evidence. This method can identify a probable branch even when individual transceivers have similar signatures. Its limitations are additional measurement hardware, time-synchronization error, calibration effort, sensitivity to harness changes, and reduced spatial resolution on short networks. Localization also does not stop transmission; it must be paired with a switch, routed coupler, or controllable port.

== 5.14 Why the Methods Must Be Combined

The candidate methods solve different portions of the problem. Emergency IDs and traffic suppression help only while some bus access remains. Baud-rate or voltage changes may temporarily confuse a simple attacker but lack durable isolation. Single-wire or redundant channels preserve coordination but do not remove the source. Fingerprinting identifies a probable source but needs an enforcement mechanism. Switches provide containment but require reliable source of evidence and safe degraded behavior. CARES combines these functions into an escalation sequence: detect actual deadline impact, classify the attack path, use local or gateway enforcement where possible, assess protocol compliance, localize a direct noncompliant source, quarantine through a zonal coupler, maintain minimum service over the shadow plane, and recover gradually.

|===
|**Conclusion from the alternative-method assessment**
No single candidate mitigation reliably handles every arbitration DoS scenario. The strongest architecture combines independent coordination, accurate source of attribution, controllable physical boundaries, local fallback, and staged recovery. CARES formalizes that combination and selects the least disruptive effective response for each attack path.
|===

= 6. Proposed Solution in This Paper

This paper proposes CARES, the CAN Availability Recovery and Escalation System. CARES is not positioned as a replacement for industry IDPS, secure transceivers, gateways, authentication, or safe-state logic. CARES coordinates these controls and adds the missing system-level escalation and recovery behavior.

CARES treats availability as a measurable service property. The architecture does not declare success merely because suspicious traffic was detected or because the attacker became temporarily silent. Success is established when the protected message set again satisfies its configured deadlines; the unaffected zones remain operational, the minimum safety-service catalogue is available, and the affected zone has entered a controlled recovery state.

The architecture is deliberately response oriented. Detection evidence is converted into an ordered decision: determine whether the condition has real timing impact, determine where the traffic enters, select the least disruptive enforceable boundary, test whether a direct transmitter can be neutralized through normal protocol behavior, and escalate to zonal isolation only when that assumption is invalid. Every transition is logged and reversible except where vehicle safety requires a latched service state.

== 6.1 System-Level Design Objectives

The first objective is bound for detection and response. The vehicle-specific safety analysis defines the maximum number of consecutive deadline misses and the maximum service-outage interval that may occur before degraded operation is required. CARES uses these limits as engineering requirements for detection window, coupler switching time, shadow-plane latency, and recovery authorization.

The second objective is containment proportionality. A gateway session should be terminated before an entire zone is isolated; a compromised local ECU should be silenced through its trusted gate before neighboring ECUs are disconnected; and a direct rogue transmitter should be quarantined only after its source zone is supported by independent evidence. This ordering reduces the functional cost of the response.

The third objective is continued safety coordination. CARES assumes that normal distributed functions may be reduced during attack, but a defined minimum set of safety statuses and commands must remain available through local control, unaffected zones, or the shadow plane. The minimum set is derived from the vehicle item definition and hazard analysis rather than chosen generically.

The fourth objective is secure recovery. Reconnection is treated as a new authorization decision, not as the automatic inverse of isolation. A zone must demonstrate acceptable electrical behavior, authorized identifiers, bounded rates, healthy message deadlines, valid software state where observable, and no recurrence during progressively less restrictive forwarding modes.

== 6.2 Architectural Assumptions and Deployment Boundaries

CARES assumes that externally accessible or lower-trust CAN branches terminate at a controllable coupler before reaching the protected backbone. The backbone is routed and packaged to reduce routine physical access. A node wired directly to an unsegmented final backbone cannot be selectively removed by a software decision, so the physical partitioning is part of the security mechanism rather than a deployment convenience.

The shadow plane must not share a single point of failure with the attacked data plane. Independence should be considered across conductors, transceivers, power domains, software drivers, gateway paths, security keys, and diagnostic accessibility. Complete physical independence may not be achievable in every platform, but shared dependencies must be documented and included in the assurance claim.

CARES also assumes that critical ECUs can recognize stale or unavailable remote data and enter predefined local behavior. The architecture coordinates degraded operation; it does not create local controllability where the ECU design has no suitable sensors, estimators, or safety state.

|===
|**CARES design principle****
**Apply the least disruptive control that can restore availability. Escalate only when evidence shows that the current response cannot succeed. Preserve minimum safety service throughout the response, and never reconnect a quarantined zone without staged validation.
|===

== 6.3 Proposed Contributions

* Deadline-aware attack confirmation: classify an event as a cybersecurity availability attack only when unauthorized priority use correlates with actual protected-message deadline failure.

* Attacker-path classification: determine whether the traffic crosses a gateway, originates from a protected local ECU, or appears directly on a physical zone.

* Protocol-compliance assessment: Observe whether a directly connected transmitter exhibits standard error-confinement progression under a tightly bounded response.

* Zonal source localization: combine coupler direction evidence, multi-point physical measurements, and optional validated topology-response profiles.

* Evidence-driven quarantine: isolate a zone only when deadline impact, unauthorized dominance, source confidence, and independent attack voting agree.

* Shadow availability control plane: preserve trusted response coordination independently of the attacked CAN data plane.

* Minimum safety-service operation: keep unaffected zones and local fallback functions active while the attacked zone is isolated.

* Staged recovery and persistence trapping: reintroduce the zone through passive observation, receive-only synchronization, restricted forwarding, and full restoration.

= 7. CARES Architecture

*Figure 3. Proposed CARES architecture with protected zones, smart couplers, a protected backbone, and an independent shadow control plane.*

image::image3.png[CARES architecture,width=85%,align=center]

The CARES architecture is organized into two cooperating planes. The primary CAN data plane carries ordinary vehicle communication and remains subject to CAN arbitration. The shadow availability control plane carries only trusted evidence, state transitions, quarantine authorization, minimum-service status, and recovery commands. Separating the planes ensures that a node monopolizing normal CAN cannot also prevent the vehicle from coordinating the response to that monopolization.

The physical architecture converts one large shared arbitration domain into several smaller local domains connected through policy-enforcing couplers. This limits the electrical reach of a rogue transmitter. The logical architecture then overlays deadline monitoring, source-direction evidence, protocol-compliance assessment, safety-service management, and staged recovery on those physical boundaries.

A central supervisor may coordinate the decision, but enforcement remains distributed. Trusted PHY gates can suppress a compromised local ECU; gateways can terminate managed ingress, and couplers can independently enter a fail-safe isolation state. This distribution prevents the supervisor from becoming the only element capable of acting and supports deterministic response close to the affected interface.

[cols="1,1"]
|===
|**CARES component** | **Responsibility**

|Availability and Safety Supervisor | Decision engine that fuses IDPS, timing, direction, fault-state, physical, and safety-service evidence; authorizes escalation and recovery.
|CAN IDPS and Deadline Monitor | Detects policy violations and measures whether protected messages meet freshness and deadline requirements.
|Trusted PHY Gate | Enforces local identifier, rate, state, and emergency-transmission policy for protected ECUs.
|Smart Zonal Coupler | Terminates separate arbitration domains, validates forwarded frames, attributes direction, isolates a zone, and supports staged re-entry.
|Protected CAN Backbone | Interconnects zonal couplers while being physically protected from routine external access.
|Shadow Availability Control Plane | Carries authenticated votes, health, quarantine commands, emergency coordination, and recovery authorization independently of primary CAN.
|Local Degraded Controller | Maintains restricted local function using local sensors, cached state, bounded estimates, and predefined safety strategies.
|Recovery Manager | Controls quarantine observation, receive-only synchronization, allowlisted forwarding, and final requalification.
|===

== 7.1 Zonal Communication Model

CARES divides the vehicle into local CAN zones. Each zone is connected to a protected backbone through a smart coupler. The coupler receives a complete frame on one arbitration domain, evaluates policy, and re-originates an accepted frame on the destination domain. A malicious transmitter can monopolize the local zone, but the coupler can stop that traffic from propagating to the backbone.

This design introduces forwarding, queueing, and renewed arbitration delay. Those delays must be included in worst-case response-time and network-schedulability analysis. The paper treats timing analysis as a core validation activity, not an implementation detail.

Each coupler therefore acts as a controlled store-and-forward boundary. On reception, the local-side controller timestamps the frame, records the ingress interface, checks identifier and state policy, evaluates the configured burst/rate envelope, and places an accepted frame in a destination queue. The backbone-side controller then arbitrates normally on the protected backbone. The reverse direction follows an equivalent policy appropriate to the destination zone.

Direction information gives CARES a source-region property that classical CAN does not provide. If an unauthorized identifier is first observed on the local interface of Coupler C and is subsequently seen on the backbone only after Coupler C forwards it, Zone C becomes the probable origin. If the frame appears first on the backbone, Coupler C must not attribute the source to its local zone.

Queue management must preserve safety priorities without allowing one zone to exhaust shared memory. Each zone should have bounded queues, per-class admission rules, and overload behavior defined before deployment. Critical frames may receive reserved forwarding capacity, while diagnostics, comfort traffic, and unusually bursty sources are delayed or discarded according to vehicle-state policy.

Partitioning also changes failure semantics. A local wiring fault or rogue transmitter can render one zone unusable while the backbone and other zones remain healthy. The safety analysis must identify which functions disappear with each zone and ensure that no single quarantinable branch contains every source needed for the minimum safety-service set.

== 7.2 Shadow Availability Control Plane

The shadow plane carries only security and minimum-service coordination. It is intentionally low bandwidth and independent of the attacked CAN data plane. A typical control message includes message type, source, zone, event identifier, freshness counter, decision epoch, and authentication tag.

The shadow plane is not intended to carry ordinary sensor and actuator traffic. Its limited scope keeps bandwidth, verification effort, and attack surface small. A representative deployment may carry periodic health heartbeats, event-triggered evidence reports, two-source quarantine votes, coupler state acknowledgements, minimum-service availability, and recovery-stage authorization.

Every control message is bound to a decision epoch and event identifier. The event identifier prevents evidence from one incident being applied to another; the epoch allows all couplers to agree on the current response state; and a monotonic freshness counter prevents replay of an earlier quarantine or recovery command. The authentication mechanism must provide source authenticity and integrity within the response-time budget.

Quarantine authority should be separated from observation. A monitor can submit evidence but should not be able to isolate a zone by itself. A practical policy requires agreement between the supervisor and one independent protected node, or between two independently powered monitors, before the coupler accepts a high-impact isolation command. Emergency local isolation remains possible when a coupler detects an internal hardware fault that requires a fail-safe reaction.

Loss of the shadow plane must lead to a defined state. During normal operation, a temporary heartbeat loss can be logged while primary CAN remains healthy. During a confirmed attack, loss of authenticated coordination should block progression toward less restrictive recovery states. Depending on the safety concept, the coupler may retain its current quarantine state, critical ECUs may remain in degraded mode, and service intervention may be required.

The security keys and counters for the shadow plane should be stored in protected hardware where available. Key provisioning, replacement, vehicle-service access, manufacturing diagnostics, and end-of-life handling are part of the deployment design because unauthorized control of the shadow plane would give an attacker influence over containment and recovery.

[cols="1,1,1,3"]
|===
|**Shadow-plane message** | **Purpose**

|Authenticated heartbeat | Proves availability of supervisor, couplers, and critical ECUs
|Attack vote | Reports deadline failure, unauthorized dominance, or source evidence
|Quarantine command | Authorizes a coupler to isolate an identified zone
|Minimum-service status | Shares brake, steering, speed-estimate, torque-limit, and safe-stop availability
|Recovery authorization | Advances a zone from quarantine to observation, receive-only, restricted, or full forwarding
|===

== 7.3 Smart Zonal Coupler

The coupler is the main physical enforcement point. It contains two CAN interfaces, policy and timing monitoring, a fail-safe isolation element, local buffering, event storage, and a shadow-plane interface. The coupler should default to a defined safety state if its control logic, power, or watchdog fails. Isolation must preserve correct termination and electrical behavior for the backbone and remaining local network.

In normal forwarding mode, the coupler validates each completed frame before it is re-originated. Policy checks include permitted direction, allowed identifier, expected source zone, vehicle operating state, minimum inter-arrival time, maximum burst, diagnostic-session state, and optional authenticity status. Frames that violate policy are not forwarded, although local capture continues for forensic analysis.

The isolation element must separate the local transceiver from the protected backbone without leaving the backbone unterminated or creating a stub that violates signal-integrity assumptions. Depending on topology, isolation may use two independently controlled transceivers, an automotive-qualified analog switch, relay, or bridge architecture. The safe electrical state, power-off behavior, welded-contact detection, and diagnostic coverage must be defined.

The coupler maintains a rolling incident buffer containing ingress timestamps, identifiers, frame length, forwarding decision, error observations, local and backbone load, shadow-plane commands, and state transitions. This data supports post-event analysis and allows the supervisor to reconstruct whether a suspicious frame originated locally or arrived from another zone.

During staged recovery, the coupler becomes a controlled one-way or restricted gateway. Receive-only synchronization allows the isolated ECUs to observe valid network state while preventing local transmissions from reaching the backbone. Restricted forwarding then enables only an allowlisted subset at conservative rates. The coupler returns to full forwarding only after receiving authenticated authorization and completing a clean observation interval.

A coupler is itself a safety-relevant and cybersecurity-relevant component. Independent watchdogs, protected configuration, secure boot, memory protection, plausibility checks between the two CAN interfaces, command timeouts, and a diagnosable isolation element are required to prevent a compromised or faulty coupler from creating an uncontrolled bridge or unnecessary outage.

== 7.4 Availability and Safety Supervisor

The supervisor executes the CARES state machine and maintains the authoritative incident context. Inputs include normalized IDPS events, per-message age counters, bus-load statistics, coupler ingress direction, secure-transceiver status, error-confinement observations, PHY evidence, safety-service status, and authenticated votes. The supervisor converts these heterogeneous inputs into deterministic evidence flags rather than relying on an opaque anomaly score for quarantine.

The supervisor separates cybersecurity confidence from safety urgency. Cybersecurity confidence answers whether unauthorized priority abuse and source evidence are credible. Safety urgency answers how close protected functions are to their deadline or outage limits. A high-confidence anomaly with no service impact may remain under observation, while rapidly deteriorating safety availability can trigger immediate degraded operation even before physical attribution is complete.

The supervisor also owns response sequencing. It records which control was attempted, the expected observable result, the timeout for that result, and the next escalation path. For example, a gateway shutdown should produce disappearance of the suspicious traffic; a local trusted-gate command should produce a silent host; and protocol neutralization should produce altered retransmission behavior. Failure to observe the expected effect is itself evidence for escalation.

== 7.5 CAN IDPS and Deadline Monitor

The IDPS supplies policy evidence, while the deadline monitor supplies availability evidence. IDPS rules detect prohibited identifiers, excessive rates, invalid diagnostics, payload or signal anomalies, unexpected source direction, and unusual load. The deadline monitor independently tracks the last accepted instance of every protected message and computes message age using hardware or synchronized timestamps.

This separation prevents a high utilization value from being treated automatically as an attack. A legitimate stress condition can be accepted when all protected deadlines remain healthy. Conversely, a targeted starvation attack can be confirmed even when average bus load is moderate because one or more protected messages exceed their individual age limits.

For every protected message, configuration should include nominal period, minimum and maximum expected inter-arrival time, absolute deadline, maximum consecutive misses, criticality, authorized source zone, expected security state, receiving functions, and fallback action. The catalogue becomes a version-controlled cybersecurity and safety artifact that must be updated whenever network scheduling or function allocation changes.

== 7.6 Trusted PHY Gate

The trusted PHY gate is the enforcement boundary between a protected ECU application and the shared local bus. It may be implemented by a secure transceiver, a dedicated hardware state machine, or a protected MCU communication peripheral. The gate observes the host transmit request and permits only identifiers, rates, bursts, and operational states authorized for that ECU.

The gate supports immediate containment of a compromised application without removing neighboring ECUs. On command, it can force the local transmitter silent, restrict the ECU to a small emergency allowlist, or block transmission while leaving reception enabled. Status information is reported over the shadow plane or another trusted interface so that the supervisor can verify that the commanded action actually occurred.

The gate does not solve the external-tap case because an independently powered transmitter does not pass through it. CARES therefore uses the gate at the local-host stage of escalation and uses coupler-based isolation for a direct physical node.

== 7.7 Protected CAN Backbone

The backbone carries inter-zone traffic after coupler validation and should have stronger physical and logical protection than routine edge buses. Access points, service connectors, unused stubs, test pads, and harness transitions should be minimized or placed behind controlled interfaces. Backbone traffic should consist only of frames re-originated by known couplers or trusted backbone nodes.

Because the backbone is a separate arbitration domain, its schedule must reserve capacity for forwarded critical messages and recovery traffic. Worst-case analysis must include local reception, policy processing, queueing, backbone arbitration, and destination-zone forwarding. Admission control should prevent a healthy but overloaded zone from consuming the complete backbone schedule.

== 7.8 Local Degraded Controller and Minimum-Service Catalogue

Every safety-relevant zone defines what can be maintained without normal distributed communication. A brake controller may continue using local wheel-speed and actuator feedback; a steering controller may provide limited local assistance; a powertrain controller may enforce torque limitation; and the vehicle supervisor may request a controlled stop. These are vehicle-specific safety strategies, not generic CARES commands.

The minimum-service catalogue records each required service, supporting local sensors, dependencies on other zones, permitted estimation duration, confidence or validity status, transition conditions, and exit criteria. During attack, CARES distributes only the status needed to coordinate these restricted functions. Missing or invalid minimum-service status must itself cause a conservative transition.

== 7.9 Recovery Manager

The recovery manager prevents a silent or intermittent attacker from exploiting immediate reconnection. It controls a sequence of increasingly permissive states: quarantine, passive local observation, receive-only synchronization, restricted forwarding, and full service. Each state has entry conditions, a minimum dwell time, monitored invariants, and an immediate rollback trigger.

Recovery authorization depends on both cybersecurity and functional readiness. Traffic must remain within identifier and rate policy, physical measurements must remain acceptable, critical ECUs must report valid local state, and unaffected-zone deadlines must remain healthy. A zone is not promoted merely because the malicious identifier disappeared.

== 7.10 End-to-End CARES Information Flow

During normal operation, couplers and protected ECUs emit heartbeats and health summaries while the supervisor maintains baselines. When a suspicious event occurs, local evidence is timestamped and associated with a common event identifier. The supervisor requests additional observations, confirms timing impact, classifies the ingress path, and selects a response boundary. The selected actuator acknowledges both command receipt and achieved physical state.

After enforcement, the supervisor verifies the outcome using independent measurements. Successful gateway or local-host containment should remove the suspicious traffic and restore deadlines. If a direct transmitter remains active, CARES evaluates compliance and source-zone evidence, obtains quarantine agreement, isolates the zone, confirms backbone recovery, and activates the minimum-service plan. Recovery messages then progress the coupler through restricted states.

== 7.11 Trust Boundaries, Failures, and Safe Defaults

CARES has four major trust boundaries: the application-to-trusted-gate boundary, each local-zone-to-coupler boundary, the coupler-to-backbone boundary, and the shadow-plane control boundary. Security requirements and diagnostic coverage should be assigned explicitly at each boundary. A component is trusted only for the narrow function it must perform, such as enforcing a local allowlist or opening an isolation path.

Safe default behavior is context dependent. A coupler power failure may need to isolate a lower-trust diagnostic zone but preserve a safety-critical zone long enough for controlled degradation. A supervisor reset should not automatically reconnect quarantined zones. Loss of time synchronization should prevent use of stale votes and block promotion to a less restrictive recovery state.

False-positive quarantine is treated as a designed failure mode. The safety analysis must determine the consequence of removing each zone, the maximum acceptable isolation duration, and the local functions that remain. Multi-evidence confirmation reduces probability, while the minimum-service catalogue limits consequence. Both probability and consequence must be included in the assurance argument.

= 8. CARES Operating Stages

*Figure 4. CARES stage model from offline configuration through monitored restoration.*

image::image4.png[CARES operating stages,width=85%,align=center]

== 8.1 Stage 0: Configuration and Calibration

Create the signed communication matrix and safety-service catalogue. For each protected message define source zone, authorized transmitter, identifier, period, deadline, burst limit, criticality, timeout behavior, and fallback action. Calibrate normal bus utilization, coupler direction, physical measurements, and optional safe network-response profiles. Define the functional consequence of isolating each zone.

Configuration is security-controlled and versioned. The signed communication matrix is checked at startup and after software update. Invalid or inconsistent configuration prevents automatic quarantine authority because the system cannot safely infer whether an identifier or rate is authorized.

Calibration establishes normal ranges but does not grant permission by itself. Learned timing, load, and PHY characteristics are bounded by engineering policy so that malicious behavior during a learning period cannot become an accepted baseline. Environmental calibration should cover voltage, temperature, transceiver variants, harness tolerances, and aging assumptions.

Exit condition: configuration signatures are valid, all required nodes report the same policy version and decision epoch, coupler self-tests pass, shadow-plane freshness counters are initialized, and the minimum-service catalogue is available.

== 8.2 Stage 1: Normal Monitoring

Continuously inspect identifiers, periodicity, rates, network load, diagnostics, DLC, payload policy, error frames, and message age. Trusted PHY gates enforce local transmit policy. Couplers record source direction and local/backbone health. The shadow plane exchanges authenticated heartbeats. A rolling forensic buffer preserves the traffic immediately preceding an incident.

The system operates with no disruptive action. Deadline counters are updated on accepted protected messages, not merely on any frame using the expected identifier. Couplers maintain local and backbone utilization, forwarding delay, queue occupancy, and policy-denial counters.

Heartbeats prove that the response components are alive. A missing heartbeat is classified separately from an arbitration attack and may reduce CARES response capability. The rolling buffer retains enough pre-trigger data to reconstruct the beginning of an event without unbounded storage.

Exit condition: a configured suspicion trigger is met, or a component fault requires a safety transition.

== 8.3 Stage 2: Suspicion

Enter SUSPECT when an unassigned low identifier appears, an authorized identifier exceeds its envelope, one identifier dominates traffic, load changes sharply, or a protected message approaches its deadline. Freeze forensic history, increase monitoring resolution, request observations from couplers and critical ECUs, and avoid disruptive action until persistence and impact are assessed.

The purpose of SUSPECT is evidence collection without unnecessary disruption. Sampling resolution and event logging increase, but ordinary forwarding continues unless a protected deadline is already endangered. The supervisor creates an event identifier and asks all relevant couplers to report first-seen interface, timestamp, load, queue state, and matching policy violations.

The system checks alternative explanations such as vehicle startup, diagnostic programming, legitimate burst communication, clock resynchronization, wiring errors, and known fallback modes. A short anomaly that disappears without deadline impact can return to NORMAL after a clean holdoff interval.

Exit condition: the evidence decays and the event is closed, the condition is classified as a non-cyber network fault, or persistence plus service impact advances the event to confirmation.

== 8.4 Stage 3: Availability-Impact Confirmation

Confirm the incident only when security evidence and service impact agree. A typical confirmation requires unauthorized priority use, persistence across multiple observation windows, and one or more protected-message deadline violations. At least two independent observations should support a high-impact response. High utilization with healthy deadlines remains a monitored overload, not a confirmed cyber containment event.

Confirmation requires both an attack indicator and a service indicator. Attack indicators include an unauthorized low ID, an authorized ID outside its rate envelope, rotating priority abuse, or prohibited diagnostics. Service indicators include deadline misses, rising message age, queue starvation, or loss of a defined minimum service.

Independent agreement prevents one faulty sensor from forcing isolation. The supervisor can require, for example, a deadline report from a critical ECU plus direction evidence from a coupler, or matching observations from two independently powered couplers. Thresholds are derived from the vehicle safety and cybersecurity analyses.

Exit condition: confirmed attack with an assigned confidence level and affected-service set, or reclassification as benign overload/fault.

== 8.5 Stage 4: Attacker-Path Classification

Determine where the malicious traffic enters. If the traffic crosses a gateway, filter or shut down that ingress. If a protected ECU produces it, use the trusted PHY gate or secure transceiver to block local transmission. If the traffic originates directly on a local zone, proceed to compliance assessment and localization. This stage ensures CARES does not isolate an entire zone when a less disruptive boundary control is sufficient.

Path classification determines the cheapest trustworthy enforcement point. Gateway ingress is identified by frames present on an external interface before the protected side. A compromised local ECU is identified when its trusted gate reports policy violation or when a uniquely assigned local path is known. Direct-zone injection is inferred when local traffic appears without a corresponding trusted-host transmission or managed ingress.

The supervisor records the selected actuator and expected verification evidence. If gateway shutdown is selected, suspicious traffic must disappear downstream. If a trusted gate is selected, the host TX path must become silent. Failure to observe that effect advances the event to the direct-physical-node branch.

Exit condition: managed ingress contained, local host contained, or direct physical path selected for compliance assessment.

== 8.6 Stage 5: Protocol-Compliance Assessment

Apply only a tightly bounded and independently authorized response to confirmed malicious transmissions, then observe transmitter behavior. A standards-compliant controller is expected to alter retransmission behavior as fault confinement progresses. Continued full-rate transmission without expected suspension indicates that protocol-level shutdown cannot be trusted. The implementation must remain within electrical ratings and undergo controlled laboratory validation before any vehicle use.

This stage is authorized only for an already confirmed malicious stream and is bounded by count, duration, target pattern, and safety timeout. The objective is to determine whether the transmitter behaves like a conventional CAN controller whose fault state changes in response to repeated transmission failures.

CARES observes retry spacing, error signaling where visible, changes associated with Error Passive operation, interruption of transmission, and later recovery attempts. The supervisor does not need to know the exact internal error counter; it evaluates whether externally observable behavior is consistent with standard confinement.

Exit condition: malicious traffic stops with compliant progression, the observation window expires without progression and the transmitter is classified noncompliant, or safety timing requires immediate quarantine/degraded operation.

== 8.7 Stage 6A: Compliant-Transmitter Neutralization

When the transmitter follows standard confinement behavior, continue the bounded response until malicious traffic stops. Verify restoration of protected-message deadlines and observe for automatic recovery or repeated attack. Restrict the identified path if recurrence occurs. Move to monitored restoration rather than immediately declaring the network healthy.

After the malicious stream stops, CARES verifies that disappearance is causally associated with the response rather than a voluntary attacker pause. Critical-message deadlines, bus load, queue occupancy, and error levels must return to an acceptable range. The source path remains restricted if it is known.

Automatic controller recovery is treated as a recurrence risk. Reappearance of the prohibited stream within the requalification window returns the incident to containment and can cause the source zone to be quarantined even though the transmitter is protocol compliant.

Exit condition: protected service is restored for the validation interval and the source transitions to staged recovery, or recurrence triggers escalation.

== 8.8 Stage 6B: Noncompliant-Transmitter Localization

When the transmitter does not exhibit expected confinement behavior, stop relying on protocol shutdown. Use smart-coupler direction first. If direction evidence is insufficient, compare synchronized edge timing, amplitude, rise/fall behavior, and settling at multiple points. An optional research extension applies a small set of prevalidated, electrically safe network profiles and compares the response vector with calibrated zone models.

Localization begins with deterministic topology evidence: which coupler interface first observed the stream and whether the backbone saw the frame only after forwarding. This evidence is preferred over analog fingerprinting because it is easier to validate and interpret.

If topology evidence is ambiguous, CARES combines multi-point timestamps and PHY measurements. Optional topology-response profiles are applied only when prevalidated for the current network state. The output is a zone-confidence estimate with uncertainty, not an unsupported claim of exact ECU identity.

Exit condition: one zone exceeds the quarantine-confidence threshold, localization remains ambiguous and the vehicle enters a broader safe response, or the attack stops and the zone proceeds to monitored observation.

== 8.9 Stage 7: Evidence-Driven Zone Quarantine

Authorize quarantine only when four conditions agree: protected-message deadline failure, unauthorized arbitration dominance, source-zone confidence above its threshold, and authenticated independent voting. Open the coupler isolation path so the attack remains in the local zone and cannot reach the backbone. Preserve forensic data and verify termination and health of the remaining network.

Before opening the isolation path, the supervisor verifies the expected functional loss and confirms that required minimum services are available elsewhere or locally. The quarantine command contains event identifier, zone, target coupler, decision epoch, allowed post-isolation mode, freshness value, and authentication tag.

The coupler acknowledges command acceptance and then reports achieved electrical state. The supervisor independently confirms that the suspicious traffic no longer reaches the backbone, termination remains healthy, unaffected-zone deadlines recover, and no secondary overload is created by fallback traffic.

Exit condition: stable quarantine with restored backbone availability, quarantine actuation failure requiring a broader safety state, or evidence showing that the wrong zone was selected and requiring rollback/reassessment.

== 8.10 Stage 8: Minimum Safety-Service Operation

Activate local degraded functions and coordinate only essential status over the shadow plane or unaffected zones. Prioritize brake availability, steering availability, reliable speed estimation, torque limitation, safe-stop request, and zone health. Suspend nonessential traffic, diagnostics, comfort features, and recovery actions that could destabilize the system.

The vehicle intentionally reduces functionality to the predefined minimum-service set. Critical ECUs use local sensors and bounded estimates, while the shadow plane communicates compact availability and safe-state information. All status includes validity and freshness so that stale fallback information is not treated as current.

Nonessential diagnostics, comfort functions, software-update activity, and recovery experiments are suspended. Traffic generated by degraded operation is budgeted in advance so that fallback itself does not overload the surviving zones or backbone.

Exit condition: vehicle reaches a safe operational state, service intervention is requested, or the isolated zone becomes eligible for staged requalification.

== 8.11 Stage 9: Staged Recovery and Requalification

Keep the zone isolated while observing local traffic. Then permit receive-only resynchronization, followed by tightly allowlisted and rate-limited forwarding, and finally normal forwarding after a clean validation interval. Any prohibited identifier, abnormal rate, deadline degradation, or suspicious physical behavior returns the zone immediately to quarantine and lengthens the observation interval.

Passive observation measures local traffic without reconnecting it. Receive-only mode allows ECUs to rebuild network state while blocking outward transmission. Restricted forwarding admits only explicitly authorized identifiers at conservative rates. Full forwarding is the final state, not the default recovery action.

Each promotion requires a clean dwell interval, valid health reports, stable electrical measurements, correct software/configuration state where available, and healthy deadlines in unaffected zones. Any prohibited frame, burst violation, suspicious PHY behavior, deadline regression, missing heartbeat, or inconsistent vote immediately returns the zone to quarantine.

Exit condition: full operation after the validation interval, continued restricted service, or latched quarantine requiring maintenance.

= 9. CARES Decision Logic

The decision logic is implemented as a deterministic state machine with guarded transitions. Each transition records the evidence set, configuration version, decision epoch, affected services, selected actuator, expected result, timeout, and rollback path. This makes the response reviewable for cybersecurity assurance and functional-safety confirmation.

CARES maintains separate confidence values for attack confirmation, source-zone attribution, protocol compliance, and recovery readiness. A high value in one dimension cannot silently substitute for missing evidence in another. For example, very high attack confidence does not establish which zone should be disconnected.

== 9.1 Multi-Evidence Confirmation

Evidence is time-correlated within the event window. Reports outside the configured freshness interval are ignored, and reports with inconsistent epochs are not combined. The supervisor also accounts for evidence dependence: two reports produced from the same sensor or gateway path do not count as two independent votes.

Negative evidence is retained. Healthy protected deadlines reduce the urgency of an anomaly; absence of source-direction agreement reduces localization confidence; and successful local-host shutdown removes the need for zone isolation. This prevents a one-way accumulation of suspicion.

Vehicle-state context modifies permissible actions. Isolation thresholds, minimum-service requirements, and recovery dwell times may differ between parked service mode, low-speed maneuvering, highway operation, charging, and manufacturing. The policy must remain deterministic for every supported state.

[cols="1,1,1,3"]
|===
|**Evidence family** | **Example indicator** | **Role in decision**

|Authorization | Identifier is unassigned or source is not permitted | Establishes policy violation
|Timing | Protected message exceeds age/deadline or misses consecutive cycles | Establishes actual availability impact
|Dominance | Low identifier occupies abnormal share of frames or arbitration opportunities | Identifies priority abuse
|Direction | Coupler observes suspicious traffic first on its local side | Identifies probable source zone
|Compliance | Transmitter lacks expected fault-confinement progression | Determines that protocol shutdown is unreliable
|Independent vote | Critical ECU and coupler report consistent incident evidence | Reduces false containment risk
|===

|===
|**Recommended quarantine predicate****
**Quarantine = confirmed deadline failure AND unauthorized priority abuse AND source-zone confidence above threshold AND authenticated independent agreement. The exact thresholds are vehicle- and safety-analysis-specific.
|===

== 9.2 Example Decision Sequence

* ID 0x000 appears at an excessive rate and is absent from the authorized communication matrix.

* Brake and steering status messages exceed the configured deadline.

* The local side of Coupler C observes the suspicious frame before the backbone side.

* The transmitter continues without the expected fault-confinement behavior during the bounded assessment.

* The supervisor classifies a direct noncompliant transmitter in Zone C.

* Coupler C and a protected critical ECU submit authenticated attack votes.

* The supervisor commands Coupler C to isolate Zone C.

* Zones A and B verify restored deadline satisfaction and enter minimum-service mode.

* Zone C is restored only through staged observation and restricted forwarding.

= 10. Implementation Blueprint

Implementation should proceed incrementally. A first prototype can use two or three isolated CAN domains, dual-channel microcontroller couplers, a separate authenticated serial shadow link, simulated safety ECUs, and an FPGA/MCU test transmitter. The prototype should demonstrate the complete state transition rather than only detection accuracy.

The production partitioning is derived from threat exposure and safety dependencies. Diagnostic, body, aftermarket, or externally accessible branches are natural lower-trust zones. Safety-critical functions should be distributed so that isolation of one zone does not remove all sensing and actuation needed for the minimum service.

Timing budgets must be allocated before implementation. The total outage includes detection windows, evidence collection, command authentication, coupler actuation, CAN rescheduling, and receiving-ECU timeout behavior. If the total exceeds the safety bound, the architecture must trigger local degraded operation earlier rather than waiting for full attribution.

[cols="1,1,1"]
|===
|**Subsystem** | **Suggested realization** | **Purpose**

|Availability supervisor | Automotive MCU or domain controller with multiple CAN channels, secure key storage, high-resolution timers, watchdog, event memory, and shadow interface | Decision fusion, response authorization, evidence logging, and state machine
|Smart coupler | Dual-CAN MCU, two transceivers, queueing and policy engine, fail-safe isolation element, optional measurement front end | Separate arbitration domains, attribute direction, isolate and restore zones
|Trusted PHY gate | Secure transceiver, dedicated state machine, or protected MCU peripheral | Local ID/rate enforcement and silent-mode control
|Shadow link | Independent low-rate differential, serial, safety, or separately switched communication path | Authenticated attack votes and safety/recovery coordination
|Critical ECU fallback | Local sensors, bounded estimators, safe-state software, watchdogs | Minimum service when distributed information is unavailable
|Instrumentation | CAN analyzer, oscilloscope, synchronized logic capture and event logger | Validate traffic, timing, error-state, physical response, and containment
|Rogue-node test fixture | Controlled FPGA/MCU test node with standards-compliant and noncompliant laboratory modes | Evaluate the full CARES escalation path on an isolated bench
|===

== 10.1 Software Architecture

The software is divided into safety- and security-relevant services with explicit interfaces. The deadline monitor operates from hardware timestamps and protected configuration. The evidence-fusion engine consumes normalized events rather than vendor-specific alarms. The state machine is single-writer for incident state, while monitors and couplers remain independent evidence producers.

Configuration changes require authenticity, rollback protection, compatibility checks, and controlled activation. A new communication matrix must not become active on only part of the vehicle because disagreement about authorized identifiers or deadlines could generate false attacks or unsafe forwarding.

Persistent event storage must be bounded and wear-aware. High-rate raw traffic can remain in circular memory, while transition summaries, evidence hashes, selected frame windows, command acknowledgements, and recovery outcomes are promoted to nonvolatile storage. Time synchronization quality is stored with each record.

Diagnostic access to CARES must be state controlled. Service tools may read evidence and initiate authorized maintenance workflows, but routine diagnostics must not directly force quarantine release, overwrite attack history, disable deadline monitoring, or change shadow-plane keys without a protected procedure.

* Configuration manager for the signed communication matrix and protected-message catalogue.

* Real-time deadline monitor using hardware timestamps and per-message age counters.

* IDPS adapter that normalizes identifier, timing, diagnostic, signal, and network-load events.

* Evidence-fusion engine with deterministic, reviewable rules for confirmation and quarantine.

* Coupler manager for forwarding policy, isolation state, termination health, and staged recovery.

* Shadow-plane security module for authentication, freshness, replay detection, and voting.

* Forensic recorder with bounded storage, synchronized timestamps, and event snapshots.

* Safety-service manager that activates local fallback and reports minimum-function availability.

== 10.2 Functional Safety and Cybersecurity Co-Engineering

The hazard analysis should consider false negative, false positive, delayed response, wrong-zone isolation, failure to isolate, failure to maintain termination, loss of shadow communication, stale recovery authorization, and simultaneous failure of local fallback. The cybersecurity analysis should consider spoofed evidence, compromised couplers, replayed commands, denial of the shadow plane, malicious configuration and abuse of recovery interfaces.

Safety mechanisms and security controls must not undermine each other. A watchdog reset must not erase quarantine state; a secure-boot failure must lead to a defined coupler mode; emergency diagnostic access must not bypass forwarding policy; and cryptographic verification time must fit the response budget. The resulting safety case and cybersecurity case should share assumptions and verification evidence.

A practical assurance argument demonstrates four properties: unauthorized arbitration dominance is detected within a bounded interval; a suitable enforcement boundary is selected with controlled false-isolation risk; minimum safety service remains available after containment; and a quarantined source cannot regain unrestricted backbone access without passing staged requalification.

CARES containment actions can intentionally remove communication from a zone. The decision logic, coupler hardware, degraded modes, and recovery process must therefore be developed with cybersecurity and functional-safety analyses together. The paper specifies safety assumptions, single-point failures, latent failures, timeout behavior, watchdog behavior, isolation-element failure states, and the consequences of false-positive quarantine.

= 11. Physical Enforcement for Directly Connected Rogue Transmitters

*Figure 5. CARES Enforcement Fabric: independently terminated ports or zones prevent a rogue local transmitter from directly driving the protected backbone.*

image::image5.png[CARES enforcement fabric,width=85%,align=center]

The most demanding CARES threat case is an independently powered FPGA or controller attached directly to CANH and CANL. The existing architecture must distinguish between detecting this device and actually preventing it from influencing protected communication. On an uninterrupted multidrop bus, software running in another ECU cannot selectively cancel the dominant bits produced by a noncompliant transmitter. Detection, classification, and physical-layer attribution therefore require an architectural actuator that changes the electrical attack domain.

|===
|**Fundamental shared-bus limitation****
**A noncompliant transmitter attached to the same uninterrupted CAN conductors cannot be selectively silenced by an IDS, gateway rule, message authentication, or another ECU’s secure transceiver. Selective containment requires a controllable electrical boundary, control of the attacker path or power, isolation of the complete branch, or an independent safety communication path.
|===

image::CARES_Enforcement_Fabric_independently_terminated_ports_or_zones_prevent_a_rogue_local_transmitter_from_directly_driving_the_protected_backbone.png[width=800, align="center"]

== 11.1 CARES Enforcement Fabric

CARES is therefore divided into a control architecture and an enforcement fabric. The control architecture performs deadline-aware detection, evidence fusion, response selection, authenticated voting, minimum-service coordination, and staged recovery. The enforcement fabric consists of individually terminated CAN ports or zones connected through active security couplers. The fabric is the mechanism that makes quarantine physically enforceable.

A security coupler is not a transparent analog repeater. It terminates the local arbitration domain, receives a complete frame using a local CAN controller, associates the frame with a known ingress port, checks policy, and re-originates an accepted frame using a separate backbone CAN controller. A dominant bit asserted in the local zone is therefore not electrically reproduced on the backbone. Only a completed and authorized frame is transmitted into the protected domain.

The coupler can enforce identifier, direction, rate, burst, diagnostic-state, and vehicle-state policy before forwarding. It can block only the offending identifier when the local bus remains usable, restrict the zone to an emergency allowlist, or disable the complete local-facing port when a rogue transmitter monopolizes the local arbitration domain. Electrical isolation must leave the backbone correctly terminated and must not create an uncontrolled stub.

== 11.2 Isolation Granularity

[cols="1,1,1,3"]
|===
|**Isolation option** | **Architecture** | **Ability to stop rogue path** | **Primary tradeoff** | **Recommended use**

|Per-ECU or per-connector port | Every ECU or exposed interface connects through an independent security port | Can disable one physical port without removing unrelated ECUs | Highest transceiver, controller, connector, power, timing, and validation cost | Diagnostic connector, telematics, aftermarket interface, highest-criticality ECUs
|Per-zone coupler | A small group of ECUs shares a local CAN segment behind one coupler | Contains the rogue device to its zone and protects the backbone | All legitimate ECUs in the attacked zone are quarantined together | Most practical CARES production deployment
|Exposed-interface isolation only | Only diagnostic, service, telematics, or accessory paths receive protected ports | Blocks the most likely managed entry paths | Does not stop a direct internal harness splice downstream of the protected interface | Cost-constrained legacy platform retrofit
|===

== 11.3 Concrete Response to a Rogue FPGA or Controller

* Local policy enforcement begins continuously, before central confirmation. The coupler records the ingress port and prevents prohibited identifiers or excessive rates from being forwarded to the backbone.

* If the rogue transmitter still allows the coupler to receive completed frames, the coupler can selectively discard the malicious frames while forwarding authorized local traffic.

* If repeated ID 0x000 transmissions monopolize the local segment and legitimate ECUs cannot obtain service, the supervisor confirms deadline impact and source-zone evidence.

* The supervisor and an independent protected node issue authenticated quarantine votes. The command identifies the event, decision epoch, target coupler, requested isolation mode, and freshness value.

* The coupler disables local-to-backbone forwarding and opens the isolation element. The rogue transmitter may continue driving the local conductors but cannot consume arbitration time on the protected backbone.

* The supervisor verifies achieved isolation through coupler feedback, disappearance of attack traffic on the backbone, restored deadlines in unaffected zones, valid termination status, and active minimum-service reports.

* The affected zone remains under passive observation. Reconnection progresses through receive-only synchronization, restricted forwarding, and full forwarding. Any recurrence causes immediate re-quarantine.

== 11.4 Optional Port Power and Transceiver Control

For vehicle-powered external accessories or diagnostic interfaces, the enforcement port can include an automotive load switch or eFuse. After confirmed abuse, CARES first blocks forwarding, then disables the port transceiver and can remove connector power. This can stop a bus-powered rogue adapter and provides a stronger response than frame filtering alone.

Power removal is not sufficient for an independently powered attacker and must not be assumed to provide electrical isolation. Back-power paths through signal pins must also be assessed. Accordingly, switched power is an optional supporting actuator; the security coupler remains the primary boundary.

== 11.5 Security Isolation Versus Galvanic Isolation

Galvanic isolation protects power and signal domains against ground differences, transients, and conducted disturbances. It does not automatically block a syntactically valid malicious frame. CARES requires data-path or security isolation: the receiving controller terminates the source domain, checks the completed frame and decides whether a different controller may transmit a corresponding frame into the destination domain.

A normal isolated transceiver or transparent repeater can protect electronics while still propagating malicious traffic. A CARES security coupler must therefore expose ingress identity, apply policy, support forwarding suppression, report achieved state, and provide a diagnosable physical isolation mode.

== 11.6 Revised CARES Assurance Claim

|===
|**Revised claim****
**CARES cannot selectively silence an arbitrary noncompliant transmitter on an unsegmented shared bus. CARES restores bounded availability when exposed or lower-trust attachments are placed behind independently terminated security ports or zonal couplers. Under that deployment assumption, a rogue transmitter can monopolize only its local domain; the corresponding coupler blocks propagation and isolates the domain while unaffected zones continue minimum safety service.
|===

= 12. Alternative Architectures Beyond CARES

CARES is one system-level approach, but other architectures can reduce the same risk. These alternatives differ in whether they prioritize exact physical containment, functional redundancy, retrofit cost, or preservation of only a minimal safe state.

[cols="1,1,1,3"]
|===
|**Alternative architecture** | **How it addresses the attack** | **Strength** | **Limitation / residual risk**

|Active-star CAN network | Every branch terminates at a central active star that receives and reproduces communication; a malicious branch can be disabled individually | Port-level attribution, branch isolation, controlled termination | Central hardware, added latency, additional failure modes, and redesign cost
|Fully switched Automotive Ethernet backbone | Zonal controllers terminate edge CAN and exchange inter-zone traffic over a switched network with port policing and shutdown | Strong source-port identity, traffic shaping, segmentation, and redundant routing options | Legacy CAN zones remain vulnerable locally; gateway and Ethernet assurance complexity
|Dual independently routed CAN networks | Critical information is transmitted over two physically and logically independent CAN paths | Improves functional availability during single-bus attack | Does not remove attacker; cost and common-cause independence must be managed
|Dedicated point-to-point safety links | A small set of commands such as torque inhibit or safe-stop request use independent links or discrete signals | Deterministic and independent of CAN arbitration | Limited bandwidth and scalability; preserves only selected functions
|Per-node transmit-disable hardware | Every protected ECU has a guardian-controlled transceiver-enable or silent-mode path | Strong against compromised legitimate ECU software | Cannot disable an unrelated external tap elsewhere on shared conductors
|Controlled branch power management | Externally accessible or accessory branches are powered through a protected load switch | Can disable bus-powered malicious or faulty accessories | Independently powered attacker remains active; legitimate branch devices lose power
|Harness tamper prevention and admission monitoring | Protect harness access and monitor loading, resistance, capacitance, edge behavior, or connector state | May prevent or detect attachment before flooding begins | Environmental variation, servicing changes, and high-impedance devices can reduce confidence
|Independent safety controller / safety island | Local sensors and actuators maintain a defined safe function without primary CAN | Strongest preservation of safe behavior despite complete network loss | High cost, duplicated functionality, and does not restore normal network service
|===

Table 11. Alternative architecture for rogue-transmitter resilience.

== 12.1 Active-Star CAN

An active-star topology is the closest alternative to the CARES enforcement fabric. Each branch has an independent receiver and driver, so a node cannot directly dominate every branch. A central star can attribute traffic to a branch and disconnect only the affected branch. CARES can use an active star as its physical deployment form, with the supervisor, evidence logic, minimum-service coordination, and recovery protocol layered above it.

== 12.2 Switched Ethernet Backbone with CAN Edge Zones

A future vehicle platform can terminate legacy CAN locally in zonal controllers and carry inter-zone communication over switched Automotive Ethernet. The Ethernet switch provides identifiable ingress ports, queues, traffic policing, virtual separation, and port shutdown. CARES concepts remain useful at the CAN edge because a rogue transmitter can still monopolize one local CAN zone; the zonal controller prevents that failure from becoming a vehicle-wide backbone outage.

== 12.3 Redundancy and Safety Islands

Dual communication paths and safety islands address a different objective: they preserve function even when the attacked CAN cannot be recovered immediately. Independent wheel-speed sensing, a direct torque-inhibit line, redundant CAN, or a local safety controller can support a controlled safe state. These methods complement physical containment but do not identify or remove the attacker.

== 12.4 Prevention and Attachment Detection

Physical-access prevention, protected connectors, reduced test points, harness packaging, tamper switches, branch-current monitoring, and electrical-admission baselines can reduce the probability or dwell time of an unauthorized attachment. These measures are valuable preventive layers, but they should not be the sole availability claim because a well-designed attacker may be externally powered and present little additional loading before transmission.

== 12.5 Recommended Complete Architecture

For the direct noncompliant FPGA case, the strongest overall design combines independently terminated per-zone or per-port security couplers, deadline-aware detection, source-port attribution, forwarding suppression, physical isolation, an independent shadow safety channel, local degraded operation, and staged recovery. Per-port isolation should be prioritized for exposed connectors and critical interfaces, while per-zone isolation provides a practical balance for the broader vehicle network.

|===
|**Key correction to the CARES design****
**Without a physical enforcement fabric, CARES is primarily an intrusion-detection, decision, and recovery coordinator. With independently terminated security ports or zonal couplers, CARES can genuinely prevent a rogue local transmitter from influencing the protected backbone.
|===

= 13. Experimental Evaluation Plan

== 13.1 Research Questions

* Can CARES distinguish legitimate high load from an availability attack using deadline-aware evidence?

* Can CARES correctly select gateway filtering, local-host shutdown, protocol response, or zonal quarantine based on attack path?

* Can transmitter compliance be classified reliably under different identifiers, payload lengths, bit rates, and bus loads?

* Can source-zone localization remain accurate across harness, temperature, supply, transceiver, and termination variations?

* Does quarantine restore critical-message deadlines in unaffected zones within the required outage bound?

* Does staged recovery detect pause-and-resume behavior before full restoration?

* What hardware, CPU, memory, latency, bus-load, and authentication overhead does CARES introduce?

== 13.2 Test Scenarios

[cols="1,1,1,3"]
|===
|**Scenario** | **Attack path / behavior** | **Expected CARES action**

|T1 | Unauthorized low-ID flood crosses diagnostic gateway | Filter, rate-limit, terminate session, preserve downstream zone
|T2 | Compromised installed ECU exceeds allowed ID/rate | Trusted gate blocks TX and records source
|T3 | Direct external node follows normal CAN fault confinement | Confirm impact, assess compliance, neutralize, monitor recovery
|T4 | Custom FPGA ignores standard fault confinement | Localize source zone, quarantine coupler, maintain minimum service
|T5 | Rotating low identifiers target selected lower-priority messages | Detect persistent deadline loss using authorization and per-message timing
|T6 | Attacker pauses during quarantine and resumes after re-entry | Detect during receive-only/restricted stage and immediately re-quarantine
|T7 | Legitimate 75-80% stress load with healthy deadlines | Log and monitor; do not quarantine
|T8 | Physical wiring fault produces errors but no unauthorized dominance | Classify as fault path; apply network fault handling rather than cyber containment
|===

Table 12. Experimental test scenarios and expected CARES responses.

== 13.3 Evaluation Metrics

* Detection latency from first malicious start-of-frame to confirmation.

* Protected-message deadline satisfaction ratio and maximum service outage.

* Attack-path classification accuracy.

* Compliant versus noncompliant transmitter classification accuracy.

* Source-zone localization accuracy and confidence calibration.

* False-positive and false-quarantine rates under stress and fault conditions.

* Containment latency and operational availability of unaffected zones.

* Shadow-plane message latency, authentication overhead, CPU and memory use.

* Recovery duration and probability of detecting persistent or pause-and-resume attacks.

* Added coupler forwarding delay and effect on worst-case CAN response time.

= 14. Limitations and Assumptions

* CARES requires controllable zonal boundaries. A rogue device connected directly to a final unsegmented protected backbone cannot be selectively removed by software.

* Quarantine removes legitimate ECUs together with the attacker; critical functions must be distributed and capable of local fallback.

* The shadow control plane must be independently protected. Simultaneous compromise of the data and control planes is outside the basic assurance claim.

* Physical shorts, destructive wiring attacks, power attacks, or out-of-rating electrical injection require separate fault and tamper controls.

* Optional topology-response localization is an experimental element and requires electrical, EMC, temperature, aging, and harness validation.

* Coupler store-and-forward operation changes the original CAN scheduling model and requires full worst-case timing analysis.

* The system cannot guarantee preservation of every vehicle feature. The target is a clearly defined minimum safety-service set.

* Publication claims of novelty require a formal literature, patent, standard, and product search beyond this engineering draft.

= 15. Conclusion

Arbitration-based CAN denial of service abuses a fundamental access-control property: dominant identifier bits give low numerical identifiers priority, while losing nodes defer without reporting arbitration loss as an error. Repeated malicious frames can therefore deny timely access to safety-critical messages even when the frames are syntactically valid. The principal consequences are deadline violations, stale information, function degradation, cascading communication faults, and difficult recovery.

Industry controls address important parts of the problem. IDPS detects anomalous traffic and network load. Secure transceivers prevent local-host flooding and unauthorized identifiers. Gateways filter managed ingress, authentication protects application data, and redundancy or local fallback preserves selected functions. The unresolved case is a directly connected transmitter already on the protected segment, particularly one that cannot be trusted to follow standard fault confinement.

CARES addresses that gap as an escalation and recovery architecture. The system confirms real availability impact, classifies the attack path, uses the least disruptive applicable control, assesses whether protocol-level containment is trustworthy, localizes a noncompliant physical source, quarantines only with corroborated evidence, maintains minimum safety service through independent coordination, and restores the affected zone through staged validation. The research contribution is therefore not a standalone detector or isolated hardware control, but a complete transition from detection to bounded service recovery.

|===
|**Proposed paper claim****
**CARES is a system-level architecture for restoring and maintaining bounded CAN service availability under arbitration DoS. Its core contribution is evidence-driven escalation from existing boundary and local-host controls to zonal containment and staged recovery when a directly attached transmitter cannot be trusted to self-confine.
|===

= References

[1] ISO, ISO 11898-1:2024, Road vehicles - Controller area network (CAN) - Part 1: Data link layer and physical coding sublayer, 2024.

[2] CAN in Automation, “Arbitration mode,” CiA glossary, terminology sourced from ISO 11898-1:2024, accessed October 2026.

[3] PlaxidityX, “CAN Protection: Intrusion Detection and Prevention System for CAN bus networks,” product information, accessed October 2026.

[4] NXP Semiconductors, “TJA1153A Secure HS-CAN Transceiver,” product information, accessed October 2026.

[5] NXP Semiconductors, “Secure TJA115x CAN Transceiver Family,” product information, accessed October 2026.

[6] AUTOSAR, “Specification of Secure Onboard Communication,” Classic Platform, Release R24-11, 2024.

[7] Texas Instruments, “Introduction to the Controller Area Network (CAN),” application report SLOA101B, revised 2016.

[8] ISO/SAE 21434:2021, Road vehicles - Cybersecurity engineering.

[9] ISO, ISO 11898-2:2024, Road vehicles - Controller area network (CAN) - Part 2: High-speed physical medium attachment sublayer, 2024.

[10] CAN in Automation, “CAN high-speed transmission,” physical-layer overview and nominal bus levels, accessed October 2026.

[11] SAE International, SAE J2411, Single Wire CAN Network for Vehicle Applications, issued 2000.

[12] National Instruments, “Single Wire CAN,” NI-XNET User Manual, updated September 2026.

[13] ISO, ISO 11898-3:2006, Road vehicles - Controller area network (CAN) - Part 3: Low-speed, fault-tolerant, medium-dependent interface, confirmed 2026.

= Appendix A - Industry Control to CARES Integration

[cols="1,1,1,3"]
|===
|**Existing control** | **CARES integration point** | **CARES added behavior**

|CAN IDPS | Supplies normalized identifier, timing, diagnostic, signal, and load evidence | Correlates security events with protected-message deadline failure
|Secure transceiver | Enforces local ECU identifier/rate policy | Uses local containment first and avoids unnecessary zone isolation
|Gateway firewall | Filters and rate-limits managed ingress | Classifies attack path and selects gateway response before physical quarantine
|Secure onboard communication | Provides authenticity and freshness status | Treats authentication failure as evidence while independently protecting bus-time availability
|Zonal network architecture | Provides controllable communication boundaries | Adds source attribution, quarantine criteria, minimum service, and staged recovery
|Local safe-state logic | Maintains selected function during missing communication | Coordinates minimum-service state through the shadow plane
|Vehicle SOC / backend | Receives security events and forensic evidence | Provides structured event state, affected zone, response, and recovery outcome
|===
