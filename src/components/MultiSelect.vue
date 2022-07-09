<template>
  <div id="dropdown">
    <select v-show="false" multiple class="bordered">
      <option v-for="(option, idx) in selectedOptions" :key="idx">
        {{ option }}
      </option>
    </select>
    <div class="relative text-left">
      <div class="rounded bordered border-grey hover:border-blue">
        <ul class="items-center flex flex-wrap list-reset pb-1 pr-1 m-0">
          <li
            v-for="(option, idx) in selectedOptions"
            :key="idx"
            class="mt-1 ml-1 mb-0 flex items-center"
          >
            <slot v-bind="{ option, idx }" :remove="removeOption">
              <span class="pill">
                <span class="text-sm">{{ option }}</span>
                <span
                  class="pl-2 text-grey-darker mt-px"
                  @click.stop="removeOption(idx)"
                >
                  <svg
                    class="text-sm w-3 h-3 fill-current"
                    aria-hidden="true"
                    viewBox="0 0 512 512"
                  >
                    <path
                      d="M256 8C119 8 8 119 8 256s111 248 248 248 248-111 248-248S393 8 256 8zm121.6 313.1c4.7 4.7 4.7 12.3 0 17L338 377.6c-4.7 4.7-12.3 4.7-17 0L256 312l-65.1 65.6c-4.7 4.7-12.3 4.7-17 0L134.4 338c-4.7-4.7-4.7-12.3 0-17l65.6-65-65.6-65.1c-4.7-4.7-4.7-12.3 0-17l39.6-39.6c4.7-4.7 12.3-4.7 17 0l65 65.7 65.1-65.6c4.7-4.7 12.3-4.7 17 0l39.6 39.6c4.7 4.7 4.7 12.3 0 17L312 256l65.6 65.1z"
                    ></path>
                  </svg>
                </span>
              </span>
            </slot>
          </li>
          <li class="flex mt-1 ml-1 mb-0 justify-between flex-1 items-center">
            <input
              type="text"
              class="input box-size p-1 inline mr-1 w-full h-full outline-none border-none leading-tight"
              @click="searchText = ''"
              @keyup.enter="setPossibleOption($event)"
              @keyup.delete="popSelectedOption"
              :placeholder="placeholder"
              v-model="searchText"
            />
            <span
              class="cursor-pointer p-1 text-left flex"
              @click="removeAllOptions"
            >
              <svg
                class="text-sm w-3 h-3 fill-current text-grey-darker"
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                viewBox="0 0 24 24"
              >
                <path
                  d="M24 20.188l-8.315-8.209 8.2-8.282-3.697-3.697-8.212 8.318-8.31-8.203-3.666 3.666 8.321 8.24-8.206 8.313 3.666 3.666 8.237-8.318 8.285 8.203z"
                />
              </svg>
            </span>
          </li>
        </ul>
      </div>

      <ul
        v-show="matchingOptions"
        class="dropdown absolute w-full overflow-auto mt-px text-left list-reset"
      >
        <li
          v-for="(option, idx) in matchingOptions"
          :key="idx"
          class="cursor-pointer outline-none"
          @click.prevent="setOption(option)"
        >
          <slot v-bind="{ option, idx }">
            {{ option }}
          </slot>
        </li>
      </ul>
    </div>
    <h3>Your skills are:</h3>
    <p v-for="(option, index) in selectedOptions" :key="index">
      {{ index + 1 + "." }} {{ option }}
    </p>
  </div>
</template>
<script>
export default {
  created() {
    this.mutableOptions = this.options;
  },
  mounted() {
    document.addEventListener("click", this.handleClickOutside);
  },
  data() {
    return {
      mutableOptions: [],
      selectedOptions: [],
      searchText: null,
      dropdownOpen: false,
    };
  },
  props: {
    taggable: {
      type: Boolean,
      default: () => false,
    },
    options: {
      type: Array,
      default: () => [],
    },
    placeholder: {
      type: String,
      default: () => "Search Here",
    },
  },
  watch: {
    searchText(curr, prev) {
      if (curr === prev) {
        return;
      }
      this.pointer = -1;
    },
  },
  methods: {
    handleClickOutside(e) {
      if (this.$el.contains(e.target)) {
        return;
      }
      this.searchText = null;
    },
    createOption(option) {
      this.$emit("option-created", option);
      return option;
    },
    popSelectedOption() {
      if (this.searchText === null) {
        this.selectedOptions.pop();
        return;
      }
      if (this.searchText === "") {
        this.searchText = null;
      }
    },
    findExistingOption(option, givenOptions) {
      return givenOptions.find((opt) => opt === option);
    },
    addOption(value) {
      if (value instanceof Event) {
        value = value.target.value;
      }
      let opt = this.findExistingOption(value, this.mutableOptions.slice(0));
      if (typeof opt === "undefined") {
        this.setOption(this.createOption(value));
        return;
      }
      this.setOption(opt);
    },
    setPossibleOption(value) {
      if (this.taggable && this.pointer === -1) {
        this.addOption(value);
        return;
      }
      this.setOption(this.matchingOptions[this.pointer]);
    },
    removeAllOptions() {
      this.selectedOptions = [];
    },
    setOption(option) {
      this.selectedOptions.push(option);
      // this.searchText = null;
    },
    removeOption(idx) {
      this.selectedOptions.splice(idx, 1);
    },
  },
  computed: {
    matchingOptions() {
      if (this.searchText === null) {
        return null;
      }
      return this.mutableOptions
        .filter(
          (option) =>
            this.selectedOptions.findIndex((selected) => selected === option) <
            0
        )
        .filter((option) =>
          option
            .toString()
            .toLowerCase()
            .includes(this.searchText.toString().toLowerCase())
        );
    },
  },
};
</script>
<style scoped>
.list-reset {
  list-style: none;
  padding: 0;
}
.overflow-auto {
  overflow: auto;
}
.text-grey-darker {
  color: #606f7b;
}
.leading-tight {
  line-height: 1.5;
}
.text-sm {
  font-size: 0.875em;
}
.w-full {
  width: 100%;
}
.inline {
  display: inline;
}
.flex {
  display: flex;
}
.flex-1 {
  flex: 1;
}
.flex-wrap {
  flex-wrap: wrap;
}
.justify-between {
  justify-content: space-between;
}
.bordered {
  border-width: 1px;
  border-style: solid;
}
.border-none {
  border: none;
}
.hover\:border-blue:hover {
  border-color: #3490dc;
}
.border-grey {
  border-color: #b8c2cc;
}
.absolute {
  position: absolute;
}
.relative {
  position: relative;
}
.items-center {
  align-items: center;
}
.p-1 {
  padding: 0.25em;
}
.pb-1 {
  padding-bottom: 0.25em;
}
.pl-2 {
  padding-left: 0.5em;
}
.pr-1 {
  padding-right: 0.25em;
}
.m-0 {
  margin: 0;
  margin-bottom: 0 !important;
}
.mt-1 {
  margin-top: 0.25em !important;
}
.mr-1 {
  margin-right: 0.25em !important;
}
.ml-1 {
  margin-left: 0.25em;
}
.mt-px {
  margin-top: 1px;
}
.mb-0 {
  margin-bottom: 0;
}
.text-left {
  text-align: left;
}
.w-full {
  width: 100%;
}
.w-3 {
  width: 0.75em;
}
.h-3 {
  height: 0.75em;
}
.fill-current {
  fill: currentColor;
}
.outline-none {
  outline: 0;
}
.rounded {
  border-radius: 0.25em;
}
.cursor-pointer {
  cursor: pointer;
}
.dropdown {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.12), 0 2px 4px 0 rgba(0, 0, 0, 0.08);
  background-color: #fff;
  color: #606f7b;
  border-radius: 0.25em;
  line-height: 1.5;
  text-align: left;
  display: inline;
  width: 99.8%;
}
.dropdown > li {
  padding: 0.5em 0.75em;
}
.input {
  padding: 0.5em;
  overflow: visible;
}
.box-size {
  box-sizing: border-box;
}
.pill {
  border-width: 1px;
  border-style: solid;
  border-color: #42b883;
  background-color: #42b883;
  color: #fff;
  cursor: pointer;
  border-radius: 0.25em;
  line-height: 1.5;
  padding: 0.15em 0.5em;
  letter-spacing: -0.05em;
}
</style>
