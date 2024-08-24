# v02enc

`v02enc` is a password-based encryption application that supports several recipients and has the option to update an encrypted file as long as the user has access to the password of a single recipient.

`vim02enc` is a small script that enables a user to transparently edit a `v02enc`-encrypted file. After closing the file, it will be encrypted for all former recipients.

## Limitations

`v02enc` does not implement streaming encryption and therefore does not support large files.

## Origin

The encryption scheme originates from [Shared-Secrets](https://github.com/yahesh/shared-secrets). Shared-Secrets implements `v00` as a password-based encryption scheme supporting a single recipient and `v01` as an RSA-based encryption scheme supporting multiple recipients.

`v02enc` extend the existing encryption schemes with `v02` as a password-based encryption scheme supporting multiple recipients.

## License

This application is released under the BSD license. See the [LICENSE](LICENSE) file for further information.
