---
如何引用SDK
---
```
npm install action-sdk git+ssh://github.com:actionorg/js-action-sdk.git
```
```javascript
var StellarSdk = require('action-sdk'); 
StellarSdk.Network.usePublicNetwork();
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
var server = new StellarSdk.Server('https://api.action.bi');
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
var sourceKeys = StellarSdk.Keypair.fromSecret('AC4AJNNFEHX5VOWXGQ63KPJEA6M6K6EKUQBY7FHGK67W66B5OJEZKB5Q的私钥');
var destinationId = 'AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC';
var transaction;

var server = new StellarSdk.Server('https://api.action.bi');
server.loadAccount(destinationId)
  .catch(StellarSdk.NotFoundError, function (error) {
    throw new Error('The destination account does not exist!');
  })
  .then(function() {
    return server.loadAccount(sourceKeys.publicKey());
  })
  .then(function(sourceAccount) {
    transaction = new StellarSdk.TransactionBuilder(sourceAccount)
      .addOperation(StellarSdk.Operation.payment({
        destination: destinationId,
        asset: StellarSdk.Asset.native(),
        amount: "10"
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
## 账户余额信息

##
```javascript
var server = new StellarSdk.Server('https://api.action.bi');
var i = 'AAYPSO6Y3OOZWQOT5F4F5LYT3G4BUPX2VOYAPVFW7F65TWVH2UPZJZQC';
server.loadAccount().then(function(account) {
  console.log('Balances for account: ' + pair.publicKey());
  account.balances.forEach(function(balance) {
    console.log('Type:', balance.asset_type, ', Balance:', balance.balance);
  });
});
```

