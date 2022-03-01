# Babylon Finance Security Process

This document describes the Security Process for Babylon Finance, including vulnerability disclosures and its [Bug Bounty program](#bug-bounty-program). We are committed to conduct our Security Process in a professional and civil manner. Public shaming, under-reporting, or misrepresentation of vulnerabilities will not be tolerated.

To submit a finding, please follow the steps outlined in receiving disclosures [section](#receiving-disclosures).

## Responsible Disclosure Standard

Babylon Finance follows a community [standard](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#the-standard) for responsible disclosure in cryptocurrency and related software. This document is a public commitment to following the standard.

This standard provides detailed information for:

- [Initial Contact](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#initial-contact): how to establish initial contact with Babylon Finance's security team.
- [Giving Details](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#giving-details): what details to include with your vulnerability disclosure after having received a response to your initial contact.
- [Setting Dates](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#setting-dates): how to agree on timelines for releasing updates and making details of the issue public.

Any expected deviations and necessary clarifications around the standard are explained in the following sections.

## Receiving Disclosures

### Directly to Babylon

Babylon Finance is committed to working with researchers who submit security vulnerability notifications to us, to resolve those issues on an appropriate timeline, and to perform a coordinated release, giving credit to the reporter if they would so like.

Please submit issues to **all** of the following main points of contact for
security related issues according to the
[initial contact](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#initial-contact)
and [giving details](https://github.com/RD-Crypto-Spec/Responsible-Disclosure#giving-details)
guidelines.

For all security related issues, Babylon Finance has the following main points of contact:

| Contact                | Public key                                                                                                   | Email                             | Key ID                                          |
| ---------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------- | ----------------------------------------------- |
| Security               | [PGP](https://github.com/babylon-finance/security/blob/master/keys/security.asc)                             | security at babylon.finance       | 0323 BF77                                       |

Include all contacts in your communication, PGP encrypted to all parties.

You can also reach out informally over keybase encrypted chat to one or more of the contacts as per the details above.

## Sending Disclosures

In the case where we become aware of security issues affecting other projects that has never affected Babylon Finance, our intention is to inform those projects of security issues on a best effort basis.

In the case where we fix a security issue in Babylon that also affects the following neighboring projects, our intention is to engage in responsible disclosures with them as described in the adopted [standard](https://github.com/RD-Crypto-Spec/Responsible-Disclosure), subject to the deviations described in the deviations [section](#deviations-from-the-standard) of this document.

## Bilateral Responsible Disclosure Agreements

_Babylon does not currently have any established bilateral disclosure agreements._

## Bug Bounty Program (COMING SOON)

Babylon is planning to launch a Bug Bounty program to encourage security researchers to spend time studying the protocol in order to uncover vulnerabilities. We believe these researchers should get fairly compensated for their time and effort, and acknowledged for their valuable contributions.

### Rules

1. Bug has not been publicly disclosed.
2. Vulnerabilities that have been previously submitted by another contributor or already known by the Babylon Finance development team are not eligible for rewards.
3. The size of the bounty payout depends on the assessment of the severity of the exploit. Please refer to the rewards [section](#rewards) below for additional details.
4. Bugs must be reproducible in order for us to verify the vulnerability.
5. Rewards and the validity of bugs are determined by the Babylon Finance security team and any payouts are made at their sole discretion.
6. Terms and conditions of the Bug Bounty program can be changed at any time at the discretion of Babylon Finance.
7. Details of any valid bugs may be shared with complementary protocols utilized in the Babylon Finance ecosystem in order to promote ecosystem cohesion and safety.

### Classifications

- **Critical:** Highly likely to have a material impact on availability, integrity, and/or loss of funds.
- **High:** Likely to have impact on availability, integrity, and/or loss of funds.
- **Medium:** Possible to have an impact on availability, integrity, and/or loss of funds.
- **Low:** Unlikely to have a meaningful impact on availability, integrity, and/or loss of funds.

### Rewards

- TBD

_Paid out in USD equivalent of USDC, DAI, ETH, BABL, or their Garden lp tokens counterparts._

Actual payouts are determined by classifying the vulnerability based on its impact and likelihood to be exploited successfully, as well as the process working with the disclosing security researcher. The rewards above represent the _maximum_ that will be paid out for a disclosure.

### Scope

The scope of the Bug Bounty program spans production smart contracts utilized in the Babylon Finance ecosystem.

#### Repositories

For exact smart contracts, refer to:

- [babylon-finance/protocol](https://github.com/babylon-finance/protocol/)

#### Production Contracts

Babylon Finance adds and removes contracts from Production on an ongoing basis. 

Initial list of smartcontracts:

- [Deployments](https://docs.babylon.finance/protocol/deployments)


Note: Other contracts, outside of the ones mentioned above, might be considered on a case by case basis, please, reach out to the Babylon Finance development team for clarification.

## Deviations from the Standard

The standard describes reporters of vulnerabilities including full details of an issue, in order to reproduce it. This is necessary for instance in the case of an external researcher both demonstrating and proving that there really is a security issue, and that security issue really has the impact that they say it
has - allowing the development team to accurately prioritize and resolve the issue.

In the case of a counterfeiting or fund-stealing bug affecting Babylon Finance, however, we might decide not to include those details with our reports to partners ahead of coordinated release, as long as we are sure that they are not vulnerable.

## More Information

Additional security-related information about the Babylon Finance project including disclosures, signatures and PGP public keys can be found in the [babylon-finance/security](https://github.com/babylon-finance/security) repository.

## Credits

Parts of this document were inspired by [Yearn Finance security policy](https://github.com/yearn/yearn-security/master/SECURITY.md) and [Grin's security policy](https://github.com/mimblewimble/grin/blob/master/SECURITY.md).
