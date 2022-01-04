---
title: 以太坊hash与签名
date: 2022-01-04 14:49:04
tags: 以太坊
categories: 区块链
---

## 1.交易签名
### 1.1 交易签名的构成
下面以授权为例：
0xf8ac8201368502540be400830c350094425c2d431065fcf120e78d47b88067349795fe3e80b844095ea7b300000000000
0000000000000629aa1e54564fe9f917143bed33fc280a8fdd93f0000000000000000000000000000000000000000000000
000000b5e620f480002ba0474d4e4e464e415abba86600fd7f0a75261cd89a3857c0e0d3e9676f028d534ea028b64c84c86
dc60eba4186db8922946c2b30d42f7bcd698a86cd65740ce6580f

- 0136 表示nonce
- 02540be400 表示gas price，转换为10进制为10000000000，即10Gwei
- 0c3500 表示gas limit，转为10进制即800000
- 425C2D431065fcf120E78d47b88067349795Fe3e 即to，表示合约地址0x425C2D431065fcf120E78d47b88067349795Fe3e
- b844表示value，按理应该是0，不知道具体含义
- 095ea7b3 methodId
- 000000000000000000000000629aa1e54564fe9f917143bed33fc280a8fdd93f 授权的钱包或者合约地址0x629aa1e54564fe9f917143bed33fc280a8fdd93f
- 0000000000000000000000000000000000000000000000000000b5e620f48000 授权的金额即0.0002个ETH
- 2ba0474d4e4e464e415abba86600fd7f0a75261cd89a3857c0e0d3e9676f028d534ea028b64c84c86dc60eba4186db892294
  6c2b30d42f7bcd698a86cd65740ce6580f为签名数据
![](eth-01.png)	

### 1.2 js版本交易签名
```
/**
 * 授权
 * @author wangjf 
 * @version v1.0
 * @date 2021-12-7
 */
var Tx     = require('ethereumjs-tx').Transaction
var ethUtil = require('ethereumjs-util');
const Web3 = require('web3')
const rpcURL = "http://xxx.xxx.xxx.xxx:18545"

const web3 = new Web3(rpcURL)

/** 
 * 买家、卖家、平台地址、私钥 
 */
const buyer = "0xCD2bedb135ba3e16F54d2B51959506297e78739D"
const buyer_privateKey = Buffer.from('xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx', 'hex')

/** 
 * 交易金额： 0.0002 ether
 */
const value = '0.0002'

/**
 * 代币合约地址
 */
const usdtAddress = '0x425C2D431065fcf120E78d47b88067349795Fe3e'
/**
 * 业务合约地址
 */
const contractAddress = '0x629aa1e54564fe9f917143bed33fc280a8fdd93f'

/**
 * 代币合约的abi文件
 */
const contractABI = [
	{
		"constant": false,
		"inputs": [
			{
				"name": "spender",
				"type": "address"
			},
			{
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "approve",
		"outputs": [
			{
				"name": "",
				"type": "bool"
			}
		],
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"constant": false,
		"inputs": [
			{
				"name": "from",
				"type": "address"
			},
			{
				"name": "to",
				"type": "address"
			},
			{
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "transferFrom",
		"outputs": [
			{
				"name": "",
				"type": "bool"
			}
		],
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [
			{
				"name": "who",
				"type": "address"
			}
		],
		"name": "balanceOf",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": false,
		"inputs": [
			{
				"name": "to",
				"type": "address"
			},
			{
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "transfer",
		"outputs": [
			{
				"name": "",
				"type": "bool"
			}
		],
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [
			{
				"name": "owner",
				"type": "address"
			},
			{
				"name": "spender",
				"type": "address"
			}
		],
		"name": "allowance",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"name": "from",
				"type": "address"
			},
			{
				"indexed": true,
				"name": "to",
				"type": "address"
			},
			{
				"indexed": false,
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "Transfer",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"name": "owner",
				"type": "address"
			},
			{
				"indexed": true,
				"name": "spender",
				"type": "address"
			},
			{
				"indexed": false,
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "Approval",
		"type": "event"
	}
]

/**
 * 创建合约对象
 */
const contract = new web3.eth.Contract(contractABI, usdtAddress)

/**
 * 生成调用合约的abi对象
 */
const data = contract.methods.approve(contractAddress, web3.utils.toWei(value, 'ether')).encodeABI()

/**
 * 获取nonce并调用合约
 * @note  注意付款由买家发起
 */
web3.eth.getTransactionCount(buyer, (err, txCount) => {

    // 创建交易对象
    const txObject = {
        nonce:    web3.utils.toHex(txCount),
        gasLimit: web3.utils.toHex(800000),
        gasPrice: web3.utils.toHex(web3.utils.toWei('10', 'gwei')),
        to: usdtAddress,
		data: data,
		value : 0,
      }
  
    // 签署交易
	const tx = new Tx(txObject, { chain: 'rinkeby', hardfork: 'petersburg' })
	// 买家私钥签名
	tx.sign(buyer_privateKey)
	
	// 查看即将序列化的数据
	console.log(tx.raw);
	const serializedTx = tx.serialize()
	const raw = '0x' + serializedTx.toString('hex')
	// 查看16进制数据
	console.log(raw);

	
  
    // 广播交易
    web3.eth.sendSignedTransaction(raw, (err, txHash) => {
      console.log('txHash:', txHash)
      // 可以去ropsten.etherscan.io查看交易详情
    })
  })  
```
js输出：
```
$ node viv-contract-usdt-aprrove
[
  <Buffer 01 36>,
  <Buffer 02 54 0b e4 00>,
  <Buffer 0c 35 00>,
  <Buffer 42 5c 2d 43 10 65 fc f1 20 e7 8d 47 b8 80 67 34 97 95 fe 3e>,
  <Buffer >,
  <Buffer 09 5e a7 b3 00 00 00 00 00 00 00 00 00 00 00 00 62 9a a1 e5 45 64 fe 9f 91 71 43 be d3 3f c2 80 a8 fd d9 3f 00 00 00 00 00 00 00 00 00 00 00 00 00 0
0 ... 18 more bytes>,
  <Buffer 2b>,
  <Buffer 47 4d 4e 4e 46 4e 41 5a bb a8 66 00 fd 7f 0a 75 26 1c d8 9a 38 57 c0 e0 d3 e9 67 6f 02 8d 53 4e>,
  <Buffer 28 b6 4c 84 c8 6d c6 0e ba 41 86 db 89 22 94 6c 2b 30 d4 2f 7b cd 69 8a 86 cd 65 74 0c e6 58 0f>
]
0xf8ac8201368502540be400830c350094425c2d431065fcf120e78d47b88067349795fe3e80b844095ea7b3000000000000000000000000629aa1e54564fe9f917143bed33fc280a8fdd93f000000
0000000000000000000000000000000000000000000000b5e620f480002ba0474d4e4e464e415abba86600fd7f0a75261cd89a3857c0e0d3e9676f028d534ea028b64c84c86dc60eba4186db892294
6c2b30d42f7bcd698a86cd65740ce6580f
txHash: 0x578e7508be076fbf9c7c29b1440b2719b8f593e2ef8be8cf278d3a90c5d0c03c
```
从输出可以看出签名数据的整个结构

### 1.3 java版本交易签名
```
public String transferContractTransaction(String fromAddress, String fromPrivateKey, String contractAddress, String contractFunction, BigInteger gasPrice, BigInteger gasLimit, BigInteger value, List<Type> inputParameters, List<TypeReference<?>> outputParameters) throws Exception {
        Function function = new Function(
                contractFunction,
                inputParameters,
                outputParameters);
        String encodedFunction = FunctionEncoder.encode(function);
        Credentials credentials = Credentials.create(fromPrivateKey);
 
        BigInteger nonce = getNonce(credentials.getAddress());
        RawTransaction rawTransaction = RawTransaction.createTransaction(nonce, gasPrice, gasLimit, contractAddress, value, encodedFunction);
 
        //签名
        byte[] signedMessage = TransactionEncoder.signMessage(rawTransaction, vivConfig.getEth().getChainId(), credentials);
        String hexValue = Numeric.toHexString(signedMessage);
        log.info("transferContract hexValue: {}", hexValue);
 
        //发起交易
        EthSendTransaction ethSendTransaction =
                Web3jUtil.getInstance().ethSendRawTransaction(hexValue).send();
        if (ethSendTransaction.hasError()) {
            throw new CustomException(500, ethSendTransaction.getError().getMessage());
        }
        String transactionHash = ethSendTransaction.getTransactionHash();
        return transactionHash;
    }
```

### 1.4 java解析签名
```
public EthApproveVO getUsdtApprove(String hexValue) throws Exception {
        SignedRawTransaction rawTransaction = (SignedRawTransaction)TransactionDecoder.decode(hexValue);
        String data = rawTransaction.getData();
        String methodHex = "0x" + data.substring(0, 8);
        if(!CommonConst.Contract.ERC20.METHOD_APPROVAL.getCode().equals(methodHex)) {
            log.error("16进制数据有误，非授权！{}", hexValue);
            return null;
        }

        // function approve(address spender, uint256 value) external returns (bool);
        List<Type> result = FunctionReturnDecoder.decode(data.substring(8), Utils.convert(Arrays.<TypeReference<?>>asList(
                new TypeReference<Address>() {},
                new TypeReference<Uint>() {}
        )));
        EthApproveVO ethApproveVO = new EthApproveVO();
        ethApproveVO.setNonce(rawTransaction.getNonce());
        ethApproveVO.setGasPrice(Convert.fromWei(new BigDecimal(rawTransaction.getGasPrice()), Convert.Unit.ETHER));
        ethApproveVO.setGasLimit(new BigDecimal(rawTransaction.getGasLimit()));
        ethApproveVO.setSpender(((Address)result.get(0)).getValue());
        ethApproveVO.setValue(Convert.fromWei(new BigDecimal(((Uint) result.get(1)).getValue()), Convert.Unit.ETHER));
        ethApproveVO.setFrom(rawTransaction.getFrom());
        return ethApproveVO;
    }
```

## 2.普通签名
### 2.1 需求
我们经常需要在solidity中通过恢复签名的钱包地址来判断用户身份，需要在remix自带计算的签名、java中计算的签名、javascript中计算的签名均保持一致，并在solidity中可以恢复出地址。

### 2.2 测试数据
remix计算hash和签名需要先转为ascii码。457542584823209984====>0x343537353432353834383233323039393834

测试账户如图
![](eth-02.png)
原始串：457542584823209984

原始串ASCII码：0x343537353432353834383233323039393834

钱包地址：0x865Af88273B9218b94c46d12f34be460865B0A0d

### 2.3 remix签名
#### 2.3.1 按如下图切换账户


用于签名的数据：457542584823209984
![](eth-03.png)


#### 2.3.2 需要MetaMask授权
![](eth-04.png)

#### 2.3.3 得到签名结果
下面是加密结果：
hash：
0x8a590edede2c24dad749a698c8f018863fda7ae9d26b9d32716d45b46ee632c5
signature：
0x2d27aaeebc445d4fa1fba84e1261fa2431406fb49554ff88e76e892b5b59031a6adf3d151dd088e1bae2e488b40305965995ec2a9ea5c9b84f792beb6bedb1161b
![](eth-05.png)



### 2.4 solidity中还原出钱包地址
在solidity中通过签名和原始串还原出钱包地址
```
// SPDX-License-Identifier: GPL-3.0  solidity
pragma solidity ^0.8.4;
contract Test {
 
    /**
    * get hash
    * @param message The message will be used to hash.
    */
   function getHash(bytes memory message) internal pure returns (bytes32) {
        return
            keccak256(
                abi.encodePacked("\x19Ethereum Signed Message:\n18", message)
            );
    }
 
    /**
     * Extract the public key
     * @param hash hash for sign
     * @param sig signed
     */
    function ecrecovery(bytes32 hash, bytes memory sig) internal pure returns (address) {
        bytes32 h;
        bytes32 r;
        bytes32 s;
        uint8 v;
     
        if (sig.length != 65) {
          return address(0);
        }
     
        assembly {
          h := hash
          r := mload(add(sig, 32))
          s := mload(add(sig, 64))
          v := and(mload(add(sig, 65)), 255)
        }
     
        if (v < 27) {
          v += 27;
        }
     
        if (v != 27 && v != 28) {
          return address(0);
        }
     
        return ecrecover(h, v, r, s);
  }
 
    function test(bytes memory tid, bytes memory sign) external pure returns (address){
        bytes32 hashValue = getHash(tid);
        return ecrecovery(hashValue, sign);
    }
     
}
```
输入如下数据：

注意传给solidity必须是ASCII码格式的0x343537353432353834383233323039393834

tid：0x343537353432353834383233323039393834
sign：0x2d27aaeebc445d4fa1fba84e1261fa2431406fb49554ff88e76e892b5b59031a6adf3d151dd088e1bae2e488b40305965995ec2a9ea5c9b84f792beb6bedb1161b

![](eth-06.png)


获取到钱包地址：0x865Af88273B9218b94c46d12f34be460865B0A0d



### 2.5 javascript中的签名
sign-3.js
```
let ethUtil = require("ethereumjs-util");
let msg = "457542584823209984"; //457542584823209984
 
/**
 * 计算hash
 * @param {*} msg 源字符串
 */
function getHash(msg) {
  const data = ethUtil.toBuffer(Buffer.from(msg, "utf8"));
  const buf = Buffer.concat([
    Buffer.from(
      "\u0019Ethereum Signed Message:\n" + data.length.toString(),
      "utf8"
    ),
    data
  ]);
  const hash = ethUtil.keccak256(buf);
  return ethUtil.bufferToHex(hash);
}
 
const tid = ethUtil.toBuffer(Buffer.from(msg, "utf8"));
console.log("原始串：0x" + msg);
console.log("ASCII码：" + ethUtil.bufferToHex(tid));
 
var msgHash = ethUtil.toBuffer(getHash(msg));
 
/** 平台 */
let prikey = ethUtil.toBuffer("0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");
const rsv2 = ethUtil.ecsign(msgHash, prikey);
console.log("===============平台==================");
console.log("hash: 0x" + msgHash.toString('hex'));
console.log("r: 0x" + rsv2.r.toString('hex'));
console.log("s: 0x" + rsv2.s.toString('hex'));
console.log("v: " + rsv2.v.toString(16));
console.log("sign: 0x" + rsv2.r.toString('hex') + rsv2.s.toString('hex') + rsv2.v.toString(16));
```
输出：
```
DELL@WJF MINGW64 /d/workspace-viv/viv-solidity/viv/test
$ node sign-3
原始串：0x457542584823209984
ASCII码：0x343537353432353834383233323039393834
===============平台==================
hash: 0x8a590edede2c24dad749a698c8f018863fda7ae9d26b9d32716d45b46ee632c5
r: 0x2d27aaeebc445d4fa1fba84e1261fa2431406fb49554ff88e76e892b5b59031a
s: 0x6adf3d151dd088e1bae2e488b40305965995ec2a9ea5c9b84f792beb6bedb116
v: 1b
sign: 0x2d27aaeebc445d4fa1fba84e1261fa2431406fb49554ff88e76e892b5b59031a6adf3d151dd088e1bae2e488b40305965995ec2a9ea5c9b84f792beb6bedb1161b
```
可见hash与sign均与remix计算出来的相同





### 2.6 java代码中的签名
引入web3j
```
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>core</artifactId>
    <version>5.0.0</version>
</dependency>
```
hash与签名
```
package com.uecent.viv.provider.blockchain.module.common.util;
 
import org.bouncycastle.asn1.x9.X9IntegerConverter;
import org.bouncycastle.crypto.params.ECDomainParameters;
import org.bouncycastle.math.ec.ECAlgorithms;
import org.bouncycastle.math.ec.ECPoint;
import org.bouncycastle.math.ec.custom.sec.SecP256K1Curve;
import org.web3j.crypto.Credentials;
import org.web3j.crypto.ECDSASignature;
import org.web3j.crypto.ECKeyPair;
import org.web3j.crypto.Hash;
import org.web3j.utils.Assertions;
import org.web3j.utils.Numeric;
 
import java.math.BigInteger;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
 
import static org.web3j.crypto.Sign.CURVE_PARAMS;
 
/**
 * Web3j工具类
 *
 * @author ：wangjf
 * @date ：2021/7/23 17:06
 * @description：provider-viv-blockchain
 * @version: v1.1.0
 */
public class Test {
 
    static final ECDomainParameters CURVE;
    static {
        CURVE = new ECDomainParameters(CURVE_PARAMS.getCurve(), CURVE_PARAMS.getG(), CURVE_PARAMS.getN(), CURVE_PARAMS.getH());
    }
 
    public static void main(String[] args) {
        String text = "457542584823209984";
        String privateKey = "0xf95816c196aec67bdae2f72005d4b11203162c72ed4e3833958e052df8a32edc";
        System.out.println("hash: " + getHashHexString(text));
        System.out.println("sign: " + signMessage(text, privateKey));
    }
 
    /**
     * 签名
     *
     * @param message    消息
     * @param privateKey 私钥
     * @return
     */
    public static String signMessage(String message, String privateKey) {
        Credentials credentials = Credentials.create(privateKey);
        ECKeyPair ecKeyPair = credentials.getEcKeyPair();
        BigInteger publicKey = ecKeyPair.getPublicKey();
        byte[] messageHash = getHash(message);
        ECDSASignature sig = ecKeyPair.sign(messageHash);
        int recId = -1;
 
        int headerByte;
        for (headerByte = 0; headerByte < 4; ++headerByte) {
            BigInteger k = recoverFromSignature(headerByte, sig, messageHash);
            if (k != null && k.equals(publicKey)) {
                recId = headerByte;
                break;
            }
        }
 
        if (recId == -1) {
            throw new RuntimeException("Could not construct a recoverable key. Are your credentials valid?");
        } else {
            headerByte = recId + 27;
            byte[] v = new byte[]{(byte) headerByte};
            byte[] r = Numeric.toBytesPadded(sig.r, 32);
            byte[] s = Numeric.toBytesPadded(sig.s, 32);
 
            StringBuilder sign = new StringBuilder();
            sign.append(Numeric.toHexString(r)).append(Numeric.toHexString(s).substring(2)).append(Numeric.toHexString(v).substring(2));
            return sign.toString();
        }
    }
 
    public static BigInteger recoverFromSignature(int recId, ECDSASignature sig, byte[] message) {
        Assertions.verifyPrecondition(recId >= 0, "recId must be positive");
        Assertions.verifyPrecondition(sig.r.signum() >= 0, "r must be positive");
        Assertions.verifyPrecondition(sig.s.signum() >= 0, "s must be positive");
        Assertions.verifyPrecondition(message != null, "message cannot be null");
        BigInteger n = CURVE.getN();
        BigInteger i = BigInteger.valueOf((long) recId / 2L);
        BigInteger x = sig.r.add(i.multiply(n));
        BigInteger prime = SecP256K1Curve.q;
        if (x.compareTo(prime) >= 0) {
            return null;
        } else {
            ECPoint R = decompressKey(x, (recId & 1) == 1);
            if (!R.multiply(n).isInfinity()) {
                return null;
            } else {
                BigInteger e = new BigInteger(1, message);
                BigInteger eInv = BigInteger.ZERO.subtract(e).mod(n);
                BigInteger rInv = sig.r.modInverse(n);
                BigInteger srInv = rInv.multiply(sig.s).mod(n);
                BigInteger eInvrInv = rInv.multiply(eInv).mod(n);
                ECPoint q = ECAlgorithms.sumOfTwoMultiplies(CURVE.getG(), eInvrInv, R, srInv);
                byte[] qBytes = q.getEncoded(false);
                return new BigInteger(1, Arrays.copyOfRange(qBytes, 1, qBytes.length));
            }
        }
    }
 
    /**
     * 计算hash，该hash与solidity的keccak256(abi.encodePacked(message))一致
     *
     * @param message
     * @return
     */
    public static byte[] getHash(String message) {
        message = Numeric.cleanHexPrefix(message);
        String str = "\u0019Ethereum Signed Message:\n18" + message;
        byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
        return Hash.sha3(bytes);
    }
 
    /**
     * 计算hash，该hash与solidity的keccak256(abi.encodePacked(message))一致
     *
     * @param message
     * @return
     */
    public static String getHashHexString(String message) {
        return Numeric.toHexString(getHash(message));
    }
 
    private static ECPoint decompressKey(BigInteger xBN, boolean yBit) {
        X9IntegerConverter x9 = new X9IntegerConverter();
        byte[] compEnc = x9.integerToBytes(xBN, 1 + x9.getByteLength(CURVE.getCurve()));
        compEnc[0] = (byte) (yBit ? 3 : 2);
        return CURVE.getCurve().decodePoint(compEnc);
    }
 
}
```
输出：
```
"C:\Program Files\Java\jdk1.8.0_221\bin\java.exe" "-javaagent:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2021.1.3\lib\idea_rt.jar=6631:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2021.1.3\bin" -Dfile.encoding=UTF-8 -classpath C:\Users\DELL\AppData\Local\Temp\classpath1808972391.jar com.uecent.viv.provider.blockchain.module.common.util.Test
hash: 0x8a590edede2c24dad749a698c8f018863fda7ae9d26b9d32716d45b46ee632c5
sign: 0x2d27aaeebc445d4fa1fba84e1261fa2431406fb49554ff88e76e892b5b59031a6adf3d151dd088e1bae2e488b40305965995ec2a9ea5c9b84f792beb6bedb1161b

Process finished with exit code 0
```
可见计算结果跟上面两种都一致。
![](eth-07.png)




