Okay, let's break down how BMO.com is currently tracking location based on the script provided and then explore ways to get that specific location data into your Qualtrics surveys.

**1. How BMO.com is Currently Tracking Location (Based on the Script):**

The JavaScript you provided reveals the current method:

* **Primary Source:** The script first attempts to fetch geolocation data from the OneTrust Geolocation API at `https://geolocation.onetrust.com/cookieconsentpub/v1/geo/location`.
* **Secondary Source:** If the primary OneTrust API call fails, it falls back to a secondary, internal BMO geolocation API at `https://geo.bmo.com/location.json`.
* **Data Retrieved:** Both APIs are expected to return location information, specifically aiming to capture the `country` and `state` (from OneTrust) or `stateOrProvince` (from the BMO internal API).
* **Storage:** The retrieved location data, formatted as "Country,State/Province" (e.g., "US,NY" or "CA,ON"), is stored in two places:
    * A global JavaScript variable: `window.OT_GeoLoc_BMO`
    * Browser Session Storage: `sessionStorage.setItem("OT_GeoLoc_BMO_SS", ...)` using the key `"OT_GeoLoc_BMO_SS"`. This ensures the data persists for the duration of the user's session on the website and doesn't need to be fetched again on subsequent page loads within the same session.
* **Timing:** This location data fetching happens relatively early in the page load process, *before* the main Adobe Launch (or DTM) tag management script is loaded. This makes the location data available for use by other scripts, likely including analytics and potentially personalization tools managed via Adobe Launch.

In summary, BMO.com uses external and internal APIs to get the user's country and state/province, stores it in a JavaScript variable and session storage, and makes it available for other scripts on the page.

**2. Ways to Add This Specific BMO Location Data to Qualtrics:**

Since the location data is already determined and available client-side (in `window.OT_GeoLoc_BMO` and `sessionStorage`), you can access this information using JavaScript on the BMO.com page and then pass it to your Qualtrics survey. The best method depends on how your Qualtrics survey is being presented to the user (e.g., a link they click, or an embedded survey/feedback tab on the page).

Here are a few common approaches:

* **Method A: Passing Data via URL Query Parameters (If Linking to a Qualtrics Survey Page)**
    * **Concept:** When a user clicks a link or is redirected from BMO.com to a standalone Qualtrics survey page, you dynamically add the location data to the survey URL as query parameters.
    * **How it works:**
        1.  On the BMO.com page, use JavaScript to retrieve the location data from `window.OT_GeoLoc_BMO` or `sessionStorage.getItem("OT_GeoLoc_BMO_SS")`.
        2.  Identify the URL of your Qualtrics survey.
        3.  Append query parameters to the URL, e.g., `?location=${encodedLocationData}` where `${encodedLocationData}` is the URL-encoded value of `window.OT_GeoLoc_BMO`.
        4.  Ensure the link the user clicks (or the redirect URL) uses this modified URL.
        5.  **In Qualtrics:** Set up "Embedded Data" fields in your Survey Flow (e.g., a field named `location`). Qualtrics is automatically configured to read query parameters matching Embedded Data field names.
    * **Pros:** Relatively straightforward for links/redirects, standard web technique.
    * **Cons:** Data is visible in the URL, limited in the amount of data you can pass, requires modifying the link dynamically.

* **Method B: Using JavaScript with an Embedded Qualtrics Survey/Feedback Tab**
    * **Concept:** If the Qualtrics survey is embedded directly into the BMO.com page (like a feedback widget, pop-over, or iframe), you can often use JavaScript on the parent page to interact directly with the Qualtrics survey object and pass data.
    * **How it works:**
        1.  Ensure the Qualtrics embedded code is set up to allow communication from the parent window (often involves specific Qualtrics setup).
        2.  On the BMO.com page, after the Qualtrics survey script has loaded and initialized, use JavaScript to retrieve the location data from `window.OT_GeoLoc_BMO` or `sessionStorage.getItem("OT_GeoLoc_BMO_SS")`.
        3.  Use the Qualtrics JavaScript API (if available for the embedded method you're using) to set "Embedded Data" fields within the survey instance. This might look something like `QualtricsAPI.SurveyEngine.setEmbeddedData('location', locationData);` (The exact syntax depends on the Qualtrics implementation type - feedback tab, iframe embed, etc.).
        4.  **In Qualtrics:** Set up "Embedded Data" fields in your Survey Flow (e.g., a field named `location`).
    * **Pros:** Cleaner than URL parameters for embedded surveys, data isn't in the URL, more robust for passing data within the same page.
    * **Cons:** Requires the Qualtrics survey to be embedded, needs knowledge of the Qualtrics embedded survey API/methods, might have cross-origin restrictions if the survey is in an iframe from a different domain.

* **Method C: Having Qualtrics Read from a Data Layer (If Applicable)**
    * **Concept:** If the BMO.com script (or the Adobe Launch tags loaded afterwards) pushes the `window.OT_GeoLoc_BMO` data into a standard web data layer (like `window.dataLayer` or a custom object), Qualtrics *might* be configured to read data from this data layer.
    * **How it works:**
        1.  Confirm if the location data (`window.OT_GeoLoc_BMO`) is being pushed into a data layer object accessible on the page.
        2.  **In Qualtrics:** Use features within Qualtrics (potentially requiring Qualtrics Professional Services or specific configurations) to read values from the data layer when the survey loads or is interacted with.
        3.  **In Qualtrics:** Set up "Embedded Data" fields to store the data read from the data layer.
    * **Pros:** Leverages existing data layer structure, keeps data collection centralized.
    * **Cons:** Requires a data layer setup that includes the location data, requires specific Qualtrics configuration which might be more complex.

**Recommendation for Your Team:**

Explain the current method as described in point 1. Then, present the options in point 2 (Methods A, B, C). The most suitable method depends entirely on *how* the Qualtrics survey will be presented to BMO.com users:

* If it's a *link* or *redirect*, **Method A (URL Query Parameters)** is the most common and simplest approach.
* If it's an *embedded survey* or *feedback widget*, **Method B (JavaScript Interaction)** is more appropriate.
* If BMO has a well-defined *data layer*, **Method C (Reading Data Layer)** is a possibility worth investigating, especially if other tools already read from it.

They will need to coordinate between the BMO.com development team (to implement the JavaScript for passing the data) and the Qualtrics administrator/team (to set up the corresponding Embedded Data fields in the survey). Method B might require input from Qualtrics support regarding the specific API for their embedded survey implementation.
