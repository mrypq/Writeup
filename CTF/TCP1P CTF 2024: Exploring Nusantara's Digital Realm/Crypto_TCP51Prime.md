## [Crypto] TCP51Prime
```
plzz check my prime
```

`chall.py` が渡される。  
```python
from secret import a,b,p,flag
from hashlib import sha512
from pwn import xor
from Crypto.Util.number import isPrime

assert p == a**51 + 51*b**51 and isPrime(p) and a > 0 and b > 0
print(hex(p)[2:])
print(xor(flag,sha512((str(a)+str(b)).encode()).digest()).hex())

# 1cec7c3ff93ca538e71f334e83d905eabd894729a1b515b89dc2392036bc7e5d59fad2c35dbb0a8903c8bb2e9cd5e4779a92d3f361eb1ce9fa2530c47881a8719763f828360138373ffa2ce627f8ccad08e9b5ead178c614f7899adc6a14fa33aa2216c463a04041e78cffa2c68963c6ff422a076bedd32236282eb3bd1b7ba870a3b1c8f639cd536cba329b10a6cf7b4ef78bd11052ff8a0d432fb6d3b221742aa1da6914876e94aca5abdaeef30acdfc90cbc621245ad288a634e8bdf4152ea8ed0062c872ace7b4011dc5743fa9c424413f4e3e8fa5c5513fd4a711141f2ef68c01177f78945db623ac6cc762a6813f11cc278a143ea657bf309e281ef59048a29f279c9ad8b37f221ac06242f577bba985a2aaec051d95391a9681f905472f4e7d1322da492639ee4a5ac776a476cece55f9dfb782c1ef869deed2226691d3157fbb6b131968ebfb1fe5bc1e44a158f1e2321c001355cc9cb3344f6b09b78d965a807cd60d58a9efbab8c6d4f75c8e5ac7c9cf0e5409b55bb2133129272685913be02166c6bffe3747ccd186b6c26fc9f09
# 43edcf6275293ce97d716f49875c4bdba37f6ab30f15a53f09b72bf3816edf6b92618fb56d569d911b2f6fe7a36d4a854022dddf671dc89b4800bc1605822aab72d3
```

### 準備
`a`, `b`, `flag` がわからない状態。  
assert文より
- $p$ は素数
- $p = a^{51} + {51}b^{51}$
- $a, b > 0$

なので$a^{51} + {51}b^{51} \equiv 0\pmod p$  

$a^{51} = -{51}b^{51} + p$だから
- $a^{51} \equiv -{51}b^{51}\pmod p$  

両辺を${51}b^{51}$で割って
- $(\frac{a}{b})^{51} \equiv -{51}\pmod p$  

51乗根をとって
- $\frac{a}{b} \equiv \sqrt[51]-{51}\pmod p$  

### 解法
$n \equiv \frac{r}{s}\pmod m$から有理数$\frac{r}{s}$を求められる[有理数の再構成](https://en.wikipedia.org/wiki/Rational_reconstruction_(mathematics))を利用する。  
今回は$r$, $s$, $m$がそれぞれ$a$, $b$, $p$に対応している。  
この方法は$a$, $b$に対して$p$が十分に大きい必要があるが、$p = a^{51} + {51}b^{51}$なので問題なさそう。  

$r$, $s$の取りうる範囲を
- $|a| < N$
- $0<b<D$
- $p>2ND$

で仮定した時の解が一意の解になる。  
今回は$p = a^{51} + {51}b^{51}$より、ざっくり
- $N = p^{\frac{1}{51}}$
- $D = (\frac{p}{51})^{\frac{1}{51}}$

とする。($p>2ND$も満たす)  

- 入力 `x` に対して$mod$ = `mod` での $n$乗根を求める関数 `nth_root()`
- 有理数の再構成を行う関数 `rational_reconstruction()`

をそれぞれ実装し、flagを復号した。


```python
import math
import mpmath
from sympy import mod_inverse
from hashlib import sha512
from pwn import xor


def nth_root(x: int, n: int, mod: int):
    base = x % mod
    exp = mod_inverse(n, mod-1)
    return pow(base, exp, mod)


def rational_reconstruction(n: int, m: int, N: int, D: int):
    # puts v = (m, 0) and w = (n, 1)
    v1, v2 = m, 0
    w1, w2 = n, 1

    # One then repeats the following steps until the first component of w becomes ≤ N
    while w1 > N:
        # puts q = v1/w1
        q = math.floor(v1 / w1)
        # puts z = v - qw
        z1, z2 = v1 - q*w1, v2 - q*w2
        # The new v and w are then obtained by putting v = w and w = z
        v1, v2 = w1, w2
        w1, w2 = z1, z2

    return abs(w1), abs(w2)


p = 0x1cec7c3ff93ca538e71f334e83d905eabd894729a1b515b89dc2392036bc7e5d59fad2c35dbb0a8903c8bb2e9cd5e4779a92d3f361eb1ce9fa2530c47881a8719763f828360138373ffa2ce627f8ccad08e9b5ead178c614f7899adc6a14fa33aa2216c463a04041e78cffa2c68963c6ff422a076bedd32236282eb3bd1b7ba870a3b1c8f639cd536cba329b10a6cf7b4ef78bd11052ff8a0d432fb6d3b221742aa1da6914876e94aca5abdaeef30acdfc90cbc621245ad288a634e8bdf4152ea8ed0062c872ace7b4011dc5743fa9c424413f4e3e8fa5c5513fd4a711141f2ef68c01177f78945db623ac6cc762a6813f11cc278a143ea657bf309e281ef59048a29f279c9ad8b37f221ac06242f577bba985a2aaec051d95391a9681f905472f4e7d1322da492639ee4a5ac776a476cece55f9dfb782c1ef869deed2226691d3157fbb6b131968ebfb1fe5bc1e44a158f1e2321c001355cc9cb3344f6b09b78d965a807cd60d58a9efbab8c6d4f75c8e5ac7c9cf0e5409b55bb2133129272685913be02166c6bffe3747ccd186b6c26fc9f09
enc = bytes.fromhex('43edcf6275293ce97d716f49875c4bdba37f6ab30f15a53f09b72bf3816edf6b92618fb56d569d911b2f6fe7a36d4a854022dddf671dc89b4800bc1605822aab72d3')
N = int(mpmath.root(p, 51))
D = int(mpmath.root(p//51, 51))

root = nth_root(-51, 51, p)

a, b = rational_reconstruction(root, p, N, D)

flag = xor(enc, sha512((str(a)+str(b)).encode()).digest())
print(flag.decode())

```

`TCP1P{prime_prime_prime_prime_prime_prime_prime_prime_prime_prime}`