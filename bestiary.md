BEGINNER LEVEL

This guide help you and I hope you, cases are classified by 😈 : evil (never do that), 😶 : maybe we're secure.

It's a melt of behaviour to adopt (human) and process to apply.

This page aim to give a start point for those which are not sure on what they can do about **security** on computer (not **privacy**)


Secure browser:
 - https://us-cert.cisa.gov/publications/securing-your-web-browser
 - extensions:<br />
      - https every where prevent transmission over insecure http channel:
        - ~~hsts~~
        - 😶 Parts of this page are not secure (such as images)
            - 😶 all links of page are SRI (sha256, sha512)
            - 😈 some are not
        - 😈 insecure http channel
        - 😶 all elements on the page over https (that's not enough)
      - 😶 (optionnal) no script if you want control over which scripts are executed (eg if you need communication over insecure channel HTTP)
      - 😈 accept https exception, communication over insecure channel (any)
      - 😈 install root certificat into browser, plz install certificat only for site you wish to visit
      - 😈 passwords without master password
      - 😶 passwords with master password

Secure com:
  - SSL/TLS
    - 😶 Warning when installing new root CA
    - 😶 SSL pinning is a great feature but is not enough (reset after each cache erase) maybe a random [third party checker](https://releasestandard.github.io/hints/thirtPartyChecker), or check on a complete different network (eg vpn)

