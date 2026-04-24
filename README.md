# HackerOne-Report
HackerOne bug bounty report

You can paste this directly into a repo as README.md

Improper Input Handling Leading to 500 Internal Server Error
# Overview

During testing of the Dynatrace SSO authentication endpoint, I observed inconsistent input validation on the arcvalue parameter. Certain inputs cause the backend to return a 500 Internal Server Error, indicating unhandled exceptions.

This behavior suggests weak input validation and improper error handling on the server side.

# Target Endpoint
POST /sso/authentication/login
> Steps to Reproduce
Intercept a request to the login endpoint using Burp Suite.
Modify the request body:
{
  "login": "test@example.com",
  "password": "invalidpassword",
  "rememberMe": false,
  "arcvalue": true
}
Send the request.

# Observed Behavior

> Payload	Response
"arcvalue": true	500 Internal Server Error
"arcvalue": false	500 Internal Server Error
"arcvalue": 123	500 Internal Server Error
"arcvalue": {}	400 Bad Request
"arcvalue": []	400 Bad Request
"arcvalue": "AAAA"	JSON parsing error
Parameter removed	400 Bad Request

# Analysis
The application appears to expect a specific data type or schema for arcvalue.
Inputs such as booleans and integers lead to server-side crashes, rather than being rejected gracefully.
Structured inputs ({}, []) are correctly rejected, indicating partial validation.

This inconsistency suggests:
> Lack of strict type enforcement
> Improper exception handling in backend logic

# Impact
> Attackers can reliably trigger 500 Internal Server Errors
> Repeated requests may lead to:
  *** degraded performance
  *** potential service instability
  *** Error responses may assist attackers in understanding backend behavior

Note: No direct data exposure or authentication bypass was observed during testing.

# Recommendation
Enforce strict input validation (type checking for arcvalue)
Implement proper error handling to avoid unhandled exceptions
Return consistent client errors (e.g., 400 Bad Request) instead of 500

# Additional Notes
This issue was identified through systematic parameter fuzzing using Intruder in Burp Suite, focusing on response behavior such as status codes and error patterns.




### 🧠 What I Learned

This test reinforced the importance of observing how different data types affect backend 
