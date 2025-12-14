---
title: GPG
parent: notes
grand_parent: 'Module 17 security'
layout: page
---

### Basic Tutorial on gpg

https://www.youtube.com/watch?v=eLKOIjNFwVs

### gpg generate key 

```bash
gpg --full-generate-key
gpg --gen-key
```

### gpg list keys 

```bash
#gpg --list-keys
```

### gpg export key 

```bash
#gpg --export-secret-keys  > private.key
````

### gpg import key 

```bash
#gpg --import private.key
```

### gpg decrypt file 

```bash
#gpg --decrypt file.txt.gpg
```

### gpg sign file 

```bash
gpg --sign file.txt
```


### gpg encrypt file 

```bash
gpg --encrypt file.txt
```


### gpg verify file 

```bash
gpg --verify file.txt.asc file.txt
```

### gpg upload key 

```bash
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 
```
