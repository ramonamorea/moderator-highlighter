# moderator-highlighter

A cross-browser solution that improves readability and navigation in Discourse-powered forums by automatically highlighting posts from moderators and staff members with a distinctive dark red background and white text.

## Installation 

### Step 1: Install Tampermonkey
Install the Tampermonkey extension for your browser:
- [Chrome](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)
- [Edge](https://microsoftedge.microsoft.com/addons/detail/tampermonkey/iikmkjmpaadaobahmlepeloendndfphd)
- [Safari](https://apps.apple.com/us/app/tampermonkey/id1482490089)
- [Opera](https://addons.opera.com/en/extensions/details/tampermonkey-beta/)

### Step 2: Install the Userscript
1. Click on the Tampermonkey icon in your browser toolbar
2. Select "Create a new script"
3. Delete any placeholder code
4. Copy and paste the following script below (the header too), then Save the script (File → Save or Ctrl+S / Cmd+S)

```javascript
// ==UserScript==
// @name         Moderator Highlighter
// @namespace    http://tampermonkey.net/
// @version      2.0
// @description  Highlights staff, admin, and moderator posts on Discourse forums with a dark red background and white text.
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    function isDiscourseSite() {
        const metaGenerator = document.querySelector('meta[name="generator"]');
        return !!(
            metaGenerator &&
            metaGenerator.content &&
            metaGenerator.content.toLowerCase().includes('discourse')
        );
    }

    if (!isDiscourseSite()) {
        return;
    }

    const highlightClass = 'custom-staff-highlight';

    // Staff/admin/moderator markers
    const staffBadgeSelector =
        '.category-moderator, .moderator, .staff, .admin, .group-staff';

    // Restrict search to likely author/header areas to avoid false positives
    const authorAreaSelectors = [
        '.topic-meta-data',
        '.names',
        '.post-info',
        '.creator',
        '.topic-post .topic-meta-data',
        '.topic-body .topic-meta-data'
    ];

    function injectStyles() {
        if (document.getElementById('custom-staff-highlight-styles')) return;

        const style = document.createElement('style');
        style.id = 'custom-staff-highlight-styles';
        style.textContent = `
            .${highlightClass} {
                background-color: #451a1e !important;
                border: 1px solid #6b272d !important;
                border-radius: 5px !important;
                color: #ffffff !important;
            }

            .${highlightClass},
            .${highlightClass} * {
                color: #ffffff !important;
            }

            .${highlightClass} blockquote,
            .${highlightClass} aside.quote,
            .${highlightClass} .quote,
            .${highlightClass} .blockquote,
            .${highlightClass} .quote-controls,
            .${highlightClass} .title {
                background-color: #5a2228 !important;
                color: #ffffff !important;
                border-color: #6b272d !important;
            }

            .${highlightClass} a {
                color: #ffffff !important;
                text-decoration: underline;
            }
        `;
        document.head.appendChild(style);
    }

    function isStaffPost(post) {
        for (const areaSelector of authorAreaSelectors) {
            const area = post.querySelector(areaSelector);
            if (area && area.querySelector(staffBadgeSelector)) {
                return true;
            }
        }
        return false;
    }

    function colorizeStaffPosts() {
        const posts = document.querySelectorAll('article');

        posts.forEach(post => {
            if (post.dataset.staffHighlighted === 'true') return;

            if (isStaffPost(post)) {
                const postBody = post.querySelector('.topic-body');
                const target = postBody || post;

                target.classList.add(highlightClass);
                post.dataset.staffHighlighted = 'true';
            }
        });
    }

    injectStyles();
    colorizeStaffPosts();

    const observer = new MutationObserver(() => {
        colorizeStaffPosts();
    });

    observer.observe(document.body, { childList: true, subtree: true });
})();
