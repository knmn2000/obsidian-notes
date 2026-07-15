
# Todo -
- building cards should lead to building details -> karna hai
- MongoDB F&B icon coming twice - ECOM ISSUE
- bundle issue from ondemand dashboard to website -> 

# blocked -
- add pop up to ondemand referral page with big blurs

# Later
- kyc issue on admin dashboard (fetch kyc info from ums on od server)
- events to digital keycard on BE p0
- fix 500 for ecom 
- wifi task - graceful message p1
# Done - 
- get map sync live - x
- update FAQs 
- google auth
- fetch loyalty from oms - x
- update virtual tours (2) - x
- add events to omni.
- get FAQ live. (its slow)


# Backburner: 
may not pickup. low priority
- backend event names
	- Email_OTP_Sent (with platform)
	- Phone_OTP_Sent (with platform)
	- /Login (login_successful)
	- /confirm-contact (contact_confirmed)
--- 

---
# revalidation
```
curl --location '[https://wwistaging.in/api/revalidate/?path=%2F](https://wwistaging.in/api/revalidate/?path=%2F)' \  
--header 'Authorization: 2511917277d5479a9c00d23680380221'
```











---
things to try
- cloudfront (ask kishan)
- some scripts can be made [[async]] (need to check)
- check if images are properly optimised for mobile
- verify if nextjs [[CSE stuff/General terms/Caching|caching]] is setup correctly
- lazy loaded components
- check libphonenumber, why is it so big