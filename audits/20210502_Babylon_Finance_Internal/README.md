# Internal Security Audit v1

Created: May 10, 2021 7:49 PM
Last Edited Time: March 3, 2022 8:13 PM
Status: Completed

**Code date: since 2021 May the 7th to Team Alpha release**

last commit ID: 4cd7359ce7db0f297d40681ab4ef92daf763b0cc (alpha tag) 

**Findings**

All findings are divided in pages depending on their severity. The order goes from the most important to the less important issues.

| Severity                |     #findings                                                                                                   | Report   |                    
| ---------------------- | ---------------------------------------------------------------- | --------------------------------- |
| Criticals               | #0                            | [**Criticals**]([NO CRITICALS FOUND])  | 
| High              | #13                            | [**High**](./High.md)    |
| Medium             | #19                            | [**Medium**](./Medium.md) |
| Low            | #11                            | [**Low**](./Low.md) |
| Informative            | #8                            | [**Informative**](./Informative.md) |



**(click on the title of each page to see all the issues classified in that category)**

[**Critical**](./README.md) 

**Critical** severity issues need to be fixed as soon as possible. They are usually allowing a drastic reduction of the balance of the system or their users' without consent, they put in danger the whole integrity of the system or they can even stuck the whole system stopping it from normal and expected functioning. They damage and impact the future of the whole system and its brand reputation.

[**High**](./High.md) 

**High** severity issues will probably bring problems and should be fixed. They are potentially allowing a reduction of the balance of the system or their users' without consent, they potentially put in danger the  integrity of the system or they can potentially stuck the  system stopping it from normal and expected functioning. They potentially damage the company brand reputation and put in high risk the feasibility and future of the system.

[**Medium**](./Medium.md)

**Medium** severity issues could potentially bring problems and should eventually be fixed.

[**Low**](./Low.md)

**Low** severity issues are minor details and warnings that can remain unfixed but should preferably be fixed at some point in the future.

[**Informative**](./Informative.md)

**Informative** severity are raised in order all the team is aware of this behavior of the code as they eventually brings new discussions and decisions to be taken.

**Files reviewed so far:**

BabController.sol

GardenValuer.sol

IshtarGate.sol

Treasury.sol

PriceOracle.sol

gardens/Garden.sol

gardens/GardenFactory.sol

gardens/GardenNFT.sol

strategies/Strategy.sol

strategies/StrategyFactory.sol

strategies/StrategyNFT.sol

token/BABLToken.sol

token/TimeLockedToken.sol

token/VoteToken.sol

token/RewardsDistributor.sol