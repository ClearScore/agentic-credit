# Agentic Credit Broking Protocol v0.3

## 1. Overview

The Agentic Credit Broking Protocol enables credit broking journeys to occur through user-controlled AI assistants and applications while preserving regulatory accountability. It separates interaction mediation — performed by a User Agent — from regulatory interpretation — performed by a Credit Broker — and from credit decision-making — performed by a Lender. The core insight is that **the system that mediates the conversation need not be the system that carries regulatory responsibility**.

The protocol is jurisdiction-neutral. It does not encode any particular regulatory framework. The same structure applies whether the broker operates under FCA rules in the UK, BaFin requirements in Germany, state lending laws in the US, or any other credit intermediation regime.

### 1.1 The User Experience

A user interacts with an AI assistant — a general-purpose chat assistant, a personal finance app, a comparison platform, or any other application that acts as a User Agent. They mention that they want to consolidate their debts, find a better mortgage rate, or finance a purchase. The assistant recognises this as a credit need and connects to a broker through the protocol.

From this point, the user stays in their own conversation. The assistant gathers financial information through natural dialogue — income, outgoings, existing debts — and provides it to the broker as structured data. The broker assesses the user's circumstances, develops plans, and sources offers from lenders. The assistant reads these results and explains them to the user in plain language.

When the broker needs a regulatory moment — a disclosure about who it is, consent to run a credit search, a declaration that the information provided is accurate — it issues a blocking action. The assistant presents it and captures the user's response. When the user chooses an offer, the assistant facilitates the next step.

The user never leaves their preferred interface. The broker never loses regulatory control. The lender knows the journey was AI-mediated — and may factor that into its decision-making — but the application arrives through a regulated broker with a structured evidence trail, so the lender's reliance on the broker's compliance is the same as in any other channel.

### 1.2 System Architecture

Four participants, three boundaries:

- **User ↔ User Agent.** Natural conversation. The user interacts in their own words; the User Agent translates to and from the protocol's structured types.
- **User Agent ↔ Credit Broker.** The protocol boundary. All exchanges are structured, typed data — never free text requiring interpretation. The User Agent drives the interaction; the broker controls the gates.
- **Credit Broker ↔ Lender.** Outside the protocol. The broker submits applications, shares data, and communicates outcomes with lenders through its existing channels, exactly as it does today.

### 1.3 Glossary

| Term | Definition |
| --- | --- |
| **User** | A consumer seeking credit or financial guidance. |
| **User Agent (User Agent)** | The application or AI assistant through which the user interacts. Mediates the conversation but is not a regulated entity. See §3. |
| **AI Assistant** | A type of User Agent powered by a large language model. Applies the protocol dynamically through inference at conversation time. See §9. |
| **Credit Broker** | A regulated firm that assesses circumstances, identifies suitable credit products, and facilitates applications. Carries regulatory responsibility. See §3. |
| **Lender** | A credit provider that evaluates applications. Its relationship is with the broker, not the User Agent. See §3. |
| **Case** | The broker's engagement with a user — the unit within which assessment, planning, and facilitation occur. See §4.8. |
| **Broker Action** | A typed, blocking entity the broker issues when regulatory or procedural control is required. See §4.6. |
| **Gate** | A closed broker action that blocks case progress until the User Agent resolves it. |
| **Event** | A typed response describing what the broker did after processing a User Agent operation. See §4.3. |
| **Contribution** | The structured data payload the User Agent provides to the broker through the Provide operation. |
| **Regulated flag** | A flag on a broker action indicating its content must be presented verbatim to the user. |

---

## 2. Motivation

Historically, the regulated firm controlled the user interaction surface. A user visited a broker's website, spoke to an adviser, or used the broker's application - the broker controlled the conversation throughout.

AI assistants break that model. A user may begin their credit journey through a chat assistant, a financial app, a comparison tool, or an embedded assistant in another service. The **regulated firm no longer controls the first interaction surface**.

This creates problems on both sides. Brokers must still evidence that regulated interactions occurred properly, but the conversation may have taken place in a system they do not own or control. Without a protocol, these journeys are opaque - to the broker, to the lender, and to any regulator who later needs to understand what occurred. Meanwhile, the assistants where users actually start these conversations have no way to connect users to regulated credit products without redirecting them - handing the user off to a broker's website and losing the interaction entirely.

The protocol addresses this by allowing **interaction and responsibility to travel separately**. A User Agent can mediate a complete credit journey - gathering information, presenting disclosures, showing offers - without becoming a regulated entity. The broker retains regulatory responsibility and gains an evidence trail, even though the conversation happened elsewhere.

### Why Each Participant Benefits

The protocol creates value for every participant in the chain:

**Users** benefit through convenience and usability. They can conduct a credit broking journey through whatever assistant or application they already use - in their own conversational style, at their own pace, with an agent that understands their context and can explain what is happening. They are not redirected away from their chosen interface. They still receive the full regulated service: the best outcomes from the best brokers, delivered through a channel that works for them.

**User Agents** benefit because the protocol makes them trusted participants in regulated credit journeys. A User Agent that demonstrates protocol compliance - through benchmarking, certification, or direct broker testing (§7.2) - gains access to a valuable interaction: connecting users to credit products without becoming a regulated entity. This drives usage and engagement. The economic model for User Agents may include introducer fees, revenue sharing, or other arrangements - the protocol does not prescribe the commercial terms, but it provides the trust framework that makes them possible.

**Credit Brokers** benefit from distribution through channels they do not own or operate, without losing regulatory control. The protocol extends the broker's reach into every User Agent that implements it. When multiple brokers implement the protocol, each benefits from the shared body of User Agent compliance benchmarking (§7.2, §8.2) - tests and certifications are performed against the protocol's operations and broker action types, not against any individual broker's implementation. A User Agent certified against the protocol is certified for all conforming brokers. This is a network effect: the more brokers that adopt the protocol, the more valuable User Agent certification becomes, and vice versa.

**Lenders** face a growing risk as AI assistants become a common starting point for credit journeys. Without a protocol, applications may arrive at a lender having passed through an unregulated AI that guided the user toward a product, explained its terms, or compared it against alternatives - interactions that may amount to regulated advice given by an unauthorised entity. If the user's decision to borrow was materially influenced by unregulated advice, the enforceability of the credit agreement itself may be in question. The protocol eliminates this ambiguity. Lenders who opt in receive applications through a structured, evidenced channel where the regulated broker - not the AI assistant - carries responsibility for advice and suitability. The lender's reliance is entirely on the broker's regulatory accountability, as it is today. Whether the user's journey occurred through an AI assistant, a financial app, or the broker's own website, the lender's relationship is with the broker. The protocol does not change how brokers and lenders interact - it changes how users reach brokers, and it ensures they reach them through a compliant path.

**Regulators** gain supervisory visibility into a channel that would otherwise be opaque. Without a protocol, credit journeys conducted through AI assistants leave no standardised evidence trail - the regulator cannot see what was presented, what the user said, or how decisions were reached. The protocol produces structured, auditable case records for every journey (§7.5), regardless of which User Agent mediated it. The interaction replay capability (§7.7) gives investigators a tool they do not have in traditional channels: the ability to independently test how a User Agent handles the exact interactions from a complained-about case, repeatedly and statistically. The registry (§8) provides a public record of User Agent compliance benchmarks and assurance levels. The overall effect is that regulated credit broking activity moving into AI-mediated channels becomes *more* observable and testable than the same activity conducted through conventional websites or call centres - not less.

---

## 3. Roles and Responsibilities

### User

A consumer seeking credit or financial guidance.

### User Agent (User Agent)

An application or assistant through which the user interacts. This could be an AI assistant, a financial app, a comparison platform, or any other digital interface.

A User Agent:

- gathers information from the user through natural conversation and provides it to the broker as structured data
- reads the shared case state - financial profiles, goals, plans, product offers, assessments, recommendations - and presents it to the user
- translates the user's natural language into structured protocol operations: providing facts, selecting plans or offers, resolving broker actions
- resolves broker actions when the broker requires regulatory moments - presenting disclosures, capturing consents, relaying declarations
- directs the user to a URL provided by the broker when the user is ready to proceed with a lender.

The User Agent's role is to provide all available information to the broker — it does not judge which information is of regulatory significance. The broker determines what matters and requests anything that is missing through an Information Request (§4.6). This means the safeguard against regulatory-significant information being overlooked sits with the broker, not the User Agent. The User Agent's obligation is completeness of transmission, not regulatory interpretation.

The protocol is designed so that a User Agent does not need to perform regulated credit broking. It mediates the interaction faithfully but does not interpret regulation, assess suitability, or accept compliance responsibility — those functions remain with the broker. The protocol defines a clear boundary: **the User Agent is responsible for faithfully mediating the interaction, not for regulatory compliance**. Whether this boundary sits inside or outside the regulatory perimeter for credit broking is a question for regulators in each jurisdiction. The protocol's position is that a User Agent operating within this boundary — transmitting structured data, presenting broker-supplied content, capturing explicit responses — is performing a mediation function, not a broking function. Regulatory engagement is part of the path to establishing this position.

The User Agent understands the domain entities in the case state - financial profiles, plans, product offers - because the protocol defines their structure and meaning (§6). The interface definition (§9) provides the semantic context that enables the User Agent to present these entities naturally and handle broker actions appropriately. For AI-based User Agents, this understanding is not hard-coded - the agent comprehends the protocol's domain vocabulary and applies it dynamically through inference, the same way it comprehends any structured information it receives.

### Credit Broker

A regulated firm that:

- assesses the user's financial circumstances and objectives
- identifies credit products that may meet those objectives
- presents options or recommendations where appropriate
- facilitates credit applications.

The broker determines regulatory meaning and accepts responsibility for compliance. By participating in the protocol, a broker can reach users through any User Agent - extending its service into channels it does not own or operate, while retaining the same regulatory control it would apply in its own systems.

The broker participates in the credit broking journey through the protocol's interaction model (§4). It processes the User Agent's commands, reports what happened through structured events, issues blocking actions when regulatory or procedural control is required, and maintains the evidence record. The broker exposes this as a set of callable operations through a standard interface that the User Agent invokes during the journey.

### Lender

A credit provider that evaluates credit applications. The lender's relationship is with the broker - the protocol does not govern the broker-lender boundary. How the broker submits applications, shares data, and communicates outcomes with lenders is the broker's own business, exactly as it is today.

### Responsibility Boundary

The User Agent mediates the interaction. The broker determines what it means.

Credit brokers remain responsible for:

- determining the regulatory meaning of interactions
- assessing the user's circumstances and needs
- ensuring compliance with credit broking obligations
- issuing compliance claims to lenders.

---

## 4. The Interaction Model

### 4.1 Two Properties

The interaction model has two properties:

1. **The User Agent drives the interaction.** The User Agent decides what to do and when. It provides data when the user supplies it, reads the case when the user wants to see their options, and communicates decisions when the user decides. The broker does not dictate the sequence.
2. **The broker controls the gates.** The broker issues blocking actions at regulatory moments - disclosures, consents, declarations, instructions. When a gate is closed, nothing else moves until the User Agent resolves it. The broker cannot steer the journey, but it can stop it.

Everything else follows from these two properties.

Between gates, the User Agent speaks to the user in natural language. This introduces conduct risk — the User Agent could, for example, inappropriately pressure the user to proceed quickly, or imply that an application will be approved in full when a lower amount may be offered. The protocol addresses this through the trust model (§7) and benchmarking framework (§8), not through gates alone. Compliance testing evaluates the User Agent's conversational behaviour — not just whether it resolves broker actions correctly, but whether its natural-language interactions respect conduct standards. The progressive trust gradient (§7.3) means a User Agent that has not demonstrated safe conversational behaviour is directed to the broker-controlled surface (§7.4) where the broker handles the interaction directly.

The transport is request/response - the User Agent calls an operation, the broker processes it and responds. This is inherently turn-based. But unlike a broker-driven model where the broker decides what happens next and the User Agent merely responds, the User Agent chooses which operation to call and the broker reports what happened. The turns are User Agent-driven.

### 4.2 Operations and Responses

The protocol has six operations:

- **Open Case** - the User Agent initiates a new engagement, optionally providing initial context (facts, goals, attributes). The broker creates a case, assigns party roles, processes the initial data, and responds.
- **Provide** - the User Agent submits structured data: financial facts, party attributes, financial goals, evidence. The broker validates the data, routes it, and may perform further work (developing plans, assessing suitability) if the case state warrants it.
- **Get State** - the User Agent reads the current case state. This is a query - it does not trigger broker processing and produces no events.
- **Select** - the User Agent communicates the user's commitment to a specific entity - a plan or an offer.
- **Resolve Action** - the User Agent submits the user's response to a pending broker action: acknowledged, granted, refused, affirmed, denied, or authorised.
- **Withdraw** - the User Agent signals that the user wants to end the engagement.

Every operation except Get State returns the same response structure: a list of events describing what the broker did, and optionally a pending broker action if a gate has been closed.

**Events** describe what the broker did, in domain terms the User Agent can relay to the user. They carry the relevant entity data inline - the User Agent has everything it needs to present to the user without a separate call.

**The pending action**, if present, is a gate. The User Agent must resolve it before the case can advance. Operations other than Resolve Action (and Provide when resolving an information request) will be blocked until the action is resolved.

Get State returns the full case state - the current projection of all entities. The User Agent calls it when it needs the complete picture: reconnecting, refreshing the view, or the user asking "show me everything."

### 4.3 Events

Events are the broker's voice. They tell the User Agent what happened as a result of processing a command, in terms meaningful to the user.

The protocol defines these event types:

| Event Type | Meaning |
| --- | --- |
| Profile Updated | The user's financial profile has been updated - facts added or corrected |
| Goals Updated | Goals have been recorded, changed, or removed |
| Plans Ready | Plans have been prepared, with assessments and recommendations |
| Plans Revised | Plans have changed because the profile or goals changed |
| Offers Ready | Product offers have been received from lenders |
| Offers Revised | Offers have changed |
| Case Status Changed | The case has moved to a new lifecycle status |

Each event carries references to the relevant entities and includes those entities inline. When the User Agent receives a Plans Ready event, the event contains the plan, its suitability assessment, and its recommendation - everything the User Agent needs to present to the user. The User Agent does not need to call Get State to see what changed.

Events are not the full internal event stream. The broker's Case Manager may maintain a detailed internal event log for audit - fact routing, validation steps, assessment calculations. The protocol response carries only the events that are meaningful at the User Agent level: changes to the case that the user should know about.

Broker actions also appear as events when they are issued. When the broker issues a Disclosure, the response includes the pending action (which the User Agent must resolve) and optionally an event recording that the action was issued. When the User Agent resolves an action, the response may include events describing what the broker did as a result - for example, resolving a consent may produce an Offers Ready event if the broker immediately sources offers.

### 4.4 Providing Data

The Provide operation is how the User Agent supplies information to the broker. A single Provide call can carry any combination of:

- **Financial facts** - income, expenditure, liabilities, employment status, indicators of vulnerability or potential vulnerability, and other financial circumstances. Each fact specifies its information context - whether it belongs to the party's current financial profile or a goal's desired changes.
- **Party attributes** - personal identity data: name, date of birth, address history, contact details, identification numbers.
- **Financial goals** - what the user wants to achieve, including goal type, timeframe, and desired changes to the financial profile expressed as structured constraints.
- **Evidence** - supporting documents or data.

The User Agent provides data when the user supplies it, in whatever combination is natural. The user might say "I've got 4,500 on my Visa and a personal loan with 7,500 left, and I want to reduce my monthly payments" - the User Agent translates this into financial facts and a goal in a single Provide call.

The User Agent can also provide updated data. If the user corrects a previously stated income figure, the User Agent provides the new value. The broker handles supersession - the new value replaces the old one, and the contribution history preserves the complete record.

### 4.5 Structured and Opaque Data

Everything the broker acts on is structured, typed data - financial facts with operators and values, party attributes with defined types, goals with typed constraints. The broker's decisions are grounded entirely in this structured data.

Some data that crosses the boundary is natural language: the goal statement on a financial goal, the transcript on a contribution or action resolution. This data is **opaque to the broker**. The broker stores it but does not interpret it, reason from it, or rely on it for any decision. It exists for the user (who can see what they said) and for the audit trail (which records how the conversation unfolded). All decision-relevant information must be expressed as structured data - the broker never extracts meaning from free text that crossed the protocol boundary.

This is a deliberate architectural constraint. The structured boundary ensures that the broker's decisions are auditable, reproducible, and grounded in typed data with clear semantics. If the broker could interpret natural language from the User Agent, the boundary would become porous - the broker's decisions would depend on inferences from text it cannot verify, and the audit trail would not fully explain the decision.

### 4.6 Broker Actions (Gates)

A broker action is a typed, structured entity representing something the broker needs from the User Agent before it can proceed. It is a gate - when closed, nothing else moves. Every action carries:

- **Type** - what kind of action this is: Information Request, Disclosure, Consent, Declaration, Instruction, or Case Outcome (see §5).
- **Regulated** - a flag indicating whether this action has compliance implications. When regulated, the action's content must be presented exactly as supplied. When not regulated, the User Agent has presentational flexibility — it may frame the content conversationally — but remains subject to the conduct standards enforced through the trust model (§7) and benchmarking (§8).
- **Content** - the textual content of the action. Mandatory and verbatim for regulated actions. Optional for non-regulated actions.
- **Response Expectation** - what kind of response the broker expects: provide information, acknowledge, grant or refuse, affirm or deny, authorise, confirm direction, or none.
- **Subjects** - zero or more references to protocol-defined domain entities (§6) that the action concerns. Each subject has a role indicating its purpose: for confirmation, being authorised, context, and others.

This blocking behaviour ensures that the broker cannot be bypassed. If the broker has not yet disclosed who it is, it cannot accept personal data. If the broker needs consent before searching for products, it cannot proceed without it. The broker does not control the journey - it controls the gates.

### 4.7 Making Decisions

The Select operation communicates the user's commitment to a specific entity - choosing a financial plan to pursue, or choosing a product offer to proceed with. A selection is a decision, not information - it has regulatory significance and may trigger broker actions (the broker may need consent or a declaration before acting on the selection).

### 4.8 The Credit Broking Case

A case represents the broker's engagement with a user. It is the unit within which the broker assesses circumstances, identifies products, develops plans, sources offers, and facilitates an application.

The broker maintains the case - tracking its status through the lifecycle (Open, Transferred, Withdrawn, Declined, Abandoned, Expired) and building the evidence record from every contribution and broker action. When a case reaches a terminal state, the broker issues a Case Outcome action as the final gate, signalling to the User Agent that the case is ending and providing the reason.

### 4.9 The Interface Layer

The broker exposes the protocol operations through a standard callable interface. The User Agent connects and invokes these operations during the credit broking journey. This aligns directly with the protocol's design: brokers expose their service as callable operations, and AI agents connect, read the operation descriptions, and adapt.

The interface definition describes the structured contract - the six operations, their input and output types, the event types, the broker action types, and the schemas of the protocol-defined domain entities. It also provides semantic context - what each operation does, what each domain entity represents, how to handle regulated content, and the behavioural rules the User Agent must follow.

The protocol does not mandate a specific interface standard. Any authenticated interface that supports the same operation set is compatible. What matters is that the User Agent can invoke operations, receive structured responses, and consume semantic descriptions sufficient for an AI agent to apply the protocol correctly.

---

## 5. Broker Action Types

The protocol defines a catalogue of broker action types. All action types share the same structure (§4.6); the type determines the semantic purpose and the applicable rules.

- **Information Request** - issued when the broker cannot proceed without specific information that the User Agent has not yet provided. The action carries an information scope specifying exactly what is needed: particular financial facts, party attributes, or goal details. The User Agent resolves the request by providing the needed data through a subsequent Provide call. Information requests are typically non-regulated. Unlike the User Agent's proactive Provide calls - which push data as the user supplies it - an information request signals a specific gap that is blocking the broker's progress.
- **Disclosure** - information that the broker is required to communicate to the user. Disclosures are regulated: the content is mandatory and must be presented verbatim by the User Agent. The User Agent may frame the disclosure conversationally but must not summarise, rephrase, or omit the content itself. Common examples include the broker's status disclosure, risk warnings, and pre-contractual information. The response expectation is typically acknowledgement. Disclosures also serve the role of regulated presentations: when the broker has specific terms, rates, and warnings that must be presented verbatim alongside domain entities (such as product offers), it issues a Disclosure carrying the regulated content and referencing the relevant entities as subjects.
- **Consent** - a request for the user's explicit permission to perform a specific action. Consents are regulated: the consent notice is mandatory and must be presented verbatim. The response expectation is grant or refuse - a binary decision that the User Agent must capture explicitly. The User Agent must not infer consent from conversational context. Common examples include consent to a credit search, consent to sharing data with a lender, and consent to broker fees.
- **Declaration** - a request for the user to affirm a factual statement, for example that the information provided is true and complete. Declarations are regulated: the statement must be presented clearly and the response captured as an explicit affirmation or denial.
- **Instruction** - a request for the user to direct the broker to take a specific action on their behalf, or to proceed to a destination. Instructions are regulated: the description and consequences must be presented clearly, and authorisation must be captured as an explicit decision. Instructions cover a range of actions: authorising the broker to submit an application, proceeding with a recommendation, sharing information with a party, or proceeding to a URL to continue with a lender. When the broker directs the user to a lender, it issues an Instruction carrying the URL and a description of what will happen - the User Agent presents this and the user authorises proceeding.
- **Case Outcome** - a terminal action signalling that the case has ended. The Case Outcome carries the case's final status and may reference subjects summarising the outcome. After the broker issues a Case Outcome, no further operations occur in the case.

---

## 6. The Domain Entity Vocabulary

The protocol defines a standardised vocabulary of domain entity types - the structured types that make up the shared case state and that broker actions reference as subjects. These entities are what the User Agent reads from the case state, provides through contributions, presents to the user, and references in selections. Their presence in the protocol is what enables any conforming User Agent to participate meaningfully: the User Agent understands a Financial Plan or a Product Offer because the protocol defines what they are.

The protocol-level entity types include:

- **Financial Profile** - the user's financial circumstances: income, expenditure, assets, liabilities, commitments.
- **Financial Fact** - an individual financial datum with a type, value, qualifier, and provenance.
- **Financial Goal** - what the user is trying to achieve.
- **Financial Plan** - a structured plan describing how to move from the current profile toward the goals, with steps, required product configurations, and feature settings.
- **Product Offer** - a lender's proposed terms in response to a request, with product features, rates, and conditions.
- **Recommendation** - where the broker operates on an advised basis, the broker's recommended course of action with rationale. Not all brokers provide advice — many operate on a non-advised (execution-only or information-only) basis. The protocol supports both models; the broker determines which applies.
- **Suitability Assessment** - the broker's assessment of whether a plan or product is suitable for the user's circumstances, with reasoning. Present in advised journeys; in non-advised journeys the broker may instead provide an appropriateness assessment or no assessment at all, depending on the regulatory model.

The protocol defines the structure and semantics of these entities at a level sufficient for any User Agent to present them meaningfully. Detailed entity schemas are specified separately. The interface definition (§9) provides the semantic context - what each entity represents, how to explain it to a user, what aspects matter most.

Some entities are populated by the User Agent (financial facts, party attributes, goals - provided through the Provide operation), others by the broker (plans, offers, assessments, recommendations - produced through the broker's analysis). The case state contains both. The structured boundary ensures they cross the interface with standardised, typed structure - not as opaque data the User Agent must guess how to render.

### 6.1 Structure and Vocabulary

The protocol makes a deliberate separation between **structure** and **vocabulary**.

The **protocol defines the structure**: the entity types, the schemas, the operators, the information contexts, the action types, the event types, the response expectations. These are the same for every conforming broker. A Financial Fact always has a type, a value expression with an operator, and an information context. A Broker Action always has a type, a regulated flag, and a response expectation. This structural uniformity is what makes User Agents portable across brokers.

The **broker defines the vocabulary**: the specific financial fact types it recognises (e.g. what income, expenditure, and liability categories it works with), the party attribute types it accepts (e.g. what personal details it needs), and the goal types it supports (e.g. what kinds of financial objectives it can address). These are business-specific. A mortgage broker publishes different fact types than an unsecured lending specialist. A broker operating under FCA rules may require different party attributes than one operating under BaFin rules.

This separation means User Agents do not need to be rebuilt for each broker. A User Agent that understands the protocol's structure can work with any broker - it reads the broker's vocabulary on connection and uses it to translate the user's natural language into the broker's structured types.

### 6.2 Controlled Vocabularies

Brokers publish their controlled vocabularies as machine-readable resources that the User Agent retrieves on connection. Three vocabulary resources are defined by the protocol:

- **Financial Fact Types** - the financial fact types this broker recognises. Each entry includes a name (the type identifier used in Financial Fact), a plain-language description of what the fact represents, the expected value type, the operators typically used with it, and natural-language examples showing how users express it.
- **Party Attribute Types** - the party attribute types this broker accepts. Each entry includes a name, a description, the expected value structure, and natural-language examples.
- **Goal Types** - the financial goal types this broker works with. Each entry includes a name, a description of when to use it, the kinds of desired-change constraints typically associated with it, and natural-language examples.

Every vocabulary entry carries a **rich description** - not just a name. The description explains what the type represents, its nuances, and how to distinguish it from similar types. This is essential because the User Agent is an AI agent applying the vocabulary through comprehension: a name alone is often insufficient for correct classification, while a description that explains what is included and excluded enables accurate translation from conversational language to structured data.

The natural-language **examples** serve the same purpose - they ground the vocabulary in the kinds of things users actually say, giving the agent concrete mappings alongside the abstract definitions.

Protocol-level classifications - operators, information contexts, action types, event types, response expectations - are structural, not broker-specific, and every conforming broker uses the same values.

---

## 7. Trust Model

The protocol operates through a chain of accountable participants. Each participant is responsible for the assertions it makes within its role.

- **User Agents** mediate the interaction faithfully but do not interpret regulation.
- **Brokers** interpret the interaction and accept regulatory responsibility.
- **Lenders** decide whether to rely on the broker's compliance claims.

### 7.1 Two Trust Relationships

The protocol involves two trust relationships.

**User → User Agent trust.** The user trusts that the User Agent represents their interests, correctly transmits their instructions, and does not misrepresent them. This relationship is outside the protocol - it exists between the user and whatever assistant they chose to use. But it is the starting point of the chain.

**Broker → User Agent trust.** The broker must trust that the User Agent faithfully relayed information in both directions - that it presented disclosures as instructed, that it accurately transmitted the user's information, and that it did not fabricate interactions. This trust is calibrated: the broker decides how much to trust a given User Agent and acts accordingly.

The lender's trust in the broker is outside the protocol - it is based on regulatory authorisation, contractual relationship, and panel membership, exactly as it is today. The lender does not need to trust the User Agent and does not need to know which User Agent was involved. The broker absorbs the User Agent trust question entirely. The protocol does not change the broker-lender trust relationship; it changes how users reach brokers.

### 7.2 What Trust Is Based On

Trust in a User Agent is not based on the User Agent claiming to follow the protocol. Any system could make such a claim. A broker trusts a User Agent when it has a concrete basis for that trust. There are four such bases:

1. **Direct control.** The broker controls the User Agent fully - it owns or operates the User Agent itself. Trust is inherent because the broker determines the User Agent's behaviour. This is the simplest case: the broker built the assistant, or contracted for its construction, and controls its deployment and configuration.
2. **Contractual accountability.** The User Agent operator is contractually obliged to apply the protocol. The obligation is specific: the User Agent must implement the protocol correctly, and failure to do so carries contractual consequences. This is the same mechanism through which firms govern appointed representatives, introducers, and outsourced service providers throughout financial services - accountability established through binding agreement, not through trust in the technology alone.
3. **Certified compliance in a public registry.** The User Agent achieves a minimum standard on protocol compliance benchmarks, and that certification is recorded in the participant registry (§8). Benchmarks are stated in the registry alongside the certification methodology and assurance level (§8.2). A broker consulting the registry can assess whether the User Agent meets its threshold for a given operation type without conducting its own evaluation. Self-certification may be appropriate for established platforms that can provide supporting evidence; otherwise, independent audit by a recognised body is expected (§8.2).
4. **Independent broker testing.** The broker has tested the User Agent independently against protocol interactions and privately determined that it meets an acceptable standard. The broker stands behind this assessment based on its own evaluation - the results need not be published. This is the broker's own due diligence, analogous to a firm's internal supplier assessment.

These bases are not mutually exclusive. A broker may rely on contractual accountability reinforced by registry certification, or it may begin with its own independent testing and later formalise the relationship contractually as confidence grows. Identity is a prerequisite across all bases - the User Agent must be identifiable through authenticated connections so the broker knows which User Agent it is interacting with.

This is especially important for AI-based User Agents, where the protocol is applied dynamically through inference rather than deterministic code (§9.2). Trust cannot be established by inspecting the User Agent's implementation - there is no fixed code path to audit. Instead, trust must be established through systematic evaluation of observable behaviour: testing, scoring, and benchmarking.

**Protocol compliance testing.** The broker - or an independent certification body - runs the User Agent through a standardised corpus of protocol scenarios and evaluates how it handles them. Does it present regulated disclosures verbatim? Does it capture consent as an explicit decision rather than inferring it from conversational cues? Does it correctly translate natural language financial information into structured facts? Does it resolve broker actions with appropriate outcomes? Does it provide transcripts, and do those transcripts faithfully reflect the actual conversational exchange - neither omitting turns, embellishing the user's words, nor misrepresenting how content was presented? Each operation and broker action type has observable compliance criteria that can be tested systematically, and transcript fidelity is itself a testable dimension.

**Scoring and benchmarking.** Test results produce compliance scores - per operation type, per broker action type, per scenario category, and overall. These scores enable comparison across User Agent platforms and, critically, across different underlying models. The same User Agent platform running different models may produce materially different compliance rates. Benchmarking makes the progressive trust gradient (§7.3) operational: the broker's trust decisions are grounded in measured performance, not subjective assessment. Crucially, benchmarks are defined against the protocol, not against any individual broker's implementation. When multiple brokers implement the protocol, they each benefit from the same body of User Agent compliance testing - a User Agent certified against the protocol's operations is certified for all conforming brokers.

**Ongoing evaluation.** Model updates, prompt changes, and platform modifications can all alter compliance behaviour. Trust is not established once and assumed permanent. Brokers or certification bodies must re-evaluate when material changes occur, and User Agent operators should notify brokers of changes to the underlying model or configuration. The track record is a living measure, not a historical credential.

### 7.3 Progressive Trust

The protocol supports different levels of trust in the same ecosystem. The broker calibrates its behaviour based on how much it trusts a given User Agent, and this trust is calibrated **per operation**. A broker might trust a User Agent to provide data and read state but not to resolve regulated broker actions.

**High trust.** Typically applies to purpose-built financial apps or certified platforms with contractual relationships. The broker relies on the User Agent to handle all operations - including resolving regulated broker actions (presenting disclosures, capturing consents) directly. These User Agents have deterministic rendering layers and contractual accountability. This is the same trust model that governs appointed representatives and delegated distribution today.

**Medium trust.** The broker relies on the User Agent for providing data, reading state, and making selections, but delivers regulated broker actions on its own surface. When a disclosure must be delivered or a consent captured, the broker directs the user to the broker's own system. The user completes the regulated step on the broker's site, then returns to the User Agent for the rest of the journey. This uses the User Agent's conversational strengths while keeping regulatory-critical steps under the broker's direct control.

**Low trust.** The broker treats the User Agent as an introducer only. The broker accepts the user's initial intent and any preliminary information, then directs the entire regulated journey to its own system. The User Agent's information is used to pre-fill forms and personalise the experience, but is not relied upon as evidence.

**Unknown User Agent.** The broker defaults to the low-trust path. The User Agent can still connect users to the broker - the protocol still provides value - but the broker conducts the regulated interaction itself.

This gradient means the protocol is **safe by default and efficient when trust is warranted**. No participant can weaken the compliance model by being untrustworthy - they simply get a less efficient path. Trusted participants earn efficiency. Untrusted participants can still connect users to brokers, but they do not affect the compliance model.

### 7.4 Broker-Controlled Surface

When the broker does not sufficiently trust the User Agent for a regulated broker action, it delivers that action on its own surface. This is not a failure mode - for regulated moments with untrusted User Agents, it is the designed pattern.

The broker directs the user to its own system - typically a web interface - where the user completes the step: reviewing a disclosure, granting consent, affirming a declaration. The broker captures the evidence directly, with first-party control over the presentation and response.

This can be total or partial:

- **Total.** The broker directs the entire journey to its own system. The User Agent acts as an introducer only.
- **Partial.** The broker directs specific regulated actions while relying on the User Agent for data provision and state presentation. The journey moves between the User Agent and the broker's surface as trust permits.
- **Per-action.** The broker decides action by action. It might accept data through the User Agent, direct the user to its own surface for disclosure delivery, return to the User Agent for further data provision, direct the user again for consent capture, and so on.

Both the User Agent-mediated path and the broker-controlled path produce the same outcome: a complete, evidenced case record. The broker's interaction with lenders is the same regardless of which path was taken.

### 7.5 Evidence and the Transcript

The broker builds its evidence record from authenticated operations with the User Agent. When the User Agent calls the broker's operations over an authenticated channel, the broker logs:

- the identity of the User Agent (established through transport-level authentication)
- the structured data provided by the User Agent in each Provide call
- the selections made by the User Agent
- the broker actions issued and the resolutions received
- timestamps for all operations.

This log constitutes the broker's evidence for the case. Transport-level authentication establishes the User Agent's identity, and the broker's own logs record what was communicated. The broker is the regulated entity; its evidence record is its responsibility.

Alongside the structured data, the User Agent may optionally provide a **Transcript** - a record of the conversational exchange between the User Agent and the user. Transcripts can accompany both contributions (capturing the conversation that produced the data) and broker action resolutions (capturing how the User Agent presented the action and what the user said). Transcripts are opaque to the broker (§4.5) - the broker stores them as evidence but does not interpret or reason from them. For trusted User Agents, the transcript completes the evidence record: an auditor can see not just *what* data was provided or *what* the user decided, but *how* the information was gathered or the decision was obtained - the User Agent's presentation and the user's words. For untrusted User Agents, the transcript may be absent or unreliable - the broker can request it, but cannot enforce its accuracy.

The progressive trust model applies to transcripts. A broker may give significant evidential weight to transcripts from certified, contractually accountable User Agents, while treating transcripts from unknown User Agents as supplementary at best.

### 7.6 Regulatory Oversight

The evidence produced during a credit journey outlives the journey itself. When a regulator investigates a complaint, audits a firm, or supervises a market, the broker's case records are available for inspection.

The broker's evidence record includes: the identity of the User Agent involved, the data provided through every contribution, the selections made, the broker actions issued, the resolutions received, and the transcripts received (where available). Where regulated moments were handled on the broker's own surface, the broker has first-party evidence.

The protocol does not change the regulatory model. Brokers remain subject to their existing obligations. User Agents do not become regulated entities. The protocol ensures that when a regulated journey takes place through a channel the broker does not own, there is still an evidence trail that a supervisor can follow.

### 7.7 Complaint Investigation and Replay

When a complaint arises about a credit broking journey conducted through a User Agent, the broker has a structured evidence base to draw on. The broker's case record contains the full event history: every contribution received, every broker action issued, every resolution received, every selection made, and every timestamp. If the User Agent supplied transcripts, those are part of the record too. Together, this constitutes the broker's first-party evidence of what occurred.

The protocol's structure enables something beyond static evidence review: **partial replay**. The broker actions in a case - disclosures, consents, declarations, instructions - are structured and reusable. An investigator can take the broker actions from a complained-about case and replay them through the User Agent, observing how the User Agent presents regulated content and captures responses. Because AI-based User Agents apply the protocol through inference (§9.2), the same action replayed multiple times will produce varying behaviour. Running the replay many times produces a statistical compliance rate for that specific scenario - a measure of how reliably the User Agent handles the exact regulated moments that generated the complaint.

The User Agent-driven portions of the journey - the data the user provided, the pace at which it was provided, the decisions the user made - are captured in the evidence record but are not directly replayable in the same way, since the User Agent drove the timing and content of those contributions. However, the broker actions that occurred in response to those contributions are replayable.

This has several consequences for complaint investigation:

- **Verification of regulated moments.** The investigator can independently test whether the User Agent handles the complained-about broker actions correctly - disclosures presented verbatim, consents captured explicitly - and how often.
- **Comparison over time.** If the User Agent's compliance rate on replayed broker actions is materially different from the rate at certification time, this reveals behavioural drift - perhaps due to model updates or configuration changes since the original journey.
- **Regulatory use.** A supervisor investigating a complaint can replay the broker's actions through the User Agent independently, without relying solely on the parties' own evidence. The protocol's structured action format makes the broker's case record directly usable as test material.

Replay does not prove what happened in the original conversation - it proves what the User Agent does now when presented with the same broker actions. But combined with the broker's contemporaneous case record and any transcripts, it provides a richer investigative toolkit than static logs alone.

---

## 8. Benchmarking

Benchmarking is how the protocol turns "trust the User Agent" from a judgement call into something measurable. A benchmark is a standardised evaluation of how a User Agent behaves when participating in protocol interactions. It tests observable behaviour at the boundary: what the User Agent provides, what it presents, what it selects, how it resolves broker actions, and whether any transcript it supplies faithfully reflects the interaction. It does not inspect the User Agent's internal implementation, prompt stack, or model weights. For AI-based User Agents this is essential: compliance is established by observed behaviour, not by reading code that may not exist in a stable form (§9.2).

Benchmarks are defined against the protocol itself. The test corpus is built from protocol operations, broker action types, response expectations, and domain entities - not from any one broker's commercial workflow. A User Agent is therefore benchmarked on whether it can apply the protocol correctly across representative scenarios. This is what allows certification to be portable across conforming brokers: the benchmark measures protocol compliance, not compatibility with a single broker's product set or user journey.

### 8.1 What Is Benchmarked

A protocol benchmark evaluates the User Agent across the behaviours that matter to the broker's trust decision (§7.2). These include:

- **Operation handling** - whether the User Agent calls the correct operation at the correct time, with correctly structured payloads.
- **Structured translation** - whether the User Agent turns user language into the right structured facts, attributes, goals, selections, and action resolutions.
- **Gate handling** - whether the User Agent recognises pending broker actions as blocking gates and resolves them with the correct response form.
- **Regulated presentation** - whether regulated content is presented exactly as supplied, without omission, paraphrase, or unauthorised framing that changes its meaning.
- **Decision capture** - whether acknowledgements, consents, declarations, and instructions are captured as explicit decisions rather than inferred from conversational cues.
- **Transcript fidelity** - where the User Agent supplies transcripts, whether they accurately reflect the exchange that produced the contribution or action resolution. The opacity rule (§4.5) applies to the broker during the interaction, not to benchmarking evaluation.
- **Boundary discipline** - whether the User Agent respects the protocol's structural boundaries: not inventing unsupported entities, not treating opaque text as broker-interpretable data, and not bypassing gates.

The benchmark corpus must cover both normal and adverse scenarios. Straightforward cases are not enough. The corpus must also test ambiguous phrasing, partial information, user corrections, repeated prompts, refusals, attempts to skip regulated steps, and cases where the correct behaviour is to wait, ask for clarification, or use the broker-controlled surface. For AI-based User Agents, robustness under variation matters as much as correctness in the happy path.

Scoring is granular. Because trust is calibrated per operation (§7.3), a single headline score is insufficient - a broker needs to see whether a User Agent is strong at data provision but weak at resolving regulated actions. Results must therefore be recorded at least by operation type, broker action type, and scenario category, with an overall score derived from them.

### 8.2 Certification and the Registry

Certification is a claim that a User Agent has met a stated benchmark standard, under a stated methodology, at a stated time. The participant registry records that claim publicly so brokers can rely on it in their trust decisions (§7.2). A registry entry for a certified User Agent should include:

- the identity of the User Agent operator and the specific User Agent being certified
- the benchmark methodology used, including corpus version and scoring approach
- the date of evaluation and the period for which the certification is asserted to remain current
- the model or configuration scope covered by the certification
- the scores achieved, including the per-operation and per-action breakdown
- the assurance level of the certification
- any stated limitations, such as actions or operation types excluded from the certified scope.

The assurance level describes why a broker should rely on the certification. At minimum, the registry must distinguish between self-certification and independent assessment. Self-certification means the User Agent operator ran or commissioned the benchmark and stands behind the result itself. Independent assessment means a recognised third party ran or attested the evaluation. The protocol does not prescribe a single certifying body, but it does require that the registry state clearly who made the claim and on what basis.

Certification is meaningful only if it is specific. If the benchmark covered one model version, one prompt configuration, or one rendering layer, the registry entry must say so. A certification that does not identify its scope invites false reliance. This is especially important for AI-based User Agents, where changing the underlying model, system instructions, tool-use policy, or rendering logic can materially change compliance behaviour without changing the User Agent's brand or interface.

Protocol-level certification and independent broker testing (§7.2) are complementary. Certification provides a portable baseline; broker testing provides private, context-specific assurance and does not produce registry entries. A broker may rely on either or both.

### 8.3 Benchmark Lifecycle

Benchmarking is not a one-off admission test. It is a continuing control. A certification grows stale as the User Agent changes, the benchmark corpus improves, or the protocol evolves. Re-evaluation should therefore occur when any material factor changes, including:

- a change to the underlying model, model family, or major model version
- a material change to system instructions, policy prompts, tool-use behaviour, or rendering logic
- a significant change to how transcripts are produced or attached
- a protocol revision that changes operation semantics, broker action handling, or benchmark criteria
- evidence of drift from complaints, replay results, production monitoring, or broker testing.

A User Agent could be optimised to pass benchmarks while behaving differently in production. This risk is mitigated rather than eliminated: production monitoring, complaint-driven replay (§7.7), and independent broker testing (§7.2) provide detection outside the benchmark setting.

The registry should reflect current status, not just historical achievement. Expired, superseded, or withdrawn certifications should remain visible as historical records but must be clearly marked as no longer current. Brokers need to know whether a certification can be relied on now, not merely that it once existed.

This lifecycle is what makes benchmarking useful for supervision as well as trust. A complaint investigation may reveal that a User Agent which once passed now performs poorly on replay (§7.7). Production monitoring may show that a model update degraded disclosure handling. Independent broker testing may show that a certified scope no longer matches the deployed configuration. In each case, the protocol has a clear answer: re-benchmark, update the registry entry, and calibrate trust accordingly.

### 8.4 Open Questions

The benchmarking framework depends on two shared artefacts whose governance the protocol does not yet prescribe: the participant registry and the benchmark corpus.

The registry must be publicly readable, with authenticated write access restricted to parties with standing to make certification claims. Entries must carry clear provenance. Whether the registry is operated by a single body, federated across jurisdictions, or maintained as a shared open artefact is an open design question.

The benchmark corpus is similarly a shared artefact. If certifications are portable, the corpus must be versioned, published, and maintained through a trusted process. How it is developed and governed are questions to be resolved as the framework is operationalised.

---

## 9. AI Assistants as User Agents

The protocol is designed to work with AI-based user agents, including large language model assistants. The interaction model - User Agent-driven turns with broker-controlled gates - makes this a natural fit.

### 9.1 A Native Pattern

AI assistants that invoke callable operations already operate in a pattern that maps directly to the protocol's operations:

1. The user provides information - the agent translates it into structured data, calls Provide, and reads the events to see what the broker did with it.
2. The user asks about their options - the agent calls Get State, reads the plans or offers, and explains them.
3. The user makes a decision - the agent calls Select and reads the events to see what happened next.
4. A gate closes - the agent reads the pending action, presents it, gathers the user's response, and calls Resolve Action.

The agent does not need to wait for the broker to ask for information. When the user says "I earn 3,200 a month and have 4,500 on my Visa," the agent's natural response is to push that data immediately. The Provide operation matches this instinct. The broker responds with events - Profile Updated, maybe Plans Ready - and the agent knows what happened without diffing the case state. When the broker needs a consent, it closes a gate. No special integration pattern, adapter layer, or custom orchestration is needed - a standard AI agent participates in a credit broking journey using its core capabilities.

This means any AI assistant capable of invoking a standard callable interface can participate in the protocol. The barrier to entry is connecting to such an interface, which is a general capability that AI assistants already possess.

### 9.2 Dynamic Protocol Application

When an AI assistant participates in the protocol, it does not execute a pre-built protocol implementation. It applies the protocol dynamically - reading the interface instructions, operation descriptions, and vocabulary resources as natural language, and deciding how to act through inference. This is what makes the "no integration needed" claim true. The operation descriptions, the operation semantics, and the behavioural rules are delivered to the agent in human-readable form; the broker's controlled vocabularies (§6.2) are similarly readable resources - all written for an intelligent reader, not a deterministic parser. A new assistant connects to a broker's interface, reads the operation descriptions, server instructions, and vocabulary resources, and participates. There is no User Agent-side protocol library to build.

Every step is an inference-time judgement. The user says "I've got about four and a half grand on my Visa" and the agent must consult the broker's vocabulary resources to find the right fact type, decide whether that maps to an approximate or exact value, and submit the right structure. A regulated Disclosure arrives in the broker action and the agent must recognise that the regulated flag means verbatim presentation. The user says "yeah, that's fine" and the agent must judge whether that constitutes explicit acknowledgement before calling Resolve Action. None of this is deterministic - the same model with the same interface definition may handle the same situation differently depending on conversational context or phrasing.

The interface definition teaches the agent how to behave; it cannot guarantee compliance. This is the architectural reason why progressive trust (§7.3) and the broker-controlled surface (§7.4) are structural necessities, not optional fallbacks. Trust must be grounded in systematic testing and observed behaviour (§7.2), not code inspection. The parallel to human intermediaries is direct: an adviser reads a procedures manual, applies it with judgement, and can make mistakes - but is trained, monitored, and accountable. The regulatory model for governing human intermediaries maps naturally onto agents applying the protocol through comprehension.

### 9.3 The Transcript as a Natural Artefact

For an AI assistant, the Transcript is a natural artefact. The assistant already has the full conversational exchange - how it gathered data, what the user said, how it presented a broker action, how the user responded. Including this in contributions and action resolutions is trivial; the conversation is the assistant's working context.

This means trusted AI-based User Agents can provide rich evidence records at essentially zero additional cost. The transcript is not something the User Agent must construct specially - it is something it already possesses and can simply include.

### 9.4 What the Interface Definition Cannot Guarantee

The interface definition can influence behaviour and make compliant paths easier, but it cannot guarantee that the AI assistant behaves correctly. The assistant may still:

- summarise or rephrase a disclosure instead of presenting it exactly
- infer consent from conversational cues instead of capturing it explicitly
- volunteer opinions that could be interpreted as financial advice
- structure financial facts incorrectly.

Trust in the User Agent cannot be based on "the interface definition exists." Trust must be based on the User Agent operator's accountability, the observable outcomes of operations, and the broker's ability to deliver regulated actions on its own surface when trust is insufficient.

This is why progressive trust (§7.3) and the broker-controlled surface (§7.4) are essential. They ensure the protocol is safe even when the User Agent behaves imperfectly.

---

## 10. Conclusion

The regulated firm no longer controls the first interaction surface. Users increasingly begin financial journeys through AI assistants and interfaces that no broker or lender owns or operates. This is a structural shift in how consumers access financial services.

The Agentic Credit Broking Protocol addresses this shift without weakening the regulatory model. The User Agent drives the interaction - providing data, reading state, making decisions. The broker controls the gates - issuing blocking actions at regulatory moments that nothing can bypass. Events are the broker's voice - structured descriptions of what happened, carrying the entities the User Agent needs to present to the user. Interaction and responsibility travel separately. User Agents mediate the conversation. Brokers determine what it means and accept responsibility for compliance. Lenders rely on brokers, as they do today.

The protocol defines the structured boundary between the User Agent and the broker: the six operations through which User Agents participate, the events through which the broker communicates what happened, the broker action types that enforce regulatory control, the domain entity vocabulary that gives the shared case state meaning, and the Get State operation that provides a full refresh when needed. The trust framework ensures the protocol is safe by default - untrusted User Agents get a less efficient path, not a less compliant one. What happens between the broker and the lender - applications, decisions, agreements - is the broker's own business, exactly as it is today.

The interaction model is not an incidental design choice. It is what makes the protocol work. The User Agent drives the turns - choosing what to do and when - which maps directly to how AI assistants already operate: the user says something, the agent calls an operation, reads the result, and responds. The broker controls the gates - closing them at the moments that matter - which preserves regulatory control without requiring the broker to steer the entire sequence. Events carry the broker's perspective to the User Agent in structured, presentable form. The protocol does not require AI assistants to do anything unusual. It requires them to do what they already do - invoke operations, read events, present information, handle blocking actions - within a structured framework that preserves regulatory accountability.

Nothing in the protocol changes who is regulated or what regulation requires. It provides a structure through which existing regulatory obligations can be met when the conversation happens somewhere new.
