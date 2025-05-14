
# 📦 SegmentedInput Component

A custom Vue 2 component for segmented, character-by-character input. Useful for input formats that require strict structure and validation:

- `invoice`: Format `AA-12345678` (two uppercase letters followed by 8 digits)
- `number`: Digits only, with customizable length

## ✅ Use Cases

- Invoice number entry
- OTP (One-Time Password) fields
- Custom structured codes

---

## ⚙️ Props

| Name      | Type     | Default             | Description                                                   |
|-----------|----------|---------------------|---------------------------------------------------------------|
| `type`    | String   | `'invoice'`         | Input type: `'invoice'` or `'number'`                         |
| `scale`   | Number   | `1`                 | Scale factor for input size and font                          |
| `length`  | Number   | `4`                 | Only applies if `type="number"`                               |
| `value`   | String   | `''`                | Supports `v-model` for two-way binding                        |
| `message` | String   | `'All fields required'` | Error message to display. If empty (`''`), message is hidden |

---

## 🧪 Methods (via `ref`)

| Method             | Returns   | Description                                                  |
|--------------------|-----------|--------------------------------------------------------------|
| `getValue()`       | `String`  | Returns full input value (e.g. `'AB-12345678'`)              |
| `validate()`       | `Boolean` | Validates if all characters are filled. Triggers error flash |
| `resetValidation()`| `void`    | Clears error state                                           |

---

## ✍️ Basic Example

```vue
<template>
  <div>
    <SegmentedInput
      v-model="invoice"
      type="invoice"
    />
    <v-btn @click="submit">Submit</v-btn>
  </div>
</template>

<script>
import SegmentedInput from '@/components/SegmentedInput.vue';

export default {
  components: { SegmentedInput },
  data() {
    return {
      invoice: ''
    };
  },
  methods: {
    submit() {
      const isValid = this.$refs.invoiceInput.validate();
      if (isValid) {
        alert('Submitted: ' + this.invoice);
      }
    }
  }
};
</script>


⸻

📋 With Vuetify v-form

<template>
  <v-form ref="form">
    <v-text-field
      v-model="name"
      label="Name"
      :rules="[v => !!v || 'Name is required']"
      required
    />
    <v-select
      v-model="option"
      :items="['A', 'B', 'C']"
      label="Select Option"
      :rules="[v => !!v || 'Selection required']"
      required
    />
    <SegmentedInput
      v-model="invoice"
      type="invoice"
      message="Please enter a valid invoice"
    />
    <SegmentedInput
      ref="codeInput"
      type="number"
      :length="4"
      scale="0.8"
    />
    <v-btn color="primary" @click="submit">Submit</v-btn>
  </v-form>
</template>

<script>
import SegmentedInput from '@/components/SegmentedInput.vue';

export default {
  components: { SegmentedInput },
  data() {
    return {
      name: '',
      option: '',
      invoice: ''
    };
  },
  methods: {
    submit() {
      const formValid = this.$refs.form.validate();
      const invoiceValid = this.$refs.codeInput.validate();

      if (formValid && invoiceValid) {
        alert('Form submitted!');
      }
    }
  }
};
</script>


⸻

📦 Dependencies

This component requires:
	•	Vue 2.x
	•	Vuetify 2.x (v-row, v-col, v-btn, v-form, etc.)

Make sure Vuetify is properly installed and set up in your project.

⸻

🌟 Features
	•	Auto-uppercase and auto-tab for each character
	•	Format validation per character
	•	Visual error flash animation
	•	Optional error message display
	•	v-model and ref support for full control

⸻

📁 Suggested File Structure

Place the component at:

src/components/SegmentedInput.vue


⸻