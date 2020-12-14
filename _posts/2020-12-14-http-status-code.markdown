---
layout:     post
title:      "HTTP Status Code"
subtitle:   ""
date:       2020-12-14 11:30:00
author:     "Xiaoxianma"
header-img: "img/home-bg-oakley-on-keyboard.jpg"
tags:
    - http
    - restapi
---

# About HTTP status codes
In HTTP,  you send a numeric status code of 3 digits as part of the response.  

These status codes have a name associated to recognize them, but the important part is the number.  

In short:
- `100` and above are for "Information". You rarely use them directly. Responses with these status codes cannot have a body.  
- `200` and above are for "Successful" responses. These are the ones you would use **the most**.  
  - `200` is the default status code, which means everything was "OK".  
  - Another example would be `201`, "Created". It is commonly used after creating a new record in the database.  
  - A special case is `204`, "No Content". This response is used when there is no content to return to the client, and so the response must not have a body.  
- `300` and above are for "Redirection". Responses with these status codes may or may not have a body, except for `304`, "Not Modified", which must not have one.  
- `400` and above are for "Client error" responses. Theses are the type you would probably use **the second most**.  
  - An example is `404`, for a "Not Found" response.  
  - For generic errors from the client, you can just use `400`.  
- `500` and above are for server errors. You almost never use them directly. When something goes wrong at some part in your application code, or server, it will automatically return one of these status codes.