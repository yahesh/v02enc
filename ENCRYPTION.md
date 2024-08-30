# Encryption Scheme v02

Encryption scheme v02 is a password-based Encrypt-then-MAC (EtM) scheme which uses AES-256-CTR as the encryption algorithm and HMAC-SHA-256 as the MAC algorithm. The key to derive the encryption key and the message authentication key is randomly generated. The subkeys used to protect the key are each derived via a 512.000 rounds PBKDF2-SHA-256 on the passwords and a 256 bits long random salt.

## Message Format

Messages in the v02 format have the following format:

```
[version:01][headersalt:32][headernonce:16][messagenonce:16][subkeycount:02][subkeyheader:32][...][subkeyheader:32][headermac:32][message:nn][messagemac:32]
```

## Message Fields

Messages in the v02 format have the following fields:

* **version** is 1 byte in size and **MUST** have the value `00h`
* **headersalt** is 32 bytes in size and **SHOULD** contain a cryptographically strong random number
* **headernonce** is 16 bytes in size and **SHOULD** contain the UNIX timestamp as the first 8 bytes, `FFh` as the next 4th byte and `00h` as the last 4 bytes
* **messagenonce** is 16 bytes in size and **SHOULD** contain the UNIX timestamp as the first 8 bytes and `00h` as the last 8 bytes
* **subkeycount** is 2 bytes in size and **MUST** denote the number of upcoming subkey blocks
* **subkeyheader** is the AES-256-CTR encrypted subkey
* **headermac** is 32 bytes in size and **MUST** contain the HMAC-SHA-256 MAC of all previous fields in their given order
* **message** is the AES-256-CTR encrypted message
* **messagemac** is 32 bytes in size and **MUST** contain the HMAC-SHA-256 MAC of all previous fields in their given order

## Key Derivation

Messages in the v02 format use the following keys:

* **key** is a cryptographically strong random number
* **enckey** is derived from **key** as the key and the string `enc` as the message using HMAC-SHA-256
* **headermackey** is derived from **key** as the key and the string `mac-header` as the message using HMAC-SHA-256
* **messagemackey** is derived from **key** as the key and the string `mac-message` as the message using HMAC-SHA-256
* **headersalt** is a cryptographically strong random number
* **subkey** is derived from the given password and **headersalt** using a 512.000 rounds PBKDF2-SHA-256

## Key Usage

Keys in the v02 format have the following purposes:

* **enckey** in combination with **nonce** is used to encrypt the message using AES-256-CTR
* **headermackey** is used as the key to calculate the **headermac** value using HMAC-SHA-256 over the following fields: `[version:01][salt:32][subkeynonce:16][subkeycount:02][subkeymessage:32][...][subkeymessage:32][nonce:16]`
* **messagemackey** is used as the key to calculate the **messagemac** value using HMAC-SHA-256 over the following fields: `[version:01][salt:32][subkeynonce:16][subkeycount:02][subkeymessage:32][...][subkeymessage:32][nonce:16][headermac:32][message:nn]`
* **subkey** in combination with **headernonce** is used to encrypt the key using AES-256-CTR

## Example

The following Bash command encrypts a message with a given password using the above encryption scheme v02. A current version of OpenSSL/LibreSSL is needed:

```
# version 02 symmetric encryption
INPUT="message to encrypt" &&
PASSWORD0000="password0001" &&
PASSWORD0001="password0002" &&
VERSION="02" &&
HEADERSALT=$(openssl rand -hex 32) &&
HEADERNONCE=$(printf "%016xFFFFFFFF00000000" "$(date +%s)") &&
MESSAGENONCE=$(printf "%016x0000000000000000" "$(date +%s)") &&
SUBKEYCOUNT="0002" &&
KEY=$(openssl rand -hex 32) &&
ENCKEY=$(echo -n "enc" | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$KEY" -sha256 | xxd -p | tr -d "\n") &&
HEADERMACKEY=$(echo -n "mac-header" | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$KEY" -sha256 | xxd -p | tr -d "\n") &&
MESSAGEMACKEY=$(echo -n "mac-message" | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$KEY" -sha256 | xxd -p | tr -d "\n") &&
SUBKEY0000=$(openssl kdf -binary -kdfopt "digest:SHA256" -kdfopt "hexsalt:$HEADERSALT" -kdfopt "iter:512000" -kdfopt "pass:$PASSWORD0000" -keylen 32 PBKDF2 | xxd -p | tr -d "\n") &&
SUBKEY0001=$(openssl kdf -binary -kdfopt "digest:SHA256" -kdfopt "hexsalt:$HEADERSALT" -kdfopt "iter:512000" -kdfopt "pass:$PASSWORD0001" -keylen 32 PBKDF2 | xxd -p | tr -d "\n") &&
SUBKEYHEADER0000=$(echo -n "$KEY" | xxd -p -r | openssl enc -aes-256-ctr -iv "$HEADERNONCE" -K "$SUBKEY0000" -nopad | xxd -p | tr -d "\n") &&
SUBKEYHEADER0001=$(echo -n "$KEY" | xxd -p -r | openssl enc -aes-256-ctr -iv "$HEADERNONCE" -K "$SUBKEY0001" -nopad | xxd -p | tr -d "\n") &&
HEADERMAC=$(echo -n "$VERSION$HEADERSALT$HEADERNONCE$MESSAGENONCE$SUBKEYCOUNT$SUBKEYHEADER0000$SUBKEYHEADER0001" | xxd -p -r | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$HEADERMACKEY" -sha256 | xxd -p | tr -d "\n") &&
MESSAGE=$(echo -n "$INPUT" | openssl enc -aes-256-ctr -iv "$MESSAGENONCE" -K "$ENCKEY" -nopad | xxd -p | tr -d "\n") &&
MESSAGEMAC=$(echo -n "$VERSION$HEADERSALT$HEADERNONCE$MESSAGENONCE$SUBKEYCOUNT$SUBKEYHEADER0000$SUBKEYHEADER0001$HEADERMAC$MESSAGE" | xxd -p -r | openssl dgst -binary -mac "HMAC" -macopt "hexkey:$MESSAGEMACKEY" -sha256 | xxd -p | tr -d "\n") &&
echo "-----BEGIN V02ENC MESSAGE-----" &&
echo -n "$VERSION$HEADERSALT$HEADERNONCE$MESSAGENONCE$SUBKEYCOUNT$SUBKEYHEADER0000$SUBKEYHEADER0001$HEADERMAC$MESSAGE$MESSAGEMAC" | xxd -p -r | openssl base64
echo "-----END V02ENC MESSAGE-----"
```
