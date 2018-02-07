..

DES
===

基本介绍
--------

DES全称为Data Encryption Standard，是典型的块加密，其基本信息如下

-  使用64位秘钥中的56位，剩余的8位要么丢弃，要么作为奇偶校验位
-  输入64位
-  输出64位。
-  明文经过16轮迭代得到密文。

给出一张 `简单的 DES 流程图 <http://homepage.usask.ca/~dtr467/400/>`__ 。

.. figure:: /crypto/symmetric/figure/des.gif
   :alt: 简单的 DES 流程图

再给一张比较 `详细的 DES 流程图 <http://bbs.pediy.com/thread-90593.htm>`__ 。

.. figure:: /crypto/symmetric/figure/des_details.jpg
   :alt: 详细的 DES 流程图

其中

-  S盒的设计标准并未给出。
