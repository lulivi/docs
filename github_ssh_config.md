
# GPG key

## GPG key generation

First we should generate a gpg key:

	$ gpg --full-generate-key
	RSA and RSA
	0 = key does not expire
	y It is correct
	Real name: Luis Liñán
	Email address: luislivilla@gmail.com
	<paraphrase>
	
Then, list it in long format:

	$ gpg --list-secret-keys --keyid-format LONG
	/Users/hubot/.gnupg/secring.gpg
	------------------------------------
	sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
	uid                          Hubot 
	ssb   4096R/42B317FD4BA89E7A 2016-03-10
	
Take the key ID (`3AA5C34371567BD2`) and show the GPG key:

	$ gpg --armor --export 3AA5C34371567BD2
	#-----BEGIN PGP PUBLIC KEY BLOCK-----
	...
	#-----END PGP PUBLIC KEY BLOCK-----
	
## Github GPG key adding

Now go to `Settings > SSH and GPG keys > New GPG key`. Then paste the key and click in `Add GPG key`.

# SSH key

## SSH key generation

Generate the key with your email:

	$ ssh-keygen -t rsa -b 4096 -C "luislivilla@gmail.com"

Start SSh-agent:
	
	$ eval "$(ssh-agent -s)"
	
Add the SSH private key to ssh-agent:

	$ ssh-add ~/.ssh/id_rsa
	
## Github SSH key adding

Go to `Settings > SSH and GPG keys > New SSH key`. Then paste the content of `~/.ssh/id_rsa.pub` and click in `Add SSH key`.
