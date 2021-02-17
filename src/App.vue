<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col
            v-for="(link, index) in links"
            :key="index"
            cols="12"
            md="4"
            sm="6"
          >
            <v-btn
              block
              color="primary"
              :to="link.to"
            >
              {{ link.text }}
            </v-btn>
          </v-col>
        </v-row>
        <router-view/>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import store from '@/store/index.js'

export default {
  name: 'App',

  components: {
  },

  data: () => ({
    store,
    links: [
      {
        to: '/',
        text: 'Streams'
      },
      {
        to: '/signin',
        text: 'Sign in / out'
      },
      {
        to: '/about',
        text: 'About'
      }
    ]
  })
};
</script>
