# Medium

1. // R [ISSUE][MEDIUM] Remove github access to external security auditors
    <aside>
    💡 SOLVED
    
    </aside>
    
    `As part of the audit we removed last auditors access to our github.`

2. // R [ISSUE][MEDIUM] Set SPF, DKIM and DMARC to avoid malicious domain related attacks and to minimize the risk for our email addresses (e.g. email marketing platform) to be tagged as spam
    <aside>
    💡 SOLVED
    
    </aside>
    
    `The team activated all security records`

3. // R [ISSUE][MEDIUM] Improve web hardening by security headers
    <aside>
    💡 SOLVED
    
    </aside>
    
    `As part of the audit the team detected and implemented some improvements with regard to Content Security Policies (CSP) and other Security Headers. The team also added all policy violations in their real time incident reporting.`

    Some of the policies implemented so far are:
    * X-Frame-Options
    * X-XSS-Protection
    * Referrer-policy
    * Clickjacking protection
    * CSP X-Frame (frame-ancestors)
    * CSP X-Content-Type nosniff
    * And other CSP directives: script-src, object-src, base-uri, frame-src, etc.

    Note: CSP is still on report-only mode

4. // R [ISSUE][MEDIUM] Disable support to TLS 1.0
    <aside>
    💡 SOLVED
    
    </aside>
    
    `The remote service accepted connections encrypted using TLS 1.0. TLS 1.0 has a number of cryptographic design flaws so we disable support of TLS 1.0.`
    `As of March 31, 2020, Endpoints that aren’t enabled for TLS 1.2 and higher will no longer function properly with major web browsers and major vendors.`

5. // R [ISSUE][MEDIUM] Activate Strict Transport Security
    <aside>
    💡 SOLVED
    
    </aside>
    
    `As part of the audit we activated STS`

6. // R [ISSUE][MEDIUM] Discord bots
    <aside>
    💡 PARTIALLY SOLVED
    
    </aside>
    
    `As part of the audit we improved the enrolment process in Discord to make a bit more difficult for bots and botnets to spam and scam users. We migrated from captchabot (used by most protocols) to wick as well as made some intents with call to action icons. It is hard to mention that Discord has a very big problem with bots due to its API.`
    `In order to have better detection and accounting (audit) mechanisms we implemented an admin log.`

7. // R [ISSUE][MEDIUM] Updated blacklist check for improved alerts
    <aside>
    💡 SOLVED
    
    </aside>
    
    `As part of the audit we created and maintained a blacklist which is used either for real-time "Alerts" as well as for offline manual or script checking against our user beta program to improve our detection and response.`

8. // R [ISSUE][MEDIUM] Brand monitoring for NFT sale
    <aside>
    💡 SOLVED
    
    </aside>
    
    `It is recommended to set alerts based on brand monitoring at least during Prophets NFT sale or important milestones. Recommended to include typo-squatting as well.`
    
    `Update after NFT sale: Despite all our efforts and alerts (which were actually detecting before domains were activated) still some domains were activated without noticing in advanced. Some users were scammed unfortunately but we believe that prevention, detection mechanisms as well as fast communication to users worked pretty well as the situation could have been worse.`


