<template>
  <div class="typeahead">
    <div class="input-box">
      <input
        ref="input"
        v-model="typedValue"
        :class="{'space-right': allowNew}"
        :required="required && !multiple"
        :placeholder="getPlaceholder()"
        @focus.prevent="handleInputFocus"
        @keyup.prevent="handleKeyUp"
        @keydown="handleKeyDown"
        @input="handleInput"
      />
      <div
        v-if="multiple"
        ref="multipleInput"
        class="multiple-input"
      >
        <div
          v-for="(v, idx) in value"
          :key="idx"
        >
          {{ getLabelByValue(v) }}
          <i @click.prevent="handleOptionRemove(v)">X</i>
        </div>
      </div>
      <button
        v-if="allowNew"
        class="allow-new-btn"
        :class="{disabled: !typedValue}"
        :disabled="!typedValue"
        @click="handleOptionCreate"
      >+</button>
    </div>
    <div v-if="open" class="options" ref="options">
      <div
        v-for="(option, idx) in filteredOptions"
        :key="idx"
        class="option"
        :class="{ hover: idx === hoverIdx }"
        @click.prevent="handleOptionSelect(option)"
        @mouseover="hoverIdx = idx"
      >
        {{ getLabelFromOption(option) }}
      </div>
      <div
        class="option hover"
        @mouseover="hoverIdx = -1"
        v-if="allowNew && !filteredOptions.length && typedValue"
        @click="handleOptionCreate"
      >
        Создать: <b>{{ typedValue }}</b>
      </div>
      <div
        class="option"
        @mouseover="hoverIdx = -1"
        v-if="!allowNew && !filteredOptions.length && typedValue"
      >
        Ничего не найдено
      </div>
    </div>
    <div class="backdrop" @click.prevent="handleBackdropClick" />
  </div>
</template>

<script>

export default {
  name: "Typeahead",
  props: {
    required: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String
    },
    options: {
      type: Array,
      required: true
    },
    labelKey: {
      type: String,
      default: "label"
    },
    valueKey: {
      type: String,
      default: "id"
    },
    value: {
      type: [String, Number, Object, Array],
    },
    allowNew: {
      type: Boolean,
      default: false,
    },
    multiple: {
      type: Boolean,
      default: false,
    }
  },
  data() {
    return {
      open: false,
      typedValue: null,
      filteredOptions: [],
      hoverIdx: -1,
      defaultInputElStyle: {}
    }
  },
  created() {
    if (this.multiple) {
      this.setInputStyle();
    } else {
      this.setTypedValue();
    }

    this.filteredOptions = this.getFilteredOptions();
  },
  mounted() {
    if (this.multiple) {
      this.defaultInputElStyle = {
        ...getComputedStyle(this.$refs.input)
      };
    }
  },
  updated() {
    if (this.multiple) {
      this.setInputStyle();
    }
  },
  watch: {
    typedValue() {
      this.hoverIdx = -1;
      this.filteredOptions = this.getFilteredOptions();

      if (this.typedValue === "" && !this.multiple) {
        this.$emit("input", null);
      }
    },
    value() {
      if (!this.multiple) {
        this.setTypedValue();
      }
      this.filteredOptions = this.getFilteredOptions();
    },
    options() {
      this.filteredOptions = this.getFilteredOptions();
    }
  },
  methods: {
    select(option) {
      if (this.multiple) {
        this.$emit("input", [
          ...this.value,
          this.getValueFromOption(option)
        ]);
        this.typedValue = "";
      } else {
        this.$emit("input", this.getValueFromOption(option));
        this.typedValue = this.getLabelFromOption(option);
      }

      this.hoverIdx = -1;
    },
    setInputStyle() {
      const multipleInputEl = this.$refs.multipleInput;
      const inputEl = this.$refs.input;
      const optionsEl = this.$refs.options;
      if (multipleInputEl && inputEl) {
        inputEl.style.height = multipleInputEl.scrollHeight + "px";

        const {
          x: inputX,
          y: inputY,
          height: inputHeight
        } = inputEl.getBoundingClientRect();

        let inputPaddingLeft = this.defaultInputElStyle.paddingLeft;
        let inputPaddingTop = this.defaultInputElStyle.paddingTop;

        const { lastChild } = multipleInputEl;
        if (lastChild) {
          const {
            x: lastChildX, width: lastChildWidth,
            y: lastChildY
          } = lastChild.getBoundingClientRect();

          inputPaddingLeft = `${lastChildX + lastChildWidth - inputX}px`;
          inputPaddingTop = `${lastChildY - inputY}px`;
        }

        inputEl.style.paddingLeft = inputPaddingLeft;
        inputEl.style.paddingTop = inputPaddingTop;

        if (optionsEl && this.open) {
          optionsEl.style.top = `${inputHeight}px`;
        }
      }
    },
    setTypedValue() {
      const selectedOption = this.options.find((option) =>
        this.getValueFromOption(option) === this.value
      );

      if (!selectedOption) {
        return;
      }

      this.typedValue = this.getLabelFromOption(selectedOption);
      this.hoverIdx = -1;
    },
    getLabelFromOption(option) {
      return (["string", "number"].includes(typeof option))
        ? option
        : option[this.labelKey];
    },
    getValueFromOption(option) {
      return (["string", "number"].includes(typeof option))
        ? option
        : option[this.valueKey];
    },
    getLabelByValue(value) {
      return this.getLabelFromOption(
        this.options.find(option => this.getValueFromOption(option) === value)
      );
    },
    getFilteredOptions() {
      return this.options
        .filter((option) => !this.typedValue ||
          this
            .getLabelFromOption(option)
            .toLowerCase()
            .includes(this.typedValue.toLowerCase())
        )
        .filter((option) => !this.multiple ||
          !this.value.includes(this.getValueFromOption(option))
        )
    },
    getPlaceholder() {
      return this.multiple && Array.isArray(this.value) && this.value.length > 0
        ? undefined
        : this.placeholder;
    },
    handleOptionCreate() {
      this.$emit('create', this.typedValue)
    },
    handleInputFocus() {
      this.open = true;
    },
    handleBackdropClick() {
      this.open = false;
    },
    handleOptionSelect(option) {
      this.select(option);
      this.open = false;
    },
    handleOptionRemove(removableValue) {
      if (!this.multiple) {
        return;
      }

      this.$emit("input", this.value.filter(
        (v) => removableValue !== v)
      );
    },
    handleInput() {
      this.open = true;
    },
    handleKeyDown(event) {
      if (["Tab", "Enter"].includes(event.code)) {
        this.open = false;
      }
    },
    handleKeyUp(event) {
      if (["ArrowDown", "ArrowUp"].includes(event.code)) {
        const direction = event.code === "ArrowDown" ? 1 : -1;

        if (this.hoverIdx === -1) {
          this.hoverIdx = 0;
        } else {
          this.hoverIdx += direction;
        }

        const totalOptions = this.filteredOptions.length;

        this.hoverIdx = Math.min(Math.max(this.hoverIdx, 0), totalOptions - 1);

        if (!this.$refs.options) {
          return;
        }

        const { firstChild, scrollTop } = this.$refs.options;
        if (firstChild &&
          (
            direction > 0 && this.hoverIdx > 2 ||
            direction < 0 && this.hoverIdx < totalOptions - 2
          )
        ) {
          this.$refs.options.scrollTo(0, scrollTop + direction * firstChild.scrollHeight);
        }
      }
      if (["Enter"].includes(event.code)) {
        event.stopPropagation();
        const selectedOption = this.filteredOptions.find((_, i) => i === this.hoverIdx);
        if (selectedOption) {
          this.select(selectedOption);
        } else if (!this.filteredOptions.length && this.typedValue) {
          this.$emit('create', this.typedValue)
        }
      }
    },
  },
};
</script>

<style lang="less" scoped>
.typeahead {
  display: flex;
  flex-direction: column;
  position: relative;
  .input-box {
    display: flex;
    position: relative;
    input {
      height: 33px;
    }
    input ~ .allow-new-btn {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 30px;
      height: 30px;
      align-self: center;
      z-index: 1031;
    }
    .multiple-input {
      min-height: 33px;
      display: flex;
      justify-content: flex-start;
      flex-wrap: wrap;
      position: absolute;
      width: 100%;
      align-items: center;
      padding: 3px 10px;
      gap: 3px;
      & > div {
        max-width: 200px;
        padding-right: 20px;
        position: relative;
        i.fa-close {
          position: absolute;
          right: 10px;
          cursor: pointer;
        }
      }
    }
  }
  .options {
    width: 100%;
    background-color: white;
    position: absolute;
    top: 33px;
    border-bottom-left-radius: 4px;
    border-bottom-right-radius: 4px;
    max-height: 156px;
    overflow-x: hidden;
    overflow-y: auto;
    z-index: 1031;
    .option {
      padding: 5px;
      cursor: pointer;
      &:not(:last-child) {
        border-bottom: 1px solid gray;
      }
      &:hover, &.hover {
        background-color: lightgray;
      }
    }
  }
  .options + .backdrop {
    position: fixed;
    opacity: 0;
    top: 0;
    left: 0;
    z-index: 1030;
    background-color: red;
    width: 100%;
    height: 100%;
  }
}
</style>
