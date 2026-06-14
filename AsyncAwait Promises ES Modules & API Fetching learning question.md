> **Async/Await, Promises, ES Modules & API Fetching**
>
> **Part 1:** Theory Part
>
> **Conceptual Questions**
>
> 1\. **JavaScript Event Loop: Task Deferral**
>
> **Question:** Inside src/app.js, a custom debounce function is
> implemented to delay search execution. Explain how the JavaScript
> Event Loop, Call Stack, Web APIs, and Callback Queue interact when an
> intern types \"Mac\" rapidly (3 characters in 100ms) into the search
> input.
>
> 2\. **Event Loop: Single-Threaded Blocking**
>
> **Question:** What would happen to the dashboard UI responsiveness and
> any active network requests if a heavy, blocking synchronous loop
> (e.g. while(true) {}) was executed in the main body of src/app.js?
> Why?
>
> 3\. **Fetch API: Rejection Conditions**
>
> **Question:** In src/api.js, the helper handleResponse checks if
> (!response.ok). Why is this check necessary? What types of network
> events cause fetch() to actually *reject* (trigger a catch block) vs.
> what results in a resolved Promise with response.ok === false?
>
> 4\. **Async/Await: Sequential vs. Concurrent Operations**
>
> Question: In src/app.js, the function loadAndRenderInventory()
> performs a single fetch request. If we wanted to fetch the devices
> list AND a separate system configuration list (e.g., categories from
> /categories), writing: const devices = await getDevices(); const
> categories = await getCategories(); would create a network waterfall.
> How would you refactor this using Promise combinators to fetch them
> concurrently, and what is the benefit?
>
> 5\. **Promises: Settled States**
>
> **Question:** In src/details.js, once a device update is submitted,
> the code awaits updateDevice(). When a Promise transitions from
> Pending to Fulfilled (or Rejected), can it ever return to Pending or
> change its value? How does this impact how we handle UI states during
> network latency?
>
> 6\. **Fetch API: Request Headers**
>
> **Question**: In src/api.js, the createDevice function sends a POST
> request. Why is it critical to include \'Content-Type\':
> \'application/json\' in the headers? What happens on the server side
> if this header is omitted?
>
> **7. ES Modules: Scope Isolation**
>
> **Question:** In src/api.js, the variable API_BASE_URL is defined at
> the top level of the file but is not exported. Can src/app.js access
> or modify this variable? Why does this design represent a good
> software engineering pattern?
>
> **8. CSS Grid: Minmax & Auto-Fit Dynamics**
>
> **Question:** In styles.css, the device grid layout is defined as:
>
> .devices-grid {
>
> display: grid;
>
>    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
>
>    gap: 1.5rem;
>
>     }
>
> Contrast auto-fit with auto-fill in this context. What happens when
> there are only two device cards on a wide 1440px monitor?
>
> **9. CSS Variables: Dynamic Theming**
>
> **Question:** In styles.css and src/app.js, the app switches between
> dark and light themes by modifying the data-theme attribute on the
> \<html\> element. Explain how the CSS variables change in response to
> this attribute swap.
>
> **10. Async Error Handling: Try/Catch Blocks**
>
> **Question:** In src/app.js, the form submission event listener uses a
> try\...catch\...finally construct. Explain what each block does and
> why the finally block is specifically useful for UI states like
> buttons and spinners.
>
> **11. REST APIs: PUT vs. PATCH Methods**
>
> **Question:** In src/api.js, the function updateDevice uses the PATCH
> method, while the REST API cheatsheet mentions both PUT and PATCH.
> What is the technical difference between these two HTTP methods when
> updating a device\'s checkout status in db.json?
>
> **12. JavaScript DOM Manipulation: DocumentFragment vs. Direct
> Appends**
>
> **Question:** In src/app.js, the renderCards function loops through a
> list of devices and appends each card directly to elements.devicesGrid
> using appendChild(). Explain why using a DocumentFragment or mapping
> to a single string and setting innerHTML once is generally preferred
> for performance when dealing with large lists.
>
> **13. ES Modules: Default vs. Named Exports**
>
> **Question:** Look at src/api.js. It uses named exports (e.g., export
> async function getDevices()). Compare this to default exports. How
> does the import statement change in src/app.js if getDevices were
> exported as a default export instead?
>
> **14. Web APIs: URLSearchParams Object**
>
> **Question:** In src/details.js, how is the device ID parsed from the
> URL http://localhost:5173/item-details.html?id=dev-01? Which Web API
> is utilized here, and why is this better than manually splitting
> strings?
>
> **15. JavaScript Closures: Debounce Timer Preservation**
>
> **Question:** The debounce function in src/app.js relies on a closure.
> What variable is captured in the closure, and how does this prevent
> the search filter from executing multiple times?
>
> **16. HTTP Status Codes: Client vs. Server Errors**
>
> **Question:** If the mock database server (json-server) is stopped,
> causing fetch() in src/api.js to fail, what error is caught in
> src/app.js, and how does the UI handle it?
>
> **17. REST APIs: JSON Server ID Auto-generation**
>
> **Question:** In src/api.js, the createDevice function does not
> specify an id field in the payload. However, when you check db.json
> after adding a device, it has a unique id. How does this ID get
> generated, and what is its standard data type?
>
> **18. CSS Grid: Mobile-First Override Logic**
>
> **Question:** In styles.css, the layout container for the details page
> split screen is .spec-details-grid. How does mobile-first styling
> dictate how it renders on a 375px mobile screen versus a 1024px
> desktop screen?
>
> **19. Relative Date Calculations: Time Offsets**
>
> **Question:** In src/app.js, the function formatRelativeTime uses
> JavaScript Date math. If the database reports a device was updated at
> 2026-06-10T21:28:00Z and the client\'s current time is
> 2026-06-10T21:28:50Z, what string is returned and how is it
> calculated?
>
> **20. ES Modules: Script Loading and Deferral**
>
> **Question:** Standard \<script\> tags block HTML parsing unless they
> have defer or async attributes. What is the default loading behavior
> of \<script type=\"module\"\>? Does it block the DOM parsing of
> index.html?
>
> **Part 2: Practical Part (Reconstruction & Challenges)**
>
> **Rebuilding TechOps from Scratch**
>
> Navigate to [the JavaScript
> repository](https://github.com/Oluwateezzy/javascript.git?utm_source=chatgpt.com)
> and open the **api-intern-project** folder.
>
> Your task is to recreate the project from scratch, following the
> implementation in that folder, and ensure that it runs successfully.
>
> **New Learning Challenges (To Gain Confidence)**
>
> **Challenge 1:** Asynchronous Serial Number Validation (Duplicate
> Prevention)

-   **Goal:** Teach interns asynchronous validation and API querying.
    > Prevent users from registering multiple devices with the exact
    > same serial number.

-   **Requirements:**

> When registering a device through the Add Form in index.html, check
> the database for duplicate serial numbers before completing the
> creation.

1.  Implement a helper in src/api.js:\
    > export async function checkSerialDuplicate(serialNumber) {const
    > response = await
    > fetch(\`http://localhost:3000/devices?serialNumber=\${encodeURIComponent(serialNumber)}\`);

> const matchedDevices = await response.json();
>
> return matchedDevices.length \> 0; }

1.  Inside the form submit handler of src/app.js, check the result of
    > await checkSerialDuplicate(serialNumber).

```{=html}
<!-- -->
```
1.  If a duplicate is found, prevent form submission, add an invalid
    > class to the Serial input field, and show an error toast stating
    > *\"Registration Failed: Serial Number is already registered!\"*.
