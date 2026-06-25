# Spec Glossary

Definitions of every term used in a spec, kept verbatim from the two sources. Read this when you need the precise meaning of a requirement type or term while authoring.

- **[Wiegers]** — Wiegers & Beatty, *Software Requirements*, 3rd Edition, Ch. 1 Table 1-1.
- **[29148]** — ISO/IEC/IEEE 29148:2011, Clause 4.1 (term numbers shown).

---

## Requirement types [Wiegers, Table 1-1]

- **Business requirement:** "A high-level business objective of the organization that builds a product or of a customer who procures it."
- **Business rule:** "A policy, guideline, standard, or regulation that defines or constrains some aspect of the business. Not a software requirement in itself, but the origin of several types of software requirements."
- **Constraint:** "A restriction that is imposed on the choices available to the developer for the design and construction of a product."
- **External interface requirement:** "A description of a connection between a software system and a user, another software system, or a hardware device."
- **Feature:** "One or more logically related system capabilities that provide value to a user and are described by a set of functional requirements."
- **Functional requirement:** "A description of a behavior that a system will exhibit under specific conditions."
- **Nonfunctional requirement:** "A description of a property or characteristic that a system must exhibit or a constraint that it must respect."
- **Quality attribute:** "A kind of nonfunctional requirement that describes a service or performance characteristic of a product."
- **System requirement:** "A top-level requirement for a product that contains multiple subsystems, which could be all software or software and hardware."
- **User requirement:** "A goal or task that specific classes of users must be able to perform with a system, or a desired product attribute."

---

## Standard terms [29148, Clause 4.1]

- **4.1.5 condition:** "measurable qualitative or quantitative attribute that is stipulated for a requirement."
- **4.1.6 constraint:** "externally imposed limitation on system requirements, design, or implementation or on the process used to develop or modify a system." NOTE: "A constraint is a factor that is imposed on the solution by force or compulsion and may limit or modify the design."
- **4.1.8 derived requirement:** "requirement deduced or inferred from the collection and organization of requirements into a particular system configuration and solution."
- **4.1.13 mode:** "set of related features or functional capabilities of a product."
- **4.1.17 requirement:** "statement which translates or expresses a need and its associated constraints and conditions." NOTE: "Requirements exist at different tiers and express the need in high-level form (e.g. software component requirement)."
- **4.1.18 requirements elicitation:** "process through which the acquirer and the suppliers of a system discover, review, articulate, understand, and document the requirements on the system and the life cycle processes."
- **4.1.19 requirements engineering:** "interdisciplinary function that mediates between the domains of the acquirer and supplier to establish and maintain the requirements to be met by the system, software or service of interest."
- **4.1.20 requirements management:** "activities that ensure requirements are identified, documented, maintained, communicated and traced throughout the life cycle."
- **4.1.22 requirements validation:** "confirmation by examination that requirements (individually and as a set) define the right system as intended by the stakeholders." ("The right system has been built.")
- **4.1.23 requirements verification:** "confirmation by examination that requirements (individually and as a set) are well formed." ("The system has been built right.")
- **4.1.24 software requirements specification (SRS):** "structured collection of the requirements (functions, performance, design constraints, and attributes) of the software and its external interfaces."
- **4.1.25 stakeholder:** "individual or organization having a right, share, claim, or interest in a system or in its possession of characteristics that meet their needs and expectations." (Includes end users, supporters, developers, producers, trainers, maintainers, disposers, acquirers, customers, operators, suppliers, accreditors, regulatory bodies.)
- **4.1.26 state:** "condition that characterizes the behaviour of a function/subfunction or element at a point in time."
- **4.1.29 system requirements specification (SyRS):** "structured collection of the requirements (functions, performance, design constraints, and attributes) of the system and its operational environments and external interfaces."

---

## Measures & related acronyms [29148]

- **MOP — Measures of Performance:** quantitative measures of how well the system performs functions; verifiable individually.
- **TPM — Technical Performance Measures:** technical measures tracked during development to confirm progress toward performance requirements.
- **HSI — Human Systems Integration:** incorporation of human factors (safety, performance, usability, well-being) into requirements definition.
- **TBD / TBS / TBR — To Be Defined / To Be Specified / To Be Resolved:** placeholder clauses. A complete requirement set contains none at baseline.
- **OpsCon (Operational Concept, Annex A, normative):** the specific system-of-interest from the user's viewpoint.
- **ConOps (Concept of Operations, Annex B, informative):** organization-level concept; leadership's intended way of operating; may treat systems as "black boxes."

---

## Requirement categories [29148, §5.2.8.2]

- **Functional:** "describe the system or system element functions or tasks to be performed."
- **Performance:** "defines the extent or how well, and under what conditions, a function or task is to be performed... quantitative requirements of system performance and are verifiable individually."
- **Usability / Quality-in-Use:** "provide the basis for the design and evaluation of systems to meet the user needs."
- **Interface:** "definition of how the system is required to interact with external systems (external interface), or how system elements within the system, including human elements, interact (internal interface)."
- **Design Constraints:** "limits the options open to a designer... by imposing immovable boundaries and limits."
- **Process requirements:** stakeholder/acquirer requirements imposed through contract or statement of work; include compliance with laws, administrative requirements, work directives, mandated design methods.
- **Non-Functional:** "specify requirements under which the system is required to operate or exist or system properties... how a system is supposed to be." Includes **Quality requirements** (the "ilities": transportability, survivability, flexibility, portability, reusability, reliability, maintainability, security) and **Human Factors requirements** (safety, performance, effectiveness, efficiency, reliability, maintainability, health, well-being, satisfaction).

## Quality attribute lists [Wiegers, Ch. 14]

- **External attributes** (users care most): Availability, Installability, Integrity, Interoperability, Performance, Reliability, Robustness ("the ability of the system to function correctly despite invalid inputs"), Safety, Security, Usability.
- **Internal attributes** (developers/maintainers care most): Efficiency, Modifiability, Portability, Reusability, Scalability, Verifiability (testability).
