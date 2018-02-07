

Coppersmith Related Attack
==========================

基本原理
--------

首先，我们来简单介绍一下\ **Coppersmith method** 方法，该方法由\ `Don Coppersmith <https://en.wikipedia.org/wiki/Don_Coppersmith>`__
提出，可以用来找到单变量或者二元变量的多项式在模某个整数下的根，这里我们主要以单变量为主，假设我们有如下的一个在模N意义下的多项式F

.. math::


   F(x)=x^n + a_{n-1} x^{n-1} + \cdots + a_1x + a_0

假设该多项式在模N意义下有一个根\ :math:`x_0` ，这里我们令\ :math:`x_0 < M^{\frac{1}{n}}` 。如果等号成立的话，显然只有\ :math:`x^n` 这一项，那0就是，也满足。

**Coppersmith method** 主要是通过\ `Lenstra–Lenstra–Lovász lattice basis reduction
algorithm <https://en.wikipedia.org/wiki/Lenstra%E2%80%93Lenstra%E2%80%93Lov%C3%A1sz_lattice_basis_reduction_algorithm>`__ (LLL) 方法来找到与该函数具有相同根\ :math:`x_0`
但有更小系数的多项式。关于更加详细的介绍，请自行搜索。

Basic Broadcast Attack
----------------------

攻击条件
~~~~~~~~

如果一个用户使用同一个加密指数e加密了同一个密文，并发送给了其他e个用户。那么就会产生广播攻击。这一攻击由 Håstad 提出。

攻击原理
~~~~~~~~

这里我们假设e为3，并且加密者使用了三个不同的模数\ :math:`n_1,n_2,n_3` 给三个不同的用户发送了加密后的消息m，如下

.. math::


   \begin{align}
   c_1&=m^3\bmod n_1 \\
   c_2&=m^3\bmod n_2 \\
   c_3&=m^3\bmod n_3 \\
   \end{align}

这里我们假设\ :math:`n_1,n_2,n_3​` 互相互素，不然，我们就可以直接进行分解，然后得到d，进而然后直接解密。

同时，我们假设\ :math:`m<n_i, 1\leq i \leq 3` 。如果这个条件不满足的话，就会使得情况变得比较复杂，这里我们暂不讨论。

既然他们互素，那么我们可以根据中国剩余定理，可得\ :math:`m^3 \equiv C \bmod n_1n_2n_3` 。

此外，既然\ :math:`m<n_i, 1\leq i \leq 3` ，那么我们知道\ :math:`m^3 < n_1n_2n_3` 并且\ :math:`C<m^3 < n_1n_2n_3` ，那么\ :math:`m^3 = C` ，我们对C开三次根即可得到m的值。

对于较大的e来说，我们只是需要更多的明密文对。

SCTF RSA3 LEVEL4
~~~~~~~~~~~~~~~~

参考http://ohroot.com/2016/07/11/rsa-in-ctf。

这里我们以SCTF RSA3中的level4为例进行介绍，首先编写代码提取cap包中的数据，如下

.. code:: shell

    #!/usr/bin/env python

    from scapy.all import *
    import zlib
    import struct

    PA = 24
    packets = rdpcap('./syc_security_system_traffic3.pcap')
    client = '192.168.1.180'
    list_n = []
    list_m = []
    list_id = []
    data = []
    for packet in packets:
        # TCP Flag PA 24 means carry data
        if packet[TCP].flags == PA or packet[TCP].flags == PA + 1:
            src = packet[IP].src
            raw_data = packet[TCP].load
            head = raw_data.strip()[:7]
            if head == "We have":
                n, e = raw_data.strip().replace("We have got N is ",
                                                "").split('\ne is ')
                data.append(n.strip())
            if head == "encrypt":
                m = raw_data.replace('encrypted messages is 0x', '').strip()
                data.append(str(int(m, 16)))

    with open('./data.txt', 'w') as f:
        for i in range(0, len(data), 2):
            tmp = ','.join(s for s in data[i:i + 2])
            f.write(tmp + '\n')

其次，利用得到的数据直接使用中国剩余定理求解。

.. code:: python

    from functools import reduce
    import gmpy
    import json, binascii


    def modinv(a, m):
        return int(gmpy.invert(gmpy.mpz(a), gmpy.mpz(m)))


    def chinese_remainder(n, a):
        sum = 0
        prod = reduce(lambda a, b: a * b, n)
        # 并行运算
        for n_i, a_i in zip(n, a):
            p = prod // n_i
            sum += a_i * modinv(p, n_i) * p
        return int(sum % prod)


    nset = []
    cset = []
    with open("data.txt") as f:
        now = f.read().strip('\n').split('\n')
        for item in now:
            item = item.split(',')
            nset.append(int(item[0]))
            cset.append(int(item[1]))

    m = chinese_remainder(nset, cset)
    m = int(gmpy.mpz(m).root(19)[0])
    print binascii.unhexlify(hex(m)[2:-1])

得到密文，然后再次解密即可得到flag。

.. code:: shell

    H1sTaDs_B40aDcadt_attaCk_e_are_same_and_smA9l

题目
~~~~

-  2017 WHCTF OldDriver

Broadcast Attack with Linear Padding
------------------------------------

对于具有线性填充的情况下，仍然可以攻击，这时候就会使用\ **Coppersmith method** 的方法了，这里暂不介绍。可以参考

-  https://en.wikipedia.org/wiki/Coppersmith%27s_attack#Generalizations

Related Message Attack
----------------------

.. 攻击条件-1:

攻击条件
~~~~~~~~

当Alice使用同一公钥对两个具有某种线性关系的消息M1与M2 进行加密，并将加密后的消息C1，C2发送给了Bob时，我们就可能可以获得对应的消息M1与M2。这里我们假设模数为N，两者之间的线性关系如下

:math:`M_1 \equiv f(M_2) \bmod N`

其中f为一个线性函数，比如说\ :math:`f=ax+b`\ 。

在具有较小错误概率下的情况下，其复杂度为\ :math:`O(elog^2N)` 。

这一攻击由Franklin，Reiter提出。

.. 攻击原理-1:

攻击原理
~~~~~~~~

首先，我们知道\ :math:`C_1 \equiv M_1 ^e \bmod N` ，并且\ :math:`M_1 \equiv f(M_2) \bmod N` ，那么我们可以知道\ :math:`M_2` 是\ :math:`f(x)^e \equiv C_1 \bmod N`
的一个解，即它是方程\ :math:`f(x)^e-C_1` 在模N意义下的一个根。同样的，\ :math:`M_2` 是\ :math:`x^e - C_2` 在模N意义下的一个根。所以说\ :math:`x-M_2`
同时整除以上两个多项式。因此，我们可以求得两个多项式的最大公因子，如果最大公因子恰好是线性的话，那么我们就求得了\ :math:`M_2` 。需要注意的是，在e=3的情况下，最大公因子一定是线性的。

这里我们关注一下e=3，且\ :math:`f(x)=ax+b` 的情况。首先我们有

:math:`C_1 \equiv M_1 ^3 \bmod N` 且\ :math:`M_1 \equiv aM_2+b \bmod N`

那么我们有

:math:`C_1 \equiv (aM_2+b)^3 \bmod N` 且\ :math:`C_2 \equiv M_2^3 \bmod N`

我们需要明确一下我们想要得到的是消息M，所以需要将其单独构造出来。

首先，我们有式1

:math:`(aM_2+b)^3=a^3M_2^3+3a^2M^2b+3aM_2b^2+b^3`

再者我们构造如下式2

:math:`(aM_2)^3-b^3 \equiv (aM_2-b)(a^2M_2^2+aM_2b+b^2) \bmod N`

根据式1我们有

:math:`a^3M_2^3-2b^3+3b(a^2M_2^2+aM_2b+b^2) \equiv C_1 \bmod N`

继而我们有式3

:math:`3b(a^2M_2^2+aM_2b+b^2) \equiv C_1-a^3C_2+2b^3 \bmod N`

那么我们根据式2与式3可得

:math:`(a^3C_2-b^3)*3b \equiv (aM_2-b)( C_1-a^3C_2+2b^3 ) \bmod N`

进而我们有

:math:`aM_2-b=\frac{3a^3bC_2-3b^4}{C_1-a^3C_2+2b^3}`

进而

:math:`aM_2\equiv \frac{2a^3bC_2-b^4+C_1b}{C_1-a^3C_2+2b^3}`

进而

:math:`M_2 \equiv\frac{2a^3bC_2-b^4+C_1b}{aC_1-a^4C_2+2ab^3}=\frac{b}{a}\frac{C_1+2a^3C_2-b^3}{C_1-a^3C_2+2b^3}`

上面的式子中右边所有的内容都是已知的内容，所以我们可以直接获取对应的消息。

有兴趣的可以进一步阅读\ `A New Related Message Attack on RSA <https://www.iacr.org/archive/pkc2005/33860001/33860001.pdf>`__
以及\ `paper <https://www.cs.unc.edu/~reiter/papers/1996/Eurocrypt.pdf>`__\ 这里暂不做过多的讲解。

例子
~~~~

这里我们以SCTF rsa3中的level3为例进行介绍。首先，跟踪TCP流可以知道，加密方式是将明文加上用户的user id进行加密，而且还存在多组。这里我们选择第0组和第9组，他们的模数一样，解密脚本如下

.. code:: python

    import gmpy2
    id1 = 1002
    id2 = 2614

    c1 = 0x547995f4e2f4c007e6bb2a6913a3d685974a72b05bec02e8c03ba64278c9347d8aaaff672ad8460a8cf5bffa5d787c5bb724d1cee07e221e028d9b8bc24360208840fbdfd4794733adcac45c38ad0225fde19a6a4c38e4207368f5902c871efdf1bdf4760b1a98ec1417893c8fce8389b6434c0fee73b13c284e8c9fb5c77e420a2b5b1a1c10b2a7a3545e95c1d47835c2718L
    c2 = 0x547995f4e2f4c007e6bb2a6913a3d685974a72b05bec02e8c03ba64278c9347d8aaaff672ad8460a8cf5bffa5d787c72722fe4fe5a901e2531b3dbcb87e5aa19bbceecbf9f32eacefe81777d9bdca781b1ec8f8b68799b4aa4c6ad120506222c7f0c3e11b37dd0ce08381fabf9c14bc74929bf524645989ae2df77c8608d0512c1cc4150765ab8350843b57a2464f848d8e08L
    n = 25357901189172733149625332391537064578265003249917817682864120663898336510922113258397441378239342349767317285221295832462413300376704507936359046120943334215078540903962128719706077067557948218308700143138420408053500628616299338204718213283481833513373696170774425619886049408103217179262264003765695390547355624867951379789924247597370496546249898924648274419164899831191925127182066301237673243423539604219274397539786859420866329885285232179983055763704201023213087119895321260046617760702320473069743688778438854899409292527695993045482549594428191729963645157765855337481923730481041849389812984896044723939553
    a = 1
    b = id1 - id2


    def getmessage(a, b, c1, c2, n):
        b3 = gmpy2.powmod(b, 3, n)
        part1 = b * (c1 + 2 * c2 - b3) % n
        part2 = a * (c1 - c2 + 2 * b3) % n
        part2 = gmpy2.invert(part2, n)
        return part1 * part2 % n


    message = getmessage(a, b, c1, c2, n) - id2
    message = hex(message)[2:]
    if len(message) % 2 != 0:
        message = '0' + message

    print message.decode('hex')

得到明文

.. code:: shell

    ➜  sctf-rsa3-level3 git:(master) ✗ python exp.py
    F4An8LIn_rElT3r_rELa53d_Me33Age_aTtaCk_e_I2_s7aLL

当然，我们也可以直接使用sage来做，会更加简单一点。

.. code:: python

    import binascii

    def attack(c1, c2, b, e, n):
        PR.<x>=PolynomialRing(Zmod(n))
        g1 = x^e - c1
        g2 = (x+b)^e - c2

        def gcd(g1, g2):
            while g2:
                g1, g2 = g2, g1 % g2
            return g1.monic()
        return -gcd(g1, g2)[0]

    c1 = 0x547995f4e2f4c007e6bb2a6913a3d685974a72b05bec02e8c03ba64278c9347d8aaaff672ad8460a8cf5bffa5d787c5bb724d1cee07e221e028d9b8bc24360208840fbdfd4794733adcac45c38ad0225fde19a6a4c38e4207368f5902c871efdf1bdf4760b1a98ec1417893c8fce8389b6434c0fee73b13c284e8c9fb5c77e420a2b5b1a1c10b2a7a3545e95c1d47835c2718L
    c2 = 0x547995f4e2f4c007e6bb2a6913a3d685974a72b05bec02e8c03ba64278c9347d8aaaff672ad8460a8cf5bffa5d787c72722fe4fe5a901e2531b3dbcb87e5aa19bbceecbf9f32eacefe81777d9bdca781b1ec8f8b68799b4aa4c6ad120506222c7f0c3e11b37dd0ce08381fabf9c14bc74929bf524645989ae2df77c8608d0512c1cc4150765ab8350843b57a2464f848d8e08L
    n = 25357901189172733149625332391537064578265003249917817682864120663898336510922113258397441378239342349767317285221295832462413300376704507936359046120943334215078540903962128719706077067557948218308700143138420408053500628616299338204718213283481833513373696170774425619886049408103217179262264003765695390547355624867951379789924247597370496546249898924648274419164899831191925127182066301237673243423539604219274397539786859420866329885285232179983055763704201023213087119895321260046617760702320473069743688778438854899409292527695993045482549594428191729963645157765855337481923730481041849389812984896044723939553
    e=3
    a = 1
    id1 = 1002
    id2 = 2614
    b = id2 - id1
    m1 = attack(c1,c2, b,e,n)
    print binascii.unhexlify("%x" % int(m1 - id1))

结果如下

.. code:: shell

    ➜  sctf-rsa3-level3 git:(master) ✗ sage exp.sage
    sys:1: RuntimeWarning: not adding directory '' to sys.path since everybody can write to it.
    Untrusted users could put files in this directory which might then be imported by your Python code. As a general precaution from similar exploits, you should not execute Python code from this directory
    F4An8LIn_rElT3r_rELa53d_Me33Age_aTtaCk_e_I2_s7aLL

.. 题目-1:

题目
~~~~

-  hitcon 2014 rsaha

Coppersmith’s short-pad attack
------------------------------

.. 攻击条件-2:

攻击条件
~~~~~~~~

目前在大部分消息加密之前都会进行padding，但是如果padding的长度过短，也有\ **可能**\ 被很容易地攻击。

.. 攻击原理-2:

攻击原理
~~~~~~~~

我们假设爱丽丝要给鲍勃发送消息，首先爱丽丝对要加密的消息M进行随机padding，然后加密得到密文C1，发送给鲍勃。这时，中间人皮特截获了密文。一段时间后，爱丽丝没有收到鲍勃的回复，再次对要加密的消息M进行随机padding，然后加密得到密文C2，发送给Bob。皮特再一次截获。这时，皮特就\ **可能**\ 可以利用如下原理解密。

这里我们假设模数N的长度为k，并且padding的长度为\ :math:`m=\lfloor \frac{k}{e^2} \rfloor` 。此外，假设要加密的消息的长度最多为k-m比特，padding的方式如下

:math:`M_1=2^mM+r_1, 0\leq r_1\leq 2^m`

消息M2的padding方式类似。

那么我们可以利用如下的方式来解密。

首先定义

:math:`g_1(x,y)=x^e-C_1`

:math:`g_2(x,y)=(x+y)^e-C_2`

其中\ :math:`y=r_2-r_1` 。显然这两个方程具有相同的根M1。然后还有一系列的推导。。。

Known High Bits Message Attack
------------------------------

.. 攻击条件-3:

攻击条件
~~~~~~~~

这里我们假设我们首先加密了消息m，如下

:math:`C\equiv m^d \bmod N`

并且我们假设我们知道消息m的很大的一部分\ :math:`m_0` ，即\ :math:`m=m_0+x` ，但是我们不知道\ :math:`x` 。那么我们就有可能通过该方法进行恢复消息。

例子1
~~~~~

可以参考https://github.com/mimoo/RSA-and-LLL-attacks。

例子2
~~~~~

Factoring with High Bits Known
------------------------------

.. 攻击条件-4:

攻击条件
~~~~~~~~

当我们知道一个公钥中模数N的一个因子的较高位时，我们就有一定几率来分解N。

攻击工具
~~~~~~~~

请参考https://github.com/mimoo/RSA-and-LLL-attacks 。上面有使用教程。

.. 例子1-1:

例子1
~~~~~

参考https://github.com/mimoo/RSA-and-LLL-attacks 。这里我们关注下面的代码

.. code:: python

    beta = 0.5
    dd = f.degree()
    epsilon = beta / 7
    mm = ceil(beta**2 / (dd * epsilon))
    tt = floor(dd * mm * ((1/beta) - 1))
    XX = ceil(N**((beta**2/dd) - epsilon)) + 1000000000000000000000000000000000
    roots = coppersmith_howgrave_univariate(f, N, beta, mm, tt, XX)

其中，

-  必须满足 :math:`q\geq N^{beta}` ，所以这里给出了\ :math:`beta=0.5` ，显然两个因数中必然有一个是大于的。
-  XX是$f(x)=q’+x $ 在模q意义下的根的上界，自然我们可以选择调整它，这里其实也表明了我们已知的\ :math:`q'` 与因数q之间可能的差距。

.. 例子2-1:

例子2
~~~~~

这里我们以2016年HCTF中的RSA2为例进行介绍。

首先程序的开头是一个绕过验证的，我们大概搞搞，绕过即可，代码如下

.. code:: python

    from pwn import *
    from hashlib import sha512
    sh = remote('127.0.0.1', 9999)
    context.log_level = 'debug'
    def sha512_proof(prefix, verify):
        i = 0
        pading = ""
        while True:
            try:
                i = randint(0, 1000)
                pading += str(i)
                if len(pading) > 200:
                    pading = pading[200:]
                #print pading
            except StopIteration:
                break
            r = sha512(prefix + pading).hexdigest()
            if verify in r:
                return pading


    def verify():
        sh.recvuntil("Prefix: ")
        prefix = sh.recvline()
        print len(prefix)
        prefix = prefix[:-1]
        prefix = prefix.decode('base64')
        proof = sha512_proof(prefix, "fffffff")
        sh.send(proof.encode('base64'))
    if __name__ == '__main__':
        verify()
        print 'verify success'
        sh.recvuntil("token: ")
        token = "5c9597f3c8245907ea71a89d9d39d08e"
        sh.sendline(token)

        sh.recvuntil("n: ")
        n = sh.readline().strip()
        n = int(n[2:], 16)

        sh.recvuntil("e: ")
        e = sh.readline().strip()
        e = int(e[2:], 16)

        sh.recvuntil("e2: ")
        e2 = sh.readline().strip()
        e2 = int(e2[2:], 16)

        sh.recvuntil("is: ")
        enc_flag = sh.readline().strip()
        enc_flag = int(enc_flag[2:-1], 16)
        print "n: ", hex(n)
        print "e: ", hex(e)
        print "e2: ", hex(e2)
        print "flag: ", hex(enc_flag)

这里我们也已经得到n，e，e2，加密后的flag了，如下

.. code:: python

    n:  0x724d41149e1bd9d2aa9b333d467f2dfa399049a5d0b4ee770c9d4883123be11a52ff1bd382ad37d0ff8d58c8224529ca21c86e8a97799a31ddebd246aeeaf0788099b9c9c718713561329a8e529dfeae993036921f036caa4bdba94843e0a2e1254c626abe54dc3129e2f6e6e73bbbd05e7c6c6e9f44fcd0a496f38218ab9d52bf1f266004180b6f5b9bee7988c4fe5ab85b664280c3cfe6b80ae67ed8ba37825758b24feb689ff247ee699ebcc4232b4495782596cd3f29a8ca9e0c2d86ea69372944d027a0f485cea42b74dfd74ec06f93b997a111c7e18017523baf0f57ae28126c8824bd962052623eb565cee0ceee97a35fd8815d2c5c97ab9653c4553f
    e:  0x10001
    e2:  0xf93b
    flag:  0xf11e932fa420790ca3976468dc4df1e6b20519ebfdc427c09e06940e1ef0ca566d41714dc1545ddbdcae626eb51c7fa52608384a36a2a021960d71023b5d0f63e6b38b46ac945ddafea42f01d24cc33ce16825df7aa61395d13617ae619dca2df15b5963c77d6ededf2fe06fd36ae8c5ce0e3c21d72f2d7f20cd9a8696fbb628df29299a6b836c418cbfe91e2b5be74bdfdb4efdd1b33f57ebb72c5246d5dce635529f1f69634d565a631e950d4a34a02281cbed177b5a624932c2bc02f0c8fd9afd332ccf93af5048f02b8bd72213d6a52930b0faa0926973883136d8530b8acf732aede8bb71cb187691ebd93a0ea8aeec7f82d0b8b74bcf010c8a38a1fa8

接下来我们来分析主程序。可以看出

.. code:: python

        p, q, e = gen_key()
        n = p * q
        phi_n = (p-1)*(q-1)
        d = invmod(e, phi_n)
        while True:
            e2 = random.randint(0x1000, 0x10000)
            if gcd(e2, phi_n) == 1:
                break

我们的得到的n=p*q。而p，q，以及我们已知的e都在gen_key函数中生成。看一看gen_key函数

.. code:: python

    def gen_key():
        while True:
            p = getPrime(k/2)
            if gcd(e, p-1) == 1:
                break
        q_t = getPrime(k/2)
        n_t = p * q_t
        t = get_bit(n_t, k/16, 1)
        y = get_bit(n_t, 5*k/8, 0)
        p4 = get_bit(p, 5*k/16, 1)
        u = pi_b(p4, 1)
        n = bytes_to_long(long_to_bytes(t) + long_to_bytes(u) + long_to_bytes(y))
        q = n / p
        if q % 2 == 0:
            q += 1
        while True:
            if isPrime(q) and gcd(e, q-1) == 1:
                break
            m = getPrime(k/16) + 1
            q ^= m
        return (p, q, e)

其中我们已知如下参数

-  k=2048
-  e=0x10001

首先，程序先得到了1024比特位的素数p，并且gcd(2,p-1)=1。

然后，程序又得到了一个1024比特位的素数q_t，并且计算n_t=p*q_t。

下面多次调用了get_bit函数，我们来简单分析一下

.. code:: python

    def get_bit(number, n_bit, dire):
        '''
        dire:
            1: left
            0: right
        '''

        if dire:
            sn = size(number)
            if sn % 8 != 0:
                sn += (8 - sn % 8)
            return number >> (sn-n_bit)
        else:
            return number & (pow(2, n_bit) - 1)

可以看出根据dire(ction)的不同，会得到不同的数

-  dire=1时，程序首先计算number的二进制位数sn，如果不是8 的整数倍的话，就将sn增大为8的整数倍，然后返回number右移(sn-n_bit)的数字。其实 就是最多保留number的n_bit位。
-  dire=0时，程序直接获取number的低n_bit位。

然后我们再来看程序

.. code:: python

        t = get_bit(n_t, k/16, 1)
        y = get_bit(n_t, 5*k/8, 0)
        p4 = get_bit(p, 5*k/16, 1)

这三个操作分别做了如下的事情

-  t为n_t的最多高k/16，即128位，位数不固定。
-  y为n_t的低5*k/8位，即1280位，位数固定。
-  p4为p的最多高5k/16位，即640位，位数不固定。

此后，程序有如下操作

.. code:: python

        u = pi_b(p4, 1)

利用pi_b对p4进行了加密

.. code:: python

    def pi_b(x, m):
        '''
        m:
            1: encrypt
            0: decrypt
        ''' 
        enc = DES.new(key)
        if m:
            method = enc.encrypt
        else:
            method = enc.decrypt
        s = long_to_bytes(x)
        sp = [s[a:a+8] for a in xrange(0, len(s), 8)]
        r = ""
        for a in sp:
            r += method(a)
        return bytes_to_long(r)

其中，我们已知了秘钥key，所以只要我们有密文就可以解密。此外，可以看到的是程序是对传入的消息进行8字节分组，采用密码本方式加密，所以密文之间互不影响。

下面

.. code:: python

        n = bytes_to_long(long_to_bytes(t) + long_to_bytes(u) + long_to_bytes(y))
        q = n / p
        if q % 2 == 0:
            q += 1
        while True:
            if isPrime(q) and gcd(e, q-1) == 1:
                break
            m = getPrime(k/16) + 1
            q ^= m
        return (p, q, e)

程序将t，u，y拼接在一起得到n，进而，程序得到了q，并对q的低k/16位做了抑或，然后返回q’。

在主程序里，再一次得到了n’=p*q’。这里我们仔细分析一下

:math:`n'=p*(q+random(2^{k/16}))`

而p是k/2位的，所以说，random的部分最多可以影响原来的n的最低的\ :math:`k/2+k/16=9k/16` 比特位。

而，我们还知道n的最低的5k/8=10k/16 比特为其实就是y，所以其并没有影响到u，即使影响到也就最多影响到一位。

所以我们首先可以利用我们得到的n来获取u，如下

:math:`u=hex(n)[2:-1][-480:-320]`

虽然，这样可能会获得较多位数的u，但是这样并不影响，我们对u解密的时候每一分组都互不影响，所以我们只可能影响最高位数的p4。而p4的的高8位也有可能是填充的。但这也并不影响，我们已经得到了因子p的的很多部分了，我们可以去
尝试着解密了。如下

.. code:: python

    if __name__=="__main__":
        n = 0x724d41149e1bd9d2aa9b333d467f2dfa399049a5d0b4ee770c9d4883123be11a52ff1bd382ad37d0ff8d58c8224529ca21c86e8a97799a31ddebd246aeeaf0788099b9c9c718713561329a8e529dfeae993036921f036caa4bdba94843e0a2e1254c626abe54dc3129e2f6e6e73bbbd05e7c6c6e9f44fcd0a496f38218ab9d52bf1f266004180b6f5b9bee7988c4fe5ab85b664280c3cfe6b80ae67ed8ba37825758b24feb689ff247ee699ebcc4232b4495782596cd3f29a8ca9e0c2d86ea69372944d027a0f485cea42b74dfd74ec06f93b997a111c7e18017523baf0f57ae28126c8824bd962052623eb565cee0ceee97a35fd8815d2c5c97ab9653c4553f
        u = hex(n)[2:-1][-480:-320]
        u = int(u,16)
        p4 = pi_b(u,0)
        print hex(p4)

解密结果如下

.. code:: python

    ➜  2016-HCTF-RSA2 git:(master) ✗ python exp_p4.py 
    0xa37302107c17fb4ef5c3443f4ef9e220ac659670077b9aa9ff7381d11073affe9183e88acae0ab61fb75a3c7815ffcb1b756b27c4d90b2e0ada753fa17cc108c1d0de82c747db81b9e6f49bde1362693L

下面，我们直接使用sage来解密，这里sage里面已经实现了这个攻击，我们直接拿来用就好

.. code:: python

    from sage.all import *
    import binascii
    n = 0x724d41149e1bd9d2aa9b333d467f2dfa399049a5d0b4ee770c9d4883123be11a52ff1bd382ad37d0ff8d58c8224529ca21c86e8a97799a31ddebd246aeeaf0788099b9c9c718713561329a8e529dfeae993036921f036caa4bdba94843e0a2e1254c626abe54dc3129e2f6e6e73bbbd05e7c6c6e9f44fcd0a496f38218ab9d52bf1f266004180b6f5b9bee7988c4fe5ab85b664280c3cfe6b80ae67ed8ba37825758b24feb689ff247ee699ebcc4232b4495782596cd3f29a8ca9e0c2d86ea69372944d027a0f485cea42b74dfd74ec06f93b997a111c7e18017523baf0f57ae28126c8824bd962052623eb565cee0ceee97a35fd8815d2c5c97ab9653c4553f
    p4 =0xa37302107c17fb4ef5c3443f4ef9e220ac659670077b9aa9ff7381d11073affe9183e88acae0ab61fb75a3c7815ffcb1b756b27c4d90b2e0ada753fa17cc108c1d0de82c747db81b9e6f49bde1362693
    cipher = 0xf11e932fa420790ca3976468dc4df1e6b20519ebfdc427c09e06940e1ef0ca566d41714dc1545ddbdcae626eb51c7fa52608384a36a2a021960d71023b5d0f63e6b38b46ac945ddafea42f01d24cc33ce16825df7aa61395d13617ae619dca2df15b5963c77d6ededf2fe06fd36ae8c5ce0e3c21d72f2d7f20cd9a8696fbb628df29299a6b836c418cbfe91e2b5be74bdfdb4efdd1b33f57ebb72c5246d5dce635529f1f69634d565a631e950d4a34a02281cbed177b5a624932c2bc02f0c8fd9afd332ccf93af5048f02b8bd72213d6a52930b0faa0926973883136d8530b8acf732aede8bb71cb187691ebd93a0ea8aeec7f82d0b8b74bcf010c8a38a1fa8
    e2 = 0xf93b
    pbits = 1024
    kbits = pbits - p4.nbits()
    print p4.nbits()
    p4 = p4 << kbits
    PR.<x> = PolynomialRing(Zmod(n))
    f = x + p4
    roots = f.small_roots(X=2^kbits, beta=0.4)
    if roots:
        p = p4+int(roots[0])
        print "p: ", hex(int(p))
        assert n % p == 0
        q = n/int(p)
        print "q: ", hex(int(q))
        print gcd(p,q)
        phin = (p-1)*(q-1)
        print gcd(e2,phin)
        d = inverse_mod(e2,phin)
        flag = pow(cipher,d,n)
        flag = hex(int(flag))[2:-1]
        print binascii.unhexlify(flag)

关于small_roots的使用，可以参考\ `SAGE
说明 <http://doc.sagemath.org/html/en/reference/polynomial_rings/sage/rings/polynomial/polynomial_modn_dense_ntl.html#sage.rings.polynomial.polynomial_modn_dense_ntl.small_roots>`__\ 。

结果如下

.. code:: shell

    ➜  2016-HCTF-RSA2 git:(master) ✗ sage payload.sage
    sys:1: RuntimeWarning: not adding directory '' to sys.path since everybody can write to it.
    Untrusted users could put files in this directory which might then be imported by your Python code. As a general precaution from similar exploits, you should not execute Python code from this directory
    640
    p:  0xa37302107c17fb4ef5c3443f4ef9e220ac659670077b9aa9ff7381d11073affe9183e88acae0ab61fb75a3c7815ffcb1b756b27c4d90b2e0ada753fa17cc108c1d0de82c747db81b9e6f49bde13626933aa6762057e1df53d27356ee6a09b17ef4f4986d862e3bb24f99446a0ab2385228295f4b776c1f391ab2a0d8c0dec1e5L
    q:  0xb306030a7c6ace771db8adb45fae597f3c1be739d79fd39dfa6fd7f8c177e99eb29f0462c3f023e0530b545df6e656dadb984953c265b26f860b68aa6d304fa403b0b0e37183008592ec2a333c431e2906c9859d7cbc4386ef4c4407ead946d855ecd6a8b2067ad8a99b21111b26905fcf0d53a1b893547b46c3142b06061853L
    1
    1
    hctf{d8e8fca2dc0f896fd7cb4cb0031ba249}

.. 题目-2:

题目
~~~~

-  2016 湖湘杯 简单的RSA
-  2017 WHCTF Untitled

Boneh and Durfee attack
-----------------------

.. 攻击条件-5:

攻击条件
~~~~~~~~

当d较小时，满足\ :math:`d\leq N^{0.292}` 时，我们可以利用该工具，在一定程度上该要攻击比wiener attack要强一些。

.. 攻击原理-3:

攻击原理
~~~~~~~~

这里简单说一下原理

首先我们有

:math:`ed \equiv 1 \bmod \varphi(N)`

进而我们有

:math:`ed =k\varphi(N)+1` 即 :math:`k \varphi(N) +1 \equiv 0 \bmod e` 。

又

:math:`\varphi(N)=(p-1)(q-1)=qp-p-q+1=N-p-q+1`

所以

:math:`k(N-p-q+1)+1 \equiv 0 \bmod e`

我们假设\ :math:`A=N+1`\ ，\ :math:`y=-p-q` 那么

原式可化为

:math:`f(k,y)=k(A+y)+1 \equiv 0 \bmod e`

如果我们求得了该二元方程的根，那么我们自然也就可以解一元二次方程(\ :math:`N=pq,p+q=-y`)来得到p与q。

.. 攻击工具-1:

攻击工具
~~~~~~~~

请参考https://github.com/mimoo/RSA-and-LLL-attacks。上面有使用教程。

.. 例子-1:

例子
~~~~

这里我们以2015年PlaidCTF-CTF-Curious为例进行介绍。

首先题目给了一堆N，e，c。简单看一下可以发现该e比较大。这时候我们可以考虑使用wiener attack，这里我们使用更强的目前介绍的攻击。

核心代码如下

.. code:: python

        nlist = list()
        elist = list()
        clist = list()
        with open('captured') as f:
            # read the line {N : e : c} and do nothing with it
            f.readline()
            for i in f.readlines():
                (N, e, c) = i[1:-2].split(" : ")
                nlist.append(long(N,16))
                elist.append(long(e,16))
                clist.append(long(c,16))
        
        for i in range(len(nlist)):
            print 'index i'
            n = nlist[i]
            e = elist[i]
            c = clist[i]
            d = solve(n,e)
            if d==0:
                continue
            else:
                m = power_mod(c, d, n)
                hex_string = "%x" % m
                import binascii
                print "the plaintext:", binascii.unhexlify(hex_string)
                return

结果如下

.. code:: shell

    === solution found ===
    private key found: 23974584842546960047080386914966001070087596246662608796022581200084145416583
    the plaintext: flag_S0Y0UKN0WW13N3R$4TT4CK!
