# vue-tag-suggest
Vue.js tag search suggest components

#### [demo](https://shaba2curry.github.io/vue-tag-suggest/index.html)

This component was created as a ui components when tag regstration. The input element suggest tag name and manages the registered tags in a list.

#### install
```
npm install --save vue-tag-suggest
```

#### example

```
<!-- Some component.vue -->
<template>
  <div>
    <vue-tag-suggest
      v-model="inputValue"
      :suggestList="suggestionList"
      @updateItems="updateItems">
    </vue-tag-suggest>
  </div>
</template>

<script>
  import VueTagSuggest from 'vue-tag-suggest'

  export default {
    components: {
      VueTagSuggest,
    },
    data() {
      return {
        inputValue: ''
      }
    },
    methods: {
      updateItems(items) {
        console.log('updateItems', items)
      },
      async suggestionList() {
        return ['vue','vue.js','vue-tag-suggest']
      }
    }
  }
</script>
```
