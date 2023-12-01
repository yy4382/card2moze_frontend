<template>
  <label for="cookie-input">New Cookie:</label>
  <input id="cookie-input" v-model="newCookie" type="text">
  <button @click="updateCookie">更新Cookie</button>
</template>
  
<script>
import axios from 'axios'

export default {
  name: "SettingsPage",
  props: { backend_url: { type: String, default: "" } },
  data() {
    return {
      newCookie: "",
    }
  },
  methods: {
    updateCookie() {
      axios.post(`${this.backend_url}/update_cookie`, {
        cookie: this.newCookie,
      }).then((response) => {
        if (response.data.success) {
          alert("更新成功");
        } else {
          alert(response.data.error);
        }
      })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
  }
}
</script>