# CXone Guide Chat – Language Switching Implementation Guide

This guide outlines how to implement EN/ES language switching for a CXone Guide-powered chat widget. It covers the custom contact field used for Studio-side routing, the widget locale, and the Guide initialization locale.

---

## Overview

Language switching requires three coordinated actions triggered on the language toggle click event:

| Action | Method | Purpose |
|---|---|---|
| Set routing variable | `setContactCustomField` | Signals CXone Studio which bot/flow to route to |
| Set widget locale | `setLocale` | Translates the chat widget UI strings |
| Reinitialize Guide | `guide init` with locale | Ensures Guide rules and proactive engagement respect the selected language |

> **Important:** Because `setContactCustomField` is read at the time a contact is initiated in Studio, a **page refresh is required** after setting this value for it to take effect on the next chat session.

---

## 1. Custom Contact Field – Routing Variable

This field is invisible to the customer and is used on the CXone Studio side to route the contact to the correct language bot.

```javascript
// Set to English
cxone("chat", "setContactCustomField", "new_lang", "en-US");

// Set to Spanish
cxone("chat", "setContactCustomField", "new_lang", "es-ES");
```

This field must be set **before** the customer initiates a chat. Because the value is captured at contact start, a page reload is needed after switching languages to ensure the updated value is picked up by Studio.

---

## 2. Widget Locale

This controls the display language of the CXone chat widget UI (e.g., placeholder text, button labels, system messages).

```javascript
// Set to Spanish
cxone('chat', 'setLocale', 'es-ES');

// Set to English
cxone('chat', 'setLocale', 'en-US');
```

---

## 3. Guide Initialization Locale

The Guide `init` call should include the `locale` option so that Guide rules, proactive triggers, and engagement content are evaluated in the correct language context.

```javascript
// Spanish
cxone('guide', 'init', { locale: 'es-ES' });

// English
cxone('guide', 'init', { locale: 'en-US' });
```

If Guide has already been initialized on the page, you may need to reinitialize it with the new locale when the language is switched.

---

## 4. Language Toggle Implementation

Attach all three actions to the language dropdown or toggle click event. A page reload is required to flush the routing variable into the next contact.

```javascript
// Example: triggered by your EN/ES dropdown click handler

function switchLanguage(lang) {
  const locale = lang === 'es' ? 'es-ES' : 'en-US';

  // 1. Update Studio routing variable
  cxone("chat", "setContactCustomField", "new_lang", locale);

  // 2. Update widget display locale
  cxone('chat', 'setLocale', locale);

  // 3. Reinitialize Guide with the new locale
  cxone('guide', 'init', { locale: locale });

  // 4. Reload the page so the routing variable takes effect on next chat session
  window.location.reload();
}

// Wire up to your toggle
document.querySelector('#language-toggle').addEventListener('click', function () {
  const currentLang = document.documentElement.lang || 'en';
  const newLang = currentLang === 'en' ? 'es' : 'en';
  switchLanguage(newLang);
});
```

> **Note:** Adjust the selector (`#language-toggle`) and language detection logic to match your site's existing toggle implementation.

---

## 5. Initialization on Page Load

On initial page load, set all three based on the page's current language context **before** the chat widget is used:

```javascript
// Determine locale from page context (e.g., URL path, HTML lang attribute, cookie)
const pageLocale = document.documentElement.lang === 'es' ? 'es-ES' : 'en-US';

cxone('guide', 'init', { locale: pageLocale });  // 1. Bootstrap Guide with locale first
cxone('chat', 'setLocale', pageLocale);           // 2. Set widget UI locale
cxone("chat", "setContactCustomField", "new_lang", pageLocale); // 3. Set Studio routing variable
```

---

## 6. Scope & Applicability

This logic must be applied to **all chat-enabled channels** that use a bilingual bot, including but not limited to:

- Ocean Breeze Hotels (`oceanbreezehotels.com/en` / `.../es`)
- Elegant – VidantaWorld (`elegant.vidantaworld.com/en` / `.../es`)
- Any additional channels currently in development with EN/ES bot variants

---

## Summary Checklist

- [ ] `setContactCustomField` — `new_lang` set to correct locale value on page load
- [ ] `setContactCustomField` — updated on language toggle click
- [ ] `setLocale` — updated on language toggle click
- [ ] `guide init` — locale passed on initialization and re-init on toggle
- [ ] Page refresh triggered after language toggle
- [ ] Logic applied to all relevant chat channels
