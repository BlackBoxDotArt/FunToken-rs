# HACKUMENTA token ($MENTA) on NEAR

![Cover](https://github.com/BlackBoxDotArt/MENTA-token/blob/master/hackumentacover.png)

Video presentation:
https://youtu.be/DzT15kyFBFE

[Pesentation slides](https://github.com/BlackBoxDotArt/MENTA-token/blob/master/Hackumenta_for_Hack_the_Rainbow_%20(1).pdf)

[Website](https://hackumenta.webflow.io/)

Instructions for testing the contract:
Use the command line to call the Mint, Burn, Transfer methods 
https://docs.near.org/docs/roles/developer/contracts/test-contracts

Team: 

Sparrow: https://github.com/SLRead 
Back-end dev ; hours contributed []

Felipe Duarte: https://github.com/DuarteFelipe 
Visual communication / Website design / Presentation ; hours contributed []

Dvid: https://github.com/dvid
Front-end dev ; hours contributed []

James Simbouras: https://github.com/jsimbouras
Token economics / communication ; hours contributed []

# Deployed Contract:

# How to test the submission:



# Intro 

Built for the Hack the Rainbow Hackathon! 

Based on Mintable Fungible Token for Rainbow Bridge implementation with JSON serialization.: https://github.com/near/rainbow-bridge-rs/tree/master/mintable-fungible-token

This contract implements $MENTA (Near $MENTA) the token of Hackumenta - A full Art Fair organized peer to peer and funded through a token sale.

Every interaction with $MENTA (mint, burn, transfer) will microfund the Hackumenta Pool making funds vailable for the Art Fair participants. 

Proceeds are completely dedicated to fund the Fair effectively acting as an “Art Pre Sale” - or more cheekly an Initial Art Offering.

The goal of this token is use the token-economic design of TrojanToken.sol https://github.com/TROJANFOUNDATION/Trojan-DAO-Token-Engineering and mirror $MENTA across the Rainbow Bridge so that participants cantake advantage of the lower costs and faster speeds of the Near blockchain. This tokenomic model on Ethereum has been proven through cadCAD simulations, which is why we aim to have it mirrored on NEAR. 

This Fungible Token is the foundation stone upon which real life art experiences are built.



# Tokenomic design

$MENTA effectively creates a web3 crowdfunding mechanism, that would lend itself to several use cases for collective value creation.

$MENTA will be a “social token”, an NEP21 fungible token minted on NEAR, that will be used as a unit of exchange within the HACKUMENTA ecosystem, for the public, participating artists, visitors and collectors.

$MENTA will be used to purchase phygital artwork, unique experiences, and tickets for events, represented as NFTs.

$MENTA will exist, in effect, on a bonding curve, operating autonomously as its own market-maker, without the need for external exchanges. Art collectors and anyone else from the public wishing to participate in the HACKUMENTA experience can mint $MENTA by depositing DAI into the token contract. 

Whenever new $MENTAs are minted, burned or transferred, the token contract automatically applies a community commission, in the form of an amount of tokens that is routed to a community-governed Pool. We implemented this mechanism using 2% on Mint, 3% on Burn, 1% on transfer as an example.  This way, we are keeping the economy within Hackumenta in $MENTA - in effect having $MENTA as the way to account for exchange of value within the system.

The remaining funds are routed to a “liquidity pool: algorithmically guaranteeing a minimum buy-back price in DAI, for anyone who wishes to exit the HACKUMENTA ecosystem, at any time, permissionlessly.

Every interaction with $MENTA funds creativity and improves the experience of HACKUMENTA art fair, distributing benefits amongst a network of participating projects, and cultivating an inclusive open art economy around HACKUMENTA. 

![individual-mint-burn-trojan-simulation](https://github.com/TROJANFOUNDATION/Trojan-DAO-Monetary-System/blob/master/cadCAD_simulation/mint-burn-graph.png)

# Technical overview

* Based on: https://github.com/near/rainbow-bridge-rs/tree/master/mintable-fungible-token

The existing contract Mintable Fungible token https://github.com/near/rainbow-bridge-rs extends the standard fungible token specification contract, with a mintable function, as well as lockable functions providing for mirroring of tokens between NEAR and Ethereum. We chose to base our token contract on this because we knew the next steps would be to use the bridge to mirror the $MENTA token between NEAR and Ethereum. 

To be able to mirror $MENTA token we needed to replicate the tokenomics that will be in place on Ethereum, which requires implementing a percentage fees for mint, burn and transfer events. In our example we implemented a 2% Mint, 3% Burn, and 1% Transfer. This means that there is symmetry between the tokenomics on both blockchains. 

In order to do this, we added calculations based on function to determine percentage fee and then deduct that (and transfer it to the 'hackumenta.testnet' Pool account, before finalising the account balances on every Mint, Burn and Transfer call.


Minting $MENTA

In the Mint scenario, the fraction fee (2%) is calculated. The account that did the minting gets credited with the amount minus the fee. The fee is sent to Hackumenta.testnet pool. 

        // Mint Pool Share is 2%
        let principle = amount;
        let numerator: u128 = 2;
        let denominator: u128 = 100;

        // Find the fraction of tokens for transfer fee
        let pool_amount_mint = numerator * principle / denominator;
         

        // Set account balance to amount minted minus mint fee amount
        account.balance += amount - pool_amount_mint;
        self.total_supply += amount;
        // Set state and send mint fee amount to Hackumenta Pool and refund unused storage deposit
        self.set_account(&owner_id, &account);
        Promise::new("hackumenta.testnet".to_string()).transfer(pool_amount_mint);
        self.refund_storage(initial_storage);
        

Burning $MENTA

Burn is the reverse of that (3% fee). 3 % of the total tokens to burn are sent automatically to  the pool. We decrement the account balance of the “burner”. For tokens burned on NEAR , only equivalent the token amount minus the burn fee are unlocked on the Ethereum side. 


        // Burn Pool Share is 3%
        let principle = amount;
        let numerator: u128 = 1;
        let denominator: u128 = 100;

        // Find the fraction of tokens for transfer fee
        let pool_amount_burn = numerator * principle.0 / denominator;
         

        // Send burn fee amount of tokens
        Promise::new("hackumenta.testnet".to_string()).transfer(pool_amount_burn);

        // Account and supply accounting
        account.balance -= amount.0 - pool_amount_burn;
        self.total_supply -= amount.0;


Transferring $MENTA

In this case the total amount gets decremented from the sender, and the receiver gets the amount transferred minus the 1 % transfer fee, which is again routed to the pool. 

        // Transfer Pool Share is 1%
        let principle = amount;
        let numerator = 1;
        let denominator = 100;

        // Find the fraction of tokens for transfer fee
        let pool_amount_transfer = numerator * principle / denominator;
  

        // Set account balance for sender
        account.balance -= amount;

        // Saving the account back to the state.
        self.set_account(&owner_id, &account);
         
        // Deposit amount to the new owner and save the new account to the state.
        let mut new_account = self.get_account(&new_owner_id);
        // Depositing amount less transfer fee
        new_account.balance += amount - pool_amount_transfer as u128;
        // Send transfer fee amount to Hackumenta Pool
        Promise::new("hackumenta.testnet".to_string()).transfer(pool_amount_transfer);
        // Set account state and refund unused storage deposit
        self.set_account(&new_owner_id, &new_account);

# TODO;

Enabling the Rainbow Bridge functionality:
Enabling the relays between the NEAR clent and the Ethereum clients on the respective networks.  

DAO implementation on NEAR: Our tokenomic system on Ethereum also contains contracts for the launch of a modified Moloch DAO for collective funds management. In the future we would also be looking towards moving the costly transactions of these contracts to NEAR.











# NOTES:


Properties specific to Mintable Fungible Token:


*  - It is an extension of the standard Fungible token that can be found here: https://github.com/near/near-sdk-rs/tree/master/examples/fungible-token


*  - The total balance is not fixed, and is initially 0. When valid proof is submitted the total


*    balance increases, when the tokens are burnt the balance decreases.


*  - The contract permanently memorizes the hashes of the events that were used for


    minting tokens;


Properties inherited from the standard Fungible Token:


*  - The maximum balance value is limited by U128 (2**128 - 1).


*  - JSON calls should pass U128 as a base-10 string. E.g. "100".


*  - The contract optimizes the inner trie structure by hashing account IDs. It will prevent some


*    abuse of deep tries. Shouldn't be an issue, once NEAR clients implement full hashing of keys.


*  - The contract tracks the change in storage before and after the call. If the storage increases,


*    the contract requires the caller of the contract to attach enough deposit to the function call


*    to cover the storage cost. + extended for Buy-In


*    This is done to prevent a denial of service attack on the contract by taking all available storage.


*    If the storage decreases, the contract will issue a refund for the cost of the released storage.


*    The unused tokens from the attached deposit are also refunded, so it's safe to


*    attach more deposit than required.


*  - To prevent the deployed contract from being modified or deleted, it should not have any access


*    keys on its account.

