staging sample email: WE-IN-16968@wework.co.in
Todo: 
sample email:
WE-IN-57702@wework.co.in

## Tooljet - vo audit dashboard
- xapikey -> make sure this works in different envs (staging, dev, prod)
- find missing properties and return
- how to get initial subscription date -> for `member_since` date

---
### search and quick preview:

search functionality -> PRESENT
select building -> get buildings required -> PRESENT

- company name, - PRESENT
- primary user, - NAME NOT PRESENT
- GST/PAN, - NOT PRESENT 
- current status - NOT PRESENT

button to load more (pagination)

https://server-dot-wework-staging.el.r.appspot.com/api/v1/admin/virtual-offices

apart from the existing details, we need
{
	GST,
	PAN,
	current status (currentDate > expiry date ? true : false),
}
please give an option to fetch all of the entries at once, tooljet has its own pagination implemented.
or, please give 
### preview page

- primary username - EMAIL PRESENT, NAME NOT PRESENT
- type of plan - PRESENT
- start date- PRESENT
- tenure - PRESENT 
- member since - is this the same as created_at ?
- primary mob number - PRESENT
- all directors verified - NOT PRESENT
- end date - expiresOn is there but it is empty in the API
- gst/pan - PRESENT
- company type - NOT PRESENT

- table of director's details
	- email - NOT PRESENT
	- name - PRESENT
	- verification ID type (aadhar, pan, passport etc) - ??? all directors have pan?
	- verification id - PRESENT

- downloadable documents - is this always within payments? 
	- invoice 
	- msa
	- noc
	- bills

---

