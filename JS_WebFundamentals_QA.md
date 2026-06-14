**JavaScript & Web Fundamentals**

Async/Await, Promises, ES Modules & API Fetching

Complete Theory Question & Answer Guide – 20 Questions

**Q1. JavaScript Event Loop: Task Deferral**

**Answer:**

When an intern types “Mac” rapidly (3 characters in 100ms), the debounce function works as follows:

Each keystroke triggers the input event handler, which calls debounce(). Inside debounce, a closure variable (a timer ID) is checked. If a previous timer exists, clearTimeout() cancels it immediately from the Call Stack. Then setTimeout() is handed off to the Web APIs layer (the browser environment), which starts a countdown (e.g., 300ms delay).

Since all three keystrokes arrive within 100ms, the first two timers are cancelled before they expire. Only the third timer survives. After 300ms of silence, the Web API moves the callback into the Callback Queue (Macrotask Queue). The Event Loop continuously checks: “Is the Call Stack empty?” When it is, it picks the callback from the queue and pushes it onto the Call Stack, executing the actual search/filter logic.

Result: The filter runs only once – after the user finishes typing “Mac” – not three times.

**Q2. Event Loop: Single-Threaded Blocking**

**Answer:**

JavaScript is single-threaded, meaning only one piece of code runs at a time on the Call Stack. If a while(true) {} loop is executed in the main body of src/app.js, it permanently occupies the Call Stack and the Event Loop cannot push any other callback or task onto the stack.

Consequences:

• The UI completely freezes – no clicks, scrolls, or input events are processed.

• Any pending fetch() callbacks (like rendering device cards after an API response) are stuck in the Callback/Microtask Queue and never execute.

• The browser may display a “Page Unresponsive” warning and offer to kill the tab.

• Active network requests continue in the background (handled by the OS/browser networking layer), but their response handlers cannot run until the Call Stack is free – which never happens.

This is why long-running operations must be handled asynchronously (via Promises, Web Workers, or chunked processing with setTimeout).

**Q3. Fetch API: Rejection Conditions**

**Answer:**

The fetch() API only rejects (triggers .catch() or the catch block in async/await) for genuine network-level failures:

• No internet connection

• DNS resolution failure (hostname not found)

• CORS policy block before a response is received

• Server unreachable / connection refused

Importantly, HTTP error responses (like 404 Not Found, 500 Internal Server Error, 403 Forbidden) do NOT cause fetch() to reject. The Promise resolves successfully because the server did communicate – it just reported an error. In these cases, response.ok is false (it is true only for status codes 200–299).

The if (!response.ok) check in handleResponse is therefore essential to catch these HTTP-level errors and throw them explicitly, converting them into rejections that the calling code can handle uniformly in its catch block. Without it, a 404 or 500 would silently appear to succeed.

**Q4. Async/Await: Sequential vs. Concurrent Operations**

**Answer:**

Sequential (waterfall) approach is slow – each request waits for the previous to finish, so total time equals the sum of all request times:

_const devices = await getDevices(); // starts, waits ~200ms_

_const categories = await getCategories(); // only starts AFTER devices finish_

Concurrent with Promise.all() is fast – both requests fire simultaneously, so total time equals only the longest single request:

_const \[devices, categories\] = await Promise.all(\[_

_getDevices(),_

_getCategories()_

_\]);_

Other useful Promise combinators:

• Promise.allSettled() – waits for all, never rejects; useful when partial failure is acceptable.

• Promise.race() – resolves/rejects with the first settled Promise.

• Promise.any() – resolves with the first fulfilled one.

Benefit: For two 200ms requests, sequential takes ~400ms while concurrent takes ~200ms – a 50% improvement.

**Q5. Promises: Settled States**

**Answer:**

A Promise has exactly three states: Pending, Fulfilled, or Rejected. Once a Promise is settled (either fulfilled or rejected), it is immutable – it cannot transition back to Pending, change its resolved value, or change its rejection reason. This is a core guarantee of the Promise specification.

Impact on UI state management: Because a Promise settles exactly once and irreversibly, we can safely model UI states around it:

• Before the Promise settles (Pending): Show a loading spinner, disable the submit button.

• On Fulfilled: Hide the spinner, show a success message, re-enable the button.

• On Rejected: Hide the spinner, show an error message, re-enable the button.

Since the settled value never changes, we never need to “re-check” a fulfilled Promise – we can safely cache and reuse the result. There is no risk of a Promise unexpectedly flipping back and causing ghost state changes in the UI.

**Q6. Fetch API: Request Headers**

**Answer:**

When createDevice() sends a POST request with a JSON body, the Content-Type header tells the server how to interpret the raw bytes in the request body:

_fetch('/devices', {_

_method: 'POST',_

_headers: { 'Content-Type': 'application/json' },_

_body: JSON.stringify(deviceData)_

_});_

Without this header:

• Many server frameworks (Express.js, json-server) will not parse the body as JSON.

• Express with express.json() middleware silently leaves req.body as undefined or {}.

• The server may respond with a 400 Bad Request or silently save a malformed record.

With the header, the server's JSON body parser recognises the content type and correctly deserialises the JSON string into a JavaScript object. The header is a contract between client and server about the data format.

**Q7. ES Modules: Scope Isolation**

**Answer:**

No. src/app.js cannot access or modify API_BASE_URL. In the ES Modules system, every file has its own private scope. Variables defined at the top level of a module are not global – only values explicitly marked with export are accessible to other modules via import.

_// api.js_

_const API_BASE_URL = 'http://localhost:3000'; // private – not exported_

_export async function getDevices() { ... } // accessible to importers_

Why this is good software engineering:

• Encapsulation: If the base URL changes, only api.js needs updating.

• Preventing accidental mutation: app.js cannot overwrite API_BASE_URL.

• No global namespace pollution: Each module's scope is completely isolated.

• Maintainability: Dependencies are explicit and declared via import statements.

**Q8. CSS Grid: Minmax & Auto-Fit Dynamics**

**Answer:**

Both auto-fit and auto-fill create as many columns as fit given the minmax(280px, 1fr) constraint. The difference appears when there are fewer items than the maximum possible columns:

_.devices-grid {_

_display: grid;_

_grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));_

_gap: 1.5rem;_

_}_

• auto-fill: Creates empty ghost columns to fill remaining space. Cards do not expand beyond their natural size.

• auto-fit: Collapses empty columns to zero width. Cards stretch to fill all available space using the 1fr unit.

On a 1440px monitor with only two device cards, auto-fit causes the two cards to expand and each fill roughly 50% of the width – which can look very wide. With auto-fill, cards stay at their natural size and the rest of the row is empty space.

**Q9. CSS Variables: Dynamic Theming**

**Answer:**

CSS custom properties (variables) can be scoped to attribute selectors. In styles.css, the theme is structured like this:

_:root, \[data-theme=“light”\] {_

_\--bg-color: #ffffff;_

_\--text-color: #1a1a2e;_

_}_

_\[data-theme=“dark”\] {_

_\--bg-color: #1a1a2e;_

_\--text-color: #f0f0f0;_

_}_

When src/app.js sets document.documentElement.setAttribute('data-theme', 'dark'), the CSS engine re-evaluates which ruleset applies to the &lt;html&gt; element. The \[data-theme=“dark”\] block now matches, overriding the :root defaults.

Every element in the page that uses var(--bg-color) or var(--text-color) automatically receives the new values – the browser cascades them down the entire DOM without any JavaScript needing to touch individual elements.

This approach is highly performant: a single attribute change triggers a CSS repaint/recalculation for only the changed properties, rather than JavaScript looping through DOM nodes. It also enables OS-level dark mode detection via the prefers-color-scheme media query.

**Q10. Async Error Handling: Try/Catch Blocks**

The three blocks serve distinct purposes:

_try {_

_submitBtn.disabled = true;_

_spinner.style.display = 'block';_

_await updateDevice(id, data); // optimistic path_

_showSuccess('Device updated!');_

_} catch (err) {_

_showError(err.message); // handle failure_

_} finally {_

_submitBtn.disabled = false; // always runs_

_spinner.style.display = 'none';_

_}_

• TRY: Contains the optimistic path. If any awaited Promise rejects or a synchronous error is thrown, execution immediately jumps to catch.

• CATCH: Receives the error object. Displays error messages and prevents unhandled Promise rejections from silently failing.

• FINALLY: Executes unconditionally regardless of outcome – ideal for re-enabling buttons and hiding spinners.

Without finally, cleanup code would need to be duplicated in both try and catch, violating DRY (Don't Repeat Yourself) and risking inconsistent UI states.

**Q11. REST APIs: PUT vs. PATCH Methods**

**Answer:**

PUT performs a full replacement – the entire resource is overwritten with the provided payload. Any omitted fields are deleted or nulled:

_// PUT – replaces entire device object_

_PUT /devices/dev-01_

_{ “checkedOut”: true } // ALL other fields (name, category...) are lost_

PATCH performs a partial update – only the provided fields are changed, all others remain intact:

_// PATCH – updates only checkedOut_

_PATCH /devices/dev-01_

_{ “checkedOut”: true } // name, category, specs... all preserved_

Why PATCH is correct here: When updating only a device's checkout status, sending the full object each time (as PUT requires) is wasteful and error-prone. PATCH is semantically correct and more efficient for single-field updates.

**Q12. JavaScript DOM Manipulation: DocumentFragment vs. Direct Appends**

**Answer:**

Direct appendChild() in a loop causes DOM reflow/repaint thrashing – each append to a live DOM element may trigger a full layout recalculation. With 100 cards, this triggers up to 100 recalculations.

DocumentFragment approach – a single DOM write:

_const fragment = document.createDocumentFragment();_

_devices.forEach(device => {_

_const card = createCard(device);_

_fragment.appendChild(card); // no reflow yet_

_});_

_devicesGrid.appendChild(fragment); // single reflow/repaint_

innerHTML approach: Building all cards as an HTML string and setting innerHTML once also produces a single parse-and-render pass, but destroys existing event listeners on child elements and carries XSS risks if data is not sanitised.

For very large lists (thousands of items), virtual scrolling – rendering only visible items – is the most scalable solution.

**Q13. ES Modules: Default vs. Named Exports**

**Answer:**

Named exports (current approach in src/api.js):

_// api.js_

_export async function getDevices() { ... }_

_export async function createDevice() { ... }_

_// app.js – name must match exactly (or use alias)_

_import { getDevices, createDevice } from './api.js';_

Default export – only one allowed per module, importer chooses the name freely:

_// api.js_

_export default async function getDevices() { ... }_

_// app.js – no curly braces, any name works_

_import fetchDevices from './api.js';_

Best practice: Named exports are preferred for utility/API modules with multiple functions because they are explicit, support tree-shaking (bundlers remove unused exports), and are easier to discover via IDE autocomplete. Default exports suit modules that expose a single primary value, such as a React component.

**Q14. Web APIs: URLSearchParams Object**

**Answer:**

In src/details.js, the device ID is parsed from the URL using the URLSearchParams Web API:

_// URL: http://localhost:5173/item-details.html?id=dev-01_

_const params = new URLSearchParams(window.location.search);_

_const id = params.get('id'); // returns “dev-01”_

Why URLSearchParams is superior to manual string splitting:

• Correctness: Automatically decodes URL-encoded characters (%20, %26, etc.). Manual splitting on '?' and '=' fails with encoded values.

• Multiple parameters: ?id=dev-01&view=specs handled trivially via params.get('view').

• Edge cases: Returns null for missing keys, supports repeated keys via params.getAll(), and handles boolean flags correctly.

window.location.search provides the query string portion of the current URL (including the leading ?), passed directly to URLSearchParams.

**Q15. JavaScript Closures: Debounce Timer Preservation**

**Answer:**

The debounce function creates a closure over the variable timerId, which persists across every call to the returned inner function:

_function debounce(fn, delay) {_

_let timerId; // captured in closure_

_return function (...args) {_

_clearTimeout(timerId); // cancel previous timer_

_timerId = setTimeout( // schedule new timer_

_() => fn(...args), delay_

_);_

_};_

_}_

Because the returned function closes over the same timerId variable on every keystroke, clearTimeout(timerId) successfully cancels the previous timer each time.

Without the closure – if timerId were a local variable inside the returned function – each keystroke would create a fresh, independent variable, clearTimeout would receive undefined, and every keystroke would schedule its own execution, defeating the entire purpose of debouncing.

**Q16. HTTP Status Codes: Client vs. Server Errors**

**Answer:**

**Answer:**

When json-server is stopped and fetch() is called, the browser cannot establish a TCP connection. This is a genuine network failure – not an HTTP response.

What error is thrown:

_// Chrome / Edge_

_TypeError: Failed to fetch_

_// Firefox_

_TypeError: NetworkError when attempting to fetch resource_

How src/app.js handles it in the catch block:

• Displays a user-friendly message: “Unable to connect to the server. Please try again later.”

• The loading spinner is hidden via the finally block.

• The device grid remains empty or shows a cached/empty state.

This is distinct from a 404 or 500 error, which would be caught by the if (!response.ok) check in handleResponse – not by fetch()'s own rejection. A NetworkError means no response object exists at all.

**Q17. REST APIs: JSON Server ID Auto-generation**

**Answer:**

When createDevice() POSTs a payload without an id field, json-server automatically generates and assigns a unique ID before saving to db.json:

_// Request payload (no id provided)_

_POST /devices_

_{ “name”: “MacBook Pro”, “category”: “Laptop” }_

_// json-server response (ID auto-generated)_

_{ “id”: “a3f9k2”, “name”: “MacBook Pro”, “category”: “Laptop” }_

• json-server v0.x: Generates auto-incremented integer IDs (1, 2, 3...) – data type is number.

• json-server v1.x: Generates random string-based IDs (e.g., “a3f9k2”) – data type is string.

The type matters for strict equality comparisons (===). json-server handles type coercion in URL lookups automatically, but custom filtering code must account for the ID type. The generated id is returned in the POST response body so the client can immediately reference the new resource.

**Q18. CSS Grid: Mobile-First Override Logic**

**Answer:**

Mobile-first styling means writing base styles for the smallest screen first, then using min-width media queries to layer in complexity for larger screens:

_/\* Base – mobile (375px): single column \*/_

_.spec-details-grid {_

_display: grid;_

_grid-template-columns: 1fr;_

_gap: 1rem;_

_}_

_/\* Desktop (1024px+): split screen \*/_

_@media (min-width: 1024px) {_

_.spec-details-grid {_

_grid-template-columns: 1fr 2fr;_

_}_

_}_

• 375px mobile: Single column applies. Device image and specs stack vertically. No media query fires.

• 1024px desktop: Media query fires. Layout becomes two columns – a narrow left panel (photo, quick stats) and a wider right panel (full specifications).

Mobile-first is preferred for performance: mobile browsers apply the base styles without evaluating any media queries, and complexity is added only when needed.

**Q19. Relative Date Calculations: Time Offsets**

**Answer:**

Given device last updated at 2026-06-10T21:28:00Z and current client time of 2026-06-10T21:28:50Z:

_const updatedAt = new Date(“2026-06-10T21:28:00Z”);_

_const now = new Date(“2026-06-10T21:28:50Z”);_

_const diffMs = now - updatedAt; // 50,000 ms_

_const diffSeconds = Math.floor(diffMs / 1000); // 50 seconds_

_// Threshold check in formatRelativeTime:_

_if (diffSeconds < 60) return “just now”;_

_if (diffSeconds < 3600) return diffMinutes + “ minutes ago”;_

_// ...and so on_

Result returned: “just now” – most applications use this label for anything under 60 seconds.

Key insight: JavaScript Date objects subtract as milliseconds since the Unix epoch, making time arithmetic straightforward without any external library.

**Q20. ES Modules: Script Loading and Deferral**

**Answer:**

Standard &lt;script&gt; tags block HTML parsing. The module type changes this behaviour:

_&lt;!-- Blocks HTML parsing – avoid for app code --&gt;_

_&lt;script src=“app.js”&gt;&lt;/script&gt;_

_&lt;!-- Deferred by default – safe for DOM access --&gt;_

_&lt;script type=“module” src=“app.js”&gt;&lt;/script&gt;_

&lt;script type=“module”&gt; is deferred by default – fetched in parallel with HTML parsing but executed only after the entire DOM is constructed. It never blocks the HTML parser.

Additional module-specific behaviours:

• Always in strict mode (“use strict”) automatically.

• Own scope – top-level variables do not become global.

• Executed only once, even if imported by multiple modules (singleton behaviour).

• Adding async attribute makes the script execute as soon as fetched, without waiting for DOM – useful for analytics or non-DOM scripts.

This default-deferred behaviour means you can safely query the DOM directly in module code without waiting for the DOMContentLoaded event.