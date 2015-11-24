title: 让你的Git使用上ssh协议授权
date: 2015-11-24 18:24:45
categories: git
tags: [git,ssh]
---

[维基百科：ssh](https://zh.wikipedia.org/zh/Secure_Shell)

### 基本的ssh配置
#### 1. Check for SSH keys
First, we need to check for existing SSH keys on your computer. Open Git Bash and enter:
```java
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
```
If you see an existing public and private key pair listed (for example id_rsa.pub and id_rsa) that you would like to use to connect to GitHub, you can skip Step 2 and go straight to Step 3.
（如果存在密钥对，密钥对的账号如果不是你所需配置网站的账号，还是需要重新生成）

#### 2. Generate a new SSH key
2.1 With Git Bash still open, copy and paste the text below. Make sure you substitute in your GitHub email address.
```java
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
# Generating public/private rsa key pair.
```
or you can generate a key for the specified name use the commond below.
```java
$ ssh-keygen -t rsa -C "your_email@example.com" -f ~/.ssh/github
```
命名为github（这里叫什么随意，不要重名即可），然后会生成github和github.pub这两个文件

2.2 We strongly suggest keeping the default settings as they are, so when you're prompted to "Enter a file in which to save the key", just press Enter to continue.

2.3 You'll be asked to enter a passphrase.(为了方便，直接enter略过)

#### 3.  Add your key to the ssh-agent
```java
$ ssh-add ~/.ssh/id_rsa
```
可能会报 Could not open a connection to your authentication agent 
```java
$ ssh-agent bash
$ eval `ssh-agent`
```

#### 4: Add your SSH key to your account
打开公钥文件（id_rsa.pub），并把内容复制至代码托管平台上.

#### 4: Test the connection
各平台：
```java
$ ssh -T git@github.com
$ ssh -T git@git.coding.net
$ ssh -T git@gitcafe.com
```