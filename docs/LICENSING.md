# byv-agent Licensing and Contribution Policy

**Owner:** BYV Dev Unorganization (BYV)  
**License model:** BYV Restricted Use License (BYV-RUL)  
**Classification:** Proprietary, source-available; not an open-source license  
**Status:** License model approved; final grant wording pending distribution-scope confirmation and legal review.

## 1. Intent

byv-agent is developed and owned by BYV Dev Unorganization (BYV). BYV intends to let users run officially distributed copies of the software while retaining control over the source code and derivative works.

This is not MIT, Apache-2.0, GPL, or another OSI-approved open-source license. Do not describe the project as open source.

## 2. Intended user permissions

The intended license grant is limited to use of an official, unmodified release of byv-agent, subject to the final BYV-RUL terms.

The final license must explicitly state:
- who may use the software (individuals, organizations, or both);
- whether personal and commercial use are permitted;
- which official distribution channels and artifacts are authorized;
- whether redistribution of unmodified binaries is permitted;
- whether use is limited to a particular number of users, devices, or environments, if BYV chooses such limits.

No permission should be implied beyond the final written license grant.

## 3. Restrictions intended by BYV

Unless BYV grants separate written permission, the intended restrictions are:
- no modification, adaptation, or alteration of BYV source code;
- no creation or distribution of derivative versions;
- no redistribution or sublicensing of source code;
- no reverse engineering or circumvention to the extent such restrictions are lawful and enforceable under applicable law;
- no removal or alteration of copyright, trademark, or proprietary notices;
- no representation that a modified or unofficial build is an official BYV release.

The exact treatment of redistribution of unmodified official binaries is pending BYV's final decision. Do not assume it is permitted.

## 4. Contributions

External contributions are **not open at this time**.

- Do not advertise the repository as accepting public pull requests or community contributions.
- Do not accept or merge external code contributions without explicit BYV authorization.
- BYV may invite individual contributors under a separate written agreement, including any required intellectual-property assignment or license grant.
- Internal BYV development remains governed by repository access controls and BYV policy.

## 5. Third-party components

A proprietary license for byv-agent does not override the licenses of third-party dependencies, libraries, assets, fonts, or other included components.

Before adding a dependency:
- identify its exact license and version;
- verify compatibility with the intended distribution model;
- preserve required notices and attribution;
- include third-party notices in releases when required;
- reject dependencies whose terms conflict with BYV's distribution or source-control requirements unless BYV explicitly approves them.

Do not copy code or assets from third-party projects without verifying the applicable permissions.

## 6. Repository and release handling

- Keep the source repository private unless BYV explicitly changes that decision.
- Publish only artifacts and documentation approved by BYV.
- Do not publish source archives, source maps, debug bundles, or build artifacts that expose source unless explicitly authorized.
- Do not commit secrets, signing keys, API credentials, or release tokens.
- Include the final license text and applicable third-party notices with every public release.

## 7. Legal review and finalization

This document records the product and engineering intent; it is not a substitute for jurisdiction-specific legal advice. Before public distribution, BYV should have the final BYV-RUL reviewed by qualified counsel.

The root `LICENSE` file must contain the final approved terms, not this planning note. Do not create a generic MIT/Apache/GPL license as a placeholder.

## 8. Open decisions

- Permitted user classes and whether commercial use is allowed.
- Redistribution of unmodified official binaries.
- Any limits on users/devices/environments.
- Governing law, venue, warranty disclaimer, liability limitation, termination, and other legal terms.

These are intentionally not invented by the implementation agent.
