# initial draft
Bank will start a PoC plan with crypto vendor heres are the thing we want to test

- Crypto Transfer within Private network
- Tokenization (tokenized Deposit)
- Wallet management
- Client wallet in custody in bank main wallet and bank main wallet in custody in vendor.
- Crypto Trading pair on Test net
- Able to accept FIX for trading from OMS
- Able to accept API calls
- Able to stream On chain data back (test net and private net)
- We are able to reconcile the data on chain with our mock core banking system
- We are able to deploy our own Data Oracle to onchain smart contract

For private net 3 nodes within the test env and bank will have integration layer connecting and orchestrating

for test net try setup a node connect to test net to perform dummy trade

# comment 2026-03-04
 - remove the trading api and use case from the poc-analysis.md, it is not pioritized
 - the Bank block chain layer instead of staying in bank GCP sandbox lets move to the Fireblocks GCP sandbox as business is expecting our private net is in custody and fully managed by the vendor
 - expand a bit on the custody hirachy, currently we have the omnibus wallet but and have bank operation wallets and client wallets, we want to have other hirachy model, like each client has their own wallet, each BU has their own wallet
 - remove 2.4 as Trading is depioritized
 - 2.3 this diagram is good, but want to also add 1 more flowdiagram describing how money are move between account. for example like CUST ACCT fiat moves to RESERVE account under corebanking but at the same time there is a mirror account on core banking for customer mirroring the virtual account on the customer wallet. basically how the fiat and token flows are in the process of minting
 - based on what 2.3 has add the sequence diagram for the following scenrio FX, Transaction, Burning
 - in the test item lets remove Goal 3 only keep Goal 1 and Goal2. making sure Goal 1 also include wallet management as this will be required for tokenized Deposit but its on a private network.
 - remove all things related to FIX-to-REST.
 - also add computing resources that we will need on GCP like GCVE, repository, Container, K8s, Database, VPC etc...
 - separate the document into different business usecase no longer in the same document.