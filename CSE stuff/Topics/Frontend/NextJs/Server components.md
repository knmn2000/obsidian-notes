
### Getting things from [[Database]] in a component

add a /lib folder that has the logic of talking to the server. this /lib folder should be out of the app folder for safety reasons.
add the script like

```javascript
// import this into your component.
import sql from 'better-sqlite3'
const db = sql('stuff.db')
const getStuff async () => {
	return db.prepare('SELECT * FROM bruh').all();
}
```

->>>> ***server components can be async-awaited.***  <<<<<-

