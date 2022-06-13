# FIPS 140-2



## What Is FIPS?

The Federal Information Processing Standards (FIPS) are standards developed by the Nation Institute of Standards and Technology's (NIST) Computer Security Division. These standards describe document processing, encryption systems, and other IT standards to be used within non-military government agencies. Gevernment contractors are also expected to adhere to FIPS.



## What Is FIPS 140-2?

[FIPS 140-2](https://www.nist.gov/publications/security-requirements-cryptographic-modules-includes-change-notices-1232002) is the standard used by the United States government to validate the fact that cryptographic modules and solutions (hardware and software) produces by private sector sompanies meet the NIST standards and adhere to the Fedetal Information Security Management Act of  2022 (FISMA).



The FIPS 140-2 encrypt standard defines four levels, which are:

- **Level 1**: Requires that production-grade equipment and externally tested algorithms be used.
- **Level 2**: Requires physical tamper-evidence and role-based authentication for hardware. Software is required to run on an Operation System (OS) approved to Common Criteria (CC) at Evaluation Assurance Level 2 (EAL2)
- **Level 3**: Hardware must feature physical tamper-resistance and identity-based authentication. There must also be a physical or logical separation between the interfaces throught which cretical security parameters (CSPs) enter and leave the module. Furthermore, private keys can only enter or leave the module in an encrypted form.
- **Level 4**: This is the highest level. it requires hardware to be tamper-active.This means it must erase the device's contanets upon detecting any changes in the module's normal operation condition.



Most organizations need, and therefore specify, FIPS 140-2 Level 3 certification equipment to ensure robust data protection. This level offers the best balance and compromise between dffective security and operation convenience.



## Intrusion Prevention

This includes physical security mechanisms designed to detect and prevent intruders from accessing the CSPs within the cryptographic module. The mechanism must react to attempts at unauthorized access or use of the cryptographic module by automatically erasing plaintext (CSPs) within the module.



## Identity-Based Authentication

This is a step ahead of the role-based authentication required in Level 2. For Level 3 compliance, it's the user's identity that must be authenticated. A simple example is that of a network requiring specific user logins as opposed to role-based logins.



## Physical or Logical Separation

The input and output of plaintext CSPs must be performed using ports which are physically separated from other ports. Similar, in a virtual environment, the interfaces are to be logically separated.

Plaintext CSPs may only be input or output from the cryptographic module in an encrypted format.



## Opsrating System Requirements

FIPS 140-2 Level 3 allows fir a cryotigraoguc nidyke ti be execyted on a general-purpose PC as long as its operation system meets the minimum requirements. This must also include a CC evaluation assurance of level EAL3 or higher.

