<!-- =========================================================================================
    File Name: NewsDetails.vue
    Description: Show News Details 
    ----------------------------------------------------------------------------------------
   
    Author: Karan Krishna K
    
========================================================================================== -->
<template>
  <div>
    <v-toolbar dark color="black">
      <v-toolbar-title class="text-center"
        ><div class="text-center">The NewYork Times</div></v-toolbar-title
      >
      <v-spacer></v-spacer>

      <v-btn class="ma-2" outlined color="white" to="/">
        Back
      </v-btn>
    </v-toolbar>
    <v-main>
      <v-container fluid class=" text-center mt-5">
        <v-card class="mx-auto " max-width="1080" v-if="details">
          <v-card-text class="text-left">
            <div class="mb-2">
              <b>{{ details.headline.kicker }}</b>
            </div>
            <h3 class="title text--primary">
              {{ details.headline.main }}
            </h3>
            <p class="subtitle-2 text--primary mt-4">{{ details.abstract }}</p>
            <div class="mt-4">
              <b>By: {{ details.byline.original }}</b>
            </div>

            <p class="subtitle-2 text--primary mt-4">
              {{ details.lead_paragraph }}
            </p>
            <p class="subtitle-2 text--primary mt-4">
              {{ details.snippet }}
            </p>
          </v-card-text>
          <v-card-actions class="text-right">
            <div class="text-right">
              <v-btn
                text
                color="black"
                class="text-right"
                @click="learnMore(details.web_url)"
              >
                Learn More
              </v-btn>
            </div>
          </v-card-actions>
        </v-card>
      </v-container>
    </v-main>
  </div>
</template>

<script>
export default {
  name: "App",

  components: {},

  data: () => ({
    details: "",
  }),

  created() {
    this.articlesearch();
  },

  methods: {
    learnMore(val) {
      window.open(val, "_blank");
    },
    async articlesearch() {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

      var requestOptions = {
        method: "GET",
      };

      const rawResponse = await fetch(
        process.env.VUE_APP_BASE_URL +
          "search/v2/articlesearch.json?" +
          "q=" +
          this.$route.query.title +
          "&api-key=" +
          process.env.VUE_APP_API_KEY,

        requestOptions
      );

      const response = await rawResponse.json();
      this.details = response.response.docs[0];
      // console.log(response);
    },
  },
};
</script>
