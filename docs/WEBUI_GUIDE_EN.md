# AMMF WebUI Development Guide

[简体中文](WEBUI_GUIDE.md) | [English](WEBUI_GUIDE_EN.md)

## 📋 Overview

This document provides development and customization guidelines for the WebUI component of the AMMF framework. WebUI is a browser-based configuration interface that allows users to configure module settings, view status information, and perform common operations through a graphical interface.

## 🚀 Quick Start

## 🎨 Customizing the WebUI

### File Structure

WebUI-related files are located in the `webroot/` directory:

```
webroot/
├── index.html         # Main page
├── app.js             # Main application logic
├── core.js            # Core functionality module
├── i18n.js            # Multi-language support
├── logger.js          # Logging
├── style.css          # Main stylesheet
├── animations.css     # Animation styles
├── layout.css         # Layout styles
├── theme.css          # Theme styles
├── theme.js           # Theme handling
└── pages/             # Page modules
    ├── status.js      # Status page
    ├── logs.js        # Logs page
    ├── settings.js    # Settings page
    └── about.js       # About page
```

### Modifying the Interface Layout

To modify the WebUI interface layout, edit the `webroot/index.html` file. This file contains the basic HTML structure of the WebUI.

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>AMMF WebUI</title>
    
    <!-- Stylesheets -->
    <link rel="stylesheet" href="style.css">
    <link rel="stylesheet" href="animations.css">
    <link rel="stylesheet" href="theme.css">
    <link rel="stylesheet" href="layout.css">
</head>
<body>
    <div id="app">
        <!-- Header -->
        <header class="app-header">...</header>
        
        <!-- Main content area -->
        <main id="main-content">...</main>
        
        <!-- Bottom navigation -->
        <nav class="app-nav">...</nav>
    </div>
    
    <!-- Script references -->
    <script src="theme.js"></script>
    <script src="i18n.js"></script>
    <script src="core.js"></script>
    <script src="logger.js"></script>
    <script src="pages/status.js"></script>
    <script src="pages/logs.js"></script>
    <script src="pages/settings.js"></script>
    <script src="pages/about.js"></script>
    <script src="app.js"></script>
</body>
</html>
```

### Customizing Styles

To modify the WebUI styles, edit the `webroot/style.css` file. This file contains the CSS style definitions for the WebUI.

```css
/* Example: Modify theme colors */
:root {
    --md-primary: #006495; /* Primary color */
    --md-onPrimary: #ffffff;
    --md-primaryContainer: #cde5ff;
    --md-onPrimaryContainer: #001d31;
    --md-secondary: #50606e;
    --md-onSecondary: #ffffff;
    --md-secondaryContainer: #d3e5f5;
    --md-onSecondaryContainer: #0c1d29;
    /* More color variables */
}

/* Dark theme */
.dark-theme {
    --md-primary: #91cbff;
    --md-onPrimary: #003355;
    --md-primaryContainer: #004a78;
    --md-onPrimaryContainer: #cde5ff;
    /* More dark theme variables */
}
```

### Adding New Pages

To add new pages, follow these steps:

1. Create a new page JS file in the `webroot/pages/` directory, for example `newpage.js`:

```javascript
/**
 * AMMF WebUI New Page Module
 * Description of the new page functionality
 */

const NewPage = {
    // Initialization
    async init() {
        try {
            // Initialization code
            return true;
        } catch (error) {
            console.error('Failed to initialize new page:', error);
            return false;
        }
    },
    
    // Render page
    render() {
        return `
            <div class="page-container new-page">
                <h2 data-i18n="NEW_PAGE_TITLE">New Page</h2>
                <div class="card">
                    <p data-i18n="NEW_PAGE_CONTENT">This is the content of the new page</p>
                </div>
            </div>
        `;
    },
    
    // After render callback
    afterRender() {
        // Bind events and other operations
    }
};

// Export page module
window.NewPage = NewPage;
```

2. Include the new page script in `webroot/index.html`:

```html
<!-- Add after other page scripts -->
<script src="pages/newpage.js"></script>
```

3. Register the new page in `webroot/app.js`:

```javascript
// Page modules
this.pageModules = {
    status: window.StatusPage,
    logs: window.LogsPage,
    settings: window.SettingsPage,
    about: window.AboutPage,
    newpage: window.NewPage  // Add new page
};
```

4. Add an entry for the new page in the navigation bar:

```html
<!-- Add in app-nav -->
<div class="nav-item" data-page="newpage">
    <span class="material-symbols-rounded">extension</span>
    <span class="nav-label" data-i18n="NAV_NEW_PAGE">New Page</span>
</div>
```

5. Add translation strings for the new page in `webroot/i18n.js`:

```javascript
// Chinese translation
this.translations.zh = {
    // Existing translations
    NAV_NEW_PAGE: '新页面',
    NEW_PAGE_TITLE: '新页面标题',
    NEW_PAGE_CONTENT: '这是新页面的内容'
};

// English translation
this.translations.en = {
    // Existing translations
    NAV_NEW_PAGE: 'New Page',
    NEW_PAGE_TITLE: 'New Page Title',
    NEW_PAGE_CONTENT: 'This is the content of the new page'
};
```

## 📊 Data Processing

### Settings Handling

WebUI handles module settings through the `pages/settings.js` file. This file is responsible for fetching settings from the server, displaying settings forms, and saving user-modified settings.

```javascript
// Load settings data
async loadSettingsData() {
    try {
        this.showLoading();
        
        // Execute Shell command to get settings
        const settingsData = await Core.execCommand(`cat "${Core.MODULE_PATH}module_settings/settings.json"`);
        this.settings = JSON.parse(settingsData);
        
        this.hideLoading();
        return this.settings;
    } catch (error) {
        console.error('Failed to load settings data:', error);
        this.hideLoading();
        Core.showToast(I18n.translate('SETTINGS_LOAD_ERROR', 'Failed to load settings'), 'error');
        return {};
    }
},

// Save settings
async saveSettings() {
    try {
        this.showLoading();
        
        // Convert settings to JSON string
        const settingsJson = JSON.stringify(this.settings, null, 4);
        
        // Execute Shell command to save settings
        await Core.execCommand(`echo '${settingsJson}' > "${Core.MODULE_PATH}module_settings/settings.json"`);
        
        this.hideLoading();
        Core.showToast(I18n.translate('SETTINGS_SAVED', 'Settings saved'), 'success');
        return true;
    } catch (error) {
        console.error('Failed to save settings:', error);
        this.hideLoading();
        Core.showToast(I18n.translate('SETTINGS_SAVE_ERROR', 'Failed to save settings'), 'error');
        return false;
    }
}
```

### Status Monitoring

WebUI handles module status information through the `pages/status.js` file. This file is responsible for fetching status information from the server and displaying it in the interface.

```javascript
// Load module status
async loadModuleStatus() {
    try {
        // Execute Shell command to get status
        const statusOutput = await Core.execCommand(`cd "${Core.MODULE_PATH}" && busybox sh service.sh status`);
        
        // Parse status output
        if (statusOutput.includes('running')) {
            this.moduleStatus = 'RUNNING';
        } else if (statusOutput.includes('stopped')) {
            this.moduleStatus = 'STOPPED';
        } else if (statusOutput.includes('paused')) {
            this.moduleStatus = 'PAUSED';
        } else {
            this.moduleStatus = 'UNKNOWN';
        }
        
        return this.moduleStatus;
    } catch (error) {
        console.error('Failed to load module status:', error);
        this.moduleStatus = 'UNKNOWN';
        return this.moduleStatus;
    }
}
```

## 🔄 API Interface

WebUI communicates with the backend through the API provided by the Core module:

### Core API

`core.js` provides core functionality for interacting with Shell:

```javascript
// Execute Shell command
async execCommand(command) {
    const callbackName = `exec_callback_${Date.now()}`;
    return new Promise((resolve, reject) => {
        window[callbackName] = (errno, stdout, stderr) => {
            delete window[callbackName];
            errno === 0 ? resolve(stdout) : reject(stderr);
        };
        ksu.exec(command, "{}", callbackName);
    });
}
```

## 🌐 Multi-language Support

WebUI supports multiple languages through the `i18n.js` file. This file contains translation strings for various languages and language switching functionality.

```javascript
// Initialize language module
async init() {
    try {
        // Load default translations
        this.loadDefaultTranslations();

        // Apply default language translations
        this.applyTranslations();

        // Determine initial language
        await this.determineInitialLanguage();

        // Apply language again (using determined language)
        this.applyTranslations();

        // Initialize language selector
        this.initLanguageSelector();

        return true;
    } catch (error) {
        console.error('Failed to initialize language module:', error);
        // Use default language
        this.currentLang = 'zh';
        return false;
    }
},

// Get translation string
translate(key, fallback = '') {
    const translation = this.translations[this.currentLang]?.[key] || 
                       this.translations['zh']?.[key] || 
                       fallback || 
                       key;
    return translation;
},

// Apply translations to DOM elements
applyTranslations() {
    document.querySelectorAll('[data-i18n]').forEach(element => {
        const key = element.getAttribute('data-i18n');
        element.textContent = this.translate(key, element.textContent);
    });
}
```

## 🎭 Theme Support

WebUI supports light and dark theme switching through the `theme.js` file. This file contains theme switching functionality and theme-related style handling.

```javascript
// Initialize theme
init() {
    // Get stored theme or system theme
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
        this.setTheme(savedTheme);
    } else {
        // Use system theme
        this.useSystemTheme();
    }
    
    // Bind theme toggle button
    this.bindThemeToggle();
    
    // Watch for system theme changes
    this.watchSystemTheme();
},

// Set theme
setTheme(theme) {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
    
    // Update theme icon
    const themeIcon = document.querySelector('#theme-toggle .material-symbols-rounded');
    if (themeIcon) {
        themeIcon.textContent = theme === 'dark' ? 'light_mode' : 'dark_mode';
    }
}
```

## 📱 Responsive Design

WebUI uses responsive design to adapt to different screen sizes. Responsive design is primarily implemented through CSS media queries.

```css
/* Desktop devices */
@media (min-width: 1024px) {
    .app-nav {
        width: 80px;
        height: 100%;
        flex-direction: column;
        top: 60px;
        left: 0;
    }
    
    #main-content {
        margin-left: 80px;
        margin-bottom: 0;
    }
}

/* Tablet devices */
@media (min-width: 768px) and (max-width: 1023px) {
    .app-nav {
        height: 60px;
        bottom: 0;
    }
    
    #main-content {
        margin-bottom: 60px;
    }
}

/* Mobile devices */
@media (max-width: 767px) {
    .app-nav {
        height: 56px;
        bottom: 0;
    }
    
    #main-content {
        margin-bottom: 56px;
    }
}
```

## 🔧 Debugging Tips

1. **Use browser developer tools**: Press F12 to open developer tools, check console output and network requests.

2. **Add debug logs**: Add `console.log()` statements in JavaScript code to output debug information.

3. **Use the logger.js module**: Use the built-in logging module to record information.

```javascript
// Record logs
Logger.log('Information', 'info');
Logger.log('Warning', 'warning');
Logger.log('Error', 'error');
```

## 🔄 Version Compatibility

When upgrading the AMMF framework, pay attention to changes in the following files:

1. `webroot/app.js` - Main application logic may change
2. `webroot/core.js` - Core functionality may be updated
3. `webroot/i18n.js` - Language strings may be updated

It is recommended to back up customized WebUI files before upgrading, then carefully merge any changes.