# CallGrid CDN Documentation

## Overview

CallGrid can be loaded directly from our CDN without any build process or package installation. This guide covers all available methods for integrating CallGrid into your website.

## Table of Contents

- [Quick Start](#quick-start)
- [Installation Methods](#installation-methods)
  - [Method 1: Auto-initialization with Data Attributes](#method-1-auto-initialization-with-data-attributes)
  - [Method 2: Manual Initialization](#method-2-manual-initialization)
  - [Method 3: Dynamic Script Loading](#method-3-dynamic-script-loading)
- [Configuration Options](#configuration-options)
- [DNI Options](#dni-options)
- [API Methods](#api-methods)
- [Events](#events)
- [Examples](#examples)
- [SPA Integration](#spa-integration)

## Quick Start

The simplest way to add CallGrid to your website:

```html
<script
  src="https://cdn.callgrid.com/callgrid.js"
  data-organization-id="YOUR_ORG_ID"
  data-campaign-source-id="YOUR_CAMPAIGN_ID"
></script>
```

This will automatically initialize CallGrid and start replacing phone numbers on your page.

## Installation Methods

### Method 1: Auto-initialization with Data Attributes

The easiest way to use CallGrid is with data attributes. The script will automatically initialize when it loads.

```html
<script
  src="https://cdn.callgrid.com/callgrid.js"
  data-organization-id="YOUR_ORG_ID"
  data-campaign-source-id="YOUR_CAMPAIGN_ID"
  data-idle-timeout="300000"
  data-target-phone-number="555-123-4567"
  data-auto-enable-dni="true"
></script>
```

**Available Data Attributes:**

| Attribute                  | Required | Description                                                         | Default            |
| -------------------------- | -------- | ------------------------------------------------------------------- | ------------------ |
| `data-organization-id`     | Yes      | Your CallGrid organization ID                                       | -                  |
| `data-campaign-source-id`  | Yes      | Your campaign source ID                                             | -                  |
| `data-idle-timeout`        | No       | Time in milliseconds before releasing number due to inactivity      | 300000 (5 minutes) |
| `data-target-phone-number` | No       | Specific phone number to replace (if not set, replaces all numbers) | -                  |
| `data-auto-enable-dni`     | No       | Automatically enable Dynamic Number Insertion                       | true               |

### Method 2: Manual Initialization

Load the script without data attributes and initialize manually:

```html
<script src="https://cdn.callgrid.com/callgrid.js"></script>
<script>
  // Initialize CallGrid manually
  const callgrid = new CallGrid({
    organizationId: 'YOUR_ORG_ID',
    campaignSourceId: 'YOUR_CAMPAIGN_ID',
    idleTimeout: 300000, // 5 minutes
    targetPhoneNumber: '555-123-4567', // Optional: only replace this number
    autoEnableDNI: true, // Automatically enable number replacement
  });
</script>
```

### Method 3: Dynamic Script Loading

Load CallGrid only when needed (e.g., after user interaction):

```html
<button onclick="loadCallGrid()">Load CallGrid</button>

<script>
  function loadCallGrid() {
    // Create script element
    const script = document.createElement('script');
    script.src = 'https://cdn.callgrid.com/callgrid.js';

    // Set configuration via data attributes
    script.dataset.organizationId = 'YOUR_ORG_ID';
    script.dataset.campaignSourceId = 'YOUR_CAMPAIGN_ID';
    script.dataset.idleTimeout = '300000';
    script.dataset.autoEnableDni = 'true';

    // Append to document
    document.head.appendChild(script);

    // Optionally, listen for load event
    script.onload = function () {
      console.log('CallGrid loaded successfully');
    };
  }
</script>
```

Or load and initialize manually after an event:

```html
<button onclick="loadAndInitialize()">Start Tracking</button>

<script>
  function loadAndInitialize() {
    const script = document.createElement('script');
    script.src = 'https://cdn.callgrid.com/callgrid.js';

    script.onload = function () {
      // Initialize after script loads
      const callgrid = new CallGrid({
        organizationId: 'YOUR_ORG_ID',
        campaignSourceId: 'YOUR_CAMPAIGN_ID',
        idleTimeout: 300000,
        autoEnableDNI: true,
      });
    };

    document.head.appendChild(script);
  }
</script>
```

## Configuration Options

When initializing CallGrid manually, you can pass the following configuration options:

```javascript
{
  organizationId: 'YOUR_ORG_ID',        // Required: Your organization ID
  campaignSourceId: 'YOUR_CAMPAIGN_ID', // Required: Your campaign source ID
  idleTimeout: 300000,                  // Optional: Milliseconds before releasing number (default: 300000)
  targetPhoneNumber: '555-123-4567',    // Optional: Only replace this specific number
  autoEnableDNI: true                   // Optional: Auto-enable number replacement (default: true)
}
```

### Configuration Details

- **organizationId**: Your unique organization identifier provided by CallGrid
- **campaignSourceId**: The campaign source ID for tracking attribution
- **idleTimeout**: Time in milliseconds of user inactivity before releasing the assigned tracking number. Set to 0 to disable idle timeout.
- **targetPhoneNumber**: When specified, only this phone number will be replaced on your site. Supports various formats (e.g., "555-123-4567", "(555) 123-4567", "+15551234567")
- **autoEnableDNI**: When true, automatically starts replacing phone numbers after initialization

## DNI Options

When calling `enableDNI()` manually, you can customize the Dynamic Number Insertion behavior:

```javascript
callgrid.enableDNI({
  selector: 'body', // CSS selector(s) to search within
  excludeSelectors: ['script', 'style', 'input'], // Elements to exclude
  targetPhoneNumber: '555-123-4567', // Override target number for this call
});
```

### DNI Option Details

- **selector**: String or array of CSS selectors to search for phone numbers (default: 'body')
- **excludeSelectors**: Array of CSS selectors to exclude from replacement (default: ['script', 'style', 'input', 'meta', 'iframe'])
- **targetPhoneNumber**: Specific number to replace, overrides config setting

## API Methods

Once CallGrid is initialized, you can access these methods:

### Core Methods

```javascript
// Get the CallGrid instance (after auto-initialization)
const callgrid = window.CallGrid;

// Get a tracking number
const number = await callgrid.getNumber();

// Enable DNI (if autoEnableDNI was false)
await callgrid.enableDNI({
  selector: '.phone-number',
  targetPhoneNumber: '555-123-4567',
});

// Disable DNI and revert to original numbers
callgrid.disableDNI();

// Get the currently assigned tracking number
const assignedNumber = callgrid.getAssignedNumber();

// Clear stored data
callgrid.clearStorage();

// Destroy the instance and clean up
callgrid.destroy();
```

### Number Formatting

```javascript
// Format a phone number
const formatted = callgrid.formatNumber('+15551234567', 'national');
// Returns: (555) 123-4567
```

## Events

CallGrid dispatches custom events you can listen to:

```javascript
// Number assigned
document.addEventListener('callgrid:numberAssigned', function (event) {
  console.log('Number assigned:', event.detail);
  // event.detail contains: { phoneNumber, sourceId, timezone, timestamp }
});

// Number released (idle timeout)
document.addEventListener('callgrid:numberReleased', function (event) {
  console.log('Number released:', event.detail);
  // event.detail contains: { number, timestamp, reason }
});

// User returns from idle
document.addEventListener('callgrid:returnFromIdle', function (event) {
  console.log('User active again:', event.detail);
  // event.detail contains: { timestamp }
});

// Page change detected (for SPAs)
document.addEventListener('callgrid:pageChange', function (event) {
  console.log('Page changed:', event.detail);
  // event.detail contains: { timestamp, url, assignedNumber }
});
```

## Examples

### Example 1: Basic Implementation

```html
<!DOCTYPE html>
<html>
  <head>
    <title>CallGrid Example</title>
  </head>
  <body>
    <h1>Contact Us</h1>
    <p>Call us at: (555) 123-4567</p>
    <p>Toll-free: 1-800-555-9876</p>
    <a href="tel:+15551234567">Click to Call</a>

    <!-- CallGrid with auto-initialization -->
    <script
      src="https://cdn.callgrid.com/callgrid.js"
      data-organization-id="org_123"
      data-campaign-source-id="campaign_456"
      data-auto-enable-dni="true"
    ></script>
  </body>
</html>
```

### Example 2: Replace Specific Number Only

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Targeted Number Replacement</title>
  </head>
  <body>
    <h1>Multiple Phone Numbers</h1>
    <p>Main Office: (555) 123-4567</p>
    <!-- This will be replaced -->
    <p>Support: (555) 987-6543</p>
    <!-- This won't be replaced -->
    <p>Sales: (555) 123-4567</p>
    <!-- This will be replaced -->

    <script
      src="https://cdn.callgrid.com/callgrid.js"
      data-organization-id="org_123"
      data-campaign-source-id="campaign_456"
      data-target-phone-number="555-123-4567"
      data-auto-enable-dni="true"
    ></script>
  </body>
</html>
```

### Example 3: Manual Control

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Manual CallGrid Control</title>
  </head>
  <body>
    <h1>Contact Information</h1>
    <div class="contact-info">
      <p>Phone: (555) 123-4567</p>
    </div>

    <button onclick="startTracking()">Start Tracking</button>
    <button onclick="stopTracking()">Stop Tracking</button>

    <script src="https://cdn.callgrid.com/callgrid.js"></script>
    <script>
      let callgrid;

      function startTracking() {
        if (!callgrid) {
          callgrid = new CallGrid({
            organizationId: 'org_123',
            campaignSourceId: 'campaign_456',
            autoEnableDNI: false, // Manual control
          });
        }

        // Enable DNI for specific area
        callgrid.enableDNI({
          selector: '.contact-info',
          targetPhoneNumber: '555-123-4567',
        });
      }

      function stopTracking() {
        if (callgrid) {
          callgrid.disableDNI();
        }
      }
    </script>
  </body>
</html>
```

### Example 4: Event-Triggered Loading

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Load on User Action</title>
  </head>
  <body>
    <h1>Premium Content</h1>
    <button onclick="showContactInfo()">Show Contact Information</button>

    <div id="contact" style="display: none;">
      <h2>Contact Us</h2>
      <p>Phone: (555) 123-4567</p>
    </div>

    <script>
      function showContactInfo() {
        // Show the contact div
        document.getElementById('contact').style.display = 'block';

        // Load CallGrid if not already loaded
        if (!window.CallGrid) {
          const script = document.createElement('script');
          script.src = 'https://cdn.callgrid.com/callgrid.js';
          script.dataset.organizationId = 'org_123';
          script.dataset.campaignSourceId = 'campaign_456';
          script.dataset.autoEnableDni = 'true';
          document.head.appendChild(script);
        }
      }
    </script>
  </body>
</html>
```

## SPA Integration

CallGrid automatically detects page changes in Single Page Applications (SPAs) and re-applies number replacement. This works with:

- React Router
- Vue Router
- Angular Router
- Any framework using History API

### React Example

```jsx
import { useEffect } from 'react';

function ContactPage() {
  useEffect(() => {
    // CallGrid auto-detects the page change
    // Numbers will be automatically replaced

    // Optional: Listen for CallGrid events
    const handlePageChange = (event) => {
      console.log('CallGrid detected page change:', event.detail);
    };

    document.addEventListener('callgrid:pageChange', handlePageChange);

    return () => {
      document.removeEventListener('callgrid:pageChange', handlePageChange);
    };
  }, []);

  return (
    <div>
      <h1>Contact Us</h1>
      <p>Call: (555) 123-4567</p>
    </div>
  );
}
```

### Vue Example

```vue
<template>
  <div>
    <h1>Contact Us</h1>
    <p>Phone: (555) 123-4567</p>
  </div>
</template>

<script>
export default {
  mounted() {
    // CallGrid automatically handles the page change
    console.log('CallGrid will replace numbers on this page');
  },
};
</script>
```

## Best Practices

1. **Load Early**: Include the CallGrid script in the `<head>` or early in `<body>` for best performance
2. **Use Data Attributes**: For simple implementations, data attributes are the easiest method
3. **Target Specific Numbers**: Use `targetPhoneNumber` when you only want to track specific numbers
4. **Set Appropriate Idle Timeout**: Adjust based on your typical user session length
5. **Handle Events**: Listen to CallGrid events for analytics or custom behavior

## Troubleshooting

### Numbers Not Being Replaced

- Verify your organization ID and campaign source ID are correct
- Check that the phone numbers on your page match standard formats
- Ensure JavaScript is enabled in the browser
- If using `targetPhoneNumber`, verify the format matches exactly

### Performance Considerations

- CallGrid uses efficient DOM traversal and caches results
- The script is optimized for minimal impact on page load
- Consider dynamic loading for pages where tracking isn't always needed

## Support

For additional support or questions, please contact CallGrid support.

