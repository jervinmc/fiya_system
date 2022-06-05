<template>
  <v-img src="/background.jpeg" align="center" class="d-flex justify-center align-center" width="100vw" height="100vh" gradient="to top right, rgba(0,0,0,.38), rgba(0,0,0,0.38)"  >
       <login-container />
    </v-img>
</template>

<script>
import LoginContainer from '../components/login/LoginContainer.vue'
export default {
  components: { LoginContainer },

}
</script>

<style>

</style>