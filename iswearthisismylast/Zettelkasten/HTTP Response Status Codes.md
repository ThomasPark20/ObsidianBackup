2022-07-07 20:58
source : [Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
tags: #tips #http #burp #cybersec 

# HTTP Response Status Codes


## What
These are status codes returned after sending HTTP requests to indicate the status of the response given

## When
Mainly seen during #web hacking, [[Burp Suite]].

## How
200 - status code ok. site loaded successfully
302 - found. redirection. This can indicate that it was successful or not successful. needs investigation
400 - bad request
401 - also no auth
403 - Forbidden. No auth.
404 - 
500 - Internal server error This is generic. Might need investigation as this is a "catch-all"
502 - Bad Gateway This means the server interacting is acting as a proxy in middle but received error from server behind
