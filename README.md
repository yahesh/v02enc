# v02enc

`v02enc` is a password-based encryption application that supports several recipients and has the option to update an encrypted file as long as the user has access to the password of a single recipient.

`v02gitdiff` is a small script that requires `colordiff` to work. It can be used to enhance the `git diff` output by adding the following configuration to `~/.gitconfig`:

```
[diff]
external = /full/path/to/v02gitdiff
```

`v02hgdiff` is a small script that requires `colordiff` to work. It can be used to enhance the `hg diff` output by adding the following configuration to `~/.hgrc`:

```
[extensions]
extdiff =

[extdiff]
cmd.v02hgdiff = /full/path/to/v02hgdiff

[alias]
diff = !for FILE in $(hg status -n) ; do hg v02hgdiff "$(hg root)/${FILE}" -o "$(hg root)" ; done
```

`vim02enc` is a small script that enables a user to transparently edit a `v02enc`-encrypted file. After closing the file, it will be encrypted for all former recipients.

## Help

```
Usage:

./v02enc
  [-d|--decrypt|-e|--encrypt|-u <file>|--update <file>]
  [-a|--armor]
  [-h|-help]
  [-i <file>|--input <file>]
  [-k <file>|--key <file>]*
  [-m|--message <string>]
  [-o <file>|--output <file>]
  [-p <string>|--password <string>]*
  [<file>]

Options:

  -d          | --decrypt           Decrypt a message.
  -e          | --encrypt           Encrypt a message.
  -u <file>   | --update <file>     Update an encrypted message with the contents in <file>.
                                    <file> can be "-" to read from STDIN.
  -a          | --armor             ASCII-armor the encrypted message.
  -h          | --help              Print this help.
  -i <file>   | --input <file>      Use <file> as the input.
                                    <file> can be "-" to read from STDIN.
                                    The default is STDIN.
  -k <file>   | --key <file>        Use the contents in <file> as an encryption key.
                                    This option can be provided multiple times.
  -m <string> | --message <string>  Use <string> as the input.
  -o <file>   | --output <file>     Use <file> as the output.
                                    <file> can be "-" to write to STDOUT.
                                    <file> can be "+" to write to STDERR.
                                    The default is STDOUT.
  -p <string> | --password <string> Use <string> as an encryption key.
                                    This option can be provided multiple times.
  <file>                            Use <file> as the input.
                                    <file> can be "-" to read from STDIN.
                                    The default is STDIN.

Notes:

* You can only use one mode at a time, so either decrypt, encrypt or update.
* You can only use one input at a time.
* You can only use one output at a time.
```

## Examples

### Generate a random passphrase

```
head -c 32 /dev/random >~/.v02enc
```

### Encrypt a file with a passphrase file

```
# using STDIN and STDOUT
echo -n "example text" | v02enc --encrypt --key ~/.v02enc >./example.txt.v02enc

# using unnamed argument and STDOUT
v02enc --encrypt --key ~/.v02enc ./example.txt >./example.txt.v02enc

# using --input and --output
v02enc --encrypt --key ~/.v02enc --input ./example.txt --output ./example.txt.v02enc
```

### Encrypt a file with a passphrase argument

```
# using unnamed argument and STDOUT
v02enc --encrypt --password "example passphrase" ./example.txt >./example.txt.v02enc

# using --input and --output
v02enc --encrypt --password "example passphrase" --input ./example.txt --output ./example.txt.v02enc
```

### Encrypt a message

```
# using unnamed argument and STDOUT
echo -n "example text" | v02enc --encrypt --password "example passphrase" >./example.txt.v02enc

# using --input and --output
echo -n "example text" | v02enc --encrypt --key ~/.v02enc --input - --output ./example.txt.v02enc

# using --message and --output
v02enc --encrypt --key ~/.v02enc --message "example message" --output ./example.txt.v02enc
```

### Use ASCII-armored output

```
v02enc --encrypt --armor --key ~/.v02enc --message "example message"
```

### Decrypt a file

```
# using unnamed argument and STDOUT
v02enc --decrypt --key ~/.v02enc ./example.txt.v02enc

# using --input and --output
v02enc --decrypt --key ~/.v02enc --input ./example.txt.v02enc --output -
```

### Update a file

```
# update using STDIN
echo -n "new text" | v02enc --update - --key ~/.v02enc --input ./example.txt.v02enc --output ./example.txt.v02enc.tmp && mv ./example.txt.v02enc.tmp ./example.txt.v02enc

# update using a file
v02enc --update ./new-text.txt --key ~/.v02enc --input ./example.txt.v02enc --output ./example.txt.v02enc.tmp && mv ./example.txt.v02enc.tmp ./example.txt.v02enc
```

### Use vim to update a file

```
# preparing passphrase file in correct location
head -c 32 /dev/random >~/.v02enc

# using implicit passphrase file path
vim02enc ./example.txt.v02enc

# using explicit passphrase file path
V02ENC_KEY=~/.v02enc ./example.txt.v02enc
```

## Origin

The encryption scheme originates from [Shared-Secrets](https://github.com/yahesh/shared-secrets). Shared-Secrets implements `v00` as a password-based encryption scheme supporting a single recipient and `v01` as an RSA-based encryption scheme supporting multiple recipients.

`v02enc` extend the existing encryption schemes with `v02` as a password-based encryption scheme supporting multiple recipients.

## License

This application is released under the BSD license. See the [LICENSE](LICENSE) file for further information.
