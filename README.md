# c32check

[Crockford base-32](https://en.wikipedia.org/wiki/Base32#Crockford's_Base32) encoding library
with 4-byte checksum.

This library is meant for generating and decoding Stacks addresses on the
Blockstack blockchain.

## How it works

Each c32check string encodes a 1-byte version and a 4-byte checksum.  When
decoded as a hex string, the wire format looks like this:

```
0      1                             n+1             n+5
|------|-----------------------------|---------------|
version     n-byte hex payload          4-byte hash
```

If `version` is the version byte (a 1-byte `number`) and `payload` is the raw 
bytes (e.g. as a `string`), then the `checksum` is calculated as follows:

```
checksum = sha256(sha256(version + payload)).substring(0,4)
```

In other words, the checksum is the first four bytes of the
double-sha256 of the bytestring concatenation of the `version` and `payload`.
This is similar to base58check encoding, for example.

## c32 Addresses

Specific to Blockstack, the Stacks blockchain uses c32-encoded public key
hashes as addresses.  Specifically, a **c32check address** is a c32check-encoded
ripemd160 hash.

# Examples

```
> c32 = require('c32check')
{ c32encode: [Function: c32encode],
  c32decode: [Function: c32decode],
  c32checkEncode: [Function: c32checkEncode],
  c32checkDecode: [Function: c32checkDecode],
  c32address: [Function: c32address],
  c32addressDecode: [Function: c32addressDecode],
  versions: 
   { mainnet: { p2pkh: 22, p2sh: 20 },
     testnet: { p2pkh: 26, p2sh: 21 } } }
```

## c32encode, c32decode

```
> c32check.c32encode(Buffer.from('hello world').toString('hex'))
'38CNP6RVS0EXQQ4V34'
> c32check.c32decode('38CNP6RVS0EXQQ4V34')
'68656c6c6f20776f726c64'
> Buffer.from('68656c6c6f20776f726c64', 'hex').toString()
'hello world'
```

## c32checkEncode, c32checkDecode

```
> version = 12
12
> c32check.c32checkEncode(version, Buffer.from('hello world').toString('hex'))
'CD1JPRV3F41VPYWKCCGRMASC8'
> c32check.c32checkDecode('CD1JPRV3F41VPYWKCCGRMASC8')
[ 12, '68656c6c6f20776f726c64' ] 
> Buffer.from('68656c6c6f20776f726c64', 'hex').toString()
'hello world'
```

# c32address, c32addressDecode

**NOTE**: these methods only work on ripemd160 hashes

```
> hash160 = 'a46ff88886c2ef9762d970b4d2c63678835bd39d'
'a46ff88886c2ef9762d970b4d2c63678835bd39d'
> version = c32check.versions.mainnet.p2pkh
22
> c32check.c32address(version, hash160)
'SP2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKNRV9EJ7'
> c32check.c32addressDecode('SP2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKNRV9EJ7')
[ 22, 'a46ff88886c2ef9762d970b4d2c63678835bd39d' ]
```
