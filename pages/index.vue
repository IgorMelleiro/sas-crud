<template>
  <RulesDataTable v-if="loading === false"/>
</template>

<script>
import RulesDataTable from '../components/RulesDataTable.vue'

export default {
  name: 'IndexPage', 
  components: { RulesDataTable },
  data() {
    return {
      loading: false,
      userInfo: {
        login: {
          email: 'task@searchandstay.com',
          password: 'ph37i45K'
        }
      },
    }
  },
  created () {
    this.loginUser()
  },

  methods: {
    async loginUser() {
      try {
        this.loading = true
        let res = await this.$auth.loginWith('local', { data: this.userInfo })
        localStorage.setItem('token', res.data.data.result.access_token)
      } catch (err) {
        console.log(err)
      } finally {
        this.loading = false
      }
    }
  }
}
</script>
