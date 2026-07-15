
# Shadow DOM
_Shadow_ DOM allows hidden DOM trees to be attached to elements in the regular DOM tree — this shadow DOM tree starts with a shadow root, underneath which you can attach any element, in the same way as the normal DOM.

![SVG version of the diagram showing the interaction of document, shadow root and shadow host.](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM/shadowdom.svg)

There are some bits of shadow DOM terminology to be aware of:

- **Shadow host**: The regular DOM node that the shadow DOM is attached to.
- **Shadow tree**: The DOM tree inside the shadow DOM.
- **Shadow boundary**: the place where the shadow DOM ends, and the regular DOM begins.
- **Shadow root**: The root node of the shadow tree.

You can affect the nodes in the shadow DOM in exactly the same way as non-shadow nodes — for example appending children or setting attributes, styling individual nodes using element.style.foo, or adding style to the entire shadow DOM tree inside a [`<style>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/style) element. The difference is that none of the code inside a shadow DOM can affect anything outside it, allowing for handy encapsulation.

Before shadow DOM was made available to web developers, browsers were already using it to encapsulate the inner structure of an element. Think for example of a [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video) element, with the default browser controls exposed. All you see in the DOM is the `<video>` element, but it contains a series of buttons and other controls inside its shadow DOM. The shadow DOM spec enables you to manipulate the shadow DOM of your own custom elements.


---
# HMR hot module replacement

In React, **Hot Module Replacement (HMR)** is ==a development feature that updates your application’s code in real-time without requiring a full page refresh==. [[1](https://reactrouter.com/explanation/hot-module-replacement), [2](https://www.sanity.io/glossary/hot-module-replacement), [3](https://www.linkedin.com/pulse/understanding-hot-module-replacement-hmr-frontend-rakibul-hasan-dihan-uih7c)]

When you modify a component, the build tool (like **Vite** or **Webpack**) sends only the changed code "chunks" to the browser. The application then swaps out the old module for the new one instantly, which significantly speeds up the development cycle. [[1](https://webpack.js.org/concepts/hot-module-replacement/), [2](https://beyondthecode.medium.com/understanding-hot-module-replacement-hmr-f13f66cb5ada), [3](https://www.sanity.io/glossary/hot-module-replacement), [4](https://www.linkedin.com/pulse/understanding-hot-module-replacement-hmr-frontend-rakibul-hasan-dihan-uih7c)]

Key Benefits

- **State Preservation:** Unlike a traditional full-page reload, HMR attempts to keep the application's current state intact. For example, if you are working on a modal with a form, the modal stays open and the form data is preserved after you save a code change.
- **Faster Iteration:** You see your changes (CSS or JS) almost instantly, which maintains your development flow.
- **Efficient Debugging:** It allows you to tweak styling or logic without having to navigate back to the specific screen or state you were testing. [[1](https://reactrouter.com/explanation/hot-module-replacement), [2](https://dev.to/cronokirby/react-typescript-parcel-setting-up-hot-module-reloading-4f3f), [3](https://medium.com/@baphemot/react-hot-module-reload-f6b3d34b9b86), [4](https://www.linkedin.com/pulse/understanding-hot-module-replacement-hmr-frontend-rakibul-hasan-dihan-uih7c), [5](https://beyondthecode.medium.com/understanding-hot-module-replacement-hmr-f13f66cb5ada)]

Modern Implementation: Fast Refresh [[1](https://namastedev.com/blog/maximizing-development-speed-with-fast-refresh-in-react-applications/)]

While "HMR" is the underlying mechanism provided by bundlers, modern React development (specifically since React 16.9+) uses a more robust version called **Fast Refresh**. [[1](https://coreui.io/answers/how-to-set-up-hot-reload-in-react/), [2](https://dev.to/leapcell/beyond-hmr-understanding-reacts-fast-refresh-13h8)]

- **Fast Refresh** is the official React-aware HMR solution.
- It is more reliable than older tools like "React Hot Loader" because it is natively supported by the React engine.
- Tools like Vite and [Next.js](https://nextjs.org/docs/architecture/fast-refresh) come with this feature enabled by default. [[1](https://www.reddit.com/r/reactjs/comments/1kxiu56/whats_the_state_of_the_art_for_hmr_hot_reloading/), [2](https://leapcell.medium.com/beyond-hmr-understanding-reacts-fast-refresh-d6d80ef0fe4e), [3](https://www.reddit.com/r/reactjs/comments/yw0bdm/how_is_reacts_hot_module_reloading_implemented_at/), [4](https://dev.to/leapcell/beyond-hmr-understanding-reacts-fast-refresh-13h8), [5](https://coreui.io/answers/how-to-set-up-hot-reload-in-react/)]

Comparison: Live Reload vs. HMR

| Feature [[1](https://www.snowpack.dev/concepts/hot-module-replacement), [2](https://beyondthecode.medium.com/understanding-hot-module-replacement-hmr-f13f66cb5ada), [3](https://beyondthecode.medium.com/understanding-hot-module-replacement-hmr-f13f66cb5ada), [4](https://www.linkedin.com/pulse/understanding-hot-module-replacement-hmr-frontend-rakibul-hasan-dihan-uih7c), [5](https://reactrouter.com/explanation/hot-module-replacement)] | Live Reload                                     | Hot Module Replacement (HMR)               |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------ |
| **Action**                                                                                                                                                                                                                                                                                                                                                                                                                                          | Refreshes the entire browser page.              | Updates only the changed modules.          |
| **State**                                                                                                                                                                                                                                                                                                                                                                                                                                           | Resets all local state to the initial value.    | Preserves the current state of components. |
| **Speed**                                                                                                                                                                                                                                                                                                                                                                                                                                           | Can be slow as the whole app must re-bootstrap. | Near-instant updates for specific modules. |

---
# Content script injection
**Content script injection** is ==the process by which a browser extension "inserts" JavaScript or CSS into a webpage==. While extensions have their own background environment, they can’t see the actual content of a website (like your emails or a news article) unless they inject a script into that page's context. [[1](https://dev.to/oluwatobi2001/a-beginners-guide-to-building-content-scripts-df), [2](https://link.springer.com/chapter/10.1007/979-8-8688-1594-2_8), [3](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_scripts), [4](https://www.youtube.com/watch?v=WZ6wMVYWsd4)]

What is "Injection"?

When a script is injected, it gains access to the page's **Document Object Model (DOM)**. This allows it to: [[1](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts), [2](https://www.plasmo.com/blog/posts/content-scripts-ui)]

- **Read** what is on the screen (e.g., scraping price data).
- **Modify** existing elements (e.g., hiding ads or changing background colours).
- **Add** new UI (e.g., placing a custom button next to a "Compose" button in Gmail). [[1](https://www.youtube.com/watch?v=ezhJezGX5ak), [2](https://docs.plasmo.com/framework/content-scripts), [3](https://www.youtube.com/watch?v=WZ6wMVYWsd4), [4](https://docs.plasmo.com/framework/content-scripts-ui), [5](https://www.plasmo.com/blog/posts/how-to-inject-a-react-component-onto-a-web-page-using-a-chrome-extension)]

The Lifecycles

In Plasmo, "lifecycles" refer to the automated sequence the framework follows to ensure your UI appears at the right time and place without breaking the host website. [[1](https://docs.plasmo.com/framework/content-scripts-ui/life-cycle), [2](https://docs.plasmo.com/framework/content-scripts-ui/life-cycle)]

1. **Browser Injected Lifecycle (`run_at`):**  
    This determines _when_ the initial script hits the page:
    - **`document_start`:** Before the page even renders.
    - **`document_end`:** Once the HTML is loaded but before images/iframes finish.
    - **`document_idle`:** (Default) The browser waits for a quiet moment after the page is mostly loaded.
2. **Plasmo CSUI Lifecycle:**  
    Once the script is on the page, Plasmo manages a specialized "mounting" lifecycle for components (React/Vue/Svelte):
    - **Anchor Finding:** Plasmo searches for a specific element (an "anchor") on the page to attach your UI to.
    - **Root Creation:** It automatically creates a Shadow DOM (a private sub-DOM) to prevent the website’s CSS from messing up your component’s styling.
    - **Mounting:** It renders your component into that root container.
    - **Observation:** Plasmo uses [MutationObservers](https://docs.plasmo.com/framework/content-scripts-ui/life-cycle) to watch the page; if the website dynamically removes your anchor, Plasmo can automatically unmount or remount your UI to prevent memory leaks or broken interfaces

---
# oauth flow -> interesting 

##  Authentication System

**Key files:** [`background.ts`](https://markdownlivepreview.com/background.ts), [`background/messages/login.ts`](https://markdownlivepreview.com/background/messages/login.ts), [`background/messages/logout.ts`](https://markdownlivepreview.com/background/messages/logout.ts), [`hooks/useLeafyAuth.ts`](https://markdownlivepreview.com/hooks/useLeafyAuth.ts), [`lib/firebase.ts`](https://markdownlivepreview.com/lib/firebase.ts), [`components/LoginScreen.tsx`](https://markdownlivepreview.com/components/LoginScreen.tsx)

### OAuth Flow

Chrome extensions cannot use standard web OAuth redirects. The flow uses `chrome.identity.launchWebAuthFlow`:

```
1. User clicks "Sign in with Google"
2. popup.tsx → sendToBackground("login")
3. background/messages/login.ts:
   a. Constructs Google OAuth URL with:
      - client_id (Web Application type, NOT Chrome App)
      - redirect_uri: https://{EXTENSION_ID}.chromiumapp.org/
      - response_type: token
   b. Calls chrome.identity.launchWebAuthFlow({ url, interactive: true })
   c. Google auth popup opens → user signs in → redirect with access_token
   d. Uses access_token to create Firebase GoogleAuthProvider credential
   e. signInWithCredential(auth, credential)
4. background.ts: onAuthStateChanged fires → syncs user to Firestore and storage
```

**Key decision**: We use a "Web Application" OAuth client, NOT a "Chrome App" client. Chrome App OAuth uses `chrome.identity.getAuthToken` which returns a limited-scope token. Web Application + `launchWebAuthFlow` gives us a full Google OAuth token that works with Firebase Auth.

### Auth State Sync (background.ts)

```typescript
onAuthStateChanged(auth, async (firebaseUser) => {
  if (firebaseUser) {
    // 1. Build user object from auth data first (works even if Firestore is offline)
    // 2. If Firestore doc exists: heal-merge auth-derived name/avatarUrl
    //    over the doc and persist any missing fields. Avoids overwriting
    //    webhook-seeded subscription state (productpage-first signups).
    // 3. If doc doesn't exist: setDoc with the auth-derived defaults.
    // 4. Write to chrome.storage.local: USER + AUTH_STATUS
    // 5. Ensure personal team exists (promise-guarded against races)
    // 6. Redeem pending invites for this email
    // 7. Whitelisted-domain check → enterprise tier promotion + auto-join
    // 8. Recalculate teamCount denormalized counter
  } else {
    // Set AUTH_STATUS to "unauthenticated"
  }
})
```

**Two race conditions solved**:

1. `onAuthStateChanged` fires multiple times (initial load + each auth change). Without guarding, `ensurePersonalTeam` could create duplicate teams. Solved with a module-level `personalTeamPromise` variable — if a creation is in flight, subsequent calls await the same promise.
    
2. **Productpage-first signup**: a user can pay for Pro on `get-leafy.com` BEFORE ever installing the extension. The Razorpay webhook seeds `users/<uid>` with subscription fields + `uid`/`email`/`createdAt`/counters, but no `name`/`avatarUrl` (Razorpay's `notes` field doesn't carry display name). On the user's first extension sign-in, the heal block in step 2 above merges the auth-derived `name`/`avatarUrl` over the seeded doc and writes them back. Without this, `UserProfile.getInitials(undefined)` crashes the popup. Also, `ensurePersonalTeam` would create `"undefined's Garden"` as the team name; that's mitigated separately by [`displayTeamName`](https://markdownlivepreview.com/lib/terminology.ts).
    

**LeafyUser subscription fields**: `subscriptionTier`, `subscriptionStatus?`, `subscriptionRenewsAt?`, `activeSubscriptionId?`. All optional; the extension never writes them. Set ONLY by the productpage Razorpay webhook (Admin SDK bypasses rules). The popup live-syncs these via `onSnapshot(users/<uid>)` so cancellations/halts propagate without requiring a sign-out/sign-in.

----

# promise depduplication
1. `onAuthStateChanged` fires multiple times (initial load + each auth change). Without guarding, `ensurePersonalTeam` could create duplicate teams. Solved with a module-level `personalTeamPromise` variable — if a creation is in flight, subsequent calls await the same promise.
```typescript
let personalTeamPromise: Promise<Team> | null = null

function ensurePersonalTeam(user: LeafyUser): Promise<Team> {
  // 1. The Carpool Check: If a request is already flying, just return it!
  if (personalTeamPromise) return personalTeamPromise
  
  // 2. Start the car: Assign the actual async work to the variable
  personalTeamPromise = (async () => {
    try {
      // ... Firestore logic to check/create the team ...
    } finally {
      // 3. Clean up: Once the work is fully done (success or fail), 
      // clear the variable so future calls later on can run fresh.
      personalTeamPromise = null
    }
  })()
  
  return personalTeamPromise
}
```


interesting stuff for the blog
- promise deduplication (personalTeamPromise)
- oauth (multiple environments and talking across)
- using background workers
- firestore rules
- indexes and why 


