---
title: Terms of Service
weight: 4
---

By using the nxthdr platforms (PeerLab and Saimiris), you agree to comply with these Terms of Service. Violations may result in suspension or termination of your access.

## General Terms

**Account Responsibility**

You are responsible for all activities conducted under your account. Keep your credentials secure and do not share your account with others.

**Research and Educational Use**

The nxthdr platforms are designed for research, education, and legitimate network operations. Commercial use requires prior approval. [Contact us](/docs/reference/contact/) to discuss commercial applications.

**Data Collection and Privacy**

All data generated through the platforms (BGP announcements, measurement results) is made publicly available under the Public Domain Dedication and License. Do not use the platforms if you require private or confidential measurements. See our [privacy policy](/docs/reference/privacy/) for details.

**Fair Use**

Use platform resources responsibly. Excessive or abusive usage that impacts other users or platform stability may result in rate limiting or access suspension.

## Peering Platform Terms

### BGP Security and Routing

PeerLab is designed to support research and education, including studies on BGP security, routing dynamics, and network behavior. You may conduct research on topics such as BGP hijacking, route leaks, and path manipulation using your leased prefixes, provided that your experiments do not disrupt or harm the network, other users, or third-party networks.

**Examples of Acceptable Research**

We encourage legitimate research activities such as:

* Testing BGP security mechanisms (RPKI, BGPsec) using your leased prefixes
* Studying traffic engineering techniques and their effects on path selection
* Analyzing BGP convergence behavior and routing stability
* Investigating anycast routing and load balancing strategies
* Measuring the impact of AS path prepending on traffic patterns
* Validating routing policies and community propagation
* Educational demonstrations of BGP concepts in controlled environments

**Monitoring and Enforcement**

We continuously monitor BGP announcements and traffic patterns to detect policy violations and security incidents. Automated systems are in place to alert on suspicious activity and may automatically remediate issues such as unauthorized prefix announcements, route leaks, or abnormal traffic patterns. This monitoring helps protect the platform, our users, and the broader Internet community.

**Authorized Announcements Only**

You may only announce IPv6 prefixes that have been explicitly leased to you through the nxthdr.dev dashboard. Unauthorized prefix announcements will be filtered automatically.

**No BGP Hijacking**

Do not attempt to announce prefixes you do not own or have authorization to announce. BGP hijacking that targets third-party networks, attempts to impersonate other networks, or intercepts traffic is strictly prohibited. Research on BGP hijacking mechanisms using only your own leased prefixes in a controlled manner is permitted.

**Route Leak Prevention**

Configure your BGP sessions responsibly to prevent route leaks. Do not re-announce routes learned from AS215011 to other networks without explicit coordination. Improper route propagation can disrupt Internet routing and will result in immediate suspension. Controlled experiments studying route leak behavior using your leased prefixes are acceptable if they do not affect production traffic.

**AS Path Manipulation**

AS path prepending and other traffic engineering techniques are permitted for your leased prefixes. You may experiment with AS path manipulation for research purposes, but you must not manipulate AS paths in ways that misrepresent network topology to third parties, attempt to bypass routing policies, or cause harm to other networks.

### Hosted Content

**Legal Compliance**

Any content you host using your leased prefixes must comply with applicable laws and regulations, including but not limited to:

* French law (where nxthdr is registered)
* Laws of the jurisdictions where our infrastructure is located
* International treaties and conventions

**Prohibited Content**

You may not host or distribute:

* Malicious software, viruses, or exploits
* Content that infringes intellectual property rights
* Spam, phishing, or fraudulent content
* Child sexual abuse material or content exploiting minors
* Content promoting violence, terrorism, or illegal activities
* Content that violates privacy or data protection laws

**Harmful Activities**

Do not use your leased prefixes to:

* Launch denial of service attacks (see also [Probing Platform Terms](#probing-platform-terms) for measurement-related restrictions)
* Conduct network scanning or intrusion attempts against third parties
* Operate command and control infrastructure for botnets
* Engage in cryptocurrency mining
* Facilitate illegal activities or services

**Takedown Procedures**

We will respond promptly to legitimate abuse reports and legal requests. Content found in violation will be subject to immediate takedown, and repeat violations will result in permanent access revocation.

## Probing Platform Terms

### Ethical Measurements

**Monitoring and Enforcement**

We continuously monitor measurement traffic to detect abusive patterns and policy violations. Automated systems are in place to alert on suspicious activity such as excessive probing rates, potential denial of service patterns, or unauthorized scanning attempts. These systems may automatically rate limit or suspend accounts engaging in prohibited activities to protect target networks and maintain platform integrity.

**Responsible Probing**

Conduct measurements responsibly and ethically. Follow established best practices for active Internet measurements, including:

* Respect rate limits and credit allocations
* Avoid excessive probing of individual targets
* Respond to abuse complaints promptly
* Provide contact information for your measurements when requested

**No Denial of Service**

Do not use Saimiris to conduct or facilitate denial of service attacks. This includes:

* Flooding targets with excessive probe traffic
* Amplification attacks using reflection techniques
* Coordinated measurement campaigns designed to disrupt services
* Probing critical infrastructure with intent to cause harm

**No Network Intrusion**

Saimiris is for network topology and performance measurements, not security testing of third-party networks. Do not:

* Attempt to exploit vulnerabilities in target systems
* Conduct port scans or service enumeration without authorization
* Probe networks or systems where you lack permission
* Use measurements to gather intelligence for malicious purposes

**No Espionage or Surveillance**

Do not use the platform to:

* Monitor or track individuals without their consent
* Conduct surveillance on behalf of any entity
* Gather competitive intelligence through unauthorized means
* Circumvent privacy protections or anonymity systems

**Target Selection**

Be mindful of your measurement targets. Avoid probing:

* Critical infrastructure (power grids, healthcare, emergency services) without coordination
* Networks that have explicitly opted out of research measurements
* Residential networks excessively
* Systems in jurisdictions where such measurements may be illegal

### Abuse Response

**Handling Complaints**

If you receive an abuse complaint about your measurements:

* Cease probing the affected target immediately
* Respond to the complaint professionally
* Document the nature of your research
* [Contact us](/docs/reference/contact/) if you need assistance

## Enforcement

**Violation Response**

Violations of these terms will be handled based on severity:

* **Minor violations**: Warning and guidance on proper usage
* **Moderate violations**: Temporary suspension and mandatory review
* **Severe violations**: Immediate permanent suspension and potential legal action

**Appeals Process**

If your access is suspended, you may appeal by [contacting us](/docs/reference/contact/) with:

* Explanation of the circumstances
* Steps taken to prevent recurrence
* Commitment to comply with terms going forward

**Cooperation with Authorities**

We will cooperate with law enforcement and legal authorities in cases involving illegal activities, security incidents, or valid legal requests.

## Liability and Disclaimers

**Platform Availability**

The platforms are provided "as is" without guarantees of availability, performance, or fitness for any particular purpose. We may modify, suspend, or discontinue services at any time.

**User Responsibility**

You are solely responsible for your use of the platforms and any consequences arising from your activities. nxthdr is not liable for damages resulting from your use or misuse of the platforms.

**Third-Party Claims**

You agree to indemnify and hold nxthdr harmless from any claims, damages, or expenses arising from your violation of these terms or applicable laws.

## Changes to Terms

We may update these Terms of Service as the platforms evolve. Continued use of the platforms after changes constitutes acceptance of the updated terms. Significant changes will be announced through our website and documentation.

## Contact

If you have questions about these terms or need to report violations, please [contact us](/docs/reference/contact/).
