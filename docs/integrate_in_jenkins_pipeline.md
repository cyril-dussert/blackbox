# Integrate an encrypted repository with Jenkins pipeline

To allow Jenkins to decrypt files of a repository using Blackbox, you need to:

- [Integrate an encrypted repository with Jenkins pipeline](#integrate-an-encrypted-repository-with-jenkins-pipeline)
  - [Create a GPG key dedicated to Jenkins](#create-a-gpg-key-dedicated-to-jenkins)
    - [On Linux](#on-linux)
      - [Checking version](#checking-version)
      - [Creating a new GPG key](#creating-a-new-gpg-key)
    - [On macOS](#on-macos)
    - [On Windows](#on-windows)
      - [Checking version](#checking-version-1)
      - [Creating a new GPG key](#creating-a-new-gpg-key-1)
  - [Register GPG Key as new admin in blackbox](#register-gpg-key-as-new-admin-in-blackbox)
  - [Add the GPG key in Jenkins](#add-the-gpg-key-in-jenkins)
    - [Export private key](#export-private-key)
    - [Export GPG ownertrust](#export-gpg-ownertrust)
    - [Export passphrase](#export-passphrase)
    - [Import files to Jenkins](#import-files-to-jenkins)
  - [Use GPG key in Jenkins Pipeline](#use-gpg-key-in-jenkins-pipeline)
    - [Import `blackbox` executables](#import-blackbox-executables)
    - [Decrypt files](#decrypt-files)



## Create a GPG key dedicated to Jenkins

### On Linux

#### Checking version

**Important:** Make sure that your GnuPG utility version is above v2.8.0 using:
```bash
gpg --version
```
If it is not the case, please consider upgrading it before taking any action.

#### Creating a new GPG key

To create a new GPG key, enter the following command:

```bash
gpg --full-generate-key
```

```text
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
```
Select (1), which will enable both encrytpion and signing.

```text
What keysize do you want? (3072)
```
Enter the keysize. A keysize of 3072 (which is the default) is a good choice.

```text
Key is valid for? (0)
```
Specify 0, meaning that the key will never expire. This is the default option.

```text
GnuPG needs to construct a user ID to identify your key.

Real name:
```
Set the value you want, the most important is to provide an intelligible value as Email Address, as it will be the key identifier. For Jenkins, it could be wise to specifiy something as:

```text
GnuPG needs to construct a user ID to identify your key.

Real name: Jenkins Master
Email address: master@jenkins.local
Comment: Machine Key
You selected this USER-ID:
    "Jenkins Master (Machine Key) <master@jenkins.local>"

```
Then, you'll be asked a passphrase:

```text
You need a Passphrase to protect your secret key.
```
Enter here a randomly generated passphrase, just keep it the time needed to save it into Jenkins.

**Notice:** Depending of your environment, the passphrase can be asked using a popup window.

While generating the key, you can be asked to generate some entropy, so that randomization can be effective. Tap on the keyboard, or move the mouse to provide some elements to the gpg agent.

```text
gpg: key D8FC66D2 marked as ultimately trusted
public and secret key created and signed.

pub   1024D/D8FC66D2 2005-09-08
      Key fingerprint = 95BD 8377 2644 DD4F 28B5  2C37 0F6E 4CA6 D8FC 66D2
uid                  Jenkins Master (Machine Key) master@jenkins.local
sub   2048g/389AA63E 2005-09-08
```

Your key is correctly generated, we will now add it to Jenkins

### On macOS

To be done...

### On Windows

#### Checking version
GnuPG is directly embedded in [Git for Windows](https://gitforwindows.org/). Open Git Bash, and check your GnuPG version:

```bash
gpg --version
```
If you have gpg version v2.2.8, or above, you're good to go. Else, we highly recommand to upgrade, at least, to GnuPG [v2.2.8](https://lists.gnupg.org/pipermail/gnupg-announce/2018q2/000425.html) so that major known security flaws of GnuPG are fixed.

To upgrade, upgrade the whole `Git for Windows` stack. For instance, Git for Windows v2.26.0 features GnuPG v2.2.20.

#### Creating a new GPG key

To create a new GPG key, as said before, we are gonna use **Git Bash**. Open a new terminal, and run the following commands:

```bash
gpg --full-generate-key
```
You will be asked the kind of key you want:
```text
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
```
Select (1), which will enable both encrytpion and signing.

```text
What keysize do you want? (2048)
```
Enter the keysize. A keysize of 3072 is a good choice.

```text
Key is valid for? (0)
```
Specify 0, meaning that the key will never expire. This is the default option.

```text
GnuPG needs to construct a user ID to identify your key.

Real name:
```

As we want to create a key for Jenkins, we will use `Jenkins Master` as key owner.
As email adress, which will be the key identifier, we set `master@jenkins.local`.
As comment, put what you want. It could be nice to specify that it is a machine key.

```text
Real name: Jenkins Master
Email address: master@jenkins.local
Comment: Machine Key
You selected this USER-ID:
    "Jenkins Master (Machine Key) <master@jenkins.local>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

A passphrase should be asked to you. As this key is a machine key, it is **highly recommended** to specify a randomly generated passphrase.

```text
gpg: /c/Users/user/.gnupg/trustdb.gpg: trustdb created
gpg: key 7642590C41D6054D marked as ultimately trusted
gpg: directory '/c/Users/user/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/c/Users/user/.gnupg/openpgp-revocs.d/59AB94C73B9C05B7DBE403697642590C41D6054D.rev'
public and secret key created and signed.

pub   rsa3072 2020-04-03 [SC]
      59AB94C73B9C05B7DBE403697642590C41D6054D
uid                      Jenkins Master (Machine Key) <master@jenkins.local>
sub   rsa3072 2020-04-03 [E]
```

Your key is correctly generated, we will now add it to Jenkins

## Register GPG Key as new admin in blackbox

Go onto your encrypted repository, and run the following commands:

```bash
blackbox_addadmin master@jenkins.local
git commit -m "Added new admin: Jenkins" .blackbox/pubring.kbx .blackbox/trustdb.gpg .blackbox/blackbox-admins.txt
blackbox_update_all_files
git push
```

`blackbox_update_all_files` will re-encrypt all registered files so that new admin can decrypt it.

## Add the GPG key in Jenkins

### Export private key

We will put the private key directly into Jenkins. To export the key, run the following command:

```bash
gpg -a --export-secret-keys <email-address> > gpg-secret.key
```

### Export GPG ownertrust

```bash
gpg --export-ownertrust > gpg-ownertrust.txt
```

### Export passphrase

Just put the passphrase in a `passphrase.txt` file.

### Import files to Jenkins

This step has to be done manually, we will import those 3 files as credentials in Jenkins, so that they can be safely re-used into pipelines.

Logon to Jenkins and go to the **Credentials** section. Then, click on *System* store, and then on *Global credentials (unrestricted)*. We will now be able to add our files as secrets.
Click on *Add Credentials*, on the left menu. And file form, 3 times, with the following values:

| Name        	      | gpg-secret       	| gpg-ownertrust       	| gpg-passphrase   	|
|------------------- |------------------	|----------------------	|------------------	|
| **Kind**         	| Secret File      	| Secret File          	| Secret File      	|
| **Scope**       	| Global           	| Global               	| Global           	|
| **File**        	| `gpg-secret.key` 	| `gpg-ownertrust.txt` 	| `passphrase.txt` 	|
| **ID**          	| gpg-secret       	| gpg-ownertrust       	| gpg-passphrase   	|
| **Description** 	| GPG Private Key  	| GPG Ownertrust       	| GPG Passphrase   	|

## Use GPG key in Jenkins Pipeline

To decrypt files inside a Jenkins pipeline, we'll first need to import `blackbox` executables.

### Import `blackbox` executables

`blackbox` executables are present on a dedicated git repository. Here is a Jenkins pipeline step to clone them:

```groovy
  stage('Get latest version of blackbox files') {
    steps {
      dir('blackbox') {
        git(
          url: 'https://github.com/cyril-dussert/blackbox.git',
          branch: "master",
          credentialsId: "<Credentials-id>"
        )
      }
    }
  }
```

You also have to add those executables to your PATH:
```groovy
  environment {
    PATH = "${env.WORKSPACE}/blackbox/bin:${env.PATH}"
  }
```

We will also import GPG private key, ownertrust and passphrase as environment variables:

```groovy
  environment {
    PATH = "${env.WORKSPACE}/blackbox/bin:${env.PATH}"
    gpg_secret = credentials("gpg-secret")
    gpg_trust = credentials("gpg-ownertrust")
    gpg_passphrase = credentials("gpg-passphrase")
  }
```

In order to use the imported key, we have to load it in a pipeline stage:

```groovy
  stage('Import GPG keys') {
    steps {
      sh "gpg --batch --passphrase-file $gpg_passphrase --import $gpg_secret"
      sh "gpg --import-ownertrust $gpg_trust"
    }
  }
```

### Decrypt files

To decrypt files, just use previously loaded `blackbox` executables, as per following example:

```groovy
  stage('Decrypt all files') {
    steps {
      sh 'blackbox_decrypt_all_files'
      sh 'cat README.md'
    }
  }
```