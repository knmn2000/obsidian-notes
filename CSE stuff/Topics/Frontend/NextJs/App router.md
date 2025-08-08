
#### Difference between <a> tag and <Link/>
if we are navigating within the application

<a> -> we actually load the page. we dont get the benefits of a single page application
<Link> -> we actually make use of the page rendered on the server side, and we use JS to switch between the pages. we dont actually load a new page.

we can see this by looking at the reload button on the browser, for <a> it loads, for <Link> nothing happens, nothing has to load

