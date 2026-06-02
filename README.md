# TryHackMe WiseGuy Room Write-Up

## Room Goal

The goal of this room was to analyze a Python source code file and interact with a service running over Netcat on port `1337`. The challenge involved XOR encryption and required us to recover two flags.

The two final flags were:

`THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}`

`THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}`

---

## Step 1: Downloading and Reading the Source Code

The room provided a downloadable `.py` file.

Since it was Python source code, we safely opened it and read it instead of blindly running it.

The important part of the code showed that the server:

1. Reads the real flag from `flag.txt`
2. Creates a random 5-character key
3. Uses a fake flag first
4. XORs the fake flag with the key
5. Sends the encrypted result as hexadecimal
6. Asks us to provide the encryption key

The fake flag in the source code was:

`THM{thisisafakeflag}`

The key was generated using 5 random characters.

---

## Step 2: Connecting to the Service with Netcat

The room told us that the service was listening on TCP port `1337`.

We connected with:

`nc <TARGET_IP> 1337`

The server returned something like:

`This XOR encoded text has flag 1: <encrypted hex>`

Then it asked:

`What is the encryption key?`

At first, it was confusing because the long hexadecimal string looked like the key, but it was actually the encrypted ciphertext.

---

## Step 3: Understanding the XOR Weakness

XOR is reversible.

The key rule is:

`ciphertext XOR plaintext = key`

Because we had the source code, we knew the plaintext used for the first encrypted message:

`THM{thisisafakeflag}`

So we could recover the 5-character XOR key by XORing:

`encrypted hex XOR THM{thisisafakeflag}`

This is called a known-plaintext attack.

---

## Step 4: Recovering the Key

We used CyberChef with:

1. `From Hex`
2. `XOR`

The input was the encrypted hex from Netcat.

The XOR key/input was the known fake flag:

`THM{thisisafakeflag}`

Important mistake we fixed:

The XOR key type had to be text/UTF-8, not HEX.

The output revealed a repeating pattern. Since the Python code showed the key length was 5 characters, the repeating 5-character pattern was the key.

We entered that 5-character key back into Netcat.

---

## Step 5: Getting the First Flag

After submitting the correct key, the server gave us the first real flag:

`THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}`

We submitted it to TryHackMe and it was accepted.

---

## Step 6: Understanding Why the Key Changed

One important thing we learned was that every new Netcat connection generated a new random key.

So if we disconnected and reconnected, the old key no longer worked.

That explained why some keys failed even though the method was correct.

The workflow had to be repeated fresh each time:

1. Connect with Netcat
2. Copy the new encrypted hex
3. Recover the new 5-character key
4. Submit that exact key in the same Netcat session

---

## Step 7: Getting the Second Flag

For the second flag, the challenge was related to brute forcing XOR.

The final flag we recovered was:

`THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}`

This showed that short XOR keys are weak because they can be guessed or brute-forced.

---

## Final Outcome

We completed the room and recovered both flags:

First flag:

`THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}`

Second flag:

`THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}`

---

## Key Lessons Learned

1. Netcat is used to manually interact with network services.
2. A long hex string is usually ciphertext, not the key.
3. XOR encryption is reversible.
4. If plaintext is known, the key can be recovered.
5. Short repeating XOR keys are weak.
6. Each new connection can generate a new random key.
7. Reading source code is often the fastest way to understand a CTF challenge.

The most important formula from the room was:

`ciphertext XOR plaintext = key`
