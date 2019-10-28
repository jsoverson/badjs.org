---
title: 'RAA Ransomware'
date: '2016-06-17'
tags:
---

The RAA ransomware was distributed via email and infected windows machines, encrypting files and demanding a ~$250 payment to decrypt. Regardless of payment it also installed the password stealing software [Pony](https://www.knowbe4.com/pony-stealer). It used the open source CryptoJS library to encrypt data, as windows JavaScript environments do not expose crypto functions.

- - -

## References

- [The new RAA Ransomware is created entirely using Javascript](https://www.bleepingcomputer.com/news/security/the-new-raa-ransomware-is-created-entirely-using-javascript/)
- [New RAA Ransomware Uses Only JavaScript to Infect Computers](https://www.trendmicro.com/vinfo/nl/security/news/cybercrime-and-digital-threats/new-raa-ransomware-uses-only-javascript-to-infect-computers)

## Payload

Too large to display inline with syntax highlighting. View the payload here: <a href="/js/samples/raa-ransomware.js.txt">raa-ransomware.js</a>.