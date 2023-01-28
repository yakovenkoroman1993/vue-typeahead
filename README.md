# vue-typeahead

### Single
```vue
<Typeahead
  allow-new
  placeholder="Select..."
  :options="[{id: 1, label: "Option 1"}, {id: 2, label: "Option 2"}, {id: 3, label: "Option 3"}]"
  v-model="singleValue"
  @create=""
/>
...
data() {
  return {
    singleValue: null
  }
}
```

### Multiple
```vue
<Typeahead
  multiple
  placeholder="Select..."
  :options="[{value: 1, title: "Option 1"}, {value: 2, title: "Option 2"}, {value: 3, title: "Option 3"}]"
  label-key="title"
  value-key="value"
  v-model="multipleValue"
/>
...
data() {
  return {
    multipleValue: []
  }
}
```
