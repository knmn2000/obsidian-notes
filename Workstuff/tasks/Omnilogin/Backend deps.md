Token manager:
- used as a middleware in basically all routes (printHub, conf room etc).


---

```
Auth Token Generation  
  
  
curl --location 'https://wework-corp-staging.wework-us.auth0.com/oauth/token' \  
--header 'Content-Type: application/json' \  
--header 'Cookie: did=s%3Av0%3A66dedf90-7527-11ed-b857-43aca0a09a27.C9rRaWjysHRv8zuYx23x4D%2FRlG00oG4cIi3OXPK%2BOCg; did_compat=s%3Av0%3A66dedf90-7527-11ed-b857-43aca0a09a27.C9rRaWjysHRv8zuYx23x4D%2FRlG00oG4cIi3OXPK%2BOCg' \  
--data '{  
    "client_id": "SXJMew6C1aVW4JS5roFg0r13E3lbxTS2",  
    "client_secret": "iI3KM2kjvr1ouJJJjXmCe8tk_HgaMSskEjULc4dkCWmaBmnvFmkkr-ZV7DL7wkjr",  
    "audience": "wework-graph",  
    "grant_type": "client_credentials"  
}'

pass this token below


curl 'https://pr-557-wifi-service-token-auth-64ceb1.phoenix.dev.wwrk.co/graphql ' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Connection: keep-alive' -H 'DNT: 1' -H 'Origin: chrome-extension://kjhjcgclphafojaeeickcokfbhlegecd' -H 'graph-client-id: wework-india' -H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlJVRXlNREk0TlRnek5qVTJNelpCTWpRMk5EWkdRMEZCTkRnd05UaEVNRFV3TkRFd01VRXpOQSJ9.eyJodHRwczovL3dld29yay5jb20vc2VydmljZV91dWlkIjoiN2FkMDJjZDktNzA3MC00N2RjLWIyNTQtNjc0N2IzMDU2ODUwIiwiaHR0cHM6Ly93ZXdvcmsuY29tL293bmVyX3R5cGUiOiJPUEVSQVRPUiIsImh0dHBzOi8vd2V3b3JrLmNvbS9vd25lcl91dWlkIjoiZmU5ZTQzZWMtN2IzZi00YjE1LThhYTgtZWU4OGQ3YTg3MDIyIiwiaXNzIjoiaHR0cHM6Ly93ZXdvcmstY29ycC1zdGFnaW5nLndld29yay11cy5hdXRoMC5jb20vIiwic3ViIjoiU1hKTWV3NkMxYVZXNEpTNXJvRmcwcjEzRTNsYnhUUzJAY2xpZW50cyIsImF1ZCI6Indld29yay1ncmFwaCIsImlhdCI6MTczNzYzNzQxMiwiZXhwIjoxNzM3NzIzODEyLCJndHkiOiJjbGllbnQtY3JlZGVudGlhbHMiLCJhenAiOiJTWEpNZXc2QzFhVlc0SlM1cm9GZzByMTNFM2xieFRTMiJ9.XUW7NpeKPotOdTv_I1lPjbxI4o3oJELyE3gaKHJwB5Hf60EA0aTdzymdQ318oFc_gh-nBD_Q_cFZuflYOvgfQhxoXTQO-EYsYDfH8xeAYzm2Olzzf-dJJ-IdMQFrN7rD1OuSDKiOB4dJl6u1MqK3vskEsxaFBnlXH0VzOP_VFyHSCs1B8X3qywpsphJxglx_X8ZO2_Qm4joxEUUp1GqfiW6VJpucrnfG2x78jvb1ngtZw_WJQruNj9TIu7FaYjQ8RHIOrm9CfU9b1knhtTezAmSOyQuPQ0howguIcZE0E6ALW_EhUYLJsoXjl0lcWEY2makGGt1xlutOWmOyw_RFvQ' --data-binary '{"query":"query GetUserWiFiCredentials($input: ID!) {\n getUserWiFiCredentials(id: $input) {\n username\n password\n }\n}","variables":{"input":"1d945eb0-bab6-013d-41c3-7280ce9be44a"}}' --compressed

```


SXJMew6C1aVW4JS5roFg0r13E3lbxTS2   -> client_id
```  
{  
  getUserWiFiCredentials(id: "member_uuid") {  
    username  
    password  
  }  
}  
```  
```  
mutation {  
  doResetUserWiFiCredentials(id: "member_uuid") {  
    result {  
      username  
      password  
    }  
  }  
}  
  
```  
``` [https://pr-557-wifi-service-token-auth-64ceb1.phoenix.dev.wwrk.co/graphql](https://pr-557-wifi-service-token-auth-64ceb1.phoenix.dev.wwrk.co/graphql) `

---
