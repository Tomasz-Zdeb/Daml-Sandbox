# Creatin a private NFT using Daml

`App4PrivateNFT` is the project that acts as a resource to this guide.

* think of what the rights and obbligation around the **Token** are and express them as templates. E.g.
  * **Token template** - should contain:
    * data fields to identify all the parties involved and describe the token: **issuer**, **owner**, **description**, **admin user**, **timestamp of when it was issued**, **previous price that the token was traded on**, **currency that the payments are made in**.
    * behaviour - **a choice** that's offering the token for sale to another party. This could be performed by creating an offer based on a template representing token offer.  
  * **TokenOffer template** -  should contain:
    * all the information that the **Token** itself contains and additionally: **the price** that it's offered for sale and the **new owner** - the Party that the offer is addressed to.
    * behaviour - **a choice** that when accepted creates a new **Token** with updated **owner** and **last price** information.
