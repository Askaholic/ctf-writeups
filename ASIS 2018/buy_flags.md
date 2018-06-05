# Buy Flags [80]
> [Here](http://46.101.173.61/) is an online shop that sells flags :) but we don’t have enough money! Can you buy the flag?

First we try to buy the real flag and take a look at the request. We see that it just makes a POST request to the /pay endpoint with the following data
```
{"card":[{"name":"asis","count":1}],"coupon":""}
```
We get an error:
```
your credit not enough
```
So copy the request as cURL command and try it again with a negative count. The hope is that this will multiply with the price of the flag and then compare against our balance. -1 * 110 = -100 which is > 0

```
{"card":[{"name":"asis","count":-1}],"coupon":""}
```
Unfortunately we get another error:
```
{"result": "item count must be greater than zero"}
```
After trying some more input we find that we can pass a list or object as the count, and it will fail to convert it to a string.
Finally we try giving NaN for the count:
```
{"card":[{"name":"asis","count":NaN}],"coupon":""}
```
And we get the flag:
```
{"data": [{"data": "ASIS{th1@n_3xpens1ve_Fl@G}\n", "flag": "asis"}], "result": "pay success"}
```
This works because any comparisons with NaN will return false. So if the code checks for our input < 0, it will return false for NaN input and pass the negative input check.
