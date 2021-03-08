# Password Security: How not to Store Passwords

## Introduction

* The best password hashing algorithm available today is [Argon2](https://en.wikipedia.org/wiki/Argon2#Algorithm), and the best implementation for Python is [PyNaCl](https://pynacl.readthedocs.io/en/stable/).
* If you think everyone knows that storing plain text passwords is terrible, remember *Sony* and *Yahoo* managed to leak plain text passwords.
* Passwords are valuable, even if the site in question has nothing else of value, because people tend to reuse their passwords across services. 
> Re-using a password is like handing the key to your safety deposit box to the kid who watches your dog :dog:
* The least a user should do is to use a password manager like Lastpass. 
* What's the least you, as a service provider who stores user credentials, can do? Start storing the password hashes and discard the old plain text passwords.

## Problems with storing encrypted passwords.

* An attacker who can get your encrypted passwords might be able to get your key too.
* The length of the password determines the length of the encrypted text. And knowing the password length, or even just the possible range, is a wonderful gift to an attacker. For example, let’s say a bank allows ATM pin codes to be from four to ten digits long. Trying every possible pin number would take about eleven-billion tries. However, if you knew a particular pin was either six or seven digits, it would only take eleven million tries. It’s still hard, but significantly easier.

## Hashing Passwords

*  Hash functions let you prove you know a secret without having to say the secret. Moreover, a good hash function produces the same size output regardless of the input. 
*  Hashing also doesn’t care what characters you feed into it. You have probably encountered a site with a low max password size (like under sixteen characters) and a list of banned characters. Those could be arbitrary rules. Or it may be a sign that the site isn’t even doing this much for password storage.
*  MD5 and SHA-1 should not be used for passwords anymore. SHA-2 is still secure. There are similar algorithms that were not designed for cryptography. CRC32 and FNV, for instance, are great for hash tables or finding transmission errors, but they were never intended for cryptography and won’t survive a serious crack attempt.

## Problems with Hashing

* It’s common for several users to have the same password. If Alice and Bob used the same password and an attacker cracks Bob's password, Alice's account (and every account with that password) has been cracked. 
* There are lists of common passwords on the internet. All an attacker has to do is hash the passwords in the list and find the accounts that match in the dump. It won’t crack every password, but it will probably crack enough, and in very little time.
* Another problem is that cryptographic hashes are fast to compute on normal hardware. A GPU can make the work go faster. Some hash algorithms can be computed on custom hardware (ASICs), which is even faster than a GPU. A brute-force attack that tries every combination is becoming possible, even against strong passwords. Additionally, if computing the hashes it too much work, attackers can download a pre-computed list of password hashes, known as a rainbow table.
* Salted hashes are a big improvement. They were the recommended way to store passwords for a long time, and salts are still used by secure password hashing algorithms. However, a simple salted password hash is no longer enough.
* Plain hashes for passwords are pretty dumb, but there are still frequent dumps of passwords using nothing more than a hash. LinkedIn, for example, lost SHA-1 hashes not that long ago.

## Salted Hashes

* Benefit of salts is that hashes will be unique even when the passwords aren’t. Consequently, when an attacker cracks one password, other accounts with the same password are still safe.
* Moreover, each user can have their own unique salt. This salt is inserted into the database unobfuscated since salt isn't a secret, and it’s needed to verify the password. Therefore, one can prepend this salt to the password hash. E.g. `iamasalt.iampasswordhash`
* Salted hashes are a big improvement. However, hardware keeps getting faster, so the speed that an attacker can try passwords is increasing. GPUs and ASICs are making brute-force attacks even against salted passwords quite reasonable. So salted hashes aren't enough.
> If leaking your user database is a car wreck, salted hashes are like a seat belt. They definitely help, but you’d be better off with airbags too.

## Improving Salted Hashing

* The only problem is the speed an attacker can test guesses. They would be much more secure if they took more time. Processing power costs an attacker real money, so making them spend 1,000 times more can be an effective deterrent. This is the principle behind special-purpose password hashing algorithms: PBKDF2 and bcrypt. 
* bcrypt requires a configurable amount of CPU time, which, in theory, makes it future proof.
* bcrypt puts the salt in the returned password hash for us, so we can simply store the returned string and use it for verification.
* You can control how much **power** bcrypt uses by changing the number passed to `bcrypt.gen_salt`
* While using these special purpose hashing algo, you have to keep in mind that even if the user doesn't exist in your system, you should still do a hash calculation (on mock value or even empty string) to ensure that login-failed request doesn't return back immediately. An attacker can use the time difference to find valid user logins. Knowing who belongs to a site can be a problem on its own. *A `time.sleep` call would work too, but it can be tricky to get the time exactly right*
* The danger with using bcrypt and PKDF2 is that special-purpose hardware for cracking passwords may become significantly faster than the general-purpose hardware used to validate passwords. If this happens, the number of iterations required to stop an attacker will slow down logins beyond the user’s tolerance. 

## Strong Password Hashing

* **PBKDF2** and **bcrypt** require a configurable amount of CPU power. **Argon2** and **scrypt** take this a step further and require memory also. This makes them harder to crack even on specialized hardware. 
* One big problem with **scrypt** is that it was used in the Litecoin digital currency. This prompted the development of Litecoin miners, which are basically scrypt solvers. scrypt should be able to handle these miners, but Argon2 is a safer choice, and the design is cleaner anyway.
* A password hash generated by PyNaCl's default algorithm (which is Argon2 right now) looks something like this - `b'$argon2id$v=19$m=65536,t=2,p=1$AnQPEtCxqefx5L7dej9jKQ$TakaD0zrKXrDov5fIv4MPQAOPCfAUJBuYpdW7f21oug'` That contains the algorithm used, ops limit (CPU requirement), memory limit, and the salt all in a nifty string you can put right in your database. This way, if you change the parameters in the future, the older passwords still work.

## Steps for Migrating to More Secure Hashes

* There isn’t a sure way to get the plain text passwords back from the hash, so the only time you can upgrade the password is when the user logs in. The basic plan is:

1. Make a new database column to store secure hashes
2. Create new accounts with the secure hash
3. Change the login code to use the new secure hash
4. Add a fallback to the login code for the old method, which also re-hashes the password

* After migrating from weak hashes to more secure hash, it's important to remove the old password hashes. Otherwise, 
> it’s something like storing a key in a safe while also keeping a copy of it under your doormat. 
* The infamous Ashley Madison breach contained bcrypt hashes along with unsalted MD5 hashes for the same passwords. A possible explanation is that the company switched to bcrypt, but they kept a backup of their old password hashes.

## Hashing Algorithms

* [Argon2](https://tools.ietf.org/html/draft-irtf-cfrg-argon2-02)
* [scrypt](https://www.tarsnap.com/scrypt/scrypt.pdf)
* [bcrypt](https://www.usenix.org/legacy/events/usenix99/provos/provos.pdf)
* [PKDF2](https://tools.ietf.org/html/rfc8018)
* [SHA-1](https://tools.ietf.org/html/rfc3174) and [SHA-2](https://tools.ietf.org/html/rfc4634)

## Libraries

* [PyNaCl docs](https://pynacl.readthedocs.io/en/stable/)
* [libsodium docs](https://doc.libsodium.org/)
* [libsodium language bindings](https://doc.libsodium.org/bindings_for_other_languages)
* [bcrypt python library](https://pypi.org/project/bcrypt/)

## Password dumps

[Have I Been Pwnd](https://haveibeenpwned.com/PwnedWebsites) lists a number of password dumps.

* [Adobe](https://haveibeenpwned.com/PwnedWebsites#Adobe)
* [Ashley Madison](https://haveibeenpwned.com/PwnedWebsites#AshleyMadison)
* [LinkedId](https://haveibeenpwned.com/PwnedWebsites#LinkedIn)
* [Sony](https://haveibeenpwned.com/PwnedWebsites#Sony)
* [Yahoo](https://haveibeenpwned.com/PwnedWebsites#Yahoo)

## Password managers

* [LastPass](https://www.lastpass.com/)
* [1password](https://1password.com/)
* [KeePassXC](https://keepassxc.org/)
* [Keeper](https://www.keepersecurity.com/)
