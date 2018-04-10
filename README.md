---
如何引用SDK
---
```
npm install action-sdk git+ssh://github.com:actionorg/js-action-sdk.git
```
```javascript
var StellarSdk = require('action-sdk'); 
StellarSdk.Network.usePublicNetwork();
var server = new StellarSdk.Server('https://api.action.bi');
```
## 创建账号
首先，需要创建一个密钥对，然后由另一个账号向公钥转账一定数量的ACTN才能激活。
如下面的例子是由 AC4AJNNFEHX5VOWXGQ63KPJEA6M6K6EKUQBY7FHGK67W66B5OJEZKB5Q 账号向AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC 账号转账，从而创建了账号 AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC 
##
```javascript
var pair = StellarSdk.Keypair.random();
pair.secret();
// SAOUOAUMXFUUQU73HPE5QS3IFAUKFU4USEH3HDW6OBMJINXI2QVHFY3X
pair.publicKey();
// AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC

var sourceKeys = StellarSdk.Keypair.fromSecret('AC4AJNNFEHX5VOWXGQ63KPJEA6M6K6EKUQBY7FHGK67W66B5OJEZKB5Q的私钥');
var destinationId = 'AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC';
var transaction;
server.loadAccount(sourceKeys.publicKey())
  .then(function(sourceAccount) {
    transaction = new StellarSdk.TransactionBuilder(sourceAccount)
      .addOperation(StellarSdk.Operation.createAccount({
        destination: destinationId,
        startingBalance : "25"
      })) 
      .build();
    transaction.sign(sourceKeys);
    return server.submitTransaction(transaction);
  })
  .then(function(result) {
    console.log('Success! Results:', result);
  })
  .catch(function(error) {
    console.error('Something went wrong!', error);
    server.submitTransaction(transaction);
  }); 
```
## 账号转账
以下例子为 AC4AJNNFEHX5VOWXGQ63KPJEA6M6K6EKUQBY7FHGK67W66B5OJEZKB5Q 向  AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC 转账
##
```javascript
var sourceKeys = StellarSdk.Keypair
  .fromSecret('AC4AJNNFEHX5VOWXGQ63KPJEA6M6K6EKUQBY7FHGK67W66B5OJEZKB5Q的私钥');
var destinationId = 'GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5';
// Transaction will hold a built transaction we can resubmit if the result is unknown.
var transaction;

// First, check to make sure that the destination account exists.
// You could skip this, but if the account does not exist, you will be charged
// the transaction fee when the transaction fails.
server.loadAccount(destinationId)
  // If the account is not found, surface a nicer error message for logging.
  .catch(StellarSdk.NotFoundError, function (error) {
    throw new Error('The destination account does not exist!');
  })
  // If there was no error, load up-to-date information on your account.
  .then(function() {
    return server.loadAccount(sourceKeys.publicKey());
  })
  .then(function(sourceAccount) {
    // Start building the transaction.
    transaction = new StellarSdk.TransactionBuilder(sourceAccount)
      .addOperation(StellarSdk.Operation.payment({
        destination: destinationId,
        // Because Stellar allows transaction in many currencies, you must
        // specify the asset type. The special "native" asset represents Lumens.
        asset: StellarSdk.Asset.native(),
        amount: "10"
      }))
      // A memo allows you to add your own metadata to a transaction. It's
      // optional and does not affect how Stellar treats the transaction.
      .addMemo(StellarSdk.Memo.text('Test Transaction'))
      .build();
    // Sign the transaction to prove you are actually the person sending it.
    transaction.sign(sourceKeys);
    // And finally, send it off to Stellar!
    return server.submitTransaction(transaction);
  })
  .then(function(result) {
    console.log('Success! Results:', result);
  })
  .catch(function(error) {
    console.error('Something went wrong!', error);
    // If the result is unknown (no response body, timeout etc.) we simply resubmit
    // already built transaction:
    // server.submitTransaction(transaction);
  });
```
