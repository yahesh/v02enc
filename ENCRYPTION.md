# Encryption Scheme v02

Encryption scheme v02 is a password-based Encrypt-then-MAC (EtM) scheme which uses AES-256-CTR as the encryption algorithm and HMAC-SHA-256 as the MAC algorithm. The key to derive the encryption key and the message authentication key is randomly generated. The subkeys used to protect the key are each derived via a 512.000 rounds PBKDF2-SHA-256 on the passwords and a 256 bits long random salt.

## Message Format

Messages in the v02 format have the following format:

```
[version:01][salt:32][subkeycount:02][subkeynonce:16][subkeymessage:32][...][subkeynonce:16][subkeymessage:32][nonce:16][message:nn][mac:32]
```

## Message Fields

Messages in the v02 format have the following fields:

* **version** is 1 byte in size and **MUST** have the value `00h`
* **salt** is 32 bytes in size and **SHOULD** contain a cryptographically strong random number
* **subkeycount** is 2 bytes in size and **MUST** denote the number of upcoming subkey blocks
* **subkeynonce** is 16 bytes in size and **SHOULD** contain the UNIX timestamp as the first 8 bytes, `01h` as the 9th byte, the index of the subkey block as the 10th and 11th byte and `00h` bytes as the last 5 bytes
* **subkeymessage** is the AES-256-CTR encrypted key
* **nonce** is 16 bytes in size and **SHOULD** contain the UNIX timestamp as the first 8 bytes and `00h` bytes as the second 8 bytes
* **message** is the AES-256-CTR encrypted message
* **mac** is 32 bytes in size and **MUST** contain the HMAC-SHA-256 MAC of all previous fields in their given order

## Key Derivation

Messages in the v02 format use the following keys:

* **key** is a cryptographically strong random number
* **enckey** is derived from **key** as the key and the string `enc` as the message using HMAC-SHA-256
* **mackey** is derived from **key** as the key and the string `mac` as the message using HMAC-SHA-256
* **salt** is a cryptographically strong random number
* **subkey** is derived from the given password and **salt** using a 512.000 rounds PBKDF2-SHA-256

## Key Usage

Keys in the v02 format have the following purposes:

* **enckey** in combination with **nonce** are used to encrypt the message using AES-256-CTR
* **mackey** is used as the key to calculate the MAC of the message `[version:01][salt:32][subkeycount:02][subkeynonce:16][subkeymessage:32][...][subkeynonce:16][subkeymessage:32][nonce:16][message:nn]` using HMAC-SHA-256
* **subkey** in combination with **subkeynonce** are used to encrypt the key using AES-256-CTR

## Example

The following Bash command encrypts a message with a given password using the above encryption scheme v02. A current version of OpenSSL/LibreSSL is needed:

```
# version 02 symmetric encryption
MESSAGE="message to encrypt" &&
PASSWORD0000="password1" &&
PASSWORD0001="password2" &&
VERSION="02" &&
KEY=$(openssl rand -hex 32) &&
ENCKEY=$(echo -n "enc" | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$KEY" -sha256 | xxd -p | tr -d "\n") &&
MACKEY=$(echo -n "mac" | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$KEY" -sha256 | xxd -p | tr -d "\n") &&
NONCE=$(printf "%016x0000000000000000" "$(date +%s)") &&
ENCMESSAGE=$(echo -n "$MESSAGE" | openssl enc -aes-256-ctr -iv "$NONCE" -K "$ENCKEY" -nopad | xxd -p | tr -d "\n") &&
SALT=$(openssl rand -hex 32) &&
SUBKEYCOUNT="0002" &&
SUBKEYNONCE0000=$(printf "%016x0100000000000000" "$(date +%s)") &&
SUBKEYNONCE0001=$(printf "%016x0100010000000000" "$(date +%s)") &&
SUBKEY0000=$(openssl kdf -binary -kdfopt "digest:SHA256" -kdfopt "hexsalt:$SALT" -kdfopt "iter:512000" -kdfopt "pass:$PASSWORD0000" -keylen 32 PBKDF2 | xxd -p | tr -d "\n") &&
SUBKEY0001=$(openssl kdf -binary -kdfopt "digest:SHA256" -kdfopt "hexsalt:$SALT" -kdfopt "iter:512000" -kdfopt "pass:$PASSWORD0001" -keylen 32 PBKDF2 | xxd -p | tr -d "\n") &&
SUBKEYMESSAGE0000=$(echo -n "$KEY" | xxd -p -r | openssl enc -aes-256-ctr -iv "$SUBKEYNONCE0000" -K "$SUBKEY0000" -nopad | xxd -p | tr -d "\n") &&
SUBKEYMESSAGE0001=$(echo -n "$KEY" | xxd -p -r | openssl enc -aes-256-ctr -iv "$SUBKEYNONCE0001" -K "$SUBKEY0001" -nopad | xxd -p | tr -d "\n") &&
MACMESSAGE="$VERSION$SALT$SUBKEYCOUNT$SUBKEYNONCE0000$SUBKEYMESSAGE0000$SUBKEYNONCE0001$SUBKEYMESSAGE0001$NONCE$ENCMESSAGE" &&
MAC=$(echo -n "$MACMESSAGE" | xxd -p -r | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$MACKEY" -sha256 | xxd -p | tr -d "\n") &&
FULLMESSAGE=$(echo -n "$MACMESSAGE$MAC" | xxd -p -r | openssl base64) &&
echo "-----BEGIN V02ENC MESSAGE-----" &&
echo "$FULLMESSAGE" &&
echo "-----END V02ENC MESSAGE-----"
```
