<template>
  <VueDatePicker v-model="range" range />
  <button @click="downloadByDate" class="functionalButtons">获取选定时间内的消费记录csv</button>
</template>

<script>
import axios from 'axios'
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'

export default {
  emits: ['csvDownload'], 
  props: { backend_url: { type: String, default: "" } },
  components: {
    VueDatePicker
  },
  data() {
    return {
      range: [new Date(), new Date()],
    }
  },
  methods:{
    downloadByDate() {
      const start_time = this.range[0].getTime();
      const end_time = this.range[1].getTime();
      axios.get(`${this.backend_url}/get_csv`, {
        params: {
          start_time: start_time,
          end_time: end_time,
        }
      }).then((response) => {
        this.$emit('csvDownload', response.data);
      })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
  }
}
</script>