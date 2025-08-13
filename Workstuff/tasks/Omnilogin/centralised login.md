







---
make SDK cleaner so that the feature-flag-off case PR can be raised. 

fix tooljet config, sensible config should be fetched.

raise PR for omni login web
get kishan to host backend 
test things out with wework website



issues:
- findOrCreateUsers mutation thing is not working with our service token.
	- ask global to help out
tasks:
- 1 -> get backend with mongoDB working -> done
	- add tooljet support
		- can read and updated -> done 
		- need to figure out how to add new configs -> done
		- need to figure out how to add new applications -> done
		- figure out delete -> done
		- figure out disabling particular config
- 2 -> setup okta, auth0 and google with package -> done
	- try out auth0 login on wework using the package -> done
	- write support for exchanging auth0 and okta token with UMS token
	- try out actual flow on wework website, which happens on omnilogin page and uses auth0 
		- the login config should be fetched from backend
		- the auth0 token should be replaced by UMS token
- 3 -> deployment of backend and omnilogin page -> need kishan's help
- 4 -> need to setup logs (newrelic) and sentry, and EFK
- 6 -> 


fix the implementation in website
- pass correct props
decide on the functions in the package, what will we use for parent app, and what will be there for omnilogin frontend page


in the package: 
	decide how we can move the auth0 provider to the package, and similarly, the okta provider to the pacakge.
		- may have to use some kind of switch case here.
	- fetch the config from backend service, (at least get the response, and save that)

flow: 
1. **User clicks login on the parent app (`parentapp.com`).**
2. **Parent app redirects to `/login` (actually `login.wework.co.in` via rewrite).**
3. **OmniLogin fetches the appropriate Auth provider config (Auth0, Okta, etc.).**
4. **User logs in through the selected provider.**
5. **OmniLogin exchanges the provider token for a UMS token.**
6. **UMS token is stored in `localStorage` under `parentapp.com`.**
7. **User is redirected back to the parent app with an authenticated session.**
8. **All subsequent API requests use the UMS token.**

ask joyan to show grapqhl stuff and test out things in the playground.
- print hub
- wifi















---
> - *List down all flows*
>     - *For each flow, we need flow diagram*
>     - *rough sketch ( Not required for each flow )*
>     - *List all API’s for each flow*
>         - *explore listeners as well*
>     - *API’s Payload and Response*
>     - *FE Side components and their structure*
> - *Auth Providers( Auth0, Okta, Google Sign In and UMS) should be integrated into our package*
>     - *Support all methods of Auth Providers*
>         - *Table All the methods of each Auth Providers which we have used*
>         - *Base Class*
>             - *Methods - same structure for each Auth providers*
>         - *Extend Base Class for Auth Providers*
>             - *Auth Providers Class Will override base class methods*
>     - *Logic of All Auth Providers we will try to merge into same structure*
> - *Identify Security Issues*
> - *Login Service - It will be backend*
>     - *this will provide the configs for different applications, these configs specify exactly how the login flow will happen.*
> - *Modular Functionality - Minimum : Logo change, colors*
> - *Admin UI to edit and add Application’s login config*
>     - *Add in the config*
>     - *Edit Config*
>     - *Disable/Remove config*
> - *Development efforts*
>     - *Minimal Changes required for Authentication*
>     - *Authorization should be handled by parent applications itself (Low Priority)*



---
- at the time of purchasing anything, check what kind of login we require (auth0 or normal bhi chalega?)
	- bundle buying
	- printing 
	- VO purchase 
- if user directly visits a privileged URL like my-account
- if auth is not there or expired.
- if auth expires in between
	- user should auth again without break in flow.

---

loginUser / signup user is called
- using the auth token from auth0, we fetch the 
needed details.
- ^ this needs to be switched out with UMS
- UMS will directly return teh userDetails
- keep other things like fetchMembers, credits, wifi creds etc empty
- parse the info returned from UMS in a way that current implementation will work 
- note exactly what more we will need for when we actually migrate data.

- after the thunk completes, there is something called extrareducers, in which we have cases of "fulfilled", "rejected" and "pending"
	- edit the fulfilled part so that we set the state correctly. mock out data we dont get.

USER INFO THAT AUTH GIVES. WE NEED TO TRANSFORM UMS RESPONSE INTO THIS FORM
not sure where gravatar is coming from, when its fetched?
{
    "createdAt": {
        "_seconds": 1736098082,
        "_nanoseconds": 278000000
    },
    "shopifyCustomerId": 8582347620636,
    "auth0Id": "60aa9cf0-adb7-013d-80c0-466ab4a75104",
    "phone": "",
    "gstin_address": {},
    "name": "zienaueuq7",
    "isODRecurringCustomer": false, -> NOT USED
    "gstin": "",
    "email": "zienaueuq7@cobmk.com",
    "hasCompletedKYC": false,
    "eligible_for_free_loyalty_pass": false,
    "partnerShopifyCustomerId": 8757654421777, -> NOT USED
    "points_expire_at": null,
    "isKYCMandate": false,
    "id": "zienaueuq7@cobmk.com",
    "isSurveyCompleted": false,
    "isRepeatUser": false,
    "recentBookings": [],
    "points_balance": 0,
    "referralLinkYotpo": "http://rwrd.io/qsc37xt",
    "successfulReferralCountYotpo": 0
}

WHAT UMS RETURNS
{
	"id": 41,
	"created_at": "2025-01-05T18:10:23.838Z",
	"uid": "68dc0296-99ce-4e34-94cd-5f2306df536e",
	"user_id": 37,
	"email": null,
	"password": null,
	"is_email_verified": false,
	"auth0_id": null,
	"entity_id": 1,
	"photo_url": null,
	"primary_account": true,
	"is_active": true,
	"parent_id": null,
	"last_login_at": "2025-01-05T18:10:23.835Z",
	"home_building_code": null,
	"current_building_code": null,
	"phone_number": "8053924822",
	"linked_accounts": []
}


currently, in the login flow, we depend on the auth token to fetch things like
- user documents
- user's teammates
- credits
- wifi creds

there is backend dependency in on demand server, auth

--- 

there is a tight dependency with auth0id in places like keypasses, and conference room booking
even if we migrate all the global data to UMS, how do we handle new users? as they wont have auth0id.
need a solution for this.

### soln:
all the new users will signup using auth0, and we will store that data(auth0id) in UMS. further logins will happen via UMS, and we will obtain auth0id wherever required. 

### outcome:
migration will be easier.
we can move all the users to UMS, they can signin via omnilogin. existing users will already have auth0id, and new users will have to signup via auth0.

### dependencies:
obtaining auth0id from global for all the existing users.


---
run DB locally, add the required column (auth0id to usersv2)
understand how wework website login exactly works, where data flows and comes from
try to replace that with UMS, add dummy data for missing fields as required

p2- 
	printhub wont work, so we will have to open microfrontend again and proceed with login.
	- if we already have auth0 token, we can fetch the details required (make sure if token is not expired)
	- if we dont have auth0 token, then open microfrontend with config of auth0 login.


101851609767