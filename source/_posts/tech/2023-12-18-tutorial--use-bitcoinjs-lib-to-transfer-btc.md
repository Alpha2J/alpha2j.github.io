---
title: 使用bitcoinjs-lib进行比特币的转账
date: 2023-08-12
tags: 
  - tutorial
  - web3
  - btw
categories: technology
keywords: 'web3, block chain, etc'
---

从A地址给B地址转n个BTC，基本流程为：1. 查找A地址的UTXO的位置（在哪个transaction下产生，以及在该transaction下的哪个index下） → 2. 构造这些UTXO的解锁脚本 → 3. 生成n个BTC于B地址的UTXO → 4. 获取当前最佳的矿工费 → 5. 使用步骤一二三四的结果构造transaction → 6. 将交易广播到链上

上面的步骤一、四、六可以通过mempool这些厂商的接口来读写；而步骤二、三、五则可以使用bitcoinjs-lib这种客户端框架来完成，否则从头开始自己写一个字节出问题都会炸，例子可以看最下面代码给出。

在这个过程中，需要解决的一些问题点有：

1. 如何找到A地址的余额是否大于等于n个BTC：mempool这个接口能拿到所有的utxo，value加起来即可https://mempool.space/docs/api/rest#get-address-utxo

   - 有链上rpc能直接读吗？不详，待看
   - 好像有点麻烦，有能直接拿余额的接口吗？mempool是没有的，有时间看看quicknode或者其他的？

2. A地址中，这n个BTC的UTXO都在哪些transaction下，以及在对应的transaction下的哪个Index里：还是拿utxo的接口，txid和vout两个字段：https://mempool.space/docs/api/rest#get-address-utxo

3. 不同格式的地址有什么区别，在转账的时候需要做什么特殊的处理吗：私钥生成公钥，公钥在哈希成地址的时候，根据不同规范添加不同的前缀或后缀字节，就会出现不同格式的地址。

   不同格式的地址对应的签名脚本以及解锁脚本都不相同，从上层来看也仅此而已

   。举例说明：p2pk格式的地址A给p2pkh格式的地址B转账，首先我得识别到地址A是一个p2pk格式的地址，然后生成p2pk对应的签名脚本 ，接着得识别到地址B是一个p2pkh格式的地址，然后生成p2pkh的公钥脚本。当然，如果使用bitcoinjs-lib，我们只要能识别到地址的格式即可，脚本怎么生成框架里面会写好。

   - 注：taproot部分还没看到，细节我也不是很懂

4. 矿工费是怎么设置的？在一个transaction中，所有的所有的txin的价值之和减去所有的txout的价值之和的结果就为矿工费，所以在设置找零价格的时候要注意这点。

5. 怎么获取最佳矿工费？可以参考这里：https://mempool.space/api/v1/fees/recommended

6. 怎么将交易广播到链上？可以参考这里：https://mempool.space/docs/api/rest#post-transaction

bitcoinjs-lib使用示例：

1. p2pkh → p2pkh：

   ```jsx
   const {ECPairFactory} = require("ecpair");
   const ecc = require('tiny-secp256k1');
   const bitcoin = require("bitcoinjs-lib");
   
   const ECPair = ECPairFactory(ecc);
   
   const network = bitcoin.networks.testnet;
   const keyPair = ECPair.fromWIF('cUYZqgWKGhn7MA8ph4CHLEzvpyhmCcqE9HrmRuiXVAvquPdqLqbm', network);
   
   // 从账户oriAddr转到账户destAddr
   const oriAddr = 'mp3pY9u653PW6d21pDXcjup8uxHzP3g3xs';
   const destAddr = 'myGqmkcCpEreM9Bevjr1z3dPzQXnutRK4Y';
   
   // utxo所在的txhash以index
   const inputData = {
       hash: '041a2be11c35f47619417786ce0b6c8bb45c6a43e4be77984f4c886863fb1478',
       index: 0,
       nonWitnessUtxo: Buffer.from('02000000000101e78d6eb648472dd758b0e035ab9a59d43306e1602e98fd01d7abe660d3b5f8e10100000000ffffffff0210270000000000001976a9145d976d0b8e01b3aae21d844f962953f15fbc8c7688acf9931c000000000016001492883416825d612021722af95ab75530335df8a302483045022100b1232f7ed5738a343efa2cfca31ad665be30eca1463c082329f4574eb1e8fd2302200a7620d5539bb710a4a202e53c705739592ee73f675035d706a31adf093488bd012102577d146c56926fbd0a5eac4a0f69cda68d9a91e2f1501a4a6960454e13e82c0900000000', 'hex'),
   };
   // 转给b的金额
   const outputData = {
       address: destAddr,
       value: 1000
   };
   // 找零
   const changeData = {
       address: oriAddr,
       value: 8500
   }
   const tx = new bitcoin.Psbt({network: network})
       .addInput(inputData)
       .addOutputs([outputData, changeData])
       .signInput(0, keyPair)
       .finalizeAllInputs()
       .extractTransaction();
   
   const txHex = tx.toHex();
   console.log(txHex);
   
   // <https://blockstream.info/testnet/tx/57d0b3913c405d9a555d22fe7712e8ee6e5c16594c1436a0bda7bbb1c2f29682>
   // txHash: 57d0b3913c405d9a555d22fe7712e8ee6e5c16594c1436a0bda7bbb1c2f29682
   ```

2. p2tr → p2sh

   ```jsx
   const {ECPairFactory} = require("ecpair");
   const ecc = require('tiny-secp256k1');
   const bitcoin = require("bitcoinjs-lib");
   bitcoin.initEccLib(ecc);
   
   const ECPair = ECPairFactory(ecc);
   
   const network = bitcoin.networks.testnet;
   const keyPair = ECPair.fromWIF('cQSjty1JPbEhWJrx2aWGiExsd7WhRycTKcdi2umubWRAZDt5ntZ5', network);
   
   const oriAddr = 'tb1pfkdt80jt533fqwzg9zz5ultmmmnw86wh582w7y73dj7pd2ahl4kstdy4y5';
   const destAddr = '2NE23G6Cjzo1TNrv7oYxqYVn5FNbFvgXXLB';
   
   const utxoAmount = 1000;
   const sendAmount = 400;
   const changeAmount = 450;
   
   const inputData = {
       hash: '4765099e3c3f2dafa02676ca83ab6ad1c022315dec4e224b4da90a9eeec3f78b',
       index: 0,
       witnessUtxo: {
           script: Buffer.from('51204d9ab3be4ba46290384828854e7d7bdee6e3e9d7a1d4ef13d16cbc16abb7fd6d', 'hex'),
           value: utxoAmount
       },
       tapInternalKey: toXOnly(keyPair.publicKey)
   };
   const outputData = {
       address: destAddr,
       value: sendAmount
   };
   const changeData = {
       value: changeAmount,
       address: oriAddr,
       tapInternalKey: toXOnly(keyPair.publicKey)
   };
   const psbt = new bitcoin.Psbt({network: network})
       .addInput(inputData) // alice1 unspent
       .addOutputs([outputData, changeData]);
   
   const tweakedSigner = tweakSigner(keyPair, { network: network });
   psbt.signInput(0, tweakedSigner);
   
   psbt.finalizeAllInputs();
   const tx = psbt.extractTransaction();
   const rawTx = tx.toBuffer();
   const txHex = rawTx.toString('hex');
   console.log(txHex);
   
   function tweakSigner(signer, opts = {}) {
       let privateKey = signer.privateKey;
       if (signer.publicKey[0] === 3) {
           privateKey = ecc.privateNegate(privateKey);
       }
   
       const tweakedPrivateKey = ecc.privateAdd(
           privateKey,
           tapTweakHash(toXOnly(signer.publicKey), opts.tweakHash)
       );
   
       if (!tweakedPrivateKey) {
           throw new Error('Invalid tweaked private key!');
       }
   
       return ECPair.fromPrivateKey(Buffer.from(tweakedPrivateKey), {
           network: opts.network
       });
   }
   
   function tapTweakHash(pubKey, h) {
       return bitcoin.crypto.taggedHash(
           'TapTweak',
           Buffer.concat(h ? [pubKey, h] : [pubKey]),
       );
   }
   
   function toXOnly(pubKey) {
       return pubKey.length === 32 ? pubKey : pubKey.slice(1, 33);
   }
   
   // <https://blockstream.info/testnet/tx/10e36be62d93394c3c3f08841c40ce1e53f2dae8b55bb4b7383a74d699aaae64?input:0>
   // txHash: 10e36be62d93394c3c3f08841c40ce1e53f2dae8b55bb4b7383a74d699aaae64
   ```

3. p2wpkh

   ```jsx
   const {ECPairFactory} = require("ecpair");
   const ecc = require('tiny-secp256k1');
   const bitcoin = require("bitcoinjs-lib");
   
   const ECPair = ECPairFactory(ecc);
   
   const network = bitcoin.networks.testnet;
   const keyPair = ECPair.fromWIF('cNHwZuLZVo531CMarT4tGUcWAfZxh19AZJRr8ZvcNTMDWaCWArLQ', network);
   
   const oriAddr = 'tb1qznq0ysk7naka8ztjs0pyx0gwhj907j8shyhtal';
   const destAddr = 'tb1qj2yrg95zt4sjqgtj9tu44d64xqe4m79rm89ne3';
   const inputData = {
       hash: 'e1f8b5d360e6abd701fd982e60e10633d4599aab35e0b058d72d4748b66e8de7',
       index: 0,
       witnessUtxo: {
           script: Buffer.from('001414c0f242de9f6dd3897283c2433d0ebc8aff48f0', 'hex'),
           value: 40000
       }
   };
   const outputData = {
       address: destAddr,
       value: 1000
   };
   const changeData = {
       address: oriAddr,
       value: 38500
   }
   
   const tx = new bitcoin.Psbt({network: network})
       .addInput(inputData)
       .addOutputs([outputData, changeData])
       .signInput(0, keyPair)
       .finalizeAllInputs()
       .extractTransaction();
   
   const txHex = tx.toHex();
   console.log(txHex);
   
   // txHash: 70db29d9ef9da6531acc8b3de4018daf2a205e44b58b735dce73e710b88cbe5a
   ```