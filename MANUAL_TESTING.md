# Manual Testing Instructions for provided_index.html

This document provides instructions for manually testing the functionality of the `provided_index.html` file, which embeds a Power BI report based on a URL parameter.

**Prerequisites:**
*   A copy of the `provided_index.html` file.
*   A modern web browser (e.g., Chrome, Firefox, Edge).
*   Access to the web browser's developer tools (specifically the JavaScript console and network inspector).
*   (Optional but Recommended) A valid Power BI `reportId` and `ctid` (tenant ID) to replace the placeholder ones in `provided_index.html` if you want to test actual report loading. The current placeholders are `reportId=a485b383-e21c-41c7-b0f3-ce7e7a8ee243` and `ctid=e7397b50-73eb-4db7-84c5-cbd8fece46a7`.
*   (Optional but Recommended) Valid `Dbid` values that exist in your Power BI dataset for the field `Verkooprelatie/Debiteurnummer` to test successful data filtering.

**Testing Steps:**

Before starting, open your web browser's developer tools (usually by pressing F12) and keep the Console tab open to monitor for JavaScript errors.

**Test Scenario 1: Valid `Dbid`**

1.  **URL Construction**:
    *   Open the `provided_index.html` file in your web browser.
    *   Modify the URL in the browser's address bar by appending a valid `DbId`. For example, if `123` is a valid ID, the URL should look like:
        `file:///path/to/your/provided_index.html?DbId=123`
        (Replace `file:///path/to/your/` with the actual path to the file, or if hosted on a server, use the appropriate http/https URL).
    *   Press Enter to load the page with the new URL.

2.  **Expected Results**:
    *   **Displayed Text**:
    *   Verify that the text "Verkooprelatie: 123" (or "Verkooprelatie: [YourDbIdValue]") is displayed prominently on the page (within the `<p id="dbidDisplay">` element).
    *   **Power BI Iframe**:
        *   Verify that an iframe element is visible on the page.
        *   The iframe should attempt to load the Power BI report.
            *   If you are using the placeholder `reportId` or if `123` is not a valid `DbId` in your Power BI dataset for the specified report, the iframe might show an error from Power BI (e.g., "Report not found" or "Can't display this visual"). This is acceptable for this test as long as the iframe *attempts* to load using the constructed URL.
            *   If you have configured a valid `reportId`, `ctid`, and are using a `DbId` that exists in your dataset, you should see the report filtered for that `DbId`.
    *   **Console Errors**:
        *   Check the browser's JavaScript console. There should be no JavaScript errors reported from the page's scripts.

**Test Scenario 2: Missing `Dbid`**

1.  **URL Construction (Option A - No Dbid parameter)**:
    *   Open `provided_index.html` directly in your browser without any URL parameters.
        `file:///path/to/your/provided_index.html`

2.  **Expected Results (Option A)**:
    *   **Displayed Text**:
        *   Verify that the text "Verkooprelatie ID niet gevonden in URL. Rapport kan niet geladen worden." is displayed.
    *   **Power BI Iframe**:
        *   Verify that the Power BI iframe element is **hidden** (not visible on the page). You can confirm this by inspecting the DOM; the iframe should have `style="display: none;"`.
    *   **Console Errors**:
        *   No JavaScript errors in the console.

3.  **URL Construction (Option B - Empty DbId parameter)**:
    *   Modify the URL to have an empty `DbId` parameter:
        `file:///path/to/your/provided_index.html?DbId=`
    *   Press Enter to load the page.

4.  **Expected Results (Option B)**:
    *   **Displayed Text**:
        *   Verify that the text "Verkooprelatie ID niet gevonden in URL. Rapport kan niet geladen worden." is displayed.
    *   **Power BI Iframe**:
        *   Verify that the Power BI iframe element is **hidden**.
    *   **Console Errors**:
        *   No JavaScript errors in the console.

**Test Scenario 3: `Dbid` with a Single Quote**

1.  **URL Construction**:
    *   Modify the URL to include a `DbId` with a single quote:
        `file:///path/to/your/provided_index.html?DbId=test'quote`
    *   Press Enter to load the page.

2.  **Expected Results**:
    *   **Displayed Text**:
        *   Verify that the text "Verkooprelatie: test'quote" is displayed.
    *   **Power BI Iframe**:
        *   Verify that an iframe element is visible and attempts to load. (Similar to Test Scenario 1, actual report content depends on Power BI setup).
    *   **URL Escaping Verification**:
        *   **Method 1: Inspecting `URLCompleet` (Easier)**
            *   Go to the "Console" tab in your browser's developer tools.
            *   Type `URLCompleet` and press Enter.
            *   The console should display the constructed Power BI URL. Verify that the `DbId` part in the filter query looks like `...Debiteurnummer%20eq%20%27test%27%27quote%27`. Note the `test%27%27quote` part, where `%27` is the URL encoding for a single quote, and `''` (two single quotes) is the escaped form within the OData string. The important part is seeing `test''quote` (after URL decoding the `%27`s that surround the value) as the value being compared.
        *   **Method 2: Network Inspector (More Thorough)**
            *   Go to the "Network" tab in your browser's developer tools.
            *   Reload the page (`file:///path/to/your/provided_index.html?DbId=test'quote`).
            *   Look for a request initiated by the iframe. The name might contain `reportEmbed` or your `reportId`.
            *   Select this request and examine the full URL.
            *   Verify the `filter` parameter in the URL. It should show the `DbId` with the single quote escaped as two single quotes, and then URL encoded. For example, you should see something like: `...&filter=Verkooprelatie%2FDebiteurnummer%20eq%20%27test%27%27quote%27&...`
                *   Decoded, `Verkooprelatie%2FDebiteurnummer` is `Verkooprelatie/Debiteurnummer`.
                *   Decoded, `%20eq%20%27` is ` eq '`.
                *   Decoded, `test%27%27quote` is `test''quote`.
                *   Decoded, the final `%27` is `'`.
                *   So the filter is `Verkooprelatie/Debiteurnummer eq 'test''quote'`.
    *   **Console Errors**:
        *   No JavaScript errors in the console, particularly no errors related to unterminated string literals or syntax errors caused by the unescaped quote.

**Test Scenario 4: Code Review**

1.  **Open `provided_index.html`**:
    *   Open the `provided_index.html` file in a text editor or using your browser's "View Page Source" option.

2.  **Check for `document.write`**:
    *   Scan or search the JavaScript code within the `<script>` tags.
    *   **Expected**: Confirm that there are no instances of `document.write()` being used. All dynamic text updates should be handled by setting `textContent` of the `<p id="dbidDisplay">` element.

3.  **Confirm Comments**:
    *   Review the JavaScript sections within both `<script>` tags.
    *   **Expected**: Confirm that descriptive comments are present, explaining the purpose of different code blocks, variable declarations, and complex logic (like parameter extraction, sanitization, URL construction, and iframe loading/hiding). The comments should be in English.

---

If all the above tests pass with the expected results, the `provided_index.html` file is functioning as intended according to the recent modifications.
