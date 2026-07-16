**Automotive cyberseurity**  

Automotive cybersecurity is the practice of protecting all electronic systems used in, or connected to, an automotive vehicle from unauthorized access, manipulation, malicious modification, or damage across the entire vehicle life cycle. Protected systems include electronic devices, data, networks, control algorithms, and software.
There is no system that is 100% secure, with enough given time and computation power any system can be cracked. Security is all about making the system complicated/difficult to crack.

Cryptography is a science and practice of securing information by transforming it into an unreadable format (ciphered text), which can only be understood by those who have right key.
Pillars of cybersecurity 
-	Confidentiality 
-	Data integrity
-	Authenticity 
-	Non - repudiation
-	Availability
  
**Confidentiality**: Ensures that information is not shown to any unauthorized parties or systems. Techniques include encryption, access control and data masking.
**Data integrity**: Guarantees that data stays exact and unaltered unless authorized. It guarantees that the information you send or store stays in its original form and hasn't been tampered by an unauthorized person. Techniques include checksums, hashing and digital signatures.
**Authenticity**: Verifies the identity of users and ensures they have the authority to access specific information or systems. Techniques include passwords, digital signatures, MAC, biometric.
**Non-Repudiation**: Ensures that the sender cannot deny that the sent message was not sent by the sender. Receiver will have the proof that the message was sent by which sender. Techniques include digital signature.
**Availability**: Ensures that authorized users have constant access to data and resources. Techniques include redundancy and failover systems.

<img width="940" height="123" alt="image" src="https://github.com/user-attachments/assets/03095872-a41b-430f-b556-5124cd9258ad" />



**Symmetric cryptography**:  
Symmetric Cryptography is an encryption system where the sender and receiver of a message use same key to encrypt and decrypt messages.
Key is nothing but the logic/mathematical expression/numeric value/shift operation etc.
Symmetric encryption is highly efficient (more speed and less resource needed). Encryption is stronger with the longer key size.
Encryption and decryption keys need to be same, achieving this at the sender and receiver end in difficult. Key exchange must be made securely. Longer key size will take more time and resource for computation.

<img width="780" height="213" alt="image" src="https://github.com/user-attachments/assets/b63051fb-85df-4a35-a6a4-df4d2b869b46" />


 
Different primitives used in symmetric encryption are Data encryption standard (DES), Revest Cipher (RC), Advanced Encryption Standard (AES). Among these AES is most popular primitive used for symmetric encryption.
Adavnaced encryption standard: AES is a highly trusted encryption algorithm which is used to secure data by converting it into an unreadable format. It is developed by the National Institute of Standards and Technology (NIST) in 2001. AES encryption uses different key lengths (128, 192, or 256 bits) to provide strong protection against unauthorized access. AES is a block cipher, which means it takes 128 bits as input and outputs 128 bits of encrypted cipher text. 
Different modes used in symmetric encryption are Electronic Code Book (EBC), Cipher Block Chain (CBC), Counter Mode (CM). 

<img width="448" height="343" alt="image" src="https://github.com/user-attachments/assets/a16059f9-2587-415a-bee1-e3b173afed2f" />
<img width="374" height="347" alt="image" src="https://github.com/user-attachments/assets/7d5baac1-3110-4df8-a624-4eea7ef34aaa" />

       
Electronic Code Book (ECB)                                                            Cipher Block Chain (CBC) 

**Asymmetric Encryption**: 
In asymmetric Key cryptography, there are two keys, also known as key pairs: a public key and a private key. The public key is publicly distributed, anyone can use this public key to encrypt messages, but only the recipient, who holds the corresponding private key, can decrypt those messages. Public-key cryptography is another representation used to refer to Asymmetric Key cryptography. Asymmetric encryption is more secured compared to symmetric encryption.
Exchanging the key publicly is difficult. There are different methods to exchange the keys publicly, one of most used method is Diffie-Hellman. 
Larger data takes more time and resource for computation. This primitive is suitable for lesser data length as it takes more time for encryption and decryption. 
Commonly used algorithms are Rivest Shamir Adleman (RSA), Elliptic Curve Cryptography (ECC), Diffie-Hellman and others.


<img width="762" height="213" alt="image" src="https://github.com/user-attachments/assets/1988d207-fddb-4ddc-a7ea-3f68c6acf825" />


**Hashing**: 
Hashing is a one-way mathematical function that scrambles data into a string of nondescript text that cannot be reversed or decoded.
Hashing is a way to keep sensitive information and data including passwords, messages, and documents secure. Once the content is converted via a hashing algorithm, the resulting value (hash code) is unreadable to humans and extremely difficult to decrypt, even with the help of advanced technology.
There is no key involved in hashing. Different algorithms are SHA 256, SHA 512. Size of the hash output depends on algorithm used. Any size of input data is given for hashing, generated output length will be always fixed to algorithm size used. It follows avalanche effect where a minor change in input will lead to a massive change in the output.


<img width="586" height="184" alt="image" src="https://github.com/user-attachments/assets/9a9b952d-9019-45b2-a88f-9b5d4162569a" />


**Salting** is a process of adding a unique value at the end of the data before hashing. Usually this is done for storing passwords and confirming them.


**Digital signature**: 
A digital signature is a cryptographic technique used to verify the authenticity, integrity, and non-repudiation of a message or document. It ensures that the message was created by a known sender and that it has not been altered during transmission by anyone. Signing is a process where the sender will use its private key to encrypt the hash value generated. Receiver will use sender’s public key to decrypt the signed value and verify the signature.


<img width="704" height="314" alt="image" src="https://github.com/user-attachments/assets/62e70dd8-8ee5-46d4-9e56-6d76a7c566b8" />



**Message Authentication Code (MAC)**:
Message Authentication Code (MAC) is a cryptographic method used for integrity and authentication. For symmetric encryption, cipher-based MAC (CMAC) is commonly used. OEMs may provision keys statically at vehicle production or distribute keys dynamically via a cloud server. Usually in this primitive symmetric encryption is used to generate the CMAC of the data to be sent. 

<img width="792" height="264" alt="image" src="https://github.com/user-attachments/assets/247c8dbe-a72e-49c7-a201-81eb4fa7b1c7" />


 
Summary of all the above cryptographic methods and the supported features is represented in the below table.

<img width="820" height="274" alt="image" src="https://github.com/user-attachments/assets/1d8acb24-7b82-4e44-ba75-349b68163adf" />



 
**Automotive cybersecurity features**
Automotive cybersecurity features form the foundation of vehicle protection. These features work together in a defence-in-depth approach to protect against unauthorized access, data breaches, and malicious control of vehicle systems.

**Secure Boot**: Ensures the ECU starts only trusted software and maintains a chain of trust through the boot sequence.
**Secure Diagnostics & Field Service**: Balances serviceability with authentication, authorization, and logging.
**Secure Debug**: Protects JTAG, SWD, and diagnostic debug interfaces from unauthorized use in production vehicles.
**Secure On-board Communication (SecOC)**: Ensure the authenticity and integrity of data transmitted between ECUs and in the network.
**Secure Flashing**: Prevents unauthorized firmware installation during manufacturing, service, or OTA updates.
**HSM/TPM**: Protects keys, performs secure cryptographic operations, and supports attestation.
**Secure Storage & Memory Protection**: Safeguards secrets, calibration data, and sensitive runtime state.
**Access Control & Authentication**: Ensures only approved users, devices, and services can perform privileged actions.
**Encryption & Cryptography**: Protects confidentiality, integrity, and authenticity of data and software.
**Network Segmentation & Gateway Firewalling**: Limits propagation of compromises across vehicle domains.
**Intrusion Detection**: Detects misuse, anomalies, and signs of compromise during vehicle operation.
**Over The Air (OTA) Security**: Protects remote update delivery, verification, rollback control, and recovery.
**Secure Manufacturing**: Ensures the trust chain begins with secure provisioning and supply chain control.
**Secure Communication Protocols**: Protects vehicle-to-cloud, V2X, and service communications.
**Runtime Integrity Protection**: Detects tampering after boot while software is already executing.

**Secure boot**:
Secure Boot is the foundation of ECU security. It ensures that only authorized and unmodified software runs on automotive processors. The secure boot process sets up a “chain of trust” starting from immutable hardware-based root of trust through each software layer.
Secure boot is a security mechanism by which software is verified for integrity and authenticity before execution. In other words, secure boot allows detection (and may disallow execution) of unauthentic or modified software when booting an embedded device. Secure boot reduces an attacker’s ability to gain persistence in a device. 
Bootloader is a piece of code that executes first when an ECU/system is powered up. Secure boot is a feature that ensures the booting process is secured.
**Root of Trust**:
Immutable bootloader is stored in masked read-only memory (ROM), which cannot be modified even by authorized users. Secure bootloader executes when ECU powers on, this holds cryptographic keys which is used for signature verification.
Boot Process Flow:
1.	Power-On/Reset: Hardware initializes, executes masked ROM bootloader could be Hardware security module (HSM).
2.	Primary bootloader verification: Masked ROM bootloader cryptographically verifies primary bootloader signatures, this uses ECDSA, RSA or other approved algorithms. ECU will end up in fail safe state if the signature verification fails.
3.	Secondary bootloader verification: Primary bootloader verifies secondary bootloader verification and chain of trust validation is continued.
4.	Operating System/Application Verification: Final bootloader verifies OS kernel and applications. Each component's hash measured and stored in secure registers is verified and passed to the next stage.
5.	Runtime verification: Periodic integrity checks during execution is done.

<img width="904" height="152" alt="image" src="https://github.com/user-attachments/assets/208d0bcb-a2fc-4a87-b725-cd1aee1943e3" />


Some of the tests that can be performed to check secure boot:
Boot Integrity Verification: Confirm that the ECU validates digital signature of its firmware before loading. Test by loading valid signed firmware and verify successful boot. As part of negative test load unsigned or tampered firmware to ensure boot is blocked.
Rollback Protection Testing: Verify that the ECU rejects older firmware versions are not permitted to run, preventing downgrade attacks.
Chain of Trust Verification: Check that each state of boot enforces verified signatures, ensuring the integrity of all loaded components.
Fault Injection: Introduce faults in memory or bootloader sequences to ensure the ECU detects tampering and enters a safe state.
Firmware Tampering Attempts: Modify firmware binaries or configuration tables to simulate an attack, confirm the ECU refuses to boot the corrupted firmware.
Replay and Rollback Attack Testing: Attempt flashing /replaying old valid signatures to verify the ECU enforces freshness and secure version control.
Signature Validation: Confirms the correctness of public key or signature file used.
Key Management Verification: Validate that keys used in secure boot are stored securely and cannot be extracted or modified via debugging interfaces.
Voltage tests: Confirms secure boot is robust to supply voltage glitches and works as expected in the operating voltage range.

**Secure Diagnostics**:
Modern vehicle’s ECU manage and monitor various functions. Secure Diagnostics is a critical component of automotive cybersecurity, ensuring that only authorized personnel and authenticated tools can access sensitive vehicle systems and data. This involves implementing strong authentication, cryptographic challenges, and granular access controls to prevent unauthorized modifications or data exfiltration during service operations. By addressing the vulnerability inherent in vehicle maintenance and troubleshooting, Secure Diagnostics protocols protect against potential cyber threats and ensure the integrity of the vehicle's electronic control units (ECUs).
Unified Diagnostic Services (UDS) 0x27 is the security access and 0x29 is the device authentication services. 
Security Access 0x27:
Service 0x27 known as Security Access, is designed to prevent unauthorized access to sensitive operations within an ECU. These operations may include firmware updates, reprogramming, or accessing critical diagnostic functions that could alter the vehicle's behaviour. To safeguard these functions, Service 27 implements a challenge-response mechanism, where the ECU and the diagnostic tool exchange security keys to authenticate the tool's access.
The primary purpose of this service is to control access levels within the ECU and ensure that only authorized tools can perform certain actions. This is crucial for protecting the vehicle against tampering, theft, and unauthorized modifications.
Step-by-Step explanation of seed-key mechanism
1. Requesting Security Access
The process begins with the diagnostic tool sending a Security Access request to the ECU. The tool specifies the access level it needs (e.g., programming or diagnostic level). The request is made by sending a request seed message to the ECU.
Request: The tool sends a message requesting a seed, indicating the desired security level.
The message format typically looks like this: Request Seed Message: [0x27 0x01]
0x27: Identifier for Security Access service
0x01: Sub-function indicating a request for a seed for a certain level (level 1)
2. ECU Responds with a Seed
Upon receiving the request, the ECU generates a random seed value that will be used to authenticate the diagnostic tool. The ECU sends this seed back to the tool. This process acts as a challenge, and the tool must respond correctly to gain access.
Response: The ECU sends back the seed.
Response Seed Message: [0x67 0x01 SEED]
0x67: Positive response identifier for Security Access
0x01: Sub-function confirming seed request.
SEED: Random value generated by the ECU
3. Diagnostic Tool Calculates the Key
The tool now has the seed provided by the ECU. Using an authentication algorithm (typically a proprietary or standardized cryptographic algorithm), the tool generates a key. This key serves as proof that the tool is authorized to access the requested security level. The algorithm must match the one used by the ECU to generate the expected key from the seed.
4. Sending the Key to the ECU
The diagnostic tool sends the calculated key back to the ECU in a send key message:
Send Key Message: [0x27 0x02 KEY]
0x27: Security Access service identifier
0x02: Sub-function indicating a send key operation.
KEY: The key calculated by the diagnostic tool
5. ECU Validates the Key
The ECU receives the key from the tool and validates it by comparing it with the expected key, which it calculates internally using the same seed and algorithm. If the key matches, the ECU grants access to the requested security level. If the key does not match, the ECU denies access and may implement a delay or lockout mechanism after multiple failed attempts to prevent brute-force attacks.
Success Response:
Response: [0x67 0x02]
0x67: Positive response identifier for Security Access
0x02: Sub-function confirming successful key validation.
If the key is incorrect, the ECU will respond with a negative acknowledgment indicating that access has been denied.
6. Performing Authorized Operations
Once access is granted, the diagnostic tool can perform the authorized operations, such as reprogramming the ECU or accessing restricted diagnostic functions. After completing these tasks, the session may revert to a lower security level, or the tool may need to go through the Security Access process again if additional secure functions are required.

<img width="552" height="492" alt="image" src="https://github.com/user-attachments/assets/6cf796c6-40c4-4f5e-9298-1a9c99ddab89" />


Device authentication 0x29:
The UDS service Authentication (0x29) is used by the ECU to identify the client and provide the relevant access to diagnostic resources, based on the client’s ’role’. The access to these diagnostic resources can be limited in time or bound to certain vehicles or ECUs.
The fundamental limitation of 0x27 is that it proves knowledge of a secret but not identity. Once someone knows the algorithm and the master key - whether through legitimate access, reverse engineering, or compromise of one diagnostic tool - every ECU using that algorithm across the entire fleet is exposed. There is no granularity, either the ECU is locked or it is fully unlocked. Additionally, 0x27 has no built-in protection against impersonation. If an attacker can capture and replay a seed-key exchange, or if the algorithm is leaked, any tool can impersonate any legitimate tester.
To overcome the limitations of service 0x27, device authentication 0x29 service was introduced. This verifies who is the client requesting access? what is the role of the client? is the certificate of the client valid? should it allowed to do this specific task? Based on this access is granted to tester perform the designated action after verification is successfully completed.
Service 0x29 works in two modes according to ISO 14229, Asymmetric Proof of Possession and Certificate Exchange (APCE) and Authentication Challenge Response (ACR). Most used mode is APCE for device authentication.
Testing Diagnostic Security:
Diagnostic checks for service 0x27 and 0x29.
Attempt diagnostic access (e.g., writing secure DID) without authorization.
Check if the seed is generated random and not repetitive. if PRNG is used it easy to guess the upcoming seed value compared to TRNG
Verify access control levels enforced for different diagnostic sessions.
Validate 29 service by certificate modification or invalid certificates.

**Secure On-board Communication (SecOC)**:
Considering normal CAN messages which can be easily read and understood by CAN analysers. Also, same messages can be fed into the network by capturing the CAN logs even without analysing. To overcome these attacks SecOC was introduced, where a freshness value and authentication is added to the message before transmission.
Authentication and integrity protection of sensitive data is necessary to be protected for safe functionality of the vehicle systems. This ensures that received data comes from the right ECU and has the correct value. 

<img width="658" height="345" alt="image" src="https://github.com/user-attachments/assets/32b389d2-b4b4-43aa-80eb-0434d0ea19fd" />


The SecOC module as described provides functionality necessary to verify the authenticity and freshness of PDU based communication between ECUs within the vehicle architecture. The approach requires both the sending ECU and the receiving ECU to implement a SecOC module. To provide message freshness, the SecOC module on the sending and receiving side get freshness from an external Freshness Manager for each uniquely identifiable Secured I-PDU. On the sender side, the SecOC module creates a Secured I-PDU by adding authentication information to the outgoing Authentic I-PDU. The authentication information comprises of an Authenticator Message Authentication Code (MAC) and optionally a Freshness Value. Regardless of the Freshness Value is or is not included in the Secure I-PDU payload, the Freshness Value is considered during generation of the Authenticator. When using a Freshness Counter instead of a Timestamp, the Freshness Counter should be incremented by the Freshness Manager prior to providing the authentication information to the receiver side. On the receiver side, the SecOC module checks the freshness and authenticity of the Authentic I-PDU by verifying the authentication information that has been appended by the sending side SecOC module. To verify the authenticity and freshness of an Authentic I-PDU, the Secured I-PDU provided to the receiving side SecOC should be the same Secured I-PDU provided by the sending side SecOC and the receiving side SecOC should have knowledge of the Freshness Value used by the sending side SecOC during creation of the Authenticator.
In normal CAN message transmission, ID, data is transmitted. In secure messages CAN frames data and ID is transmitted along with freshness and authentication as show below.

<img width="403" height="120" alt="image" src="https://github.com/user-attachments/assets/6fb411bd-5949-4919-b7c9-1fd237ffb732" />

 
Generation of MAC, freshness and forming a message from sender ECU and verification of MAC, freshness from receiver ECU is show below.


<img width="623" height="610" alt="image" src="https://github.com/user-attachments/assets/52c26cde-5148-49fc-8af4-d3ce08f44264" />

 
**Freshness value generation**: 
Freshness values are nothing but a monotonously incrementing counter value that keeps changing for every transmission of message to avoid replay attack in the network. 
As per JASPER document, which is followed by most of the OEMs, counter based method is used for generation of freshness value. Consider this document there are typically three counters namely Trip counter, Reset counter and message counter.
Considering that there is Freshness Value Manager (FVM) ECU is available is the network as an individual ECU. Trip counter and reset counter are maintained by FVM ECU. Trip counter increments for every ignition cycle and reset counter increments at fixed time intervals (e.g., 1sec). Reset counter resets to zero when trip counter increments.
Message counter is maintained by both sender ECU individually. For each message transmission this counter value increments. This counter resets to value zero when reset counter increments. By concatenating these three counter values freshness values is generated. Since transmitting this in CAN network takes time usually truncated values are sent. 
Testing Secure on-board communication:
Secure messages can be tested by checking if the generated MAC is transmitted in the bus is correct (even lengths specified by OEM). 
Freshness values can be verified in the transmitted messages. 
Change the MAC key used for calculation of MAC in SecOC and check the system reactions (any DTC, abnormal communication, message rejection/acceptance).
Check for replay attacks are not affecting the system. 
Alter the freshness value and MAC value before transmission and check the system behaviour. 

**Secure Debug**:
Modern vehicles have many interconnected ECUs with debug interfaces that were designed for development and diagnostics. While automotive cybersecurity efforts focus on both wired and wireless attack vectors, physical debug ports including JTAG, UART, and SWD interfaces are potentially significant attack surface.
Debug interfaces are hardware connections used during ECU development, testing, and diagnostics. Common types include:
Joint Test Action Group (JTAG): Standard debugging interface for processors
Universal Asynchronous Receiver Transmitter (UART): Serial communication interface
Serial Wire Debug (SWD): ARM's debugging interface protocol
The security concern is many manufactures ship their ECUs with debug interfaces still enabled or easily accessible, potentially allowing attackers gain physical access and get into:
Extract firmware and reverse-engineer proprietary code.
Modify ECU behaviour or install malicious firmware.
Bypass security controls or safety systems.
Establish persistent access to vehicle networks.
Unlike remote attacks that require sophisticated wireless exploitation, debug port attacks might only require physical access and readily available hardware debugging tools. As vehicles become more connected and autonomous, securing all potential attack vectors becomes increasingly critical.
Best practices followed in industry now are: 
Implement cryptographic authentication for all debug access.
Use password-based or token-based JTAG locking to prevent unauthorized access.
Employ HSM for secure key management and cryptographic operations.
Integrate Secure Boot and firmware encryption to maintain software integrity.
It equally important to implement Secure Life Cycle in an ECU as shown below.

<img width="854" height="203" alt="image" src="https://github.com/user-attachments/assets/8cec68a9-7297-42ee-8484-11b22fe383c1" />

 
Testing secure debug:
Attempt debug access before authentication and verify denial.
Verify credentials are properly validated, check the using different algorithms/mechanism to unlock the ports and access ECU.
Test debug port disabling mechanism which are specified and not specified by OEM or manufactures.
Verify debug activity logging, check if the debug port locking and unlocking is logged.
Certain secure debug implementations have timeout for debug port unlock (e.g., Ignition reset/diagnostic session change). Evaluate those debug timeout mechanisms.

**Over the air (OTA)**:
Over-The-Air (OTA) software updates are now an integral part of many consumer electronics products. Apps on smartphones and tablets are supplied with updates practically every day. Applications as well as the firmware of the devices can thus be updated continuously and easily directly at the end user.
In the automotive sector, software updates "Over-The-Air" have already been implemented in some cases, but the functionality is then usually restricted to certain ECUs or parts of the vehicle software. Due to the increasing complexity of vehicle software and its importance for functionality, the need for software updates is growing - even for safety-relevant applications or functions. 
OTA can be used for entire software update of a target ECU, delta changes update, new features update, bug fixes remotely by avoiding the customer visit the dealer/workshop.


<img width="760" height="315" alt="image" src="https://github.com/user-attachments/assets/47e6851c-8357-4870-b837-688e8eeaa6ce" />

 
OEM or the supplier responsible for OTA updates will manage all the fleet data. Software updates/firmware updates are pushed into vehicle by using the Vehicle Identification Number (VIN). Each car would have a unique VIN number and the data of the vehicle and the software that is running in each vehicle would be maintained by the fleet management. 
For any OTA update to a specific vehicle/number vehicles can be done remotely. When an update in need to a vehicle, campaigns are created by the fleet management and pushed to vehicle via cloud to the target vehicle’s target ECU. Once the vehicle is connected to internet, connectivity module (telematics) receives an update regarding the created campaign. Updated software/patch is downloaded to central storage and notified to the OTA master ECU. This ECU decides which ECU need to be updated with the received files.
Steps involved in updating an automotive ECU via OTA
Download: Downloading the image refers to the transfer of all related and necessary ECU software, data and configuration needed for a complete update of an OTA Target ECU from the backend server to the OTA Master instance.
Installation: Installation refers to the transfer of the new ECU software from the OTA Master ECU to the OTA Target ECU. Since it is not handy to completely transfer the whole ECU software at once to the FOTA Target ECU, this process is realized using data chunks using the UDS services. In addition, the installation process also covers the actual writing of software into the inactive target partition by the program flash driver within the target ECU.
Verification: The verification process is implementation specific and shall assure the correctness of the newly flashed ECU software. This only affects the plain ECU software which was flashed into the respective FOTA Target ECU.
Activation: The activation process describes the actual switch of the ECU boot partition. 
Rollback: During the rollback process all ECU and user data from the previous running software must be restored. After the rollback has finished there must be no difference to the ECU software and user data compared to before the whole update procedure has been started.
UDS services 0x34 request download, 0x36 Transfer data, 0x37 Transfer exit are commonly used to update the target ECU by OTA master ECU. 


<img width="797" height="401" alt="image" src="https://github.com/user-attachments/assets/e97712f3-fde6-4d7d-b2d7-42f124f76248" />

 
Testing OTA as an ECU supplier: 
Checking the correct sequence of OTA flow from OTA master to target ECU. 
Flashing incorrect SW from OTA master to target ECU to check that activation fails and ECU rolls back to old SW. 
Update the older version of the software to check the downgrade protection.
Check rest of functionality of target ECU is completely available during OTA update. 
Interrupting the data transfer or any other flow in the sequence. 
Testing OTA as entire system: 
Create campaigns to a vehicle using the VIN number with valid SW and push to the vehicle, check for update and allow the OTA sequence is complete. Check if ECU is updated with latest SW update. 
Create campaigns with invalid updates (corrupt SW/older SW versions) and check OTA update fails.
Interrupt the OTA sequence in various stages, download – internet connectivity, installation – power disruption and others.
Check rest of functionality of target ECU is completely available during OTA update. 

**Key Update – Secure Hardware Extension (SHE)**:
Hardware Security Module (HSM) or Trusted Platform Module (TPM) provides secure cryptographic key storage and operations. These are dedicated secure processors separate from the main ECU processor. 
The Secure Hardware Extension (SHE) is a specification originally defined by the E-Safety Vehicle Intrusion protected Applications (EVITA) project to provide a standardized lightweight security module within microcontrollers. SHE is specifically designed to perform core cryptographic operations and manage secure storage for automotive applications.
The Hardware Security Module (HSM) is a more advanced and versatile security solution. It is a dedicated co-processor within the microcontroller or as a standalone hardware unit, designed to perform robust cryptographic operations.


<img width="619" height="380" alt="image" src="https://github.com/user-attachments/assets/ed9da2a9-4a09-407c-81da-9bbc153fa168" />

 
SHE is an on-chip extension to any given microcontroller. It is intended to move the control over cryptographic keys from the software domain into the hardware domain and therefore protect those keys from software attacks. SHE provides several memory slots to store keys and provides a feasibility to update these stored keys with additional checks through flags & counter values.
Different Types of Keys Which Will be Stored in the SHE  
SECRET_KEY: stored in the ROM. This will be persisted during chip fabrication by the semiconductor manufacturer.
MASTER_ECU_KEY: stored in the Non-Volatile Memory slot. This key will be persisted by the Owner of component (ECU Owner or OEM) using SHE protocol.
KEY_<n>:  These are SHE keys which will be stored in the Non-Volatile Memory Slot. Here n is an arbitrary number which can be vary from 0 to 20. These keys generally used for Encryption/Decryption, MAC generation /Verification.
BOOT_MAC_KEY: The BOOT_MAC_KEY is used by the secure booting mechanism to verify the authenticity of the software.
BOOT_MAC: The BOOT_MAC is used to store the MAC of the Bootloader of the secure booting mechanism. 
RAM_KEY: This is a plain text key which can be written with the knowledge of the KEY_<n> or in plain text.
Steps involved in updating the keys using SHE protocol:
Step 1: OEM or User will provide the Kauth, Knew, Counter Identifier (CID), Unique Identification Identifier (UID), Flag Identifier (FID) to the Generator and generate M1, M2, M3, M4 & M5 values using the formulas,  
Step 2: Tester sends the M1, M2, M3 values, which holds the actual key to be persisted, as well as further Authentication key details. 
Step 3: If user configures the proof, ECU calculates M4’ & M5’ and sends to the tester. 
Step 4: Tester receives the M4’ & M5’ and compares with M4 & M5 calculated and should be same.
Testing key update:
Update the master keys, MAC key and other keys with valid M1, M2, M3 values and check proper functionality (e.g., after updating MAC key, new MAC key is used in SecOC).
Update keys with invalid CID, FID, UID values and check key update fails.
Check keys are stored cryptographically and cannot be tampered.

**Cybersecurity Testing strategies**:
Security testing aims at
Checking if the functionalities are implemented with proper mitigation of possible vulnerabilities.
Checking if all the interfaces are properly secured.
Checking if the item behaves as expected, also in case of maliciously forged input.
Some of the major threats are,
Remote attackers: might exploit wireless or cellular connections to access the vehicle remotely, taking control even of critical systems such as the engine or braking systems.
Vehicle theft: Vulnerabilities in security systems and electronic keys might be exploited for vehicle theft or unauthorized door opening.
Infotainment intrusions: Internet-connected infotainment systems might be subject to attacks to compromise driver's privacy by stealing sensitive data or tracking driving habits.
Data manipulation: Attackers might modify data from vehicle sensors, such as speed or orientation data, leading to unexpected or dangerous behaviour.
Attacks on Bluetooth or Wi-Fi: Attackers might exploit vulnerabilities in the Bluetooth connection to gain access to the vehicle and steal personal information or take control of connected devices.
Attacks on OBD Port: OBD port, typically used for vehicle diagnostics at the service, might be exploited by attackers to gain access to vehicle systems and change its behaviour.
Attacks on ECU: Attackers can specifically aim an ECU and can attack ECU that might affect vehicle functions, or to intentional tampering.
Navigation system attacks: Attackers might interfere with navigation and positioning systems, causing incorrect directions or loss of accuracy, even spoofing attack can also be made.
Attacks on connected infrastructure: Connected Road infrastructure may be subject to attacks that affect road signs or cause security problems by deceiving a multiplicity of vehicles.
Vehicle fleet attacks: Corporate vehicle fleets might be targeted by attacks aimed to affect multiple vehicles simultaneously, compromising business operations.

**Penetration testing**: 
Penetration testing, also known as ethical hacking, is a crucial component in the field of automotive cybersecurity. The primary purpose of penetration testing is to identify and mitigate cybersecurity vulnerabilities that may not have been considered during the design phase. This process involves simulating cyberattacks to expose weaknesses in the system, allowing for their remediation before they can be exploited by attackers.
As discussed in the testing strategies below table represents some of the entry points where attacks can happen,


<img width="720" height="870" alt="image" src="https://github.com/user-attachments/assets/6d0c3d9c-6809-4425-a92e-eebd5b429e01" />


**Fuzz**:
Fuzz testing, or fuzzing is a software testing technique where random, unexpected, or invalid data is injected into a system to uncover security vulnerabilities, bugs, and system weaknesses before they can be exploited in the real world. As vehicles become increasingly software-driven and connected, fuzz testing helps manufacturers identify and resolve issues that could compromise safety and cybersecurity.
By supplying a large volume of random or malformed inputs, fuzz testing aims to crash the system or trigger errors, thereby revealing potential weaknesses that could be exploited by attackers.

**Fuzz testing tools**:
Fuzz testing tools are automated testing tools that generate a variety of invalid, unexpected, or random data inputs. These inputs are designed to challenge the software's ability to manage unexpected scenarios. 
Fuzz testing tools should monitor the generated inputs are injected into the target application, and the system is monitored for exceptions, crashes, or any abnormal behaviour. This process helps identify areas of the code that may be vulnerable to attacks. 
Fuzz testing tools should analyse the results for vulnerability if found, the fuzz testing platform can help determine the root cause, allowing developers to patch the identified flaws.

**Threat Analysis and Risk Assessment (TARA)**:
Threat Analysis and Risk Assessment (TARA) is a systematic process that identifies potential cybersecurity threats in vehicle systems, evaluates their likelihood, and assesses their impact on safety and functionality. The goal of TARA is to pinpoint vulnerabilities within a vehicle’s electronic architecture, communication networks, and control units, and to develop strategies that reduce the risk of exploitation.
In the context of automotive cybersecurity, TARA plays a critical role in protecting key vehicle components, such as electronic control units (ECUs), sensors, and communication interfaces, from cyberattacks. TARA focuses on potential attack vectors, threat agents, and the systems most susceptible to hacking, allowing manufacturers to address security issues before they lead to real-world breaches.
ISO/SAE 21434 Clause 15 defines the TARA process in five distinct steps. Each step builds on the outputs of the earlier one, creating a traceable chain from asset identification to risk treatment.
Step 1: Asset Identification. The first step is to find the cybersecurity-relevant assets within the system under analysis. An asset is anything that has value and requires protection - this includes data (firmware, calibration data, cryptographic keys, personal data), functions (diagnostic services, OTA update mechanisms, safety-critical control functions), and interfaces (CAN bus connections, Ethernet ports, Bluetooth, Wi-Fi, OBD-II).
Step 2: Threat Identification. For each identified asset, potential threats need be identified - actions or events that could compromise the asset's cybersecurity properties. Threat identification can be performed using structured methodologies such as STRIDE, attack trees, or catalogue-based approaches. A comprehensive threat library is critical here - experienced teams keep catalogues of hundreds of known automotive threats mapped to specific asset types and architectures.
Step 3: Impact Assessment. Each identified threat is assessed for its potential impact if successfully exploited. ISO 21434 defines four impact categories: safety (potential for physical harm), financial (economic loss to stakeholders), operational (disruption of vehicle functions), and privacy (exposure of personal data). Each category is rated on a scale — typically negligible, moderate, major, or severe. The overall impact rating is the highest rating across all four categories.
Step 4: Attack Feasibility Assessment. This step evaluates how feasible it is for an attacker to conduct each identified threat. ISO/SAE 21434 provides several approaches for assessing attack feasibility, with the attack potential-based approach being the most used. This evaluates factors such as elapsed time (how long the attack takes), specialist expertise required, knowledge of the target, window of opportunity, and equipment needed. The result is a feasibility rating: low, medium, high, or extremely high.
Step 5: Risk Determination and Treatment. The last step combines the impact rating and Attack Feasibility rating to decide the overall risk level for each threat. ISO/SAE 21434 defines risk levels from 1 (lowest) to 5 (highest), decided by a risk matrix. For each risk, the team then selects a Risk Treatment decision: avoid (drop the threat source), reduce (implement cybersecurity controls), share (transfer risk to another party), or accept (acknowledge and document the residual risk). Risk reduction decisions generate Cybersecurity Goals, which flow into cybersecurity requirements for the development phase.
Example:


<img width="940" height="238" alt="image" src="https://github.com/user-attachments/assets/da952d5a-5f3a-4762-b342-34e2a61504bc" />


Security implementation and testing can be categorized as shown below.


<img width="591" height="320" alt="image" src="https://github.com/user-attachments/assets/986c37cc-1769-43b3-83e4-180de326e4e4" />


**Intrusion Detection System (IDS)**:
Automotive Intrusion Detection Systems (IDS) are specialized security solutions designed to monitor and protect vehicles from unauthorized access and cyber threats. These systems analyse network traffic and activities within the vehicle's components to identify potential security breaches.
In practice, IDS implementations are typically deployed at two distinct levels:
Network-Based IDS (NIDS) and Host-Based IDS (HIDS).
Network IDS are deployed in Central gateway ECU/Domain Controllers which monitors Inter-ECU network traffics and detects faults like injection of malicious inputs, spoofing, DoS, replay, fuzzing. NIDS is designed to monitor traffic traversing the in-vehicle communication. It observes message exchanges across networks such as CAN, CAN FD, LIN, and Automotive Ethernet, analysing traffic patterns in real time to detect anomalies or known attack signatures Since the detection system is implemented at centralised places security is moderate.
Host IDS are deployed in individual ECUs that monitors system calls, processes, file integrity and detects privilege escalation, malware, rootkits. NIDS is designed particularly for those running on rich operating systems like Linux, Android, or QNX. Rather than monitoring network packets, it examines system calls, process behaviour, file integrity, and runtime memory to detect compromise from within. Since the detection system is implemented at individual ECU level security is higher.
IDS implementation in an automotive system is as show below.



<img width="720" height="405" alt="image" src="https://github.com/user-attachments/assets/7290a19d-f08c-4e3d-a937-9e4d4ded7f2c" />






