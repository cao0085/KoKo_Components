<template>
  <div>
    <v-row dense>
      <template v-for="(char, i) in chars">
        <v-col
          v-if="showDash && i === dashIndex"
          :key="'dash-' + i"
          cols="auto"
          class="d-flex align-center"
        >
          <span class="dash" :style="dashStyle">-</span>
        </v-col>
        <v-col :key="i" cols="auto">
          <input
            class="inv-char"
            :class="{ 'error-flash': isFlashingError }"
            :style="charStyle"
            v-model="chars[i]"
            :ref="'char' + i"
            maxlength="1"
            autocomplete="off"
            @input="onInput(i)"
            @keydown.backspace.prevent="onBackspace(i)"
          />
        </v-col>
      </template>
    </v-row>
    <v-messages v-if="errorMessage" :value="[errorMessage]" color="red" />
  </div>
</template>

<script>
export default {
  name: 'SegmentedInput',
  props: {
    type: {
      type: String,
      default: 'invoice', // or 'number'
      validator: val => ['invoice', 'number'].includes(val)
    },
    scale: { type: Number, default: 1 },
    length: { type: Number, default: 4 },
    value: { type: String, default: '' },
    message: {
      type: String,
      default: 'All fields required'
    }
  },
  data() {
    return {
      chars: [],
      errorMessage: '',
      isFlashingError: false
    };
  },
  computed: {
    inputLength() {
      return this.type === 'invoice' ? 10 : this.length;
    },
    showDash() {
      return this.type === 'invoice';
    },
    dashIndex() {
      return 2;
    },
    charStyle() {
      const w = 42 * this.scale;
      const h = 56 * this.scale;
      const fs = 24 * this.scale;
      return {
        width: `${w}px`,
        height: `${h}px`,
        fontSize: `${fs}px`,
        lineHeight: `${h}px`,
        borderRadius: `${6 * this.scale}px`,
      };
    },
    dashStyle() {
      const h = 56 * this.scale;
      const fs = 28 * this.scale;
      return {
        fontSize: `${fs}px`,
        lineHeight: `${h}px`,
        padding: `0 ${4 * this.scale}px`,
      };
    },
    hasError() {
      return !!this.errorMessage;
    }
  },
  mounted() {
    this.chars = Array(this.inputLength).fill('');
    if (this.$parent && typeof this.$parent.register === 'function') {
      this.$parent.register(this);
    }
  },
  beforeDestroy() {
    if (this.$parent && typeof this.$parent.unregister === 'function') {
      this.$parent.unregister(this);
    }
  },
  methods: {
    onInput(idx) {
      const raw = this.chars[idx];
      const val = raw.toUpperCase();
      let isValid;

      if (this.type === 'invoice') {
        const isLetterArea = idx < 2;
        isValid = isLetterArea ? /^[A-Z]$/.test(val) : /^[0-9]$/.test(val);
      } else {
        isValid = /^[0-9]$/.test(val);
      }

      if (!isValid) {
        this.$set(this.chars, idx, '');
        return;
      }

      this.$set(this.chars, idx, val);

      this.$emit('input', this.getValue());

      if (idx < this.inputLength - 1) {
        this.$refs['char' + (idx + 1)][0].focus();
      }
    },
    onBackspace(idx) {
      if (idx > 0) {
        this.$set(this.chars, idx, '');
        this.$nextTick(() => {
          this.$refs['char' + (idx - 1)][0].focus();
        });
      } else {
        this.$set(this.chars, idx, '');
      }
      this.$emit('input', this.getValue()); 
    },
    getValue() {
      if (this.type === 'invoice') {
        return (
          this.chars.slice(0, 2).join('') + '-' + this.chars.slice(2).join('')
        );
      } else {
        return this.chars.join('');
      }
    },
    validate() {
      const isComplete = this.chars.every(char => char.length === 1);
      if (!isComplete) {
        this.errorMessage = this.message || ''; // 空字串不顯示
        this.triggerErrorFlash();
      } else {
        this.errorMessage = '';
      }
      return isComplete;
    },
    resetValidation() {
      this.errorMessage = '';
    },
    triggerErrorFlash() {
      this.isFlashingError = false;
      this.$nextTick(() => {
      const firstInput = this.$refs['char0']?.[0];
      if (firstInput) void firstInput.offsetWidth;
        this.isFlashingError = true;
      });
    }
  },
  watch: {
    value: {
      immediate: true,
      handler(val) {
        if (typeof val === 'string') {
          const pure = val.replace('-', '').toUpperCase();
          this.chars = Array(this.inputLength)
           .fill('')
            .map((_, i) => pure[i] || '');
        }
      }
    }
  },
};
</script>

<style scoped>
@keyframes flash-border {
  0% {
    border-color: #ff5252;
    box-shadow: 0 0 0px #ff5252;
  }
  50% {
    border-color: #ffbaba;
    box-shadow: 0 0 1px #ff5252;
  }
  100% {
    border-color: #ff5252;
    box-shadow: 0 0 0px #ff5252;
  }
}

.error-flash {
  animation: flash-border 0.3s ease-in-out;
}
.inv-char {
  font-weight: 600;
  text-align: center;
  background: #f5f5f7;
  border: 1px solid #d0d0d0;
  outline: none;
  caret-color: transparent;
  -moz-appearance: textfield;
}
.inv-char::-webkit-outer-spin-button,
.inv-char::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
.dash {
  font-weight: 700;
  color: #555;
}
</style>
