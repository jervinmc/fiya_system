<template>
  <v-app dark>
    <!-- <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer> -->
    <v-app-bar
    v-if="$route.name!='index'"
      :clipped-left="clipped"
      fixed
      color="#068935"
      app
    >
    <v-spacer></v-spacer>
     <div class="white--text mr-5" style="cursor:pointer">
      <v-icon color="white">mdi-graph</v-icon>  <b @click="goReport" style="padding-top:10px">Reports</b>
    </div>
    <div class="white--text mr-5" style="cursor:pointer">
      <v-icon color="white">mdi-account</v-icon>  <b @click="goUsermanagement" style="padding-top:10px">Usermanagement</b>
    </div>
    <v-btn  @click="logout">Logout</v-btn>
    </v-app-bar>
    <v-main fluid>
      <v-container  fluid class="pa-0">
        <Nuxt class="pa-0"  />
      </v-container>
    </v-main>
    <v-navigation-drawer
      v-model="rightDrawer"
      :right="right"
      temporary
      fixed
    >
      <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light>
              mdi-repeat
            </v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-footer
      :absolute="!fixed"
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  methods:{
    goReport(){
     window.location.href="/dashboard"
    },
    goUsermanagement(){
      window.location.href="/usermanagement"
    },
    logout(){
      window.location.href="/"
    }
  },
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js'
    }
  }
}
</script>
