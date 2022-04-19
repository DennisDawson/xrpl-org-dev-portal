
# 4. Transfer NFTokens

_The XLS-20 standard for NFTs <span class="status not_enabled" title="This feature is not currently enabled on the production XRP Ledger."><i class="fa fa-flask"></i></span> has a preliminary implementation that can be used in test networks, but is not yet available as an [amendment](amendments.html) to the XRP Ledger protocol. An amendment may be included in a future XRP Ledger release._

This example shows how to:



1. Create NFToken Sell Offers.
2. Create NFToken Buy Offers.
3. Accept NFToken Sell Offers.
4. Accept NFToken Buy Offers.
5. Get a list of offers for a particular NFToken.
6. Cancel an offer.



![Quickstart form with NFToken transfer fields](img/quickstart13.png)

You can download the [Quickstart Samples](https://github.com/XRPLF/xrpl-dev-portal/tree/master/content/_code-samples/quickstart/quickstart.zip) archive to try each of the samples in your own browser.


# Usage


## Get Accounts



1. Open `4.transfer-nftokens.html` in a browser.
2. Get test accounts.
    1. If you have existing NFT-Devnet account seeds
        1. Paste account seeds in the **Seeds** field.
        2. Click **Get Accounts from Seeds**.
    2. If you do not have NFT-Devnet account seeds:
        3. Visit the [XRP Testnet Faucet](https://xrpl.org/xrp-testnet-faucet.html) page.
        4. Click **Generate NFT-Devnet credentials**.
        5. Copy the account **Secret**.
        6. Paste the secret in a persistent location, such as a notepad, and press return.
        7. Click **Generate NFT-Devnet credentials** to create a second account.
        8. Copy the account **Secret**.
        9. Paste the secret in the persistent location.
        10. Copy both secrets, separated by a return.
        11. Paste them in the **Account** **Seeds** field.
        12. Click **Get Accounts from Seeds**.



![Form with account information](img/quickstart14.png)



## Create a Sell Offer

To create a NFToken sell offer:



1. Enter the **Amount** of the sell offer in drops (millionths of an XRP).
2. Set the **Flags** field to _1_.
3. Enter the **Token ID** of the NFToken you want to sell.
4. Click **Create Sell Offer**.

The important piece of information in the response is the Token Offer Index, labeled as _Index,_ which is used to accept the sell offer.



![NFToken Sell Offer](img/quickstart15.png)



## Accept Sell Offer

Once a sell offer is available, another account can opt to accept the offer and purchase the NFToken.

To accept an available sell offer:



1. Enter the **Token Offer Index** (labeled as _Index_ in the token offer results. This is not the same as the Token ID).
2. Click **Accept Sell Offer**.



![Accept Sell Offer](img/quickstart16.png)



## Create a Buy Offer

You can offer to buy a NFToken from another account.

To create an offer to buy a NFToken:



1. Enter the **Amount** of your offer.
2. Enter the **Token ID**.
3. Enter the owner’s account string in the **Owner** field.
4. Click **Create Buy Offer**.



![NFToken Buy Offer](img/quickstart17.png)



## Accept a Buy Offer

To accept an offer to buy a NFToken:



1. Enter the **Token Offer ID** (the **Index** of the token offer).
2. Enter the buyer’s account string in the **Owner** field (the buyer is the owner of the token offer).
3. Click **Accept Buy Offer**.



![Accept Buy Offer](img/quickstart18.png)



## Get Offers

Click **Get Offers** to get the buy and sell offers associated with a particular account.



![Get offers](img/quickstart19.png)



## Cancel Offer

To cancel a buy or sell offer that you have created:



1. Enter the **Token Offer ID**.
2. Click **Cancel Offer**.



![Cancel offer](img/quickstart20.png)



# Code Walkthrough 

You can download the [Quickstart Samples](https://github.com/XRPLF/xrpl-dev-portal/tree/master/content/_code-samples/quickstart/quickstart.zip) archive to try each of the samples in your own browser.


## Create Sell Offer


```
// *******************************************************
// ****************** Create Sell Offer ******************
// *******************************************************
```


Connect to the ledger and get the wallet accounts.


```
      async function createSellOffer() {
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
          let net = getNet()
          const client = new xrpl.Client(net)
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('standbyResultField').value = results
        await client.connect()
          results += '\nConnected. Creating sell offer...'
          document.getElementById('standbyResultField').value = results


```


Define the transaction. The _Flags_ value of 1 indicates that this transaction is a sell offer.


```
        const transactionBlob = {
              "TransactionType": "NFTokenCreateOffer",
              "Account": standby_wallet.classicAddress,
              "TokenID": standbyTokenIdField.value,
              "Amount": standbyAmountField.value,
              "Flags": parseInt(standbyFlagsField.value)
        }


```


Submit the transaction and wait for the results.

        


```
        const tx = await client.submitAndWait(transactionBlob,{wallet: standby_wallet})
```


      


```
        results += '\n\n***Sell Offers***\n'
```


Request the list of sell offers for the token.


```


        let nftSellOffers
          try {
            nftSellOffers = await client.request({
              method: "nft_sell_offers",
              tokenid: standbyTokenIdField.value  
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)
        results += '\n\n***Buy Offers***\n'
```


Request the list of buy offers for the token.


```
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
        method: "nft_buy_offers",
        tokenid: standbyTokenIdField.value })
        } catch (err) {
          results += 'No buy offers.'
        }
        results += JSON.stringify(nftBuyOffers,null,2)


```


Report the results of the transaction.


```
        results += '\n\nTransaction result:\n' + 
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += '\n\nBalance changes:\n' + 
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
```


Get the current XRP balances for the operational and standby accounts.


```
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))
        document.getElementById('standbyBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```


        client.disconnect()
        // End of createSellOffer()
      }
```



## Create Buy Offer


```
// *******************************************************
// ***************** Create Buy Offer ********************
// *******************************************************


      async function createBuyOffer() {
```


Get the account wallets and connect to the ledger.


```
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
        let net = getNet()
        const client = new xrpl.Client(net)
        let results = 'Connecting to ' + getNet() + '...'
        document.getElementById('standbyResultField').value = results
        await client.connect()
        results = '\nConnected. Creating buy offer...'
        document.getElementById('standbyResultField').value = results


```


Define the transaction. Setting the _Flags_ value to _null_ indicates that this is a buy offer.


```
        const transactionBlob = {
              "TransactionType": "NFTokenCreateOffer",
              "Account": standby_wallet.classicAddress,
              "Owner": standbyOwnerField.value,
              "TokenID": standbyTokenIdField.value,
              "Amount": standbyAmountField.value,
              "Flags": null
        }


```


Submit the transaction and wait for the results.


```
        const tx = await client.submitAndWait(transactionBlob,{wallet: standby_wallet})
```


Request the list of sell offers for the token.


```


        results += "\n\n***Sell Offers***\n"
        let nftSellOffers
          try {
            nftSellOffers = await client.request({
            method: "nft_sell_offers",
            tokenid: standbyTokenIdField.value  
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)
```


Request the list of buy offers for the token.


```
        results += "\n\n***Buy Offers***\n"
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
        method: "nft_buy_offers",
        tokenid: standbyTokenIdField.value })
        } catch (err) {
          results += "No buy offers."
        }
        results += JSON.stringify(nftBuyOffers,null,2)


```


Report the results of the transaction.

      


```
        results += "\n\nTransaction result:\n" +
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += "\n\nBalance changes:\n" + 
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```
        client.disconnect()
        // End of operationalCreateBuyOffer()
      }
```



## Cancel Offer


```
// *******************************************************
// ******************** Cancel Offer *********************
// *******************************************************

      async function cancelOffer() {    
```


Get the standby wallet and connect to the ledger.


```
        const wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const client = new xrpl.Client("wss://xls20-sandbox.rippletest.net:51233")
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('standbyResultField').value = results
        await client.connect()
        results +=  "\nConnected. Cancelling offer..."
        document.getElementById('standbyResultField').value = results
```


Insert the token offer index as a new array. This example destroys one offer at a time. In practice you could implement the function to accept multiple token offer index values and destroy all of the token offers in one operation.


```
        const tokenOfferIDs = [standbyTokenOfferIndexField.value]


```


Define the transaction.


```
        const transactionBlob = {
              "TransactionType": "NFTokenCancelOffer",
              "Account": wallet.classicAddress,
              "TokenOffers": tokenOfferIDs
        }
```


Submit the transaction and wait for the results.


```
        const tx = await client.submitAndWait(transactionBlob,{wallet})
```


Request the list of sell offers for the token.


```
        results += "\n\n***Sell Offers***\n"
        let nftSellOffers
          try {
            nftSellOffers = await client.request({
          method: "nft_sell_offers",
          tokenid: standbyTokenIdField.value
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)
```


Request the list of buy offers for the token.


```
        results += "\n\n***Buy Offers***\n"
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
        method: "nft_buy_offers",
        tokenid: standbyTokenIdField.value })
        } catch (err) {
          nftBuyOffers = "No buy offers."
        }
        results += JSON.stringify(nftBuyOffers,null,2)
```


Report the transaction results.


```


        results += "\nTransaction result:\n" +
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += "\nBalance changes:\n" + 
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```
        client.disconnect()
        // End of cancelOffer()
      }
```



## Get Offers


```
// *******************************************************
// ******************** Get Offers ***********************
// *******************************************************


      async function getOffers() {
```


Get the standby account wallet and connect to the ledger.


```
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        let net = getNet()
        const client = new xrpl.Client(net)
        results = 'Connecting to ' + getNet() + '...'
        document.getElementById('standbyResultField').value = results
        await client.connect()
        results += '\nConnected. Getting offers...'
        document.getElementById('standbyResultField').value = results
```


Request the list of sell offers for the token.


```
        results += '\n\n***Sell Offers***\n'  
        let nftSellOffers
          try {
            nftSellOffers = await client.request({
              method: "nft_sell_offers",
              tokenid: standbyTokenIdField.value  
            })
          } catch (err) {
            nftSellOffers = 'No sell offers.'
        }
        results += JSON.stringify(nftSellOffers,null,2)
        document.getElementById('standbyResultField').value = results
```


Request the list of buy offers for the token.


```
        results += '\n\n***Buy Offers***\n'
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
            method: "nft_buy_offers",
            tokenid: standbyTokenIdField.value })
        } catch (err) {
          nftBuyOffers =  'No buy offers.'
        }
        results += JSON.stringify(nftBuyOffers,null,2)    
        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```
        client.disconnect()
        // End of getOffers()
      }


```



## Accept Sell Offer


```
// *******************************************************
// ****************** Accept Sell Offer ******************
// *******************************************************


      async function acceptSellOffer() {
```


Get the account wallets and connect to the ledger.


```
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
        let net = getNet()
        const client = new xrpl.Client(net)
        results = 'Connecting to ' + getNet() + '...'
        document.getElementById('standbyResultField').value = results
        await client.connect()
        results += '\nConnected. Accepting sell offer...\n\n'
        document.getElementById('standbyResultField').value = results


```


Define the transaction.


```
        const transactionBlob = {
          "TransactionType": "NFTokenAcceptOffer",
          "Account": standby_wallet.classicAddress,
          "SellOffer": standbyTokenOfferIndexField.value,
        }
```


Submit the transaction and wait for the results.


```
        const tx = await client.submitAndWait(transactionBlob,{wallet: standby_wallet})
```


Request the list of NFTs for the standby account.


```
        const nfts = await client.request({
          method: "account_nfts",
          account: standby_wallet.classicAddress  
        })


```


Get the balances for both accounts.


```
        document.getElementById('standbyBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))
```


Report the transaction results.


```


        results += 'Transaction result:\n'
        results +=  JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += '\nBalance changes:'
        results +=  JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        results += JSON.stringify(nfts,null,2)


        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```
        client.disconnect()
      // End of acceptSellOffer()
      }
```



## Accept Buy Offer


```
// *******************************************************
// ******************* Accept Buy Offer ******************
// *******************************************************


      async function acceptBuyOffer() {
```


Get the account wallets and connect to the ledger.


```
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
        let net = getNet()
        const client = new xrpl.Client(net)
        results = 'Connecting to ' + getNet() + '...'
        document.getElementById('standbyResultField').value = results
        await client.connect()
        results += '\nConnected. Accepting buy offer...'
        document.getElementById('standbyResultField').value = results
```


Prepare the transaction.


```
        const transactionBlob = {
              "TransactionType": "NFTokenAcceptOffer",
              "Account": standby_wallet.classicAddress,
              "BuyOffer": standbyTokenOfferIndexField.value
        }
```


Submit the transaction and wait for the results.


```
        const tx = await client.submitAndWait(transactionBlob,{wallet: standby_wallet})
```


Request the current list of NFTs for the standby account.


```
        const nfts = await client.request({
          method: "account_nfts",
          account: standby_wallet.classicAddress  
        })
        results += JSON.stringify(nfts,null,2)
        document.getElementById('standbyResultField').value = results
```


Report the transaction result.


```
        result += "\n\nTransaction result:\n" + 
            JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        result += "\nBalance changes:\n" +
            JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
```


Request the XRP balance for both accounts.


```
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))
        document.getElementById('standbyBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('standbyResultField').value = results
```


Disconnect from the ledger.


```
        client.disconnect()
        // End of acceptBuyOffer()
      }
```



## Reciprocal Transactions

These functions duplicate the functions of the standby account for the operational account. See the corresponding standby account function for walkthrough information.

```
// *******************************************************
// *********** Operational Create Sell Offer *************
// *******************************************************


      async function oPcreateSellOffer() {
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
          let net = getNet()
          const client = new xrpl.Client(net)
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('operationalResultField').value = results
        await client.connect()
          results += '\nConnected. Creating sell offer...'
          document.getElementById('operationalResultField').value = results


       // Prepare transaction -------------------------------------------------------
        const transactionBlob = {
              "TransactionType": "NFTokenCreateOffer",
              "Account": operational_wallet.classicAddress,
              "TokenID": operationalTokenIdField.value,
              "Amount": operationalAmountField.value,
              "Flags": parseInt(operationalFlagsField.value)
        }


        // Submit transaction --------------------------------------------------------


        const tx = await client.submitAndWait(transactionBlob,{wallet: operational_wallet})


        results += '\n\n***Sell Offers***\n'


        let nftSellOffers
          try {
            nftSellOffers = await client.request({
          method: "nft_sell_offers",
          tokenid: operationalTokenIdField.value  
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)
        results += '\n\n***Buy Offers***\n'
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
        method: "nft_buy_offers",
        tokenid: operationalTokenIdField.value })
        } catch (err) {
          results += 'No buy offers.'
        }
        results += JSON.stringify(nftBuyOffers,null,2)


        // Check transaction results -------------------------------------------------
        results += '\n\nTransaction result:\n' + 
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += '\n\nBalance changes:\n' + 
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))
        document.getElementById('standbyBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('operationalResultField').value = results


        client.disconnect()
      }  // End of oPcreateSellOffer()


// *******************************************************
// ************** Operational Create Buy Offer ***********
// *******************************************************


      async function oPcreateBuyOffer() {


        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
          let net = getNet()
          const client = new xrpl.Client(net)
          let results = 'Connecting to ' + net + '...' 
          document.getElementById('operationalResultField').value = results
        await client.connect()
        results += '\nConnected. Creating buy offer...'
          document.getElementById('operationalResultField').value = results


       // Prepare transaction -------------------------------------------------------
        const transactionBlob = {
              "TransactionType": "NFTokenCreateOffer",
              "Account": operational_wallet.classicAddress,
              "Owner": operationalOwnerField.value,
              "TokenID": operationalTokenIdField.value,
              "Amount": operationalAmountField.value,
              "Flags": null
        }


        // Submit transaction --------------------------------------------------------
        const tx = await client.submitAndWait(transactionBlob,{wallet: operational_wallet})


        results +="\n\n***Sell Offers***\n"
        let nftSellOffers
          try {
            nftSellOffers = await client.request({
          method: "nft_sell_offers",
          tokenid: operationalTokenIdField.value  
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)


        results += "\n\n***Buy Offers***\n"
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
          method: "nft_buy_offers",
          tokenid: operationalTokenIdField.value })
        } catch (err) {
          nftBuyOffers = "No buy offers."
        }
        results += JSON.stringify(nftBuyOffers,null,2)


        // Check transaction results -------------------------------------------------
        results +="\n\nTransaction result:\n" + 
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += "\n\nBalance changes:\n" +
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        client.disconnect()
        // End of oPcreateBuyOffer()
      }


// *******************************************************
// ************* Operational Cancel Offer ****************
// *******************************************************


      async function oPcancelOffer() {


        const wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
        const client = new xrpl.Client("wss://xls20-sandbox.rippletest.net:51233")
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('operationalResultField').value = results
        await client.connect()
        results +=  "\nConnected. Cancelling offer..."
        document.getElementById('operationalResultField').value = results


        const tokenOfferIDs = [operationalTokenOfferIndexField.value]


       // Prepare transaction -------------------------------------------------------
        const transactionBlob = {
              "TransactionType": "NFTokenCancelOffer",
              "Account": wallet.classicAddress,
              "TokenOffers": tokenOfferIDs
        }
        // Submit transaction --------------------------------------------------------
        const tx = await client.submitAndWait(transactionBlob,{wallet})


        results += "\n\n***Sell Offers***\n"
        let nftSellOffers
          try {
            nftSellOffers = await client.request({
          method: "nft_sell_offers",
          tokenid: operationalTokenIdField.value
          })
          } catch (err) {
            nftSellOffers = "No sell offers."
        }
        results += JSON.stringify(nftSellOffers,null,2)
        results += "\n\n***Buy Offers***\n"
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
        method: "nft_buy_offers",
        tokenid: operationalTokenIdField.value })
        } catch (err) {
          nftBuyOffers = "No buy offers."
        }
        results += JSON.stringify(nftBuyOffers,null,2)


        // Check transaction results -------------------------------------------------


        results += "\nTransaction result:\n" +
          JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += "\nBalance changes:\n" + 
          JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        document.getElementById('operationalResultField').value = results


        client.disconnect()
        // End of oPcancelOffer()
      }


// *******************************************************
// **************** Operational Get Offers ***************
// *******************************************************


      async function oPgetOffers() {
        const standby_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
          let net = getNet()
          const client = new xrpl.Client(net)
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('operationalResultField').value = results
        await client.connect()
          results += '\nConnected. Getting offers...'


        results += '\n\n***Sell Offers***\n'


        let nftSellOffers
          try {
          nftSellOffers = await client.request({
          method: "nft_sell_offers",
          tokenid: operationalTokenIdField.value  
          })
          } catch (err) {
            nftSellOffers = 'No sell offers.'
        }
       results += JSON.stringify(nftSellOffers,null,2)


        results += '\n\n***Buy Offers***\n'
        let nftBuyOffers
        try {
          nftBuyOffers = await client.request({
            method: "nft_buy_offers",
            tokenid: operationalTokenIdField.value })
        } catch (err) {
          nftBuyOffers =  'No buy offers.'
        }
        results += JSON.stringify(nftBuyOffers,null,2)    


        document.getElementById('operationalResultField').value = results


        client.disconnect()
        // End of oPgetOffers()
      }


// *******************************************************
// *************** Operational Accept Sell Offer *********
// *******************************************************


      async function oPacceptSellOffer() {
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
          let net = getNet()
          const client = new xrpl.Client(net)
          results = 'Connecting to ' + getNet() + '...'
          document.getElementById('operationalResultField').value = results
        await client.connect()
          results = '\nConnected. Accepting sell offer...\n\n'
        document.getElementById('operationalResultField').value = results


       // Prepare transaction -------------------------------------------------------
        const transactionBlob = {
              "TransactionType": "NFTokenAcceptOffer",
              "Account": operational_wallet.classicAddress,
              "SellOffer": operationalTokenOfferIndexField.value,
        }
        // Submit transaction --------------------------------------------------------
        const tx = await client.submitAndWait(transactionBlob,{wallet: operational_wallet}) 
        const nfts = await client.request({
        method: "account_nfts",
        account: operational_wallet.classicAddress  
        })


        // Check transaction results -------------------------------------------------




        document.getElementById('standbyBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))


        results += 'Transaction result:\n'
        results +=  JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        results += '\nBalance changes:'
        results +=  JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        results += JSON.stringify(nfts,null,2)


        document.getElementById('operationalResultField').value = results


        client.disconnect()
      // End of acceptSellOffer()
      }

// *******************************************************
// ********* Operational Accept Buy Offer ****************
// *******************************************************


      async function oPacceptBuyOffer() {
        const standby_wallet = xrpl.Wallet.fromSeed(standbySeedField.value)
        const operational_wallet = xrpl.Wallet.fromSeed(operationalSeedField.value)
        let net = getNet()
        const client = new xrpl.Client(net)
        results = 'Connecting to ' + getNet() + '...'
        document.getElementById('operationalResultField').value = results
        await client.connect()
        results += '\nConnected. Accepting buy offer...'
        document.getElementById('operationalResultField').value = results

       // Prepare transaction -------------------------------------------------------
        const transactionBlob = {
              "TransactionType": "NFTokenAcceptOffer",
              "Account": operational_wallet.classicAddress,
              "BuyOffer": operationalTokenOfferIndexField.value
        }
        // Submit transaction --------------------------------------------------------
        const tx = await client.submitAndWait(transactionBlob,{wallet: operational_wallet}) 
        const nfts = await client.request({
        method: "account_nfts",
        account: operational_wallet.classicAddress  
        })
        results += JSON.stringify(nfts,null,2)
        document.getElementById('operationalResultField').value = results

        // Check transaction results -------------------------------------------------
        result += "\n\nTransaction result:\n" + 
            JSON.stringify(tx.result.meta.TransactionResult, null, 2)
        result += "\nBalance changes:\n" +
            JSON.stringify(xrpl.getBalanceChanges(tx.result.meta), null, 2)
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(operational_wallet.address))
        document.getElementById('operationalBalanceField').value = 
          (await client.getXrpBalance(standby_wallet.address))
        document.getElementById('operationalResultField').value = results
        client.disconnect()
        // End of acceptBuyOffer()
      }  
```



## 4.transfer-nfts.html

Update the form with fields and buttons to support the new functions.


```
<!-- ************************************************************** -->
<!-- ********************** The Form ****************************** -->
<!-- ************************************************************** -->

  <body>
    <h1>Token Test Harness</h1>
    <form id="theForm">
      Choose your ledger instance:  
      <input type="radio" id="xls" name="server"
        value="wss://xls20-sandbox.rippletest.net:51233" checked>
      <label for="xls20">XLS20-NFT</label>
      &nbsp;&nbsp;
      <input type="radio" id="tn" name="server"
        value="wss://s.altnet.rippletest.net:51233">
      <label for="testnet">Testnet</label>
      &nbsp;&nbsp;
      <input type="radio" id="dn" name="server"
        value="wss://s.devnet.rippletest.net:51233">
      <label for="devnet">Devnet</label>
      <br/><br/>
      <button type="button" onClick="getAccountsFromSeeds()">Get Accounts From Seeds</button>
      <br/>
      <textarea id="seeds" cols="40" rows= "2"></textarea>
      <br/><br/>
      <table>
        <tr valign="top">
          <td>
            <table>
              <tr valign="top">
                <td>
                <td>
                  <button type="button" onClick="getAccount('standby')">Get New Standby Account</button>
                  <table>
                    <tr valign="top">
                      <td align="right">
                        Standby Account
                      </td>
                      <td>
                        <input type="text" id="standbyAccountField" size="40"></input>
                        <br>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        Public Key
                      </td>
                      <td>
                        <input type="text" id="standbyPubKeyField" size="40"></input>
                        <br>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        Private Key
                      </td>
                      <td>
                        <input type="text" id="standbyPrivKeyField" size="40"></input>
                        <br>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        Seed
                      </td>
                      <td>
                        <input type="text" id="standbySeedField" size="40"></input>
                        <br>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        XRP Balance
                      </td>
                      <td>
                        <input type="text" id="standbyBalanceField" size="40"></input>
                        <br>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        Amount
                      </td>
                      <td>
                        <input type="text" id="standbyAmountField" size="40" value="100"></input>
                        <br>
                      </td>
                    </tr>
                    <tr valign="top">
                      <td><button type="button" onClick="configureAccount('standby',document.querySelector('#standbyDefault').checked)">Configure Account</button></td>
                      <td>
                        <input type="checkbox" id="standbyDefault" checked="true"/>
                        <label for="standbyDefault">Allow Rippling</label>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">
                        Currency
                      </td>
                      <td>
                        <input type="text" id="standbyCurrencyField" size="40" value="USD"></input>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">Token URL</td>
                      <td><input type="text" id="standbyTokenUrlField"
                        value = "ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf4dfuylqabf3oclgtqy55fbzdi" size="80"/>
                      </td>
                    </tr>
                    <tr>
                      <td align="right">Flags</td>
                      <td><input type="text" id="standbyFlagsField" value="1" size="10"/></td>
                    </tr>
                    <tr>
                      <td align="right">Token ID</td>
                      <td><input type="text" id="standbyTokenIdField" value="" size="80"/></td>
                    </tr>
                    <tr>
                      <td align="right">Token Offer Index</td>
                      <td><input type="text" id="standbyTokenOfferIndexField" value="" size="80"/></td>
                    </tr>
                    <tr>
                      <td align="right">Owner</td>
                      <td><input type="text" id="standbyOwnerField" value="" size="80"/></td>
                    </tr>
                  </table>
                  <p align="left">
                    <textarea id="standbyResultField" cols="80" rows="20" ></textarea>
                  </p>
                </td>
                </td>
                <td>
                  <table>
                    <tr valign="top">
                      <td align="center" valign="top">
                        <button type="button" onClick="sendXRP()">Send XRP&#62;</button>
                        <br/><br/>
                        <button type="button" onClick="createTrustline()">Create TrustLine</button>
                        <br/>
                        <button type="button" onClick="sendCurrency()">Send Currency</button>
                        <br/>
                        <button type="button" onClick="getBalances()">Get Balances</button>
                        <br/><br/>
                        <button type="button" onClick="mintToken()">Mint NFToken</button>
                        <br/>
                        <button type="button" onClick="getTokens()">Get NFTokens</button>
                        <br/>
                        <button type="button" onClick="burnToken()">Burn NFToken</button>
                        <br/><br/>
                        <button type="button" onClick="createSellOffer()">Create Sell Offer</button>
                        <br/>
                        <button type="button" onClick="acceptSellOffer()">Accept Sell Offer</button>
                        <br/>
                        <button type="button" onClick="createBuyOffer()">Create Buy Offer</button>
                        <br/>
                        <button type="button" onClick="acceptBuyOffer()">Accept Buy Offer</button>
                        <br/>
                        <button type="button" onClick="getOffers()">Get Offers</button>
                        <br/>
                        <button type="button" onClick="cancelOffer()">Cancel Offer</button>
                      </td>
                    </tr>
                    </td>
                    </tr>
                  </table>
                </td>
              </tr>
            </table>
          </td>
          <td>
            <table>
              <tr>
                <td>
                <td>
                  <table>
                    <tr>
                      <td align="center" valign="top">
                        <button type="button" onClick="oPsendXRP()">&#60; Send XRP</button>
                        <br/><br/>
                        <button type="button" onClick="oPcreateTrustline()">Create TrustLine</button>
                        <br/>
                        <button type="button" onClick="oPsendCurrency()">Send Currency</button>
                        <br/>
                        <button type="button" onClick="getBalances()">Get Balances</button>
                        <br/><br/>
                        <button type="button" onClick="oPmintToken()">Mint NFToken</button>
                        <br/>
                        <button type="button" onClick="oPgetTokens()">Get NFTokens</button>
                        <br/>
                        <button type="button" onClick="oPburnToken()">Burn NFToken</button>
                        <br/><br/>
                        <button type="button" onClick="oPcreateSellOffer()">Create Sell Offer</button>
                        <br/>
                        <button type="button" onClick="oPacceptSellOffer()">Accept Sell Offer</button>
                        <br/>
                        <button type="button" onClick="oPcreateBuyOffer()">Create Buy Offer</button>
                        <br/>
                        <button type="button" onClick="oPacceptBuyOffer()">Accept Buy Offer</button>
                        <br/>
                        <button type="button" onClick="oPgetOffers()">Get Offers</button>
                        <br/>
                        <button type="button" onClick="oPcancelOffer()">Cancel Offer</button>
                      </td>
                      <td valign="top" align="right">
                        <button type="button" onClick="getAccount('operational')">Get New Operational Account</button>
                        <table>
                          <tr valign="top">
                            <td align="right">
                              Operational Account
                            </td>
                            <td>
                              <input type="text" id="operationalAccountField" size="40"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              Public Key
                            </td>
                            <td>
                              <input type="text" id="operationalPubKeyField" size="40"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              Private Key
                            </td>
                            <td>
                              <input type="text" id="operationalPrivKeyField" size="40"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              Seed
                            </td>
                            <td>
                              <input type="text" id="operationalSeedField" size="40"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              XRP Balance
                            </td>
                            <td>
                              <input type="text" id="operationalBalanceField" size="40"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              Amount
                            </td>
                            <td>
                              <input type="text" id="operationalAmountField" size="40" value="100"></input>
                              <br>
                            </td>
                          </tr>
                          <tr>
                            <td>
                            </td>
                            <td align="right">                        <input type="checkbox" id="operationalDefault" checked="true"/>
                              <label for="operationalDefault">Allow Rippling</label>
                              <button type="button" onClick="configureAccount('operational',document.querySelector('#operationalDefault').checked)">Configure Account</button>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">
                              Currency
                            </td>
                            <td>
                              <input type="text" id="operationalCurrencyField" size="40" value="USD"></input>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">Token URL</td>
                            <td><input type="text" id="operationalTokenUrlField"
                              value = "ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf4dfuylqabf3oclgtqy55fbzdi" size="80"/>
                            </td>
                          </tr>
                          <tr>
                            <td align="right">Flags</td>
                            <td><input type="text" id="operationalFlagsField" value="1" size="10"/></td>
                          </tr>
                          <tr>
                            <td align="right">Token ID</td>
                            <td><input type="text" id="operationalTokenIdField" value="" size="80"/></td>
                          </tr>
                          <tr>
                            <td align="right">Token Offer Index</td>
                            <td><input type="text" id="operationalTokenOfferIndexField" value="" size="80"/></td>
                          </tr>
                          <tr>
                            <td align="right">Owner</td>
                            <td><input type="text" id="operationalOwnerField" value="" size="80"/></td>
                          </tr>
                        </table>
                        <p align="right">
                          <textarea id="operationalResultField" cols="80" rows="20" ></textarea>
                        </p>
                      </td>
                      </td>
                    </tr>
                    </td>
                    </tr>
                  </table>
                </td>
              </tr>
            </table>
          </td>
        </tr>
      </table>
    </form>
  </body>
</html>

```

---


| Previous                                                 | Next  |
| :---                                                     |  ---: |
| 3. [Mint and Burn NFTokens](mint-and-burn-nftokens.html) |       |