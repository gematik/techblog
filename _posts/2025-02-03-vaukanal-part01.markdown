---
layout: post
title:  "The VAU channel to the Aktensystem"
date:   2025-02-03 10:10:10 +0200
author: Mike Kurtze
categories: tech
tags: Audits
---

A trusted runtime environment (VAU) is a technical measure to ensure that data in the ePA file system can be processed in plain text on the server side without individual employees of the operator having access to this data.

For this purpose, several web interfaces are available in the elektronische Patientenakte (ePA) Aktensystem, through which several requests can be sent via the HTTPS protocol. This is done via a TLS connection that each client (e.g. Praxisverwaltungssystem, PVS) must establish first. However, the requests are processed by a VAU instance located behind the Aktensystem, so that an additional encryption channel, the so-called VAU channel, is established between the client and the VAU instance.

Example of a VAU connection bewtween PVS and VAU:

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/overview_vau.png" alt="Overview VAU Channel" width="501" height="210"/>

## The VAU Encryption Protocol?

The purpose of the VAU protocol is to establish an encrypted channel to the VAU instance. Encryption is performed symmetrically with AES-256 and GCM mode. However, a prerequisite for encryption is that both sides agree on two symmetric keys for AES encryption. One key is called client2server (c2s) and is used to encrypt communication data sent from the PVS to the VAU instance, for example. The second key, called server2client (s2c), is used when sending communication data from the VAU instance to the PVS. In contrast to conventional TLS connections, where both sides use a shared key, two different symmetric keys are used here. Of course, both keys must be present on both sides so that each side can correctly decrypt the data from the other side.

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/vaumessages.png" alt="VAU Messages" width="530" height="221"/>

In order for both sides to calculate the same symmetric keys (key negotiation), the asymmetric algorithms Elliptic Curves in combination with Diffie Hellman, as well as the Cyberalgorithm are used. The exchange of the calculations of these algorithms takes place in the 4 messages, called “VAUMessage1, VAUMessage2, VAUMessage3 and VAUMessage4”.

## How does the Principle of Algorithms work in the VAU Protocol?

### The Algorithm Elliptic Curve in combination with Diffie Hellman
To understand how the complexity of the VAU protocol works, we will first take a closer look at the algorithms used and how they work. Similar to a TLS connection, the client and server must first agree on a shared secret (ss1) to be used to encrypt and decrypt the information. In today's TLS connections, the asymmetric algorithm Elliptic Curve is used in combination with the key exchange method Diffie Hellmann in TLS 1.3 and later. The Diffie-Hellman algorithm was developed in 1976 by Whifield Diffie and Martin Hellman to establish a secure shared key between two parties over an insecure communcation channel. The process allows to communicate in encrypted form, without the key having to be exchanged over the actual communication channel. A 32-byte shared secret is currently calculated here.

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/sharedsecretecdh.png" alt="Shared Secret with ECDH" width="531" height="711"/>

**Step 1**: The server (e.g. VAU instance) and the client (e.g. PVS) generate their own private asymmetric key pair of the elliptical curve SECP256R1.

- Elliptic curve SECP256R1: 3rd-order parabola, defines point G, n → order of the elliptic curve
- Private Key(VAU):  1 <= d1 < n, d1 → random number
- Private Key(PVS):  1 <= d2 < n, d2 → random number
- Public Key(VAU): Point Q1 = d1 * G
- Public Key(PVS): Point Q2 = d2 * G

**Step 2**: The client calculates its public key (Q2) and sends it to the server. This is done in VAU message 1. The server also calculates its public key (Q1) and sends it to the client. This is done in VAU message 2.

**Step 3**: Both sides now calculate the same shared secret (ss1). To do this, both sides have to multiply their own private key by the other's public key. This way, both sides calculate the same result. Since scalar multiplication is commutative on elliptic curves, the result of the calculation is the same on both sides. The resulting shared secret (again a point on the elliptic curve) is processed further to obtain a key derivation for the AES symmetric algorithm (ss1). 

- Server: ss1 = Q2 * d1 → d2 * d1 * G
- Client: ss1 = Q1 * d2 → d1 * d2 * G

### The Kyber-Algorithm
Unlike TLS, both sides must agree on a further shared secret (ss2). In the VAU channel, the modern encryption algorithm Kyber is also used, which was developed as part of post-quantum cryptography. The algorithm uses mathematical structures based on lattice problems, which means that the security of the method is based on the difficulty of solving certain problems in high-dimensional lattices that are difficult to crack even with quantum computers. The application of Kyber is intended to ensure that VAU encryption is more robust against future attacks using quantum computers.

In this procedure, for example, the client generates a cyber key pair consisting of a public and private key. The public key is then transferred to the server. A secret shared secret is randomly generated in the server and encrypted using the client's public cyber key. The encrypted ciphertext is then transferred to the client, which can decrypt the ciphertext using its matching private key. In this way, a key exchange for the symmetric algorithm AES-256 is also carried out using Kyber, and both sides again have the same shared secrets.

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/sharedsecretkyber.png" alt="Shared Secret with Kyber" width="521" height="701"/>

**Step 1**: The client (e.g. PVS) generates its own private asymmetric key pair for the cyber-algorithm.

**Step 2**: The client sends its public key to the server (e.g. VAU instance). The server, however, creates a randomly generated shared secret (ss2).

**Step 3**: The server now encrypts the shared secret (ss2) using a cyber algorithm and the client's public key. The procedure used by Kyber is called the “Key Encapsulation Mechanism (KEM)”. The encrypted shared secret (encss2) is now sent to the client.

**Step 4**: Since only the client has the corresponding private key, it can decrypt the encrypted shared secret (encss2) using the cyber algorithm. The result is again the shared secret (ss2). Now both sides have the same shared secret.


### The Key Derivation Function (KDF)

Both sides now have the same two shared secrets (ss1 and ss2). Finally, both sides have to derive two symmetric keys for the AES-256 from these shared secrets. The key derivation function (KDF) is used for this.

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/kdf.png" alt="Symmetric key generation with KDF" width="411" height="342"/>

**Step 1**: Both shared secrets are passed to the key derivation function. Both shared secrets are concatenated into an array. The array is then passed to the HMAC. HMAC stands for “Hash-based Message Authentication Code” and is used in the VAU channel with the hash algorithm SHA-256. The HMAC is used to create a 64-byte key, which is then halved into two symmetrical 32-byte keys.

**Step 2**: This results in two symmetrical keys, each 32 bytes long, for AES-256. The first key (AES c2s) is used to encrypt and decrypt data from the client to the server. The second key (AES s2c) is used to encrypt and decrypt data from the server to the client.


### The symmetric Algorithm AES-GCM

The VAU channel is only fully established when both sides have the two symmetrical keys, and information such as e-prescriptions can then be exchanged between the two sides in encrypted form. Depending on the direction of the data, the data is encrypted and decrypted with the corresponding symmetrical key. For this purpose, a symmetric algorithm like the AES must be used. The last algorithm we will look at is AES with its GCM mode. AES stands for Advanced Encryption Standard and is a symmetric algorithm. Unlike asymmetric algorithms such as ECC, it uses the same key for encryption and decryption. This key is either calculated between the server and client (using ECDH) or randomly generated and transmitted after being encrypted asymmetrically (Kyber). A key length of 256 bits is currently used for the VAU channel. AES is a block cipher and is used for large amounts of data due to its high speed. GCM (Galois/Counter Mode) is used as the operating mode. AES-GCM is an AEAD mode (Authenticated Encryption with Associated Data) that offers encryption and authentication in a single step.

<img src="{{ site.baseurl }}/assets/img/20250203-vaukanal-part01/aesgcmalgorithm.png" alt="Symmetric Algorithm AES-GCM" width="516" height="342"/>

The result is the encrypted data (ciphertext) and a checksum (authentication tag) that ensures that the message has not been tampered with. Since the IV is also needed to decrypt the data, it is placed in front of the ciphertext.

### Next Chapter
In the following blog posts, we will look at how the individual calculation results are transported in the four VAU messages. Only two VAU messages are needed for a key negotiation (ss1 and ss2). However, the key negotiation process between server and client is done twice, which is why two more are needed, resulting in 4 VAU messages. Why a double key negotiation is carried out here is something we will also explain in the next blog posts.

# About the author
As a senior softwar engineer, Mike Kurtze has been developing projects in the data encryption field for around 20 years and has also given video courses and webinars on this topic on serveral plattforms like Udemy and Heise-Academy.
