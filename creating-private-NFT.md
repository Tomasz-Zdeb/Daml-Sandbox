# Creating a private NFT using Daml

`App4PrivateNFT` is the project that acts as a resource to this guide.

* Think of what the rights and obbligation around the **Token** are and express them as templates bundled in a single module (`Token module`) E.g.
  * `Token template` - should contain:
    * data fields to identify all the parties involved and describe the token: **issuer**, **owner**, **description**, **admin user**, **timestamp of when it was issued**, **previous price that the token was traded on**, **currency that the payments are made in**.
    * behaviour - **a choice** that's offering the token for sale to another party. This could be performed by creating an offer based on a template representing token offer.  
  * `TokenOffer template` -  should contain:
    * all the information that the **Token** itself contains and additionally: **the price** that it's offered for sale and the **new owner** - the Party that the offer is addressed to.
    * behaviour - **a choice** that when accepted creates a new **Token** with updated **owner** and **last price** information.

  What above structures are missing is the **payment** that should take place while the offer is accepted. That could be created in separate module.

* Think of what are the obbligations and behaviours around the **payment**. Express them as templates in a module (`Payment module`) E.g.
  * `Payable template` - should contain:
    * data fields to identify all the parties involved and describe the payable - which is a claim that there is a payment to be made, in a way that will allow to perform the payment. These are: **amount to be paid**, **currency of the payment**, **party that pays**, **party that receives the payment**, **some kind of memo - why the payment takes place**.
    * behaviours - a **choice** that allows to claim they execution of payment by creating a **Payment** contract by from side that includes additionally the **transaction ID**.

    Don't forget to include the party that receives the payment in the observers, otherwise since it is not a signatory of the contract nor the controller of the choice, the **Payable** contract would not be visible to it.
  * `PaymentClaim template` - should contain:
    * data fields to identify the **payment execution**, which are basically the same data fields that `Payable template` contains + **transaction ID** to identify the transaction itself.  
    * behaviours - a choice that allows **the receiver** of the payment confirm that the payment took place by creating a **Receipt** contract
  * `Receipt template` - should contain:
    * data fields to identify the **confirmation of payment**, which are basically the same data fields that `PaymentClaim template` contains + a **timestamp** of payment taking place.  
    * behaviours - a choice that seems to be doing nothing at the first glance, but thanks to the default choice's consequence results in contract archivisation.

* Right now `Token module` and `Payment module` have no connection. There should be some itegration between those two performed.
  * First of all `Payment module` should be imported in `Token module`
  * Think of when the payment contract should be created? - the most appropriate place seems to be at the moment when **Token** offer is being accepted.
  * It is also good to return reference to created payable in the offer's choice. (If there is already any value returned, they can be combined in a tuple and then returned).
  * Remember that every transaction should also include some royalty for the **NFT** creator, therefore data field for **royalty rate** should be included in the **Token** contract template.
  * Royalty is just another `Payable` that is issued at the same moment when the payment for the old owner, but with the price based on royalty rate.

* Since this **NFT** is intended to be private, there is a need to implement the permission module: E.g. `UserAdmin module`. There should be two kind of user templates:
  * `Issuer template` - should contain:
    * data fields to represent: **NFT issuer** party and the **admin** party, that will be approving the issuer access, so the **admin** shoud be stated as **signatory** of this contract.
    * behaviours:
      * a choice that will allow the **issuer** to actualy issue - mint the token. With the default behaviour: a contract is being archived after execution of a choice, which in this case would mean that the permission would allow to mint only one token. Therefore changing the default behavior - not to archive the contract after execution of the choice, could be a way to give permission to the issuer once and allow him to mint tokens multiple times. `nonconsuming` keyword is used to do so.
      * a choice that revokes the **issuer** rights, in order to keep control over issuers. The choice can be a blank one, since the only desired outcome is to archive the contract.
  * `Issuer request template` - which should contain
    * the same data fields that the `Issuer template`, and some adnotation that will describe his request E.g. callled `reason`
    * behaviours:
      * a choice that allows **admin** to accept a request, which results in `Issuer` contract creation.
      * a choice that allows **admin** to reject a request. That choice simply archives the contract.

  > in order to controll the owners, let's add **admin** to the controllers of `Issuer` > `AcceptToken` choice, but that does not mean we do not need `Owner template` - [...]
  * `Owner template` - should contain:
    * data fields to represent: **NFT owner** party and the **admin** party, that will be approving the owner access, so the **admin** shoud be stated as **signatory** of this contract.
    * behaviours:
      * a choice to accept a **Token** offer, which will allow **owner** to acquire a **Token**. Since the desired behaviour is to allow **owner** to acquire **Tokens** unless his rights are revoked, let's make this choice: `nonconsuming`.
      * a choice that revokes the **owner** rights, in order to keep control over owners. The choice can be a blank one, since the only desired outcome is to archive the contract.
  * `Owner request template` - should contain:
    * the same data fields that the `Owner template`, and some adnotation that will descripe his request E.g. callled `reason`
    * behaviours:
      * a choice that allows **admin** to accept a request, which results in `Owner` contract creation.
      * a choice that allows **admin** to reject a request. That choice simply archives the contract.
