:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch2/keystore.rst

.. _reference-keystore:

资料篇：Keystore 与私钥保存
======================================

**私钥看上去是一串杂乱的数字，我如何保存它呢？**

首先读者能想到的是，裸露在外的私钥就等同于将自己保险库的钥匙拱手让人。
任何取到私钥的人都可以取走该账户持有的以太币。

所以将私钥妥善加密存储，是一个良好的习惯。当通过网络传输账户信息到另一个手机/电脑时，也需要通过加密来保障通讯时候的数据安全。

那么，如何合理地选择私钥加密方式呢？

曾经有人采用ZIP或者RAR压缩包的方法来加密私钥文件，这是一种妥善的好方法。

在以太坊的世界里，采用 :guilabel:`keystore` 格式加密存储是一种更加通用的方式，keystore格式可以被多数钱包APP及客户端所导入、导出。

例如，假设我们拥有一个钱包，信息如下。

+------+------------------------------------------------------------------+
| 地址 | 0x7c52e508C07558C287d5A453475954f6a547eC41                       |
+------+------------------------------------------------------------------+
| 私钥 | d88169685ceeaaaef5c619bcdaca3b44c323372e0d777720d4a28efe3c658aa2 |
+------+------------------------------------------------------------------+

好的，为了妥善保管该钱包，我们使用软件程序对该钱包进行加密，采用密码123456对其锁定。

keystore生成过程，历经4个步骤：
   - 通过 ``KDF`` 的算法的变种算法 ``Scrypt`` 算法，将我们选中的密码：123456，变换为一个 **AES-128-CTR** 对称加密算法所能采用的加密密匙 *S* 。
   - 使用该密匙 *S* 通过 **AES-128-CTR** 对称加密算法加密明文的以太坊私钥。
   - 将 2 步骤生成的结果保存为密文 *cyphertext* 。
   - 为了防止可能的篡改或数据变更，将 *cyphertext* 与 *S* 联合起来作为输入，使用 ``SHA3`` 哈希算法对该值进行带入求值，得到一个完整性校验签名。

keystore解密的过程与生成过程相反：
   - 用户输入准确的密码 123456 后，通过 ``KDF`` 的 ``Scrypt`` 算法先计算出解密密匙 *S’* 。
   - 如果密码正确，*S* = *S’* ，否则提示用户密码输入错误。
   - 最终通过 *S’*，经由 **AES-128-CTR** 对称加密算法反向计算出私钥。


加密后的keystore 如代码清单2-4所示：

.. code-block:: json
   :caption: 代码清单2-4

   {
     "version": 3,
     "id": "7d5d99c8-f455-49aa-8b89-6c795a7cdd46",
     "address": "7c52e508c07558c287d5a453475954f6a547ec41",
     "crypto": {
         "kdf": "scrypt",
         "kdfparams": {
             "dklen": 32,
             "salt": "a4f9677eaf6f72394da51e16695899ad3e9b4f2228ad4eca5ef2a5c36093fe12",
             "n": 262144,
             "r": 8,
             "p": 1
         },
         "cipher": "aes-128-ctr",
         "ciphertext": "d89df5ef74f51ae485308e6dce8991dd80674e111f8073f9efa52cb2dd6eca3f",
         "cipherparams": {
             "iv": "6b064c5b09a154d9877d3a07e610a567"
         },
         "mac": "30949eb085ce342a6a488fd51fa5e3231e45f7515efa10c19ea0d46270c73f06"
     }
   }

这串 JSON 格式的 keystore 代码可以被市面上大多数钱包 App 读取并合法导入。当导入时，仅需根据客户端程序提示输入 123456 将其解密即可。在网络中传输 keystore 是安全的，因为即使黑客窃取到了密文也不知道如何解密。

在 keystore 中的重要参数和对应的解释如下表。

+--------------+------------------------------------------------------------------------+
| 名词         | 解释                                                                   |
+--------------+------------------------------------------------------------------------+
| id           | 随机生成的一个 uuid 格式的字符串                                       |
+--------------+------------------------------------------------------------------------+
| address      | 该 keystore 对应的公开地址                                             |
+--------------+------------------------------------------------------------------------+
| crypto       | 加密后的密文区域以及加密参数                                           |
+--------------+------------------------------------------------------------------------+
| kdf          | 全称Key Derivation Function，加密密匙生成算法, 举例中选用了Scrypt 算法 |
+--------------+------------------------------------------------------------------------+
| kdfparams    | Scrypt 算法所需要的必要参数                                            |
+--------------+------------------------------------------------------------------------+
| cypher       | 加密私钥选用的对称加密算法，举例中选用了AES-128-CTR 算法               |
+--------------+------------------------------------------------------------------------+
| cyphertext   | 加密后的密码文                                                         |
+--------------+------------------------------------------------------------------------+
| cypherparams | AES-128-CTR 算法所需要的必要输入参数                                   |
+--------------+------------------------------------------------------------------------+
| mac          | cyphertext与加密密匙的校验值，防止keystore被中途篡改                   |
+--------------+------------------------------------------------------------------------+

.. Note::
   具体Javascript代码可以参考笔者的Github项目：http://github.com/laalaguer/VeChain-Address/