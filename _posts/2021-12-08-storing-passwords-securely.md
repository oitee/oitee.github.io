---
layout: post
title: "Defence Against the Dark Arts: How to Store Passwords Securely"
tags: conceptual
image: /assets/images/passwords_title.png
---

This post is all about storing passwords. What is the point of securely storing passwords? What is the difference between encryption and hashing? How are passwords stored in [Twirl](https://oteetwirl.herokuapp.com/), the per-user URL shortening app I built using NodeJs and Express framework?

## Storing passwords securely: What’s the point?

Passwords act as an all-access key to the user’s entire account. Anyone with the password of a user will have unhindered access to the user’s account. Therefore, if it is leaked, there is no way to prevent malicious actors from taking over the account.

Most apps do not require any additional level of permission other than password-based authentication. This is **different from how API keys work**. API keys are “*[task credentials](https://medium.com/swlh/api-keys-whats-the-point-8f58d7966f9#:~:text=TL%3BDR%3A%20Passwords%20are%20for,keep%20those%20API%20keys%20secret.)”:* they are used to authenticate a user and authorize a specific set of tasks on behalf of that user. The potential for malicious use is limited by the scope of actions the API keys can be used for. In the case of passwords, though, there is no distinction between authentication and authorisation: one who has the password, can do everything and anything that the user is permitted to do. This is why we need to be very careful while storing passwords.

In this post, I discuss some of the **best practices for storing passwords**. I explain some of the downsides of storing passwords in plain text, and explore relative advantages of **using encryption and by cryptographic hash functions** for storing passwords. But before that, I will discuss some of the **basic concepts of Cryptography** that are relevant to this discussion. Finally, I will show **how passwords are stored in Twirl**.

## Cryptography

Cryptography is essentially a study of techniques to protect “*communication in the presence of adversaries*”. The [four primary goals](https://doc.lagout.org/network/3_Cryptography/CRC%20Press%20-%20Handbook%20of%20applied%20Cryptography.pdf) of Cryptography are: **confidentiality** (protecting the contents of a message from a third-party/adversary), **data integrity** (detecting manipulation of data by a third-party/adversary), **authentication** (verifying the identity of the sender of a message), and non-repudiation (preventing one from [denying having performed an action](https://firstmonday.org/ojs/index.php/fm/article/view/778/687)).

### Encryption

Encryption is the “*[principal application of Cryptography](http://index-of.es/Varios-2/Serious%20Cryptography%20A%20Practical%20Introduction%20to%20Modern%20Encryption%20(2).pdf)*” which makes data *incomprehensible* in order to secure its confidentiality. It involves the transformation of a clear text (unencrypted data) into a ciphertext (encrypted data) and the reverse transformation of the ciphertext back to the original plain-text. Encryption and decryption is achieved by mathematical functions, called [cryptographic algorithms](https://link.springer.com/chapter/10.1007/978-1-4302-6383-8_8), which make use of ‘keys’ to encrypt and decrypt information. The same information can be encrypted into different ciphertexts by the same cryptographic algorithm, if different keys are used.

Largely there are two types of encryptions: **symmetric** and **asymmetric** encryptions. In symmetric encryption algorithms, only one key is used to encrypt and decrypt information. Thus, both the generator and recipient of a message *share* the same key. In asymmetric encryption algorithms,there are two keys: a public key and a private key. The public key–which is known publicly–is used only for encrypting a ciphertext. The private key–which should remain a secret–is solely used to decrypt information.

#### Encryption is not the same as ‘encoding’

Encryption should not be confused with ‘encoding’. Encoding is the process of converting one form of data into another form. It has *[nothing to do with cryptography](https://auth0.com/blog/how-secure-are-encryption-hashing-encoding-and-obfuscation/),* neither does it guarantee any of the goals of Cryptography.

The primary goal of encoding is to transform data so that it can be properly read by a different system. This transformation of data is achieved by a [publicly available scheme](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/) that can easily be reversed.

For example, certain cryptographic functions, such as `pbkdf2` (discussed below) operate at the level of bytes. The output, therefore, is also in the form of bytes. Now, when we use such functions to store passwords in a database (which does not support byte data-types), we need to convert the `byte` data-type into a data-type supported by the database.

### Cryptographic hash functions

A hash function is one which maps an [infinite set of inputs to a finite set](https://otee.dev/2021/10/01/hash-tables.html). A cryptographic hash function is a special kind of hash function which makes it a vital tool in Cryptography. A cryptographic hash function should have the following properties:

- **Non-reversibility:** Just like a normal hash function, a cryptographic hash function is a many-to-one function. Additionally, it should be computationally infeasible to *[to reverse a given output to its original input](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/)*. In other words, it should be a ***one way function**:* you can go to a hash value from a given input but not the other way round.
- **Deterministic:** A given input should always produce the same hash value (like a normal hash function)
- **Avalanche effect:** A small change in the input should cause a significant and unpredictable change in the hash value
- **Collision resistant:** It should be practically infeasible to find two inputs mapping to the same hash value. Of course, by virtue of their many-to-one nature, a hash function will necessarily produce collisions. The key is to ensure that finding such collisions computationally infeasible.

## Storing Passwords Safely

### Why we cannot store passwords just-like-that

The simplest way to store passwords of a system is to maintain them *as-is*, i.e., storing each password in clear text (or, unencrypted) in a table against its respective username. At the time of authenticating a user, we just need to verify the password entered by the user against the password stored in the database.

**Storing passwords in clear text is a dangerous idea**, for two reasons. First, if an attacker gains access to the database, they will have [unfettered access to each user account](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/) (and their respective information) in the system, thereby compromising the entire system. Second, people usually use the same password for multiple systems. Thus, one attack of one system could potentially expose multiple other systems.

### Using two-way encryption is better

Instead of storing clear text passwords, it is **significantly safer to first encrypt each password and store its encrypted value**. This way, even if the database storing the passwords of users is exposed, the attacker will not be able to generate the clear text passwords.

However, there is a catch. If the attacker gets access to the key required to decrypt the encrypted passwords, they will gain unfettered access to each password. This is the **downside of using a two-way function to store passwords**. It is always possible to get back to the original password, irrespective of how much care is taken to secure the key.

### Hash functions are ideal for storing passwords

The problem faced with two-way encryption can be avoided if we use cryptographic hash functions to encrypt passwords. As discussed above, a cryptographic hash function guarantees irreversibility. Thus, even if the database containing the hash values of each password is exposed, *and* the hash function used to generate those values is known, it is **computationally very difficult to guess the clear text passwords**.

Thus, to protect users’ passwords, the actual password never gets stored: “*[we hash it and then forget it](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/)*”.

### But attacks are still possible

Cryptographic hash functions are really powerful. If an attacker gains access to a database storing hashed passwords, all they will see is a bunch of random-looking data. By virtue of the **one-way** nature of hash functions, there will be no process to find the inverse (‘preimage’) of the hash values.

Despite the advantages of cryptographic hash functions, they are **not immune to attacks**. Sure, it is nearly impossible to generate the original password from a given hash value.

But cryptographic hash functions need to be deterministic: the same input needs to always generate the same hash result. What this means for an attacker is that they can try a **brute force attack**–trying every possible combination–to find out the hash values which match with the hash values stored in the database.

Thankfully, a brute force attack is largely ineffective for most modern cryptographic hash functions. But consider the fact that most of us use common passwords (like `password123`). The probability of two or more users sharing the same password is always high. For all such users, their hashed passwords will contain the same values. An attacker can notice this pattern; the most commonly occurring hash values will most likely map to one of the several commonly used passwords. The attacker no longer needs to attempt a brute force attack: they may simply try out the most frequently used passwords and match their hash results with the values stored in the database. Once they find one clear text password that produces a hash value present in the database, it would expose every account that used that password (as they will all have the same hash value).

Also, rainbow attacks can be used to generate the original value of a password. A rainbow table is a precomputed table that stores the hash values of different inputs. Thus, given a hash result, the table can be used to generate an input that could produce that result. Thus, instead of a brute-force search, the attacker can match the hash values from the database with a rainbow table, to arrive at the clear text password.

### Salting and Peppering to the rescue!

Using a ‘salt’ can ensure that two identical clear text passwords produce two distinct (and different) hash values, making the hash function appear non-deterministic.

What is a ‘salt’? It is a “*[unique, randomly generated string](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) that is added to each password as part of the hashing process*”. Thus, **a random string that is unique to each user, is appended to each password**, and this resultant string is hashed. This ensures that we add “*random data to the input of a hash function to [guarantee a unique output](https://auth0.com/blog/adding-salt-to-hashing-a-better-way-to-store-passwords/)*” for each password.

Now, when an attacker looks at the table of hashed passwords, they would not see any pattern of common passwords; they can no longer pass a list of most probable passwords to the hash function, to figure out a list of clear text passwords of users. **Note**, the salt should be stored in clear text in the database, along with the hashed password. This is crucial because, when we need to verify if a password entered by a user is correct, we will need to append the salt to the password and then compare the resultant hash value with the hash value stored in the database.

To an attacker: there are no visible patterns suggesting commonality of passwords; there is no computationally convenient way to invert a hashed value to its original input; the salt is visible for each hashed value. Salting creates a significant bottleneck for the attacker: they will have to try out different clear text passwords (whether by brute-force or by trying out candidate passwords) for each pair of salt and hash value. The attacker will be “*limited to [searching through passwords separately for each salt](https://www.ietf.org/rfc/rfc2898.txt)*”

As an additional layer of protection, we can encrypt the salted hashed values of each password, before storing them in the database. This is called **peppering**. Importantly:

- The same cryptographic key (or, ‘pepper’) should be used for all hash values; they should not be unique to each password (unlike salts)
- The key should not be stored in the database

If the attacker has access to the database, they will not be able to generate the hashes they see on the database, by trying out different combinations of clear text passwords.

## Storing passwords in Twirl

Salts are generated in Twirl, by using the `randomBytes` method of NodeJs’ `crypto` module. The `randomBytes` method returns a “*cryptographically [strong pseudorandom data](https://nodejs.org/api/crypto.html#cryptorandombytessize-callback)*”.

For hashing clear text passwords (and their salts), we have used the `crypto` module’s PBKDF2 implementation.

What is PBKDF2? It is a Key Derivation Function which takes (among others) a password, a salt, and an iteration count as its parameters. A key derivation function [derives a random-looking key from a given value](https://en.wikipedia.org/wiki/Key_derivation_function) (such as a clear text password) by using a cryptographic hash function (or some other pseudorandom function). PBKDF2 applies a “*pseudorandom function, such as hash-based message authentication code (HMAC), [to the input password along with a salt value and repeats the process many times](https://en.wikipedia.org/wiki/PBKDF2) to produce a derived key*”. This derived key is stored in the database, as the hash value. (HMAC is [similar to a cryptographic hash function](https://www.ibm.com/docs/en/sdk-java-technology/8?topic=cp-cryptographic-hash-functions-message-authentication-codes-digital-signatures), where a key is added “*[as part of the data hashed by the function](https://cseweb.ucsd.edu/~mihir/papers/kmd5.pdf)*”)

PBKDF2 takes the following parameters to generate a derived key (hash value): a password, a salt, the number of iterations, key length (the desired bit length of the derived key) and the hash function.

This is how passwords are salted and hashed in Twirl:

```js
import crypto from "crypto";

let password = "test123";

const salt = crypto.randomBytes(16).toString("base64");

crypto.pbkdf2Sync(password, salt, 100, 64, "sha512")

.toString("base64")

```