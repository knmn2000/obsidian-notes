sanity cms for company registration part is pending

> "Please find below the definitions and nomenclature for variants:  
> 
> - **Business Address Plan:** Premium address for your business card, website, and more with mail and package handling services.
>     
> - **GST Registration Plan:** Premium address with government-compliant documentation for GST registration
>     
> - **Company Registration Plan:** Premium address with government-compliant documentation for company registration
>     
> 
> We can make the changes in the website using this."


ask anurag if existing blogs should be edited as well?


find out if we actually need to make changes in shopify. OMS se hojayega kya
shopify me import kar sakte hai kya change karke
myhq ka sync kaise hota hai virtual office ka. pehle vo kaise work kar raha tha.

---
## Changing vo products descriptions
sanity:
- descriptions change karne hai
- need to check meta as well. multiple touch points
-> check, do the existing descriptions exist anywhere in the:
- codebase
- firebase
- shopify
---
## changing business reg -> company reg plan

shopify:
- descriptions are not on shopify. so we will have to make change of the plan change (business registration plan to company registration plan).
- can look into importing and exporting
	- how do we test on staging?
- upflex me changes karne hai. doosra wala is not in use

backend mentions:
- on demand server 
- wework co in
- wework co in CMS
- on demand dashboard
-> dont change legacy code, add a new condition for the new enum and add comments as required 

Sanity:
- change description + title

keywords:
- BUSINESS_REGISTRATION
	- understand how this key is being used
	- ye change hua toh kya hoga
- business-registration-vo-package

ye change legality and NOC pe aana hai kya?
