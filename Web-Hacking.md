#DNS (Domain Name System):

- a simple way for communicating with services on the Internet. 

## Domain Heirarchy:

- Top-Level Domain:

    - two types (gTLD and ccTLD)
    
    - .com, ,org, .edu (gTLD)
    
    - .co.uk, .np (ccTLD)
    
- Second-Level Domain:

    - name of the domain 
    
    - limited to 63 characters + the TLD can only use a-z 0-9 and hyphens (cannot start or end with hyphens or have consecutive hyphens)
    
- Subdomain:

    - support.nischal.com
    
    - same creation restrictions as Second-Level Domain
    
    - can be used for multiple subdomains (jupiter.servers.nischal.com)
    
    - length must be kept to 253 charcters or less.

## DNS Record Types:

- A Record:

    - resolves to IPv4 addresses
    
- AAAA Record:

    - resolves to IPv6 addresses
    
- CNAME Record:

    - resolves to another domain.
    
    - For eg: an ecommerce website might have a subdomain name as "store.nischal.com" which could return a CNAME record "shops.shopify.com". Here, another DNS request will be made to work out the IP address.
    
- MX Record:

    - resolves to address of the server that handles the email for the domain you are querying.
    
    - tells the client in which order to try the servers.
    
- TXT Record:

    - free text fields where any text-based data can be stored.
    
    - common uses can be to list servers that have the authority to send an email on the behalf of the domain.
    
    - can also be used to verify the ownership of the domain name when signing for third party services.
    
## Making a Request:

- First, when I request a domain name "nischal.com", the computer checks its local cache to see if I have previously visited this address recently. If yes, i will be redirected to the website with this domain, but if not, then a request to the Recursive DNS Server is made

- A Recursive DNS Server also has a local cache of recently looked up domain names. If there is a result stored in it, then the request ends here and I will be inside the website, but if here also it is not found then the DNS begins its journey to the wild to find the correct answer to our request which starts with the internet's root DNS servers.

- So, the root servers act as the DNS backbone of the internet. Their job is to redirect a client to the correct Top Level Domain Server as per the request. So, what I requested was "www.nischal.com", the root server will recognise the Top Level Domain of ".com" and then refer me to the correct TLD server that deals with ".com" addresses.

- Now, we are finally in the TLD server that holds records for where to find the authoritative server to answer the DNS request we sent. The authoritative server is also known as the nameserver for the domain. Lets assume the nameserver of my request is "kip.ns.cloudflare.com" and "uma.ns.cloudflare.com". We may often find multiple nameservers for a single domain name to act as a backup if one goes down.

- An authoritative DNS server is the server who is responsible for storing the DNS records for a particular domain name and where any updates to the domain name DNS records are made. So, depending on the record type, the DNS record is sent back to the Recursive DNS server where a local copy will be cached for future requests and the request is relayed back to the original client. DNS records all come with TTL (Time to Live) value. This value is a number represented in seconds that the response should be saved locally until a client has to look it up again. So, it basically cache saves on having to make a DNS request every time you communicate with a server.


# HTTP in Detail

- HTTP is an internet protocol which defines a sets of rules used for communicating with web servers. Basically, it handles transmission of webpage data such as HTML, CSS, JS, Images, Videos etc. While HTTP(S) is the secure version of HTTP, where the transmission of the data is encrypted.

## Requests and Responses:

- When we access a website, the browser first needs to make requests to a web server to retrieve assets such as HTML, Images, and download the responses. For this to happen, the browser needs to specify how and where to look for these resources where URLs will come to the rescue.

- What is a URL? (Uniform Resource Locator)

    - an instruction on how to access a resource on the internet. Some of its features are listed below, however it does not use all features in every request. A URL looks like this -> http://user:password@nischal.com:80/profile?id=1#profilesettings
    
    - Scheme: This feature instructs on what protocol to use for accessing the resource such as HTTP, HTTPS, FTP.
    
    - User: Some services requires authentication to log in. We can put a username and password into the URL to log in.
    
    - Host: It is the domain name or the IP address of the server we wish to access.
    
    - Port: The port we are connecting to.
    
    - Path: The file name or location of the resource we are trying to access.
    
    - Query String: Extra information which can be sent to the requested path.
    
    - Fragment: A reference to a location on the actual page requested.
    
- Making a Request:

    - We can make request to a web server by just one line "GET /HTTP/1.1"
    
    - For example:
    
    GET / HTTP/1.1
    
    Host: nischal.com
    
    User-Agent: Mozilla/5.0 Firefox/87.0
    
    Referer: https://nischal.com/
    
    - The first request is sending the GET method to view the home page "/" and telling the web server we are using HTTP protocol version 1.1. The second line tells the webserver we want the website "nischal.com". The third line specifies the browser we are requesting from. In the fourth line we are telling the web server that the web page that referred us to this one is "https://nischal.com". The HTTP requests always ends up with a blank line to inform the web server that the request has been completed.
    
    - Example Response:
    
    HTTP/1.1 200 OK
    
    Server:nginx/1.15.8
    
    Date: Thursday, 29 Jan 2026 13:14:03 GMT
    
    Content-Type: text/html
    
    Content-Length: 98
    
    
    <html>
    <head>
        <title>Nischal</title>
    </head>
    <body>
        Welcome to Nischal.com
    </body>
    </html>
    
    - What we got here is first the version of the HTTP protocol the server is using followed by the HTTP status code. Second line showed the web server software and version number. Third line showed the current date, time and timezone of the web server. Fourth line tells the client what is going to be sent such as HTML, images, videos, pdf, XML. The fifth line tells the client how long the response is. Then, a blank line indicates the end of the HTTP response and finally the information that has been requested is presented. In this case, the home page.

## HTTP Methods:

- HTTP methods are a way for the client to show their intended actions when making a request. There are many methods but most commonly we will use GET, POST, PUT, DELETE.

- GET Request: 

    - used for getting information from a web server
    
- POST Request:

    - used for submitting data into the web server and creating new record.
    
- PUT Request:

    - used to update the data to the web server.
    
- DELETE REquest:

    - used for deleting information/records from the web server.
    
## HTTP Status Codes:

- When responding to a client requests, HTTP server always responds with a status code to inform the client whether the request was successful of not. The status codes can be broken down into 5 different ranges:

    - 100-199 (Information Response): This code tells the client that the first part of the request has been accepted and they should continue sending the rest of the request.
    
    - 200-299 (Success): This range of status codes tells the client that their request was successful.
    
    - 300-399 (Redirection): These are showed when the client request has been redirected to another resource. This can be either to a different webpage or a different website.
    
    - 400-499 (Client Errors): These ranges are showed when the client makes error with their request.
    
    - 500-599 (Server Errors): These ranges point out that the server had error handling the request. This server-side not client-side errors.
    
- Common HTTP status codes:
    
    - 200-OK : Request was completed successfully.
    
    - 201-Created : Resource has been created (for eg a new user or a post)
    
    - 301-Moved Permanently : This indicated the client's browser has been redirected to a new webpage.
    
    - 302-Found : It is the same as 301 but, this is only temporary change and it may change in the near future.
    
    - 400-Bad Request : This tells the browser that something was wrong or missing in the request. This could be the requested web server resource expected a certain parameter that the client did not send.
    
    - 401-Not Authorised : This tells us that we are not allowed to view the resource until we have authorised with the web application, most commonly with a username and password.
    
    - 403-Forbidden: Even when we are logged in, we cannot view this resource because of the persmission.
    
    - 404-Page Not Found : The page/resource you requested does not exist.
    
    - 405-Method not Allowed : The resource does not allow this method request, such as sending GET request to create an account.
    
    - 500-Internal Server Error : This notifies the browser that the web server encountered some internal error with the request and it does not know how to handle it properly.
    
    - 503-Service Unavailable : The server cannot handle the request either because of the server being overloaded or down for maintainance.
    
## Headers:

- Headers are additional data sent to the webserver when making a request.

- Common Request Headers:

    - Host: Some web servers host mutiple websites, so to not get the default website for the server, we can provide host headers to tell which one we want.
    
    - User-Agent: By telling the browser software and version number to the web server, the browser software helps it to format the website properly according to the browser being used.
    
    - Content-Length: It tells the web server how much data to expect in the web request. This can ensure the availability of the data received is not missing.
    
    - Accept-Encoding: Tells the web server what types of compression methods the browser supports so the data can be made samller for transmission over the internet.
    
    - Cookie: Data sent to the server to help remember user information.
    
- Common Response Headers:

    - Set-Cookie: Information to store which gets sent back to the web server on each request.
    
    - Cache-Control: How long the browser stores the content of the response before it makes request again.
    
    - Content-Type: Tells the client the type of data being returned.
    
    - Content-Encoding: Tells the method that was used to compress the data while sending it over the internet. 
    
## Cookies:

- Small piece of data stored on the computer. Cookies are saved when we get "Set-Cookie" response header from the web server. HTTP is stateless, so cookies can be helpful to remind the web server who we are. Example of how cookies are set:

    - Client requests webpage -> Server responds back with a simple webpage with a form asking for the users name -> Client fills the form and sends it to the web server -> Server responds with Set-Cookie header telling the client to save the data name -> On next and every further request the clients send the cookie data back to the server -> Server sees the cookie and remembers the user.
    
    - Cookie value won't usually be a clear-text string, but a token that cant be human guessable.
    
# OWASP API Security Top 10

- API stands for Application Programming Interface which acts as the middleware for the communication of two software components by utilising a set of protocols and definitions. Putting in simpler terms, API is a building block for developing complex and enterprise-level applications.

## Broken Object Level Authorisation (BOLA):

- API endpoints are utilised for common practice of retrieving and manipulating data through object identifiers. BOLA refers to Insecure Direct Object Reference (IDOR), where an user uses the input function and gets access to the resources they are not authorised to see. 

- Such flaw in API results in a data leakage, or in some cases complet account takeover. Such problem exists beacuse of the endpoint not validating any incoming API call to confirm whether the request is valid. 

- A solution for such flaw in endpoint could be resolved if we implement access tokens or authorisation tokens in the request header. In this way, only headers with a valid token can make a call to the endpoint.

## Broken User Authentication (BUA):

- BUA allows an attacker to access a database or gain even higher privilege than the existing one. The primary reason behins such vulnerability is either invalid implementation of authentication like using incorrect email/password queries or the absence of security mechanisms like authorisation headers, tokens etc.

- In a broken user authentication, attackers can compromise the authenticated session or the authentication mechanism and easily access sensitive data. So, if we have information that the login endpoint only uses email to validate the user from the user table and ignores the password field in the SQL query, then we can get a valid token just from thhe email and pasword could be anything. So, after getting a valid token, we can use the token value to get the user's information which could normally be in /user/details/ endpoint.

- To prevent from such misconfiguration, a developer can update the login query logic and use both email and password for the validation which then authorises the user based on password and emial both.

## Excessive Data Exposure:

- This flaw exists on applications that tend to disclose more information than desired through an API response. Such misconfiguration can lead to an attacker to capture personal details like account numbers, phone numbers, access tokens and many more.

- One of its olution could be to avoid using generic methods such as to_string() adn to_json().

## Lack of Resources and Rate Limiting:

- This flaw means that the API does not enforce any restriction on the frequency of clients requested resources or the files size. This affects the API server performance and potentially lead to DoS or non-availability of service. 

- An attacker can use this flaw and could exploit it by writing a small script and brute force the endpoint. Many opportunity arises for the attacker such as bruteforcing password to a known username, sending emails in a few seconds, uploading files of large sizes continuosly etc.

- So, enabling rate limiting, using captcha, defining maximum data size on all parameters and payloads on the endpoint of such requests can help to mitigate such flaws.

## Broken Function Level Authorisation:

- Broken Function Level Authorisation relfects a scenario when a low privileged user bypasses system checks and get access to confidential data by impersonating a high privileged user (Admin). This flaw also reflects IDOR permission. An attacker can impersonate an authorised user and get permissions to administrative rights. 

- Suppose we have access to an employee who is at low level privilege. To perform this flaw and escalte the privilege, we can add headers of authorization-token and isAdmin=1 to get details of the admin. Here, the API only fetches employee information from the database if isAdmin=1 and Authorization-Token are correct.

- Ensuring proper design and testing off all authorisation system and denying access by default could help. Operations should only be allowed to the users belonging to the authorised group.

## Mass Assignment:

- happens where client-side data is automatically bounded with server-side objects or class variables. If necessary filtration is not enabled on the server-side, it will simply insert/update the data in the database.

## Security Misconfiguration:

- This flaw reflects an implementation of incorrect and poorly configured security controls. Such misconfigurations can give intruders complete knowledge of API components. First, it allows attackers to bypass security mechanisms. Then, stack trace or other detailed errors can provide the intruder to access confidential information and essential system details.

## Injection:

- Injection flaw occurs when user input is not filtered and is directly processed by an API which can give us chance to perform unintended API actions without authorisation. An injection comes from SQL, OS commands, XML etc.

## Improper Assets Management:

- This flaw is activated when there are more than one API versions in the system.  So, an older or unpatched API version can be used to get unauthorised access to confidential data.

## Insufficient Logging and Monitoring:

- It reflects a scenarios when an attacker performs a malicius act on the server, then when the hacker is tried to be pursued, then the development and security teams could not track the behaviour of the attacker. So, API logging and monitoring could help organizations find the details of an attacker like Ip address, endpoints accessed, inout data etc.
