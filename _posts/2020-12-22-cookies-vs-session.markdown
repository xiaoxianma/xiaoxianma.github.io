---
layout:     post
title:      "Cookies vs Sessions"
subtitle:   ""
date:       2020-12-22 22:10:00
author:     "Xiaoxianma"
header-img: "img/home-bg-oakley-on-keyboard.jpg"
tags:
    - http
    - restapi
---

Cookies and Sessions are used to store information. Cookies are only stored on client-side machine, while sessions get stored on the client as well as server.

### Session
A session creates a file in a temporary directory on the server where registered session variables and their values are stored. This data will be available to all pages on the site during that visit.  

A session ends when the user closes the browser or after leaving the side, the server will terminate the session after a predetermined period of time, commonly 30 minutes duration.

### Cookies
Cookies are text files stored on the client computer and they are kept of use tracking purpose. Server script sends a set of cookies to the browser. For example anme, age, or id number etc. The browser stores this information on a local machine fur future use.  

When next time browser sends any request to web server then it sends those cookies information to the server and server uses that information to identify the user.

### Comparison Chart
| Cookie                        | Session                        |
|-------------------------------|--------------------------------|
| Cookies are client-side files | Sessions are server-side files |
| lifetime set by exp var       | session end when browser close |
| max size is 4KB               | max size 128MB by default      |

### Rules
- Rule 1: Never trust user input : cookies are not safe. Use sessions for sensitive data.  
- Rule 2: If persistent data must remain when the user closes the browser, use cookies.  
- Rule 3: If persistent data does not have to remain when the user closes the browser, use sessions.  
- Rule 4: Read the detailed answer below.  

### Detailed answer
**Cookies**  
- Cookies are stored on the client side (in the visitor's browser).
- Cookies are not safe: it's quite easy to read and write cookie contents.
- When using cookies, you have to notify visitors according to european laws (GDPR).
- Expiration can be set, but user or browser can change it.
- Users (or browser) can (be set to) decline the use of cookies.

**Sessions**
- Sessions are stored on the server side.
- Sessions use cookies (see below).
- Sessions are safer than cookies, but not invulnarable.
- Expiration is set in server configuration (php.ini for example).
- Default expiration time is 24 minutes or when the browser is closed.
- Expiration is reset when the user refreshes or loads a new page.
- Users (or browser) can (be set to) decline the use of cookies, therefore sessions.
- Legally, you also have to notify visitors for the cookie, but the lack of precedent is not clear yet.
