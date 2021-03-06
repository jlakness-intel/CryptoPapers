# History and Progress of FHE

- What is FHE: Cloud computing on encrypted data
- Pre-FHE
    - 1978 Rivest et al "On data banks and privacy homomorphisms"
- First Generation: 
    - Single Bit bootstrapping on lattices, one bootstrap
    - 2009: "Fully Homomorphic encryption over idea lattices"
    - Fully Homomophic Encryption over the Integers
    - Implementing Gentry's Fully-Homomorphic Encryption Scheme
- Second Generation: 
    - Ciphertext Packing, multiple levels
    - Efficient Fully Homomorphic encryption from standard LWE
    - Leveled Fully homomorphic encryption without bootstrapping
    - fully homomorphic simd operations
    - fully homomorphic encryption with polylog overhead
    - 12 orders of magnitude speed improvement from 2010 to 2014 (5 papers)
- Third Generation FHE: 
    - Lower noise, Smaller parameters, single bits again
    - HE from LWE: simpler, faster, attribute-based
    - Lattice-based FHE as secure as PKE
    - Faster Bootstrapping eith Polynomial Error
    - FHEW: Bootstrapping Homomorphic Encryption
    - Faster Fully Homomorphic Encryption
- 4th Generation: Floating Point
    - CKKS: Homomorphic Encryption for Arithemetic of Approximate numbers
    - > Idea: Can CKKS be extended to arbitrary Lie algebras (e.g. quarternions)?
- Chimeric FHE
    - Switching between FHE schemes inside encrypted context
    - TFHE: fast sully homomorphic encryption over the torus
    - CHIMERA: Combining ring-lwe-based fully homomorphic encryption schemes
- 10 orders of magnitude in 10 years: 300ns/bit
- Applications
    - GWAS (linear regression)
    - Cryptonets
    - NN speech recognition (1 second inference time)
    - Private information retrieval
        - XPIR, SealPIR
        - Compressible FHE with applications to PIR
            - Plaintext/ciphertext  = 4/9
            - Implementation shows retrieval is as fast as CPU encrypting with AES
- Accelleration
    - DPRIVE
    - F1: As fast and programmable accelerator for fully homomorphic encryption
    - Over 100x faster boostrapping in fully homomorphic encryption through memory-centric optimization with GPUs
- Usability
    - github./com/jonaschn/awesome-he
    - Google: A Generap Purpose Transpiler for Fully Homomorphic Encryption
- Standardization
    - NIST post-quantum
    - Draft standard for homomorphic encryption schemes


# How homomorphic encryption schemes are built

- PKE overview (https://youtu.be/487AjvFW1lk?t=1615)
    - (sk,pk) = K(entropy)
    - c = E(pk,m)
    - m = D(sk,c)
    - Correctness: m = D(sk,E(pk,m)) for all valid parameters
    - Semantic security: distributions of E(pk,0),E(pk,1) are indistinguishable
- Homomorphic 
    - V: function performing homomophism of function on ciphertext: 
    - D(sk,V(pk,f,c)) = f(m)
    - works for family of functions f $\in$ F, such as arithmetic circuits
- Two ways to develop HE
    - Start with established crypto, make PKE, make PKE homomorphic
    - start with homomorphism, build PKE, create semantic security
- Rothblum: Homomorphic Encryption: From Private-Key to Public-Key (https://youtu.be/487AjvFW1lk?t=2242)
    - <img src="Screen Shot 2021-11-17 at 8.37.54 AM.png" width=500>
    - *Discussion:* Why do we care about forming a public key system?
    - Generate k samples of encryptions of 0 and 1 as the public key: $pk_i = {c_i,d_i}, i\in{1...k},c_i=E(sk,0),d_i=E(sk,1)$
    - *Discussion:* What is missing from the definition of the encryption function? What is being assumed about the distributions of these samples? What if the scheme is 'strongly homomorphic w.r.t the identity function'? 
    - Encrypt a single bit $m\in\{0,1\}$: 
        - pick a random set of pk with parity of the number of elements equal to the message bit: $S \subset pk, m = |S|\%2 $ 
        - Perform homomorphic addition over entire set, and return result.
        > *Discussion:* What are the semantic security requirements for this result? What then does that require of the distribution of samples?
    - > *Discussion:* What is the private key for the PKE scheme?
- Goldwasser-Micali a.la. Rothblum (https://youtu.be/487AjvFW1lk?t=2360)
    - <img src="Screen Shot 2021-11-17 at 8.37.09 AM.png" width=500>
    - start with multiplicative homomorphism of the legendre symbol (quadratic residue funtion: 0 if congruent to the base, 1 if there exists an square that is congruent to 1 under the base modulus, -1 otherwise)
    - secret key is an integer s
    - Ciphertext is an integer < s
    - Plaintext is the legendre symbol function of the ciphertext relative to the secret key base
    - > *Discussion:* Does the rothblum public key reveal the private key?
- Quantum attacks
    - "Quantum kills all HE schemes using abelian groups!"
    - Thm.(Watrous) Quantum can compute the cardinality of a group 
    - get a zero by chosen plaintext, use as generator to find cardinality
    - test ciphertext to determine whether cardinality is same as zero group
    - > *Discussion:* What if subgroup sizes are equal or close? Is quantum really this accurate?
    - > Discussion: What if we ignore quantum resistance, mitigate CPT,CCT attacks with entropy?
- Ring Homomorphisms: addition and multiplication
    - modular reduction or polynomial evaluation
    - Zeros are still form a subgroup under addition, so quantum attack applies
    - linear algebra can distinguish between subgroups (zeros are regularly-spaced)
- Modular reduction
    - Rivest, Adelman, Dertouzos
        - secret is a modulus p of the plaintext
        - public key is modulus p*q, and ciphertext operations use this modulus
        - attack: gcd of encryptions of zero is secret modulus
        - <img src="Screen Shot 2021-11-17 at 9.18.13 AM.png" width=500>
    - Van Dijk et al
        - RAD with a little bit of noise added
        - PKE HE scheme a.la. Rothblum
            - y: encryptions of zero
            - z: (encryptions of zero) + 1
        - Decryption: (c % p)%2  
        - <img src="Screen Shot 2021-11-17 at 9.21.09 AM.png" width=500>
        - > *Discussion:* Does the ciphertext meet semantic security requirements? Does the homomorphism actually break parity?
- Polynomial Evaluation
    - secret is an evaluation point s
    - ciphertext is a polynomial
    - Encryptions of zero are polynomials with common root (the secret), discoverable with linear algebra
    - adding noise can defeat linear algebra
    - LWE Assumption [regev 2005]: linear polynomnials that evaluate to a small number are indistinguishable from random linear polynomials
    - <img src="Screen Shot 2021-11-17 at 9.49.01 AM.png" width=500>
    - Decryption is evaluation of the ciphertext (polynomial) at the secret, mod 2
    > Discussion: Is the parity groups distinguishable in the ciphertext? Can a chosen plaintext attack discover properties of the zero group?
    - ciphertext polynomial grows with each multiplication
        - solution is to provide (as part of public key) polynomials with single quadratic terms from the zero group
        - those can be used to subtract off the quadratic terms
        - > Discussion: How many terms is this? Does this reveal anything about the zero group?
        - > Discussion: How else might the degree be controlled?
- Bootstrapping
    - Problem: noise terms grow to overcome the modulus
    - Solution: remove noise by decryption
    - Requires encryption and decryption performed homomorphically inside an encrypted cyphertext
    - Encryption of 1 bit results in n bits (only have 1-bit encryption)
    - Those bits are then encrypted separately to n sets of n bits
    - Operations must be performed across those n sets of bits
    - Use a bit-wise NAND homomorphism in the double-encrypted context
    - The cyphertext space forms a "magma" (complex unstructured algebraic structure)
    - > Discussion: Is a simpler algebraic structure possible?
    - > Discussion: How much simpler does this become if the plaintext symbols encrypt to a single ciphertext symbol?
    - > Idea: Probabalistic homomorphism $Pr[m'!=m]<e$, possibly combined with thresholding error-correction (e.g. Parity, Reed-Solomon or LDPC)
- New Homomorphisms to explore
    - Abelian groups are solvable, so we need to use noise, what about Non-Abelian groups?
    - Nuida: "Toward Constructing FHE without Ciphertext Noise from Group Theory
    - Interested in finding a multivariate crypto homomorphism
    - > Idea: Are there non-arithemetic homomorphisms (NAND/XOR/LUT)