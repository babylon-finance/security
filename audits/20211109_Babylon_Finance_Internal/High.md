# High

1. // R [ISSUE][HIGH] Password rotation of team encrypted password vault and FW access activation
    <aside>
    💡 SOLVED
    
    </aside>
    
    A team member was leaving Babylon. As part of the audit we confirmed legit traffic (initially suspicious) using his account from uncommon countries due his was using a new personal contracted private VPN. As a prevention mechanism and following our security risk management policy, we not only removed his access and credentials but also rotated passwords. We confirmed that all 2FA were already activated for all the team members as well as we activated a new firewall on the encrypted password vault to whitelist specific team locations denying access to any other location by default.
    
2. // R [ISSUE][HIGH] Transfer ownerships of remaining smartcontracts to MULTISIG
    <aside>
    💡 SOLVED
    
    </aside>
    
    
    Most of the smartcontracts are owner by the Babylon DAO (Babylon Governor) based on OpenZeppelin Governor which is also Governor Bravo compatible. Other remaining smartcontracts are also decentralized by using Team Multisig (gnosis 2/5). Some contracts during the audit were still not under team multisig or Babylon DAO so as part of the audit we transferred all remaining ownerships to the team multisig. Any new deployment is not only using team multisig but also Defender Admin from OpenZeppelin integrated with gnosis multisig. Now 100% smartcontracts are decentralized (Babylon Governor or Multisig).
    
