
# XSS Challenge

[CodePen Pen](https://codepen.io/cakinney/pen/rrrppd)

## About
A simple and ~~beta~~ evolving set of Cross-Site Scripting (XSS) challenges (written in [VanillaJS](http://vanilla-js.com/)) with various contexts and filtering for developer education and awareness.

## Usage
Select radio buttons to change challenge contexts and filters. Then use the input to inject XSS with a JavaScript alert box proof of concept.

## The Challenges
### #1

```html
<h2>[variable]</h2>
```

Filters: Script tags

### #2

```html
<a href="[variable]">Click Me</a>
```

Filters: Double quotes (") or angle brackets (< , >)

### #3

```html
<img src="[variable]">
```

Filters:  Angle brackets (< , >)

## Solutions
Please see the bottom of this document for challenge solutions and explanations. 

# XSS Information
Note: The following was adapted for the wider web developer community from my post on XSS Security for Vapor developers at [cakinney/VaporSecurity](https://github.com/cakinney/VaporSecurity).
  
## What is Cross-Site Scripting (XSS)?
Cross-Site Scripting (XSS) is a code injection vulnerability that allows an attacker to run malicious scripts on a victim's browser. These scripts allow an attacker to perform any action on behalf of the user, access sensitive data, and modify page content. Additionally, the scripts are run in the context of the vulnerable page and therefore are trusted and bypass the browser's Same Origin Policy (SOP) protections. 

### Types of XSS
* **Reflected XSS** is immediately (non-persistent) reflected off the server and run in the client’s browser. It is commonly exploited as a GET request but can also be a POST request.
* **Stored XSS** is stored on the server (persistent) and executed when the stored exploit is retrieved. 
* **DOM Based XSS** is exploited client side in the DOM (Document Object Model) and is never sent to the server.

## XSS Examples

### HTML Tags
If HTML tags are not filtered or encoded, attackers can use valid tags, which will be interpreted by the browser as HTML/JavaScript. 

*Exploit:*  

```html
<script>alert(1337)</script>
```

*HTML:*  

```html
<h1>Welcome [variable]!</h1>
```

*Request:*  
`http://localhost:8080/xss?variable=<script>alert(1337)</script>` 

*Response:*  

```html
<h1>Welcome <script>alert(1337)</script>!</h1>
```


### Attributes
If " or ' is not encoded, attackers can break out of attribute tags and use JavaScript event handlers to exploit XSS, even if angle brackets are filtered. 

*Exploit:*  
`" onfocus="alert(1337)" autofocus="`

*HTML:*  

```html
<input id="[variable]" type=“text”>
```

*Request:*  
`http://localhost:8080/xss?variable=foo”+onfocus=“alert(1337)”+autofocus=“`

*Response:*  

```html
<input id="foo" onfocus="alert(1337)" autofocus="" type=“text”>
```

### href/src/data Tags
Untrusted data placed in href, src or data tags are commonly not projected by templates or default encoding, therefore extra caution should be given when placing variables inside those attributes. 

*Exploit:*  
`javascript:alert(1337)`

*HTML:*  

```html
<a href="[variable]">My Profile</a>
```

*Request:*  
`http://localhost:8080/xss?variable=javascript:alert(1337)`

*Response:* 
 
```html
<a href="javascript:alert(1337)">My Profile</a>
```

### File Uploads
Allowing unrestricted file uploads could also result in XSS attacks.

For example, SVG image files are treated as XML from the browser and can contain XSS attacks:  

```xml
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <script type="text/javascript">
      alert(1337);
   </script>
</svg>
```

*Additionally, if you are processing the Exif or other metadata of image files, XSS exploits can be embedded into them and should be treated as untrusted data.*

## How can we protect against XSS attacks?
### Use a templating engine
Most tempting engines (Jinja2, Leaf, Mustache, etc) automatically HTML encode malicious characters (*unless variables are placed directly in script tags or href/src/data attributes*) protecting you from XSS attacks. 

### Content Security Policy (CSP)
CSP neutralizes XSS attacks by defining a whitelist of trusted origins that can access a page’s resources. CSP will also restrict inline JavaScript and eval functions (*if you do not include unsafe-inline or unsafe-eval in your policy*). However, there are CSP bypasses with header misconfigurations or trusted origins that can be manipulated or controlled by the attacker.

### HttpOnly Cookies
Cookies with the HttpOnly flag set instruct the browser to not allow any client side code to access the cookie's contents.  

### FIEO
*Filter Input, Escape Output* - Filter and escape malicious characters and content as context requires.

### Quote Attributes
Ensure you quote your attributes with single or double quotes. For example use `<input id="[variable]">` instead of `<input id=[variable]>`. An attacker can break out of the attribute by adding JavaScript event handlers.

### File Uploads
If you allow various files to be uploaded, ensure when serving the files that the proper `Content-Type` and `Content-Disposition` headers are set. 

## Other attacks
### HTML Injection
HTML injection is an injection attack, similar to XSS but does not include JavaScript, where pages could be injected with unescaped HTML tags. HTML injection is not protected by CSP. 

### Blind XSS
XSS that is exploited somewhere not accessible to the attacker (for example in server logs) and includes actions or a callbacks to a server owned by the attacker. 


## Resources:

### Cross-Site Scripting (XSS)
* [DEF CON 20 - Adam "EvilPacket" Baldwin - Blind XSS](https://youtu.be/8mO00rEdCrQ)
* [File Upload XSS - Brute XSS](https://brutelogic.com.br/blog/file-upload-xss/)
* [OWASP Sweden - The image that called me](https://www.owasp.org/images/0/03/Mario_Heiderich_OWASP_Sweden_The_image_that_called_me.pdf)
* [Top 10-2017 A7-Cross-Site Scripting (XSS) - OWASP](https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_%28XSS%29)
* [XSS (Cross Site Scripting) Prevention Cheat Sheet - OWASP](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)

### Content Security Policy (CSP)
* [Content Security Policy - Google Developers](https://developers.google.com/web/fundamentals/security/csp/)
* [Content Security Policy - OWASP](https://www.owasp.org/index.php/Content_Security_Policy)
* [CSP Evaluator](https://csp-evaluator.withgoogle.com)


# Solutions
*Note: There are many possible solutions to each challenge.*

## Challenge 1
The context of challenge 1 is the variable is inside `h2` tags with script tags filtered. 

```html
<h2>[variable]</h2>
```

 *Note: Script tags are automatically filtered with Element.innerHTML.* 
 
 Instead of using script tags (`<script>alert(1)</script>`), you can use additional tags with event handlers such as an anchor tag with a onclick event handler, which will alert when the link is clicked. `<a href=# onclick=alert(1)>click</a>`

```html
<h2><a href=# onclick=alert(1)>click</a></h2>
```

## Challenge 2
The context of challenge 2 is the variable is inside the `href` attribute in an `a` tag surrounded by double quotes.

```html
<a href="[variable]">Click Me</a>
```

Double quotes and angles brackets are filtered. Since you are unable to break out of the double quotes context, however you can turn the link URL into a JavaScript function (`javascript:alert(1)`) which will alert when the link is clicked. 

```html
<a href="javascript:alert(1)">Click Me</a
```
	
## Challenge 3
The context of challenge 3 is the variable is inside the `src` attribute in an `img` tag (`<img src="[variable]"> `) surrounded by double quotes. Only angle brackets are filtered. Because double quotes are not filtered you can break out of the double quote context and add an event handler `"x" onerror="alert(1)` which will alert on an image error.

```html
<img src="x" onerror="alert(1)"
```
