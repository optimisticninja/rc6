# rc6

RC6 block cipher implementation from the [paper](doc/586cc5d356330aef8a868aaa6c9bee493796.pdf) in C++ fully templated to accomodate different word sizes. RC6 was an AES candidate finalist that was [found being used in NSA implants](https://en.wikipedia.org/wiki/RC6#Possible_use_in_NSA_%22implants%22). The algorithm was an [RSA patent](https://patents.google.com/patent/US5835600A/en) but the patent expired between 2015 and 2017.

## Features

* GCM-SIV ([RFC here](doc/rfc8452.pdf)) mode of operation
    - Authentication AND encryption
* Key size up to 2048 bits
* Fully parameterized to support a variety of word lengths, key sizes and number of rounds.

### Planned Features

* Flag to use [metamorphic engine from Stone Cipher-192](doc/091101.pdf)
* Parallelize

## Usage

Doxygen generated files are in `doc/latex` and `doc/html`.

Copy [`types.h`](src/types.h), [`binops.h`](src/binops.h), and [`rc6.h`](src/rc6.h) to your source directory.

### In GCM-SIV mode

```cpp
#include "rc6/mode/aead.h"

int main()
{
    vector<u8> plaintext(8 * 32);
    vector<u8> aad(256);
    vector<u8> key_generating_key(16);
    AEAD<BlockType::BLOCK_128> aead = AEAD<BlockType::BLOCK_128>(key_generating_key);
    vector<u8> ciphertext = plaintext;
    
    // Encrypt
    aead.seal(ciphertext, aad);
    // Decrypt and authenticate
    aead.open(ciphertext, aad);
```

### As a lone block cipher

```cpp
#include "rc6/mode/rc6.h"

int main()
{
    // Initialize RC6 block cipher to use 128-bit blocks
    RC6<BlockType::BLOCK_128> rc6 = RC6<BlockType::BLOCK_128>();
    vector<u8> block(16);
    // Please create a random key, although RC5/RC6 has failed to reveal weakness in key-setup
    vector<u8> key(16);
    
    rc6.encrypt(block, key);
    rc6.decrypt(block, key);
}
```

## Building

### Requirements

* CMake >= 3.18
* GTest
* GMock

### The Build

```
$ mkdir build && cd build
$ cmake ..
$ make
```

## Running (from `build/` folder)

### Main program

The `main.cc` file runs the test vectors from the whitepaper.

```bash
$./src/rc6
========== TEST VECTOR #1 ==========
PLAIN   : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
KEY     : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
ENCRYPT : 8f c3 a5 36 56 b1 f7 78 c1 29 df 4e 98 48 a4 1e
DECRYPT : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
========== TEST VECTOR #2 ==========
PLAIN   : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
KEY     : 01 23 45 67 89 ab cd ef 01 12 23 34 45 56 67 78
ENCRYPT : 52 4e 19 2f 47 15 c6 23 1f 51 f6 36 7e a4 3f 18
DECRYPT : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
========== TEST VECTOR #3 ==========
PLAIN   : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
KEY     : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                      00 00 00 00 00 00 00 00
ENCRYPT : 6c d6 1b cb 19 0b 30 38 4e 8a 3f 16 86 90 ae 82
DECRYPT : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
========== TEST VECTOR #4 ==========
PLAIN   : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
KEY     : 01 23 45 67 89 ab cd ef 01 12 23 34 45 56 67 78
                      89 9a ab bc cd de ef f0
ENCRYPT : 68 83 29 d0 19 e5 05 04 1e 52 e9 2a f9 52 91 d4
DECRYPT : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
========== TEST VECTOR #5 ==========
PLAIN   : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
KEY     : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
          00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
ENCRYPT : 8f 5f bd 05 10 d1 5f a8 93 fa 3f da 6e 85 7e c2
DECRYPT : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
========== TEST VECTOR #6 ==========
PLAIN   : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
KEY     : 01 23 45 67 89 ab cd ef 01 12 23 34 45 56 67 78
          89 9a ab bc cd de ef f0 10 32 54 76 98 ba dc fe
ENCRYPT : c8 24 18 16 f0 d7 e4 89 20 ad 16 a1 67 4e 5d 48
DECRYPT : 02 13 24 35 46 57 68 79 8a 9b ac bd ce df e0 f1
```
### Tests

Tests run a Google Test suite that test constraints of the paper as well as the test vectors.

```bash
$ ./tests/tests
[==========] Running 39 tests from 6 test suites.
[----------] Global test environment set-up.
[----------] 11 tests from RC6
[ RUN      ] RC6.MagicConstantP32
[       OK ] RC6.MagicConstantP32 (0 ms)
[ RUN      ] RC6.MagicConstantQ32
[       OK ] RC6.MagicConstantQ32 (0 ms)
[ RUN      ] RC6.KeyAbove2048Bits
[       OK ] RC6.KeyAbove2048Bits (1 ms)
[ RUN      ] RC6.WordSize32Bit
[       OK ] RC6.WordSize32Bit (0 ms)
[ RUN      ] RC6.WordSize64Bit
[       OK ] RC6.WordSize64Bit (0 ms)
[ RUN      ] RC6.PaperTestVector1
[       OK ] RC6.PaperTestVector1 (0 ms)
[ RUN      ] RC6.PaperTestVector2
[       OK ] RC6.PaperTestVector2 (0 ms)
[ RUN      ] RC6.PaperTestVector3
[       OK ] RC6.PaperTestVector3 (0 ms)
[ RUN      ] RC6.PaperTestVector4
[       OK ] RC6.PaperTestVector4 (0 ms)
[ RUN      ] RC6.PaperTestVector5
[       OK ] RC6.PaperTestVector5 (0 ms)
[ RUN      ] RC6.PaperTestVector6
[       OK ] RC6.PaperTestVector6 (0 ms)
[----------] 11 tests from RC6 (1 ms total)

[----------] 3 tests from BinOps
[ RUN      ] BinOps.ROL
[       OK ] BinOps.ROL (0 ms)
[ RUN      ] BinOps.ROR
[       OK ] BinOps.ROR (0 ms)
[ RUN      ] BinOps.IsBigEndian
[       OK ] BinOps.IsBigEndian (0 ms)
[----------] 3 tests from BinOps (0 ms total)
...
[  PASSED  ] 39 tests.
```

## License

None, everyone deserves strong encryption.
