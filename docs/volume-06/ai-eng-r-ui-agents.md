# AI-ENG-R — UI Agents - Browser Control, Desktop Automation & Visual State

The structural integrity of interface-controlling artificial intelligence systems depends on a fundamental law of operating boundaries: user interfaces are partially observable, drift-prone state machines. Traditional automation systems treat the target interface as a static, deterministic layout with permanent locators and predictable transition latencies. This simplified assumptions layer collapses under the execution patterns of production-grade deployments, where client-side state changes, asynchronous networking, dynamic Document Object Model (DOM) mutations, and layout shifts introduce continuous operational divergence.1  
To achieve reliable execution margins, UI agents must be designed as state-verifying interface operators. This architectural model dictates that every interaction—whether page navigation, semantic field inputs, visual targeting, or bulk file transfers—must be verified against the resulting physical, accessibility, and structural state transitions before the control thread executes subsequent operations.4 This report establishes the structural state models, perception pipelines, control-channel architectures, and containment protocols required to deploy resilient and safe UI agents.

## **Doctrinal Foundations: The Interface as a Stateful Object**

A high-dimensional UI agent cannot be built merely by mapping a model's textual outputs directly to keyboard and mouse coordinates.1 When UI control is treated as ungrounded coordinate-emitting actions, systems fail via misplaced events, duplicate form submissions, security-boundary leaks, and infinite failure-recovery loops.1  
The central task of UI agents is to act as state-verifying interface operators. These systems combine DOM, accessibility, screenshot, visual, browser, desktop, and action-history evidence to plan bounded software-interface actions, execute them through sandboxed control channels, verify each resulting UI state transition, and recover safely from interface drift.1

### **The Core Doctrine of Interface Control**

UI agents must treat interfaces as partially observable, drift-prone state machines. Every click, type, navigation, drag, upload, download, and submit action must be grounded in inspected UI state, executed through a controlled channel, and verified against resulting visual, DOM, accessibility, or application state before the agent continues.4  
This principle organizes the inquiry away from simple execution metrics to a continuous verification loop. The useful question is not "Can the agent click the screen?" but "Does the agent know what interface state it is in, what target it intends to manipulate, why that target is correct, what action is permitted, what state should change afterward, whether that change occurred, and what it must do if the interface drifts?".1 Interface agents fail when they confuse dispatched input events with verified task progress.5 A UI action is not complete when the click fires; it is complete when the interface state proves the intended transition occurred.5

### **Taxonomy of Interface Controllers**

To avoid overlapping execution profiles, systems must separate programmatic scripting from model-driven interface agency.1 This comparison details the execution assumptions, failure tolerances, and target interfaces of these controllers:

| System Category | Executive Definition | Execution Assumptions | Failure Tolerances | Primary Interaction Targets |
| :---- | :---- | :---- | :---- | :---- |
| **Browser Automation Script** | A programmatic instruction sequence designed to run a predefined path over a known web page.12 | Assumes static elements, fixed timeouts, and unyielding DOM targets.10 | Zero-tolerance; fails immediately on layout shifts or selector changes.10 | Pre-defined CSS selectors and XPaths.14 |
| **Test Runner** | A scripted execution environment that runs test cases alongside repeatable fixtures and assertions.5 | Assumes isolated local servers, repeatable mock data, and test environments.5 | Fails loudly; prioritizes pinpointing discrepancies over recovery.5 | Explicit test IDs and DOM elements.14 |
| **RPA Workflow** | A robotic process automation macro that automates semi-structured business workflows across legacy interfaces.1 | Assumes stable application windows, consistent screen coordinates, and fixed layouts.1 | Brittle; requires manual re-engineering on minor application updates.1 | Coordinate-based coordinates, Win32 controls, and legacy properties.1 |
| **Browser Agent** | A model-driven agent that uses dynamic planning to operate web interfaces under variable layouts and flows.4 | Operates under dynamic layouts, changing user states, and unannounced redirects.4 | High; handles dynamic layout changes and performs automated recovery.4 | DOM structures, accessibility trees, and visual viewports.4 |
| **Desktop Agent** | A model-driven controller that plans and executes tasks across operating system windows and applications.1 | Operates across local files, multiple native windows, and global system settings.1 | High; manages focus transitions and handles system alerts.1 | Native OS accessibility APIs, system cursors, and display graphics.1 |
| **Computer-Use Agent** | A platform-agnostic agent that interacts with host operating systems via visual screenshots and synthetic inputs.1 | Assumes no direct DOM access, relying on pixel coordinates and screenshots.1 | Variable; handles visual elements but is susceptible to coordinate offsets.1 | Visual viewports and global screen coordinates.1 |

## **The Unified UI State Model**

To operate reliably, the UI agent must maintain a comprehensive, multi-channel representation of the interface. A screenshot is not just an image; it is a rendered snapshot of application state.1 A DOM is not the complete UI; it is a structural representation that may not match rendered truth.10 An accessibility tree may contain semantic controls unavailable from pixels alone.1  
The state representation must capture fifteen distinct dimensions to provide a complete picture of the UI:

| State Dimension | Physical Data Type | Extraction Mechanism | Operational Purpose | Structural Verification Signal |
| :---- | :---- | :---- | :---- | :---- |
| **1. Navigation State** | String (URI / App ID) 18 | browsingContext.get via WebDriver BiDi.18 | Confirms target domain alignment, tracking redirects and browser origins.7 | Matches the active URL path against expected domain patterns.5 |
| **2. Structural Graph** | Hierarchical markup tree 17 | CDP DOM extraction / raw XML traversal.19 | Maps parent-child element nodes and extracts HTML attributes.17 | Confirms that the target element is attached to the active DOM.5 |
| **3. Semantic Substrate** | Accessible role & name graph 8 | Native OS APIs (UIA, AX, AT-SPI2).8 | Resolves element roles, names, states, and descriptions.8 | Validates that roles match active application properties.8 |
| **4. Visual Capture** | 300 DPI high-res PNG byte array 17 | Page.captureScreenshot / OS screen capture.1 | Analyzes raw pixels to check for visual changes and layout overlays.1 | Confirms rendering contrast and verifies element layouts.17 |
| **5. Spatial Coordinates** | Normalized array: [x1, y1, x2, y2]17 | getBoundingClientRect / AX position query.11 | Coordinates element bounds for visual click fallbacks.1 | Bounding box has a non-zero area and maps inside the viewport.5 |
| **6. Focus Element** | Element reference ID 11 | activeElement traversal via CDP Runtime.13 | Pinpoints the active input cursor location, preventing blind typing.5 | Focus matches the target element's active DOM reference.5 |
| **7. Input Values Map** | Map: Selector -> String 5 | Traverses input element attributes in the DOM.5 | Tracks field text to confirm data entry and clear operations.5 | Input strings match the expected field values.5 |
| **8. Validation Alerts** | Map: Selector -> String 5 | Evaluates aria-invalid attributes and text.5 | Identifies inline input errors and block constraints.5 | Error containers are absent or hidden after correction.5 |
| **9. Modal & Overlay State** | Map: Selector -> BoundingBox | Checks z-index layers and visible overlays.5 | Detects blocking dialogs, consent pop-ups, and screens.5 | Ensures the target element is not obscured by other layers.5 |
| **10. Viewport Offset** | Tuple: (Scroll_x, Scroll_y) 11 | Queries window scroll coordinates.11 | Checks scroll position to confirm element visibility.5 | Target element is positioned within the active viewport bounds.5 |
| **11. Network Traffic** | Integer (active request count) | Monitors requests via WebDriver BiDi.18 | Identifies loading states and pending transactions.5 | Active network requests drop to zero (network idle).9 |
| **12. Runtime Console Logs** | List: ConsoleMessage 22 | LogInspector / console listener connections.22 | Catches Javascript exceptions and resource failures.22 | Zero uncaught runtime errors during action execution.22 |
| **13. Session Profile** | Map (cookies and local storage) 18 | storage.getCookies via BiDi.18 | Tracks login states, sessions, and tenant scopes.18 | Active session cookies match target auth configurations.7 |
| **14. Action History** | List: ActionRecord | Internal tracing logs. | Saves the sequence of completed steps to prevent loops. | Verifies execution progress matches the planned path. |
| **15. Risk Profile** | Enum (Low, Medium, High, Critical) | Internal configuration map. | Maps action risks to enforce safety constraints.7 | High-risk actions are gated and require explicit approvals.7 |

## **Dual-Channel UI Perception Model and Fusion Rules**

A robust UI agent cannot rely on a single channel for perception. DOM-only agents fail when elements are visually occluded, offscreen, hidden behind overlays, or rendered on custom HTML Canvases.5 Conversely, screenshot-only agents fail when semantic labels are missing, text characters are small or low-contrast, or keyboard accessibility relationships are hidden from the raw pixels.1 The agent must implement a Dual-Channel UI Perception Model that integrates semantic and visual data.

```
+---------------------------------------------------------------------------------------------------------+  
|                                    DUAL-CHANNEL PERCEPTION MODEL                                        |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                                                         |  
|         |                                                                                               |  
|         +-----------------------------------------+                                                     |  
|         v (Semantic Channel)                      v (Visual Channel)                                    |  
|                                                                                                         |  
|         +-- HTML DOM attributes [19]            +-- 300 DPI Viewport PNG                        |  
|         +-- ARIA role descriptions         +-- Deep bounding-box OCR                       |  
|         +-- Active focus state             +-- Icon recognition models                           |  
|         |                                         |                                                     |  
|         +--------------------+--------------------+                                                     |  
|                              v (Perception Fusion Engine)                                               |  
|                                                                                                         |  
|                              |                                                                          |  
|                              +--------------- (No Occlusions) ----> Safe Target Verification            |  
|                              +--------------- (Z-Index Overlays) --> Invoke Modal Playbooks             |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To resolve structural conflicts, the fusion engine enforces five rules during state construction:

| Rule Category | Conflict Scenario | Detection Diagnostic | Unified Resolving Algorithm | Expected Outcome |
| :---- | :---- | :---- | :---- | :---- |
| **1. Visibility Guard** | DOM element exists but is hidden from the viewport. | DOM has display: none or element size is 0. | Exclude element from targeting; mark as non-actionable. | Prevents misclicks on hidden layout elements. |
| **2. Occlusion Verification** | Target coordinates are covered by another DOM node. | Hit-test fails; overlay captures events. | Trace coordinate z-indexes; map overlay boundaries. | Identifies blocking overlays and routes to dismiss routines. |
| **3. Status Cross-Check** | DOM shows element as enabled, but pixels are grayed out. | Pixel analysis finds low-contrast grayout styling. | Query accessibility tree status (aria-disabled). | Confirms true element status before action dispatch. |
| **4. Canvas Resolution** | Target element is rendered on an HTML Canvas layer. | DOM shows canvas node with zero children. | Runs OCR and applies Snappy relevance propagation over the region. | Resolves coordinate offsets for custom canvas controls. |
| **5. Accessibility Audit** | DOM elements lack text labels or identifiers. | Null values for id and descriptive attributes. | Scans the accessibility tree for roles and labels. | Resolves labels for unnamed controls. |

## **Semantic Targeting and Accessibility Tree Models**

UI agents should prefer stable semantic targets over volatile visual coordinates to prevent layout shifts or resolution differences from breaking automated workflows.1 By utilizing accessibility names, roles, and structural attributes, agents construct robust targets that survive minor interface changes.8  
Windows, macOS, and Linux manage accessibility trees using distinct, platform-specific APIs.1 To support cross-platform targeting, systems must abstract these variations into a unified semantic target schema:

```JSON  
{  
  "$schema": "https://ai-engineering.canon/schemas/semantic-target-v1.json",  
  "target_id": "tgt_2026_06_10_00412",  
  "role_descriptor": {  
    "standardized_role": "button",  
    "win32_uia_control": "ButtonControlPattern",  
    "macos_ax_role": "AXButton",  
    "linux_at_spi_role": "ROLE_PUSH_BUTTON"  
  },  
  "name_descriptor": {  
    "accessible_name": "Place order",  
    "placeholder_text": null,  
    "aria_label_value": "Submit and finalize your transaction"  
  },  
  "locators": {  
    "css_selector": "button.btn-submit-order",  
    "xpath_selector": "//button[@type='submit' and contains(., 'order')]",  
    "test_id_selector": "data-testid=order-checkout-button"  
  },  
  "spatial_metadata": {  
    "bounding_box": ,  
    "scaling_factor": 1.0,  
    "viewport_attached": true  
  },  
  "actionability": {  
    "is_visible": true,  
    "is_stable": true,  
    "is_enabled": true,  
    "receives_pointer_events": true  
  },  
  "targeting_heuristics": {  
    "ambiguity_score": 0.0,  
    "confidence_rating": 0.985,  
    "risk_classification": "critical_mutation"  
  }  
}
```

### **Locator Robustness and Selection Strategy**

To minimize failures during system updates, the targeting engine evaluates element locators using a strict reliability hierarchy :

1. **Explicit Test IDs (getByTestId)**: The most robust targeting method.14 It relies on dedicated data attributes (data-testid, data-qa) that remain unchanged during layout modifications.14  
2. **Accessible Roles & Names (getByRole)**: Highly resilient.14 It locates elements by how they are perceived by assistive technologies, ensuring the agent interacts with controls identically to human operators.14  
3. **Associated Form Labels (getByLabel)**: The preferred method for form inputs.14 It binds input controls to their visible labels, preventing input misalignment.14  
4. **Text Content & Placeholders (getByText, getByPlaceholder)**: Resilient for content pages.14 However, these are susceptible to localization changes and translation drift.14  
5. **Raw CSS Selectors & XPath (locator)**: Brittle and prone to breakage.15 Changes to utility classes or component hierarchies can disrupt lookups.15 This method is reserved as a fallback of last resort.15

## **Browser Control Architecture**

The browser control layer acts as the physical link between the agent's planning models and the active browser process. Modern high-dimensional systems select from multiple specialized control channels based on the required speed, latency, cross-browser compatibility, and level of introspection needed:

| Feature Dimension | WebDriver Classic | WebDriver BiDi | Chrome DevTools Protocol | Native Accessibility Bridge |
| :---- | :---- | :---- | :---- | :---- |
| **Transport Layer** | HTTP REST requests (stateless).12 | Persistent bidirectional WebSocket connection.18 | Persistent bidirectional WebSocket connection.19 | Native OS IPC and COM/CoreFoundation calls.8 |
| **Command Response Latency** | High; introduces network overhead per HTTP call.26 | Low; bidirectional websocket messaging.26 | Minimal; direct socket communication with browser process.26 | Single-digit millisecond-level native API calls.4 |
| **Event Streaming** | Lacks streaming; requires continuous HTTP polling.28 | Native streaming of lifecycle, console, and network events.18 | Native streaming of DOM, network, and security events.13 | Native OS event notifications (observers).8 |
| **Execution Performance** | Slower; execution requires driver translation.12 | Extremely fast; direct browser protocol execution.29 | Fast; removes protocol translations.26 | Extremely fast; executes directly on background windows.4 |
| **Introspection Depth** | High-level; limited to user actions and DOM states.12 | Mid-level; captures console, script contexts, and requests.18 | Complete low-level access (memory, profiler, network).13 | Visual controls, roles, positions, and element hierarchies.1 |
| **Cross-Browser Support** | Universal standard supported by all vendors.12 | Emerging standard; supported by major browsers.12 | Proprietary; limited to Chromium-based browsers.29 | Platform-specific; separate implementations required per OS.2 |
| **Context Isolation** | Managed via driver browser parameters. | Dedicated user contexts and sandboxed evaluations.18 | Supports multiple targets and isolated page sessions.13 | Window-level isolation; tracks Frontmost focus.16 |

## **Desktop Automation Boundary Model**

Operating outside the browser sandbox introduces critical safety concerns. A desktop automation agent operates in an environment with access to local applications, filesystem structures, terminal prompts, and active system windows.1 The system must implement a multi-layered boundary model to control system inputs and isolate local processes:

```
+-----------------------------------------------------------------------------------------+  
|                               DESKTOP BOUNDARY ISOLATION                                |  
+-----------------------------------------------------------------------------------------+  
|                                                                                         |  
|                                                                                         |  
|         |                                                                               |  
|         v (Enforces process boundaries)                                                 |  
|   [ Hypervisor Layer: Firecracker MicroVM / KVM ] ------------------> [ Hardware MAC ]  |  
|         |                                                                               |  
|         v (Restricts graphical output)                                                  |  
|   ----------------> [ No Host Leak ]                                                    |  
|         |                                                                               |  
|         v (Traverses platform structures)                                               |  
|                                                                                         |  
|         +-- Windows: COM / IUIAutomation7 [8, 31]                                       |  
|         +-- macOS: CoreFoundation / AXUIElement                                  |  
|         +-- Linux: DBus / AT-SPI2                                                    |  
|         |                                                                               |  
|         v (Filters network connections)                                                 |  
|   [ Egress Filtering Gateway / Proxy ] --------------------------->|  
|                                                                                         |  
+-----------------------------------------------------------------------------------------+
```

To coordinate interaction across different operating systems, desktop bridges use native APIs that present distinct structural features:

| Platform API | Win32 UI Automation (UIA) | macOS AXUIElement API | Linux AT-SPI2 Bridge |
| :---- | :---- | :---- | :---- |
| **Interface Technology** | COM interface via uiautomationclient.h.8 | CoreFoundation C wrapper and Objective-C bindings.8 | DBus-based messaging layer over system bus.8 |
| **Role Categorization** | Strongly typed roles mapping to fixed enums.8 | Untyped role and subrole string classifications.8 | Standardized enums with custom registration support.8 |
| **Query Mechanism** | Supports bulk querying and prefetching (FindFirstBuildCache).32 | Single element-by-element queries with multiple copy calls.8 | Element-by-element DBus messaging.8 |
| **Performance Profile** | High; batch queries minimize cross-process calls.32 | Moderate; batching helps optimize multiple copy operations.8 | Slow; D-Bus round-trips introduce millisecond-level latency per node read.8 |
| **Query Optimization** | Uses TreeScope properties and cache filters.31 | Implements multi-attribute copy operations to batch requests.8 | Employs lazy evaluation to query attributes on matching nodes.8 |
| **Element Verification** | Validates roles against the active UIA cache properties.33 | Inspects elements for structural actions like AXPress.16 | Checks elements for conventional action strings.8 |
| **Stable Reference** | Tracked via cached snapshots.8 | String-based references; susceptible to stale nodes.2 | High-frequency node changes require locator re-querying.8 |
| **Execution Tool** | Microsoft UFO / xa11y Windows backend.1 | Browser Control Layer (BCL) / xa11y macOS.8 | Daytona / xa11y Linux backend.4 |

## **UI Action Taxonomy and Bounded Planning**

To simplify action planning, UI interactions are categorized into an exhaustive taxonomy of five operations. Actions must be explicitly bound to target elements and state preconditions:

| Action Category | Action Identifier | Input Parameters | Expected Interface Change | Precondition Criteria |
| :---- | :---- | :---- | :---- | :---- |
| **Observation** | ObserveState 1 | Viewport target.1 | Refreshes the active UI state model.4 | Target page context is active.18 |
|  | InspectElement 8 | Target element locator.14 | Captures structural attributes and coordinates.8 | Element node is attached to the DOM.5 |
| **Navigation** | NavigateTo 18 | Destination URL.18 | Target URL loads; page enters network idle.5 | Destination matches domain policies.7 |
|  | NavigateBack 18 | None. | Browser context rolls back to prior history state. | Browser history list contains past pages. |
|  | RefreshPage 18 | None. | Document reloads; active DOM cache is cleared.2 | Target page context is active.18 |
| **Interaction** | Click 11 | Target locator, position offset.11 | Dispatches pointer event; triggers layout change.5 | Element is visible, stable, and enabled.5 |
|  | DoubleClick 11 | Target locator, position offset.11 | Dispatches double-click sequence to target.11 | Element is visible, stable, and enabled.5 |
|  | Hover 35 | Target locator, hover delay.35 | Dispatches pointer moves; triggers tooltip rendering.5 | Target element is inside the viewport.5 |
|  | Type 35 | Target locator, character string.35 | Appends string values to target input.5 | Field is focused and editable.5 |
|  | ClearValue 11 | Target locator.11 | Selects and deletes active input strings.11 | Field is focused and editable.5 |
|  | SelectValue 21 | Target locator, option ID.21 | Selected property updates inside select node.5 | Select element is enabled and not read-only.5 |
|  | CheckControl 11 | Target checkbox locator.11 | Check state updates; aria-checked set to true.5 | Element is an unchecked checkbox.5 |
|  | DragAndDrop | Source locator, destination coordinates. | Source element is moved to target area. | Source and target elements are attached. |
| **System** | Upload 18 | Target input, file path.18 | File properties are updated inside upload node.5 | Target file exists in the sandbox.34 |
|  | Download 18 | Target action locator.18 | File is saved to the designated download path.18 | Host path is write-enabled and has space.34 |
|  | CopyText | Target source locator. | String is copied to the isolated sandbox clipboard. | Source element contains selectable text.5 |
|  | PasteText | Target destination locator. | Clipboard contents are pasted into input.5 | Destination input field is focused.5 |
|  | KeyboardShortcut 1 | Keyboard character combination.1 | System event is triggered inside active context.16 | Context window is active and focused.16 |

### **The Bounded Action Planning Model**

To prevent unverified execution paths, the planning engine evaluates interactions using a structured Bounded Action Planning Model. This schema maps preconditions and verification checks to target actions:

```
+-----------------------------------------------------------------------------------------+  
|                                BOUNDED ACTION PLAN RECORD                               |  
+-----------------------------------------------------------------------------------------+  
|                                                                                         |  
|   Target Domain: billing.internal.enterprise                                            |  
|   Active Action Type: Click                                                             |  
|   Target Element: button[id="btn_confirm_payment"]                                      |  
|                                                                                         |  
|   Execution Parameters:                                                                 |  
|   {                                                                                     |  
|     "preconditions": [                                                                  |  
|       "is_visible == true",                                                             |  
|       "is_stable == true",                                                              |  
|       "is_enabled == true",                                                             |  
|       "receives_pointer_events == true"                                                 |  
|     ],                                                                                  |  
|     "execution_payload": {                                                              |  
|       "coordinates": ,                                              |  
|       "dispatched_method": "CDP:Input.dispatchMouseEvent"                               |  
|     },                                                                                  |  
|     "post_action_expectations": [                                                       |  
|       "url_redirected == true",                                                         |  
|       "success_toast_visible == true",                                                  |  
|       "network_activity == idle"                                                        |  
|     ],                                                                                  |  
|     "timeout": 10000,                                                                   |  
|     "error_recovery_playbook": "PLY_RECV_CLICK_NO_EFFECT"                               |  
|   }                                                                                     |  
|                                                                                         |  
+-----------------------------------------------------------------------------------------+
```
## **Click and Type Verification Framework**

The Click and Type Verification Framework acts as the system's execution-level gate. It ensures that inputs are verified directly against resulting UI state transitions before subsequent actions are permitted.

```
+---------------------------------------------------------------------------------------------------------+  
|                                    CLICK & TYPE VERIFICATION TIMELINE                                   |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|   0ms                     20ms                    100ms                   250ms                         |  
|   +-----------------------+-----------------------+-----------------------+-------------------------+   |  
|   | Pre-Action Check      | Event Dispatch        | Settle Delay          | Post-Action Verification|   |  
|   |                       |                       |                       |                         |   |  
|   | - Confirms visibility | - Programmatic click  | - Monitors page load  | - Audits DOM mutations  |   |  
|   | - Evaluates stability |   or key sequences    | - Tracks transitions  | - Checks input values   |   |  
|   | - Verifies enablement |   dispatched over     | - Waits for active    | - Confirms redirects    |   |  
|   | - Checks occlusion    |   active socket       |   network idle state  | - Validates toast text  |   |  
|   +-----------------------+-----------------------+-----------------------+-------------------------+   |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To coordinate interaction across different UI elements, the verification pipeline executes across four sequential stages:

| Verification Phase | Pre-Action Checks | Action Dispatch Method | Settle & Sleep Checks | Post-Action Verification |
| :---- | :---- | :---- | :---- | :---- |
| **Click Verification** | Confirms element visibility, stability, and enablement. Verifies target is not obscured by overlay elements. | Dispatches pointer clicks via CDP input events or invokes platform AXPress. | Monitors CSS transitions and waits for animation frames to stabilize. | Verifies navigation changes, target element detachment, or success toast rendering. |
| **Type Verification** | Confirms field visibility, focus, and editable properties. | Sends character sequences via CDP insert text commands. | Waits for input events to trigger and monitors active client-side validators. | Audits resulting input values against expected strings. |
| **Select Verification** | Validates target select element and active dropdown choices. | Dispatches option selection events or updates selected DOM attributes. | Monitors select container for list collapses or overlay dismissals. | Verifies active option properties match targeted indices. |
| **Check Verification** | Validates checkbox elements and active check parameters. | Dispatches pointer events to checkboxes or updates checking properties. | Waits for toggle transitions to complete. | Evaluates resulting checkbox attributes (aria-checked="true"). |
| **Drag Verification** | Identifies source and destination coordinates. | Sends programmatic mouse-down, drag-path, and mouse-up events. | Waits for layout containers to adapt to element changes. | Verifies target element coordinate intersections inside destination boundaries. |
| **Upload Verification** | Confirms target input element has valid file upload types. | Dispatches file paths directly to target inputs via automation APIs. | Waits for processing and monitors form status. | Verifies uploaded file properties are recognized by the DOM tree. |
| **Download Verify** | Confirms download action controls are attached and enabled. | Dispatches pointer events to download triggers. | Monitors browser context for file download events. | Verifies downloaded files are written to the sandbox directory. |
| **Navigate Verify** | Confirms destination URLs match allowed domains. | Sends navigation requests over active browser connections. | Waits for page load completions and monitors redirects. | Checks resulting URL properties and audits console states. |
| **Submit Verification** | Executes Form Submission Safety checklist. | Sends click events to form submit buttons. | Waits for transitions and verifies active database state updates. | Confirms navigation changes or checks backend API records. |

### **Mitigating the React Re-render Race Condition**

A common automation failure occurs when React unmounts and replaces a DOM element between the agent's pre-action check and event dispatch.10 While the element remains visually identical, holding an outdated DOM reference throws an Element is not attached to the DOM exception 10:  
Race Window (T_race) = T_dispatch - T_check < 50ms  
This race window is typically under 50 milliseconds. The race condition is mitigated using two runtime design patterns:

* **Dynamic Re-Querying**: The execution engine must never store raw element references. Every action must accept a dynamic locator (e.g., page.getByRole("button", { name: "Place order" })) that re-queries and resolves the element target immediately before dispatching the event.10  
* **Web-First Retrying Assertions**: Verification loops must avoid immediate value checks (heading.textContent()) that fail on dynamic page loads.38 Instead, the system must utilize auto-retrying, polling assertions (e.g., expect(heading).toHaveText("Order completed")) that repeatedly evaluate criteria over a 5-second window to accommodate rendering delays.6

## **Form Submission Safety Model**

Form submissions represent critical action boundaries because they trigger side effects that often cannot be programmatically rolled back (e.g., database writes, financial transactions, or message dispatches).7 The Form Submission Safety Model establishes a rigorous verification protocol before executing any form actions:

```
+---------------------------------------------------------------------------------------------------------+  
|                                      FORM SUBMISSION SAFETY CONTROL                                     |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                                                         |  
|         |                                                                                               |  
|         v (Verification Gate 1: Origin Validation)                                                      |  
|   [ Confirms domain matches allowed list; blocks third-party framing ] ----------------> [ Passed ]     |  
|         |                                                                                               |  
|         v (Verification Gate 2: Structural Integrity Check)                                             |  
|   [ Audits input values, scans for errors & verifies required fields ] ----------------> [ Passed ]     |  
|         |                                                                                               |  
|         v (Verification Gate 3: Payload Risk Evaluation)                                                |  
|   [ Calculates transactional cost thresholds and evaluates destinatary endpoints ]                       |  
|         +------- (Below Risk Threshold) --> Direct Automated Submission Event                           |  
|         +------- (Above Risk Threshold) -->                                                |  
|                                                     |                                                   |  
|                                                     v (Requires manual signature clearance)             |  
|                                                 [ User-in-the-Loop PIN Verification ]                   |  
|                                                     |                                                   |  
|                                                     v                                                   |  
|                                                                                     |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To coordinate forms across processing states, the validation engine maps actions to distinct submission stages:

| Submission Stage | Required Verification Controls | Core Processing Algorithm | Expected Verification Signals | Exception Mitigation Protocols |
| :---- | :---- | :---- | :---- | :---- |
| **1. Draft** | Scans fields to confirm focus and checks input values.5 | Verifies input fields match planned schemas.5 | Initial form containers are attached and visible.5 | Resets inputs if data entry errors are flagged.11 |
| **2. Ready** | Evaluates required inputs and checks for errors.5 | Verifies all required fields are populated.5 | Form state shows zero validation errors.5 | Populates empty fields and re-evaluates validation states.5 |
| **3. Confirmation** | Evaluates cost limits and targets.7 | Invokes the Payload Review Pattern for high-risk actions.7 | Signed permission token is generated.24 | Halts and escalates to user for approval.7 |
| **4. Submitted** | Monitors click events and tracks context navigations.5 | Sends click events to submit buttons.5 | Navigation starts; submit buttons show loading state.5 | Retries click if target elements remain attached.10 |
| **5. Accepted** | Monitors network responses via WebSocket connections.18 | Extracts HTTP status codes from API endpoints.9 | Network confirms success response (HTTP 200/201).9 | Displays error state if request returns error codes.7 |
| **6. Successful** | Audits DOM mutations and success toast elements.5 | Scans target elements for success indicators.5 | Success toast is visible; URL path is updated.5 | Fallback to checking database records.7 |
| **7. Failed** | Scans for inline errors or system alerts.5 | Extracts error text and logs response payloads.5 | Inline error text or modal alert is visible.5 | Invokes specialized recovery playbooks. |
| **8. Duplicate** | Checks action history logs before submitting. | Compares active payloads against past transactions.7 | Preventative block triggered before click. | Cancels submit and flags transaction as duplicate.7 |

## **Browser Sandboxing, Permissions, and Containment**

When interacting with untrusted websites, the agent's browser runtime must be isolated to prevent credential theft, data leaks, and local process exploits.36 Modern configurations harden browser environments using process-level operating system sandboxes and remote browser isolation frameworks:

| Isolation Domain | Risk Surface | Programmatic Defense Configuration | Intended Containment Boundary |
| :---- | :---- | :---- | :---- |
| **Ephemeral Profiles** | Extends active session history, exposing cookie stores and local caches. | Launches browser using incognito context parameters (--incognito). | Wipes all browser data, cookies, and cache upon context teardown. |
| **Credential Masking** | Exposes passwords and tokens to the agent's prompt context. | Implements secure, headless credential brokers to inject inputs directly into the page. | Prevents credential leaks into context windows or logging outputs. |
| **Password Isolation** | Exposes local keychain vaults and system password managers. | Disables password autofill capabilities (--disable-save-password-bubble). | Blocks access to host credential vaults and system resources. |
| **Download Control** | Triggers drive-by malware downloads or writes to host filesystems. | Restricts download pathways to temporary sandboxed directories (/tmp/downloads). | Blocks execution of downloaded files on the host computer. |
| **Upload Restrictions** | Allows unauthorized exfiltration of sensitive host files. | Restricts upload access to a single, read-only workspace directory. | Prevents the agent from reading or uploading private host data. |
| **Clipboard Isolation** | Leaks sensitive clipboard data or system credentials. | Restricts sandbox browser clipboard APIs and access controls. | Prevents data exchange between host and sandbox clipboards. |
| **Extension Shield** | Vulnerable browser extensions intercept traffic and leak credentials. | Disables extensions on startup (--disable-extensions). | Limits execution to verified browser core components. |
| **Egress Filtering** | Initiates requests to malicious domains or exfiltrates data. | Implements container-level egress filters using domain whitelists. | Restricts outbound traffic to approved API and service endpoints. |
| **Permission Controls** | Accesses cameras, microphones, or location coordinates. | Rejects permission prompts automatically via browser preferences. | Prevents unauthorized monitoring and tracking. |
| **Pop-up Blocking** | Spawns multiple windows to bypass isolation boundaries. | Enforces single-tab restrictions and blocks window creation. | Restricts execution to the active, monitored browser tab. |
| **Cross-Origin Framing** | Frame injection attacks bypass security controls. | Enforces Site Isolation policies, running frames in separate processes. | Prevents malicious frames from accessing parent context data. |
| **Context Teardown** | Orphaned processes leak memory and retain session files. | Closes active connections and wipes temporary mount spaces. | Reclaims system resources and ensures zero persistent traces. |

## **Interface Drift Detection and Recovery**

Interface drift is a primary cause of automation failures, occurring when layout shifts, A/B tests, modal dialogs, or selector changes diverge from the agent's planned expectations.1 The system's central recovery principle is: **When the UI changes, the plan is stale. Re-observe before acting.**

```
+---------------------------------------------------------------------------------------------------------+  
|                                    INTERFACE DRIFT RECOVERY PROTOCOL                                    |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                                                         |  
|         |                                                                                               |  
|         v (Stage 1: Classification & Assessment)                                                        |  
|                                                            |  
|         |                                                                                               |  
|         v (Stage 2: Execution Pause)                                                                    |  
|                                                                  |  
|         |                                                                                               |  
|         v (Stage 3: Element Re-Evaluation)                                                              |  
|                                                           |  
|         +------- (Target Resolved) --> Resume Action Sequence                                           |  
|         +------- (Target Ambiguous) --->                                          |  
|                                                     |                                                   |  
|                                                     v                                                   |  
|                                                 [ Visual & Neighborhood Check ]                         |  
|                                                     +-- (Success) -> Update Locator & Resume            |  
|                                                     +-- (Fail) ----> Escalate to Human / Halt           |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To coordinate drift recovery, the system executes a structured Interface Drift Recovery Model:

| Drift Variant | Visual/DOM Root Cause | Detection Diagnostic Signal | First-Line Automated Recovery | Fallback Resolution Channel | Stop Condition Threshold |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **1. Stale Selector** | Dynamic class names change or element hierarchies shift after system updates.2 | Locator fails to resolve inside the active DOM tree.5 | Re-queries using accessible names and user-facing roles.10 | Fallback to visual coordinate matching via OCR.1 | Timeout exceeded or 3 failed query attempts.4 |
| **2. Layout Shift** | Responsive styling or dynamic advertisements reposition elements.1 | Bounding box coordinates change during actionability checks.5 | Pauses action loop and waits for element position to stabilize.1 | Re-calculates element coordinates relative to viewport bounds.1 | Target remains unstable after 2 seconds.5 |
| **3. A/B Variant** | Page routing or workflow structure changes unexpectedly.3 | URL matches base domain but layout structures diverge.5 | Re-evaluates page elements using visual layout models.17 | Re-plans the task execution path starting from the current page state.4 | Re-planning loops fail to resolve a valid path. |
| **4. Localization Shift** | Target labels change language based on location or browser settings. | Locators match role structures but text strings are missing.14 | Identifies elements using stable test IDs or ARIA roles.14 | Resolves elements using structural placement and visual positions.17 | Element matching confidence drops below 0.75. |
| **5. Responsive Hide** | Navigation menus collapse on narrow or mobile viewports. | Selector is present in DOM but hidden from visual viewport.5 | Maximizes browser window size or scrolls target into view.5 | Triggers menu expansion buttons to expose hidden links.5 | Target remains invisible after maximize and scroll.5 |
| **6. Dynamic Rearrange** | Search results or table rows reorder during task execution. | Target data row positions shift dynamically.4 | Locates target row using unique cell key value strings.17 | Filters the data table to isolate the target row element.14 | Target row is missing from the search space. |
| **7. Cookie Banner** | Consent dialogs load asynchronously, blocking interactions.1 | Hit-test indicates pointer events will be intercepted by overlay nodes.5 | Scans DOM for close buttons or consent actions.5 | Programmatically dismisses the overlay using direct AXPress commands.16 | Overlay remains visible after 2 dismissal attempts. |
| **8. Modal Block** | Dialog windows pop up, intercepting viewport controls.1 | DOM z-index analysis confirms modal layer is active.5 | Scans for modal cancel buttons, close icons, or escape keys.5 | Programmatically executes close commands on the modal wrapper. | Modal blocks interactions after 2 dismissal attempts. |
| **9. Progress Spinner** | Async network requests delay content loading and page updates.1 | Progress indicator elements remain attached to DOM.5 | Suspends action loop; waits for spinner elements to unmount.9 | Polls target element status until actionability checks pass.5 | Spinner remains visible after 15 seconds.5 |
| **10. Session Expired** | Navigation redirects to login or registration form. | Active session cookies expire or page redirects to login URL.18 | Scans for login selectors and inputs saved credentials.7 | Suspends execution and escalates to user for manual login.7 | Redirect loops repeatedly to the login form. |
| **11. Input Mask Shift** | JavaScript formatting masks dynamically update typed values.5 | Value verification finds mismatch after typing actions.5 | Clears field, resets cursor position, and types slowly.1 | Submits raw unmasked strings directly to input properties.11 | Values diverge after 2 typing attempts. |
| **12. Multi-Label Match** | Multiple identical buttons appear in layout containers.6 | Locator query returns more than 1 matching node.14 | Filters targets using parent container scopes.6 | Selects the target element based on spatial proximity.1 | Ambiguity remains after applying filters.14 |

## **UI Recovery Playbook Library**

The UI Recovery Playbook Library provides developers with predefined, deterministic routines to handle common automation exceptions and maintain workflow stability 4:

| Playbook Identifier | Exception Trigger Case | Action Sequence Strategy | Target Recover State | Escalation / Stop Conditions |
| :---- | :---- | :---- | :---- | :---- |
| **PLY_RECV_MISSING** | Target element is missing or selector fails to resolve.5 | 1. Pauses action loop for 1.5s.1 2. Clears element caches.2 3. Re-queries using accessible names.10 | Element successfully resolved and verified as active. | Aborts and escalates if element is missing after 3 attempts.4 |
| **PLY_RECV_AMBIGUOUS** | Locator resolves to multiple matching DOM elements.14 | 1. Filters results using visibility checks.14 2. Evaluates adjacent labels.6 3. Selects based on spatial coordinates.1 | Target is uniquely identified and isolated. | Immediately halts if targeting ambiguity remains.14 |
| **PLY_RECV_NO_EFFECT** | Click action succeeds but triggers no state changes.5 | 1. Retries click using visual coordinates.1 2. Attempts focus and sends Enter key inputs.5 3. Clears browser focus state.5 | Page state transition or visual mutation verified.5 | Aborts after 2 failed interaction retries. |
| **PLY_RECV_BAD_PAGE** | Navigation loads an unexpected page or error screen.5 | 1. Clears navigation context.18 2. Rewinds page history.18 3. Re-dispatches navigation requests.13 | Active context successfully loads target URL. | Halts if navigation fails twice.10 |
| **PLY_RECV_NAV_TIME** | Page navigation times out or network requests hang.5 | 1. Cancels pending requests.18 2. Refreshes active tab context.18 3. Evaluates current DOM structures.1 | Page settles on a usable, functional state. | Aborts if page fails to load within 30 seconds.10 |
| **PLY_RECV_SPINNER** | Dynamic page content is blocked by loading spinners.1 | 1. Polls loading spinner element status.9 2. Waits up to 10s for spinner unmount.9 3. Refreshes page if hang persists.18 | Progress indicators disappear; page content loads. | Escalates to user if spinner persists after page refresh.7 |
| **PLY_RECV_VALIDATE** | Input action triggers an inline validation error.5 | 1. Clears input field values.11 2. Re-types inputs slowly to trigger events.1 3. Dispatches focus blur events.5 | Validation markers are resolved and cleared. | Halts if inputs repeatedly trigger validation errors. |
| **PLY_RECV_LOGIN** | Page session expires and redirects to login.7 | 1. Identifies login form selectors. 2. Requests credentials from secure vault.7 3. Re-submits login form.7 | Target page session is restored and verified.7 | Aborts if login actions loop back to authentication screen.47 |
| **PLY_RECV_CAPTCHA** | Automation is blocked by a CAPTCHA verification prompt. | 1. Halts execution immediately.40 2. Saves the active session trace.1 3. Escalates control to user.7 | User resolves verification; session context restored. | Instantly halts and transfers control; no retries.40 |
| **PLY_RECV_PERM** | Context navigation triggers a browser permission prompt. | 1. Intercepts prompt using BiDi events.18 2. Rejects request following safety profile.18 3. Closes prompt window.18 | Context permission prompt is cleared and dismissed. | Rejects request and blocks camera/location access.46 |
| **PLY_RECV_PICKER** | Action triggers an unannounced native file picker. | 1. Scans context for active file picker events.18 2. Sets targets using file path controls.5 3. Dismisses native window.18 | File path successfully linked to upload control. | Aborts if native file dialog locks execution focus. |
| **PLY_RECV_DL_BLOCKED** | File download action is blocked by sandbox policy.34 | 1. Audits file metadata and extension types.18 2. Confirms file path matches safety scopes.34 3. Clears blocks if within bounds.18 | Download succeeds and writes to sandboxed directory. | Halts if downloaded file type violates safety limits.34 |
| **PLY_RECV_UP_FAIL** | File upload action fails or is rejected by form. | 1. Verifies local file exists and is readable.34 2. Validates format against form attributes.5 3. Re-dispatches upload path.5 | Form accepts file; upload progress finishes.5 | Aborts if form repeatedly rejects file format.5 |
| **PLY_RECV_OVERLAY** | Target click coordinate is blocked by an overlay.5 | 1. Identifies the overlay's close control.5 2. Sends clicks to dismiss pop-up. 3. Re-evaluates target actionability.5 | Overlay dismissed; target receives pointer events.5 | Aborts if overlay remains visible after 2 attempts. |
| **PLY_RECV_CRASH** | Page process crashes, displaying an error screen.46 | 1. Captures process crash dump logs.22 2. Re-creates the page context.13 3. Re-runs step sequence from history state. | Page context is restored and verified as functional. | Aborts if page crashes repeatedly during steps.46 |
| **PLY_RECV_NETWORK** | Requests fail due to local connection drops. | 1. Pauses execution and checks connection status. 2. Waits up to 10s for network to restore. 3. Retries failed navigation step.18 | Connection is restored; target page loads successfully. | Aborts if connection remains offline after 3 attempts. |
| **PLY_RECV_REDIRECT** | Navigation triggers unexpected cross-origin redirect.5 | 1. Audits target domain origin safety.7 2. Blocks inputs on unverified domains.7 3. Returns to prior history state.18 | Context is returned to a verified page domain. | Aborts if redirected domain violates safety policies.34 |
| **PLY_RECV_EXPIRED** | User session expires during workflow execution.47 | 1. Suspends active step sequence.4 2. Authenticates session via secure login broker.7 3. Restores execution context. | Active session is restored; target workflow resumes. | Halts if authentication attempts fail.47 |
| **PLY_RECV_SUB_ERR** | Form submission returns a server-side error.7 | 1. Extracts error text from target containers.5 2. Updates input fields with corrected data.11 3. Re-runs Form Submission safety checks.7 | Form accepts corrected payload; success confirmed.7 | Aborts if server returns repeatable 500 error codes. |
| **PLY_RECV_DUP_WARN** | Form returns duplicate transaction warning.7 | 1. Scans page for warning confirmation prompts.5 2. Verifies backend transaction logs.7 3. Cancels submit to prevent duplicates.7 | Submission is safely cancelled; status updated. | Halts and escalates to user for validation.7 |
| **PLY_RECV_CONFIRM** | Risky action triggers a confirmation dialog.7 | 1. Compares payload with planned variables.7 2. Scans for confirm button elements.5 3. Clicks to confirm if values match plan.5 | Dialog is dismissed; target mutation proceeds.5 | Halts and escalates if payload values diverge.7 |

## **UI Prompt Injection and Interface Instruction Collision**

UI agents read dynamic, untrusted page content, including comments, ads, and user-generated text.40 This introduces a critical vulnerability: indirect prompt injection (IDPI), where attackers embed malicious instructions in web pages to hijack the agent's prompt context.40

```
+---------------------------------------------------------------------------------------------------------+  
|                                    INDIRECT PROMPT INJECTION LIFECYCLE                                  |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                                                         |  
|         |                                                                                               |  
|         v (Stage 1: Delivery & Pre-processing)                                                          |  
|                |  
|         |                                                                                               |  
|         v (Stage 2: Parsing & Serialization)                                                            |  
|   [ Visual concealment (zero-sizing, hidden divs) stripped; raw text is serialized ]                    |  
|         |                                                                                               |  
|         v (Stage 3: Context Concatenation)                                                              |  
|   [ Untrusted page text is combined with developer system prompts ]                                     |  
|         |                                                                                               |  
|         v (Stage 4: Execution Override)                                                                 |  
|   [ LLM misinterprets data text as system-level instructions ]                                          |  
|         +------- (Exploit Prevented) --> Target processes as clean text                                 |  
|         +------- (Exploit Succeeds) ---> Compromised tool use, data exfiltration, or leaks              |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To secure the agent's prompt boundary, developers must analyze the typical delivery vectors used in injection attacks:

| Attack Delivery Vector | Programmatic Concealment Method | Parser Extraction Vulnerability | Core System Security Guard |
| :---- | :---- | :---- | :---- |
| **1. Zero-Sizing** | Hiding text elements by setting font-size: 0px and line-height: 0 inside container styles.40 | Text extractors capture hidden strings; vision encoders miss the tiny characters.40 | Pre-processing parsers must strip elements with computed height or width of zero.5 |
| **2. CSS Hidden Divs** | Hiding elements from rendering using styles like display: none or visibility: hidden.40 | Heuristic parsers extract raw text from the DOM; visual checks fail to identify the hidden elements.42 | Ensure pre-flight parsers evaluate computed styles and filter hidden elements.5 |
| **3. HTML Comments** | Placing malicious payloads inside standard HTML comments (``).40 | Comment tags are parsed into text strings; models ingest comments as instruction contexts.42 | Strip all HTML comments from the DOM tree before serializing text inputs.42 |
| **4. SVG CDATA** | Encapsulating malicious instructions inside character data (CDATA) blocks in SVG images.40 | OCR engines extract instructions; security scanners miss the embedded code.40 | Sanitize SVG inputs and block OCR text extraction over untrusted vector images.40 |
| **5. Semantic Blend** | Weaving malicious instructions directly into legitimate page text or help articles.42 | Models cannot distinguish between page content to process and instructions to follow.42 | Enforce strict role isolation, wrapping data inside untrusted text markers.7 |
| **6. Multilingual Override** | Repeating malicious instructions across multiple languages to bypass single-language filters.40 | Simple string matchers fail on alternative languages; models process instructions normally.40 | Enforce language identification filters and use multilingual model guardians. |
| **7. XML/JSON Breaks** | Injecting syntax characters (e.g., }}, </untrusted-data>) to break out of data blocks.40 | Models parse injected characters as structural breaks, exposing system prompts to hijack.40 | Escape and sanitize all syntax characters before serializing content.7 |

### **Principles of System-Data Separation**

To prevent indirect prompt injections from hijacking agent execution, pipelines must treat page content as untrusted evidence rather than executive authority 7:

* **System Dominance Training**: Models must be explicitly fine-tuned to prioritize system prompts and developer policies over instructions found in page data.48  
* **Delimited Data Contexts**: Untrusted page content must be wrapped in strict XML or markdown delimiters (e.g., <untrusted-web-data>) with system instructions stating that the enclosed text is data to be processed, not commands to be followed.7  
* **Capability Sandboxing**: Security must be enforced at the API and sandbox layers, not through text filtering.7 If an agent is summarized as a read-only scraper, it must not possess the system credentials or tools needed to write data, send emails, or execute transactions.7

## **Deceptive Interface Risk Model**

UI agents can be tricked by deceptive design patterns, phishing pages, lookalike domains, and visual spoofing designed to manipulate browser and desktop operations.47

```
+-----------------------------------------------------------------------------------------+  
|                                DECEPTIVE INTERFACE MODEL                                |  
+-----------------------------------------------------------------------------------------+  
|                                                                                         |  
|                                                                                         |  
|         |                                                                               |  
|         v (Security Gate 1: DNS & SSL Origin Check)                                     |  
|   [ Verifies domains and audits network endpoints; blocks unauthorized redirects ]      |  
|         |                                                                               |  
|         v (Security Gate 2: Semantic-Visual Validation)                                 |  
|                   |  
|         |                                                                               |  
|         v (Security Gate 3: Interactive Verification)                                    |  
|   [ Probes targets for pointer focus and checks visibility for clickjacking traps ]     |  
|         |                                                                               |  
|         v (Security Gate 4: Execution Enforcement)                                      |  
|   [ Limits action capabilities; requests human approval for critical steps ]            |  
|                                                                                         |  
+-----------------------------------------------------------------------------------------+
```

To secure agents against deceptive designs, developers must implement a multi-layered verification model:

| Threat Category | Spoofing Mechanism | Primary Detection Method | Core System Mitigation |
| :---- | :---- | :---- | :---- |
| **Phishing Pages** | Lookalike login screens designed to steal credentials.47 | Matches active context URLs against domain whitelists and SSL certificate registries.7 | Blocks credential autofill on unverified or third-party domains.24 |
| **Clickjacking** | Transparent elements overlaid on legitimate buttons to intercept clicks.36 | Evaluates element visibility and checks hit targets during pre-action verification.5 | Rejects clicks on elements obscured by overlapping layers.5 |
| **Hidden Costs** | Pre-checked optional charges or terms hidden behind small visual fonts.40 | Scans the DOM for hidden check state properties (aria-checked="true").5 | Unchecks options by default and flags unannounced changes to users.7 |
| **Spoofed Prompts** | Webpages rendering fake OS alerts or browser security warnings.50 | Checks active elements against the browser's native window handles.37 | Ignores dialogs rendered inside web context wrappers.37 |
| **Malicious Downloads** | Automated drive-by file downloads initiated upon page load.36 | Monitors browser context download events via WebSocket APIs.18 | Configures sandboxes to block download executions by default.34 |

## **UI Agent Evaluation and Observability Guidance**

Evaluating and debugging UI agents is challenging because interactions are highly dynamic and non-deterministic.2 Standard task-completion metrics are insufficient; comprehensive evaluation requires tracking detailed telemetry across perception, targeting, execution, and safety domains.1

```
+-----------------------------------------------------------------------------------------+  
|                                UI OBSERVABILITY ARCHITECTURE                            |  
+-----------------------------------------------------------------------------------------+  
|                                                                                         |  
|                                                                                         |  
|         |                                                                               |  
|         +------------------> [ Automated Metrics Evaluator ]                            |  
|         |                          +-- Task Success Rate (TSR)                    |  
|         |                          +-- Click Precision Rate                      |  
|         |                          +-- Locator Resolution Failures               |  
|         |                                                                               |  
|         +------------------>                               |  
|         |                          +-- Base URL, HTML snapshots & screenshots     |  
|         |                          +-- Bounded plan schema parameters            |  
|         |                          +-- Visual pointer coordinates                 |  
|         |                                                                               |  
|         +------------------>                                        |  
|                                    +-- Chronological slide deck rendering               |  
|                                    +-- Network intercept HAR logs                |  
|                                    +-- Integrity check verification              |  
|                                                                                         |  
+-----------------------------------------------------------------------------------------+
```

To support systematic audits and regression tests, developers must implement eighteen observability metrics:

| Metric Category | Technical Metric Identifier | Diagnostic Target | Standard Production Target |
| :---- | :---- | :---- | :---- |
| **Perception** | Character Error Rate (CER) | General phonetic text extraction accuracy over dynamic fonts.17 | < 1.5% CER on prioritized text views. |
|  | Bounding Box IoU Accuracy 17 | Precision of spatial element localization and grounding.1 | IoU >= 0.85 on target controls.17 |
|  | Context Volume Reduction | Optimization of visual tokens passed to the generator. | > 50% reduction in token consumption.17 |
|  | DOM-Screenshot Agreement | Matches DOM-extracted inputs against visual views. | > 98% agreement on visible text nodes.17 |
| **Targeting** | Grounding Hit Rate | Accuracy of finding targets using semantic locators.10 | > 95% target resolution rate.14 |
|  | Selector Timeout Frequency | Frequency of locator resolution timeouts.5 | < 2% of active execution steps.5 |
|  | Locator Resolution Delta | Latency added during selector re-querying steps.10 | < 15ms overhead per re-query.4 |
| **Execution** | Step Success Rate (SSR) | Percentage of interface actions completed without errors. | > 98% per individual workflow step. |
|  | Click Precision Rate | Percentage of clicks that land on valid hit targets.5 | < 0.1% misclick rate in active viewports. |
|  | Type Fidelity Rate | Percentage of inputs where values match target strings.5 | 100% value matching accuracy.5 |
|  | Pre-Action Check Failures | Percentage of actions blocked by pre-action checks.5 | < 5% of planned system steps.5 |
| **Verification** | State-Transition Match Rate | Matches post-action states against plans.5 | > 95% transition agreement.5 |
|  | Focus Verification Rate | Confirms field focus before sending typing actions.5 | 100% of interactive typing steps.5 |
|  | Form Submission Error Rate | Frequency of validation errors triggered by submits.5 | < 1% of active submit attempts.7 |
| **Safety** | Sandbox Isolation Violations | Attempts to access forbidden local files or settings.34 | Zero sandbox violations.34 |
|  | Egress Policy Blocks | Attempts to access unapproved domains.34 | Zero egress policy blocks.34 |
|  | Credential Access Violations | Attempts to extract secrets from password forms.7 | Zero credential access violations.7 |
| **Telemetry** | Average Task Duration | Time taken to complete a user task. | < 45s across multi-step flows. |

## **Cross-Canon Handoff Map**

The systems architecture of AI-ENG-R integrates with adjacent disciplines across the AI Engineering Systems Canon. This mapping defines the structural dependencies and operational handoff boundaries:

| Target Canon Report | Domain Area | Technical Handoff Parameters | Engineering Integration Rules |
| :---- | :---- | :---- | :---- |
| **AI-ENG-B** | Context-Tenure & State | Session token timeouts and cache profiles.18 | Enforces memory-clearing rules, wipes browser cookies, and clears local caches upon task completion.34 |
| **AI-ENG-C** | Latency-Margin Management | Execution time limits and budgets. | Coordinates dynamic step delays with active network and frame stability metrics.1 |
| **AI-ENG-L** | Serving Mechanics | Bidirectional serving backpressure | Terminate WebRTC packets at dynamic transceiver edge proxy containers.41 |
| **AI-ENG-M** | Autonomy Boundaries | Timeout parameters and limits.34 | Enforces retry budgets, restricts action loops, and triggers manual escalations on errors.4 |
| **AI-ENG-N** | Tool Contracts | API schemas and parameter controls. | Matches dispatched browser commands against validated execution schemas.5 |
| **AI-ENG-O** | Action Verification | Transition verification logs.5 | Ensures post-action interface states are verified before executing subsequent steps.5 |
| **AI-ENG-P** | Multimodal Understanding | Coordinate coordinates and crops.17 | Integrates visual segmentation and OCR engines for fallback coordinate matching.1 |
| **AI-ENG-Q** | Real-Time Interaction | Stream processing and inputs.52 | Manages input throttling and coordinate scaling for voice-driven browser contexts.52 |
| **AI-ENG-S** | Production Pathologies | Telemetry and crash logs.22 | Monitors and logs layout drift, unmounting errors, and browser process crashes.10 |
| **AI-ENG-T** | Prompt Security | Content filters and bounds.7 | Implements input sanitization and context limits to block indirect prompt injections.7 |
| **AI-ENG-U** | Dependency Risk | Sandboxing and isolation limits.34 | Hardens sandbox containers and isolates local directories to prevent resource leaks.34 |
| **AI-ENG-V** | Resource Protection | Request tracking and blocks.34 | Prevents request abuse, blocks recursive navigations, and limits action volumes.34 |
| **AI-ENG-W** | Fallback Operations | Fallback triggers and playbooks.4 | Activates visual coordinate matching when DOM engines fail.1 |
| **AI-ENG-X** | User Trust | Visual status indicators.17 | Renders real-time action steps and bounding boxes to keep users informed.6 |
| **AI-ENG-Y** | Human Review | Escalation queues and context transfer. | Suspends execution and transfers session context to users on verification failures.7 |
| **AI-ENG-Z** | Telemetry | Telemetry schemas and formats.43 | Formats and exports trace logs and session metrics using standardized telemetry schemas.43 |
| **AI-ENG-AA** | Evaluation | Automation test suites and metrics. | Runs automated regression tests against standard evaluation benchmarks (e.g., WebArena).41 |
| **AI-ENG-AB** | Replay & Debugging | Signed traces and replays.17 | Stores signed, timestamped session records to support offline audits and debugger replays.17 |
| **AI-ENG-AC** | Incident Response | Sandbox quarantine protocols.44 | Isolates and quarantines compromised container hosts and revokes active session credentials.34 |
| **AI-ENG-AJ** | Reference Blueprint | Production layouts and diagrams. | Integrates remote browser isolation hosts and containerized desktop clusters.34 |

## **Durable Principles of UI Agents**

1. **Interfaces are State Machines**: A user interface is a partially observable, drift-prone state machine. Interaction events (clicks, inputs, submissions) are hypotheses that must be visually, semantically, and structurally verified against resulting state transitions before execution can proceed.4  
2. **Dynamic Locators are Mandatory**: Storing raw, static element references causes automation failures during client-side rendering.10 To prevent race conditions, agents must use dynamic locators that re-query and resolve target elements immediately before event dispatch.10  
3. **Plausible Visuals Lack Structure**: Vision models can analyze global trends but struggle to extract small characters, read overlapping labels, or resolve semantic focus. Robust agents must combine visual analysis with semantic accessibility tree traversal to ensure accurate targeting.1  
4. **Data is Evidence, Not Authority**: Untrusted page content, ads, and comments represent data context, not commands.7 Executing actions based on unverified content introduces prompt injection vulnerabilities. Security boundaries must be enforced at the API, credential, and sandbox layers.7  
5. **Containment by Default**: UI agents must operate within disposable virtual environments, isolated framebuffers, and restricted networks.4 No agent should have ambient access to the developer's default browser profile, corporate filesystem, or host credentials.16

#### **Works cited**

1. Desktop Automation AI Agents: Beyond the Browser | by Oktay Ateş | May, 2026 | Medium, accessed June 10, 2026, [https://medium.com/@oktayates/desktop-automation-ai-agents-beyond-the-browser-031c52b0ec7a](https://medium.com/@oktayates/desktop-automation-ai-agents-beyond-the-browser-031c52b0ec7a)  
2. I built a library that gives AI agents structured UI access via accessibility APIs, like Playwright but for the entire OS - Reddit, accessed June 10, 2026, [https://www.reddit.com/r/AI_Agents/comments/1s8o7gs/i_built_a_library_that_gives_ai_agents_structured/](https://www.reddit.com/r/AI_Agents/comments/1s8o7gs/i_built_a_library_that_gives_ai_agents_structured/)  
3. Learn to Adapt: Robust Drift Detection in Security Domain - arXiv, accessed June 10, 2026, [https://arxiv.org/pdf/2206.07581](https://arxiv.org/pdf/2206.07581)  
4. computer-use: overhaul for improved agentic desktop + browser interaction · Issue #4445 · daytonaio/daytona - GitHub, accessed June 10, 2026, [https://github.com/daytonaio/daytona/issues/4445](https://github.com/daytonaio/daytona/issues/4445)  
5. Auto-waiting - Playwright, accessed June 10, 2026, [https://playwright.dev/docs/actionability](https://playwright.dev/docs/actionability)  
6. Best Practices - Playwright, accessed June 10, 2026, [https://playwright.dev/docs/best-practices](https://playwright.dev/docs/best-practices)  
7. Indirect prompt injection through enterprise data is becoming a real attack surface - Reddit, accessed June 10, 2026, [https://www.reddit.com/r/aiagents/comments/1tgjzfj/indirect_prompt_injection_through_enterprise_data/](https://www.reddit.com/r/aiagents/comments/1tgjzfj/indirect_prompt_injection_through_enterprise_data/)  
8. Cross-platform desktop automation through accessibility APIs ..., accessed June 10, 2026, [https://crowecawcaw.github.io/general/2026/05/30/accessibility-for-computer-use.html](https://crowecawcaw.github.io/general/2026/05/30/accessibility-for-computer-use.html)  
9. Mastering waits and timeouts in Playwright - CircleCI, accessed June 10, 2026, [https://circleci.com/blog/mastering-waits-and-timeouts-in-playwright/](https://circleci.com/blog/mastering-waits-and-timeouts-in-playwright/)  
10. Playwright auto-wait is great, until your component re-renders mid-action | Mergify, accessed June 10, 2026, [https://mergify.com/blog/playwright-auto-wait-element-rerenders](https://mergify.com/blog/playwright-auto-wait-element-rerenders)  
11. Locator - Playwright, accessed June 10, 2026, [https://playwright.dev/docs/api/class-locator](https://playwright.dev/docs/api/class-locator)  
12. Automation Protocols - WebdriverIO, accessed June 10, 2026, [https://webdriver.io/docs/automationProtocols/](https://webdriver.io/docs/automationProtocols/)  
13. CDP vs. BiDi: Browser Automation Protocol Internals for Scrapers - Evomi Blog, accessed June 10, 2026, [https://evomi.com/blog/cdp-vs.-bidi-browser-automation-protocol-internals-for-scrapers](https://evomi.com/blog/cdp-vs.-bidi-browser-automation-protocol-internals-for-scrapers)  
14. Locators - Playwright, accessed June 10, 2026, [https://playwright.dev/docs/locators](https://playwright.dev/docs/locators)  
15. React Playwright Testing: Complete Guide - Autonoma AI, accessed June 10, 2026, [https://getautonoma.com/blog/react-playwright-testing-guide](https://getautonoma.com/blog/react-playwright-testing-guide)  
16. Browser Control Layer: Programmatic Desktop Automation via macOS Accessibility API for AI Agent Swarms - Pubroot, accessed June 10, 2026, [https://pubroot.com/ai/agent-architecture/browser-control-layer-programmatic-desktop-automation-via-macos-accessib/](https://pubroot.com/ai/agent-architecture/browser-control-layer-programmatic-desktop-automation-via-macos-accessib/)  
17. AI-ENG-P — Multimodal Understanding - Documents, Images, Tables, Charts & Video  
18. WebDriver BiDi reference - MDN Web Docs, accessed June 10, 2026, [https://developer.mozilla.org/en-US/docs/Web/WebDriver/Reference/BiDi](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Reference/BiDi)  
19. Chrome DevTools Protocol - GitHub Pages, accessed June 10, 2026, [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)  
20. Revisiting WebDriver and Selenium: Understanding History and Internals (CDP/BiDi) - Zenn, accessed June 10, 2026, [https://zenn.dev/mima_ita/articles/61c12757495cc0?locale=en](https://zenn.dev/mima_ita/articles/61c12757495cc0?locale=en)  
21. axuiautomation package - github.com/tmc/apple/x/axuiautomation - Go Packages, accessed June 10, 2026, [https://pkg.go.dev/github.com/tmc/apple/x/axuiautomation](https://pkg.go.dev/github.com/tmc/apple/x/axuiautomation)  
22. WebDriver BiDi: 2023 status update | Blog - Chrome for Developers, accessed June 10, 2026, [https://developer.chrome.com/blog/webdriver-bidi-2023](https://developer.chrome.com/blog/webdriver-bidi-2023)  
23. Selenium Chrome DevTools Protocol (CDP) API: How Does It Work? - Applitools, accessed June 10, 2026, [https://applitools.com/blog/selenium-chrome-devtools-protocol-cdp-how-does-it-work/](https://applitools.com/blog/selenium-chrome-devtools-protocol-cdp-how-does-it-work/)  
24. Prompt injection in AI agents: are your controls keeping up?, accessed June 10, 2026, [https://nhimg.org/community/agentic-ai-and-nhis/prompt-injection-in-ai-agents-are-your-controls-keeping-up/](https://nhimg.org/community/agentic-ai-and-nhis/prompt-injection-in-ai-agents-are-your-controls-keeping-up/)  
25. Trust Playwright's Auto-Waiting for End-to-End Testing - YouTube, accessed June 10, 2026, [https://www.youtube.com/shorts/eDdsynE9Xw0](https://www.youtube.com/shorts/eDdsynE9Xw0)  
26. WebDriver vs Chrome DevTools (Speed Test) - Abstracta, accessed June 10, 2026, [https://abstracta.us/blog/testing-tools/webdriver-vs-chrome-devtools-speed-test/](https://abstracta.us/blog/testing-tools/webdriver-vs-chrome-devtools-speed-test/)  
27. WebDriver BiDi - W3C, accessed June 10, 2026, [https://www.w3.org/TR/webdriver-bidi/](https://www.w3.org/TR/webdriver-bidi/)  
28. Chrome DevTools Protocol (CDP) - Pydoll, accessed June 10, 2026, [https://pydoll.tech/docs/deep-dive/fundamentals/cdp/](https://pydoll.tech/docs/deep-dive/fundamentals/cdp/)  
29. WebDriver BiDi - The future of cross-browser automation | Blog - Chrome for Developers, accessed June 10, 2026, [https://developer.chrome.com/blog/webdriver-bidi](https://developer.chrome.com/blog/webdriver-bidi)  
30. BiDirectional functionality - Selenium, accessed June 10, 2026, [https://www.selenium.dev/documentation/webdriver/bidi/](https://www.selenium.dev/documentation/webdriver/bidi/)  
31. IUIAutomationElement7::FindAllWithOptionsBuildCache (uiautomationclient.h) - Win32 apps, accessed June 10, 2026, [https://learn.microsoft.com/en-us/windows/win32/api/uiautomationclient/nf-uiautomationclient-iuiautomationelement7-findallwithoptionsbuildcache](https://learn.microsoft.com/en-us/windows/win32/api/uiautomationclient/nf-uiautomationclient-iuiautomationelement7-findallwithoptionsbuildcache)  
32. IUIAutomationCacheRequest (uiautomationclient.h) - Win32 apps - Microsoft Learn, accessed June 10, 2026, [https://learn.microsoft.com/en-us/windows/win32/api/uiautomationclient/nn-uiautomationclient-iuiautomationcacherequest](https://learn.microsoft.com/en-us/windows/win32/api/uiautomationclient/nn-uiautomationclient-iuiautomationcacherequest)  
33. Caching UI Automation Properties and Control Patterns - GitHub, accessed June 10, 2026, [https://github.com/MicrosoftDocs/win32/blob/docs/desktop-src/WinAuto/uiauto-cachingforclients.md](https://github.com/MicrosoftDocs/win32/blob/docs/desktop-src/WinAuto/uiauto-cachingforclients.md)  
34. AI Agent Sandbox: How to Safely Run Autonomous Agents in 2026 - Firecrawl, accessed June 10, 2026, [https://www.firecrawl.dev/blog/ai-agent-sandbox](https://www.firecrawl.dev/blog/ai-agent-sandbox)  
35. Locator | Playwright Python, accessed June 10, 2026, [https://playwright.dev/python/docs/api/class-locator](https://playwright.dev/python/docs/api/class-locator)  
36. What Is Remote Browser Isolation (RBI) and How Does It Work? - NordLayer, accessed June 10, 2026, [https://nordlayer.com/learn/browser-security/what-is-browser-isolation/](https://nordlayer.com/learn/browser-security/what-is-browser-isolation/)  
37. DevTools Event Listener, accessed June 10, 2026, [https://chromium.googlesource.com/chromium/src/+/refs/heads/main/chrome/test/chromedriver/docs/event_listener.md](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/chrome/test/chromedriver/docs/event_listener.md)  
38. Playwright Assertions: Avoid Race Conditions with This Simple Fix! - DEV Community, accessed June 10, 2026, [https://dev.to/playwright/playwright-assertions-avoid-race-conditions-with-this-simple-fix-dm1](https://dev.to/playwright/playwright-assertions-avoid-race-conditions-with-this-simple-fix-dm1)  
39. Playwright Assertions: Avoid Race Conditions with This Simple Fix! - YouTube, accessed June 10, 2026, [https://www.youtube.com/watch?v=1VxkHP8vfGg](https://www.youtube.com/watch?v=1VxkHP8vfGg)  
40. Fooling AI Agents: Web-Based Indirect Prompt Injection Observed in ..., accessed June 10, 2026, [https://unit42.paloaltonetworks.com/ai-agent-prompt-injection/](https://unit42.paloaltonetworks.com/ai-agent-prompt-injection/)  
41. Why Your AI Agents Need a Sandbox: OpenSandbox by Alibaba Explained - Emelia.io, accessed June 10, 2026, [https://emelia.io/hub/opensandbox-ai-agents-guide](https://emelia.io/hub/opensandbox-ai-agents-guide)  
42. Indirect Prompt Injection in Web-Browsing Agents | Promptfoo, accessed June 10, 2026, [https://www.promptfoo.dev/blog/indirect-prompt-injection-web-agents/](https://www.promptfoo.dev/blog/indirect-prompt-injection-web-agents/)  
43. Sandbox - Chromium Docs, accessed June 10, 2026, [https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md)  
44. Site Isolation - The Chromium Projects, accessed June 10, 2026, [https://www.chromium.org/Home/chromium-security/site-isolation/](https://www.chromium.org/Home/chromium-security/site-isolation/)  
45. bureado/awesome-agent-runtime-security - GitHub, accessed June 10, 2026, [https://github.com/bureado/awesome-agent-runtime-security](https://github.com/bureado/awesome-agent-runtime-security)  
46. chrome://sandbox Diagnostics for Windows - The Chromium Projects, accessed June 10, 2026, [https://www.chromium.org/Home/chromium-security/articles/chrome-sandbox-diagnostics-for-windows/](https://www.chromium.org/Home/chromium-security/articles/chrome-sandbox-diagnostics-for-windows/)  
47. What is Remote Browser Isolation (RBI) | Nomios Group, accessed June 10, 2026, [https://www.nomios.com/resources/remote-browser-isolation-rbi/](https://www.nomios.com/resources/remote-browser-isolation-rbi/)  
48. Prompt Injection - OWASP Foundation, accessed June 10, 2026, [https://owasp.org/www-community/attacks/PromptInjection](https://owasp.org/www-community/attacks/PromptInjection)  
49. How are you handling prompt injection in AI agents that read untrusted content? - Reddit, accessed June 10, 2026, [https://www.reddit.com/r/AskNetsec/comments/1rwywvu/how_are_you_handling_prompt_injection_in_ai/](https://www.reddit.com/r/AskNetsec/comments/1rwywvu/how_are_you_handling_prompt_injection_in_ai/)  
50. how-microsoft-defends-against-indirect-prompt-injection-attacks, accessed June 10, 2026, [https://www.microsoft.com/en-us/msrc/blog/2025/07/how-microsoft-defends-against-indirect-prompt-injection-attacks](https://www.microsoft.com/en-us/msrc/blog/2025/07/how-microsoft-defends-against-indirect-prompt-injection-attacks)  
51. Architectures for Agent Systems: A Survey of Isolation, Integration, and Governance | by yunwei37 | Medium, accessed June 10, 2026, [https://medium.com/@yunwei356/architectures-for-agent-systems-a-survey-of-isolation-integration-and-governance-59224d26e666](https://medium.com/@yunwei356/architectures-for-agent-systems-a-survey-of-isolation-integration-and-governance-59224d26e666)  
52. AI-ENG-Q — Speech, Voice, and Real-Time Interaction Systems

---

[← Back to Canon Map](../canon-map.md)