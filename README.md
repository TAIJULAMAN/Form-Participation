# 🧩 Form Participation API (Function-Based Guide)

This repository demonstrates how to use the [Form Participation API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/formdata_event) with **function-based JavaScript patterns**, helping you build dynamic and modern form experiences with or without custom elements.

---

## 📖 Table of Contents

- [🔍 Overview](#-overview)
- [1️⃣ Event-Based Participation (No Custom Elements)](#1️⃣-event-based-participation-no-custom-elements)
  - [💡 Example: Inject Hidden Fields on Submit](#💡-example-inject-hidden-fields-on-submit)
- [2️⃣ Form-Associated Custom Elements](#2️⃣-form-associated-custom-elements)
  - [💡 Example: Functional Custom Input Element](#💡-example-functional-custom-input-element)
  - [💡 Example: Submit Multiple Fields](#💡-example-submit-multiple-fields)
- [🧪 Feature Detection](#-feature-detection)
- [⚖️ When to Use Which API](#️-when-to-use-which-api)
- [✅ Summary](#-summary)
- [📚 Resources](#-resources)
- [📄 License](#-license)

---

## 🔍 Overview

The **Form Participation API** allows custom elements and arbitrary JS logic to integrate seamlessly with `<form>` submission, validation, reset, and autofill.

It includes two approaches:

1. **Event-Based Participation**: Dynamically modify form data using the `formdata` event.
2. **Form-Associated Custom Elements**: Build custom form controls that act like native inputs.

---

## 1️⃣ Event-Based Participation (No Custom Elements)

This API lets **any JS object** listen for form submission and dynamically add values using the `formdata` event.

### ✅ Great for:
- Adding user/session data without modifying the DOM.
- Dynamic or app-driven form enhancements.

---

### 💡 Example: Inject Hidden Fields on Submit

```html
<form id="order-form" method="POST" action="/submit">
  <input type="text" name="email" placeholder="Email">
  <button type="submit">Submit Order</button>
</form>

<script>
  function addHiddenFieldsOnSubmit(form, fields) {
    form.addEventListener('formdata', (event) => {
      for (const [key, value] of Object.entries(fields)) {
        event.formData.append(key, value);
      }
    });
  }

  const form = document.getElementById('order-form');
  const dynamicFields = {
    'customer-id': 'cust-00123',
    'affiliate-code': 'AFF-2025',
  };

  addHiddenFieldsOnSubmit(form, dynamicFields);
</script>
```
📨 Resulting Submission:
email=test@example.com
customer-id=cust-00123
affiliate-code=AFF-2025
2️⃣ Form-Associated Custom Elements
You can build fully functional custom elements that integrate with forms using ElementInternals.
⚠️ Note: A class is required by the Web Components API, but the logic inside can be 100% functional.

✅ Great for:
Reusable form inputs like <my-input>, <date-picker>, <credit-card-form>

Full integration with <form>, <label>, and browser validation
💡 Example: Functional Custom Input Element
```
<form id="signup-form" method="POST" action="/signup">
  <label>
    Name:
    <my-input name="user-name"></my-input>
  </label>
  <button type="submit">Sign Up</button>
</form>

<script>
  function setupCustomInput(shadowRoot, internals, onValueChange) {
    const input = document.createElement('input');
    input.type = 'text';

    input.addEventListener('input', () => {
      onValueChange(input.value);
    });

    shadowRoot.appendChild(input);
    return input;
  }

  function createMyInputElement() {
    return class extends HTMLElement {
      static formAssociated = true;

      constructor() {
        super();
        this._internals = this.attachInternals();
        const shadow = this.attachShadow({ mode: 'open' });

        this._input = setupCustomInput(shadow, this._internals, (value) => {
          this._value = value;
          this._internals.setFormValue(value);
        });
      }

      get value() {
        return this._input.value;
      }

      set value(val) {
        this._input.value = val;
        this._internals.setFormValue(val);
      }

      formResetCallback() {
        this.value = '';
      }

      formDisabledCallback(disabled) {
        this._input.disabled = disabled;
      }

      formAssociatedCallback(form) {
        console.log("Attached to form:", form);
      }

      formStateRestoreCallback(state) {
        this.value = state || '';
      }
    };
  }

  customElements.define('my-input', createMyInputElement());
</script>
```
💡 Example: Submit Multiple Fields
You can also submit multiple key-value pairs using a single custom element:
```
function submitMultipleFields(internals, name, values) {
  const data = new FormData();
  for (const [key, val] of Object.entries(values)) {
    data.append(`${name}-${key}`, val);
  }
  internals.setFormValue(data);
}
```
🧪 Feature Detection
```
if ('FormDataEvent' in window) {
  console.log("✅ formdata event supported");
}

if ('setFormValue' in ElementInternals.prototype) {
  console.log("✅ Form-associated custom elements supported");
}
```
⚖️ When to Use Which API
Scenario	formdata Event	Form-Associated Element
Inject dynamic hidden data	✅	❌
Custom UI input widget	❌	✅
Label, Fieldset, Reset support	❌	✅
Native validation integration	❌	✅
No DOM manipulation	✅	❌

✅ Summary
🧠 Use formdata when you want to inject values programmatically.

🧱 Use Form-Associated Custom Elements when you’re building new form controls.

Both work beautifully together — you can even use both in the same form.
📚 Resources
MDN: formdata event

Spec: Form-Associated Custom Elements

WICG Proposal Discussion
