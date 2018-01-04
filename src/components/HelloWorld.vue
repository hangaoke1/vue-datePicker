<template>
  <div class="hello">
    {{time}}
    <date-picker v-model="time" @change="timeChange" :range="true"></date-picker>
  </div>
</template>

<script>
import datePicker from './datePicker'
export default {
  name: 'HelloWorld',
  components: {
    datePicker
  },
  data () {
    return {
      time: ['2017-11-05', '2018-01-10']
    }
  },
  methods: {
    timeChange (value) {
      console.log('时间改变了', value)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
