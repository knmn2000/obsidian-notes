the redirection after logging in should be consistent
- check in accounts page
- sometimes redirection doesnt happen -> check confirm contact case. 

fix cookie setting unsetting


wework website
ondemand (doesnt use auth package) - x
global
orders service - x
credits service - x
ecom

need to deploy everything on dev and test.
raise PR for staging
- **wework website** is already there -> should add toggle (toggleable from localhost)
	- make reset wifi work -> logic will be different

- ondemand -> take help of madhu -> PR https://github.com/wework-india/wework-on-demand-server/pull/1631/files
	- STAGING PR: https://github.com/wework-india/wework-on-demand-server/pull/1653/files#
	- PROD (COMPARISON MODE): https://github.com/wework-india/wework-on-demand-server/pull/1715/files




- global -> take from https://github.com/wework-india/wework-global/pull/136/files | https://github.com/wework-india/wework-global/pull/135/files
	- STAGING PR: https://github.com/wework-india/wework-global/pull/146 STILL ON STAGING
	- PROD (COMPARISON MODE): https://github.com/wework-india/wework-global/pull/181



- orders service
	- STAGING: https://github.com/wework-india/wework-orders/pull/184/files
	- PROD (COMPARISON MODE): https://github.com/wework-india/wework-orders/pull/201

- credits service -> cherrypick from https://github.com/wework-india/wework-credits-service/pull/120/commits
	- STAGING PR: https://github.com/wework-india/wework-credits-service/pull/124
	- still on staging
	- PROD (COMPARISON MODE): https://github.com/wework-india/wework-credits-service/pull/136
 
- ecom -> add to staging, PR: https://github.com/wework-india/ecommerce/pull/118
	- STAGING PR FOR ECOM: https://github.com/wework-india/ecommerce/pull/146
	- https://github.com/wework-india/ecommerce/pull/254
	- PROD (COMPARISON MODE): https://github.com/wework-india/ecommerce/pull/319

unable to check some things because of CORS issue
- verify the doable things then deploy everything to develop to check

maybe we should get redirect_url from the backend (omnilogin backend)

401 issue coming in 
- credits
- OD

replace profile_uid with account_uid (not in params, the token now gives account_uid)
- done in OD
- done in credits service
- done in package
	- need to update places where package is being used
	- done in global
