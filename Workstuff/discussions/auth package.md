1-> auth0 me umsauthissuer kya kar raha hai

2-> take jwksuri as arugment, instead of doing isproduction

3-> profile_uid and account_uid both are needed 
	- profile_uid is needed only for spaceops, baaki jageh account rakhenge

	check in ums.validator, there we are sending profile_uid.
		decoded.profile_uid || decoded.account_uid
		
should we have a different validator for spaceops??
or should we have a spaceops flag?

use http service wherever we are doing axios calls. 
dont do axios calls by default

change name of api.ts within api folder to ums.ts or xyz.ts, depending on whatever provider is using that api.

folder names:
	apis, configs, modules. plural rakho

remove index.ts from all folders, or do it properly.

remove guards, and global. remove unused stuff. 

---

in UMS enabled repos (like ecom)
move the ums auth guard to the package and use it there. 

