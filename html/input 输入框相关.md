##vue的单选框绑定单一属性，多选框绑定空数组，选择框可根据情况而定，选中的值都为data。
##若需要绑定的数据来自vuex，则需要
```html
<input :value="message" @input="updateMessage"/>
```

```javascript
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```
或者

```html
<input v-model="message">
```

```javascript
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```