<template>
  <div class="vue-tag-suggest">

    <div class="wrap-input">

      <slot name="searchInput">

        <ul class="wrap-seleted-items">
          <li class="seleted-item" v-for="(item, idx) in items" v-bind:key="idx">
            <span class="">{{ item }}</span>
            <div class="wrap-btn">
              <div class="closeBtn" @click="onDeleteAtion(idx)"><span></span></div>
            </div>
          </li>
        </ul>

        <input
          class="search-input"
          v-model="inputData"
          @focus="onFocus"
          @input="onInput"
          @blur="onBlur"
        >
      </slot>

      <div class="wrap-suggest" v-if="(!!showSuggestion && suggestions.length > 0)">
        <slot name="suggest" :suggestions="suggestions" :value="inputData">
          <ul class="suggestions">
            <li
              class="item"
              :class="[{
                selected: isSelected(item),
                hover: isHovered(item)
              }]"
              @mouseover="onMouseOver(item)"
              @mouseleave="onMouseLeave(item)"
              @mousedown="onSelectedAction(item)"
              v-for="(item, idx) in suggestions"
              v-bind:key="idx"
              >
                <span>{{ item }}</span>
            </li>
          </ul>
        </slot>
      </div>

      <div class="wrap-suggest" v-if="(!!showSuggestion && fuzzySuggestions.length > 0)">
        <slot name="fuzzySuggest" :suggestions="fuzzySuggestions" :value="inputData">
          <span>Looking for this?</span>
          <ul class="suggestions"
          >
            <li
              class="item"
              :class="[{
                selected: isSelected(item),
                hover: isHovered(item)
              }]"
              @mouseover="onMouseOver(item)"
              @mouseleave="onMouseLeave(item)"
              @mousedown="onSelectedAction(item)"
              v-for="(item, idx) in fuzzySuggestions"
              v-bind:key="idx"
              >
              <span>{{ item }}</span>
            </li>
          </ul>
        </slot>
      </div>
    </div>

  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'vue-tag-suggest',
  model: {
    prop: 'value',
    event: 'input'
  },
  props: {
    value: {},
    suggestList: {
      type: [Function],
      default: () => []
    },
    fuzzySuggestList: {
      type: [Function],
      default: () => []
    },
  },
  computed: {
    inputData: {
      get () {
        return this.value
      },
      set (value) {
         this.$emit('input', value)
      }
    }
  },
  data () {
    return {
      text: this.value,
      suggestions: [],
      fuzzySuggestions: [],
      showSuggestion: false,
      hoverItem: null,
      items: [],
    }
  },
  methods: {
    onInput: _.debounce(function () {
      this.callGetSuggestionFunction()
    }, 500),
    onFocus() {
      this.showSuggestion = true
    },
    onBlur() {
      this.showSuggestion = false
    },
    onMouseOver(item) {
      this.hoverItem = item
    },
    onMouseLeave() {
      this.hoverItem = null
    },
    isSelected(item) {
      return this.hoverItem == item
    },
    onSelectedAction(item) {
      var items = [item];
      items = items.filter(t => t.trim().length > 0);

      var dup = this.items.find(t => t === item);
      if (dup) return

      items.forEach(item => {
        this.items.push(item)
      });
      this.showSuggestion = false
    },
    onDeleteAtion(idx) {
      this.items.splice(idx, 1);
    },
    isHovered(item) {
      return this.hoverItem == item
    },
    async callGetSuggestionFunction() {
      var suggestions = []
      var fuzzySuggestions = []
      try {
        suggestions = await this.suggestList()
        if (suggestions.length <= 5) {
          fuzzySuggestions = await this.fuzzySuggestList()
        }
      } catch (e) {
        //console.log(e)
      }
      finally {
        this.$set(this, 'suggestions', suggestions)
        this.$set(this, 'fuzzySuggestions', fuzzySuggestions)

        if ( suggestions.length > 0 || fuzzySuggestions.length > 0) {
          this.showSuggestion = true
        }
      }
    },
    refresh() {
      this.callGetSuggestionFunction()
    }
  },
  watch: {
    // eslint-disable-next-line
    items: function(val) {
      this.$emit('updateItems', this.items);
    }
  }
}
</script>

<style>
.vue-tag-suggest {
  position: relative;
}
.vue-tag-suggest > ol,ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.vue-tag-suggest .wrap-input input {
  width: 100%;
  padding: 5px;
  font-size: 16px;
  border-radius: 3px;
  /* border: 1px solid #ddd; */
  border: 1px solid #52b56a;
  box-sizing: border-box;
}
.vue-tag-suggest .wrap-suggest {
  top: 3px;
  position: relative;
}

.vue-tag-suggest .wrap-suggest .suggestions {
  position: relative;
  left: 0;
  right: 0;
  top: 100%;
  border-radius: 3px;
  border: 1px solid #ddd;
  background-color: #fff;
  opacity: 1;
  z-index: 999999;
}
.vue-tag-suggest .suggestions .item {
  cursor: pointer;
  user-select: none;
  text-align: left;
  padding-left: 5px;
}

.vue-tag-suggest .suggestions .item.hover {
  background-color: #e4e4e4;
}

.vue-tag-suggest .wrap-seleted-items .wrap-btn {
  display: table;
  height: 100%;
}
.vue-tag-suggest .wrap-seleted-items .closeBtn {
  display: table-cell;
  width: 22px;
  position: relative;
}
.vue-tag-suggest .wrap-seleted-items .closeBtn span::before,
.vue-tag-suggest .wrap-seleted-items .closeBtn span::after {
  display: block;
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 84%;
  height: 16%;
  margin: -8% 0 0 -42%;
  background: #52b56a;
}
.vue-tag-suggest .wrap-seleted-items .closeBtn span::before {
  transform: rotate(-45deg);
}
.vue-tag-suggest .wrap-seleted-items .closeBtn span::after {
  transform: rotate(45deg);
}

.vue-tag-suggest .wrap-seleted-items{
  display: flex;
  align-content: flex-start;
  flex-wrap: wrap;
  padding-bottom: 2px;
  list-style: none;
}
.vue-tag-suggest .wrap-seleted-items .seleted-item{
  display: flex;
  margin: 3px;
  padding: 1px;
  border-radius: 3px;
  box-shadow: 0 0 0 1px #52b56a;
}

</style>
