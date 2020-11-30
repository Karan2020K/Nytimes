<!-- =========================================================================================
    File Name: Home.vue
    Description: Home Page
    ----------------------------------------------------------------------------------------
   
    Author: Karan Krishna K
    
========================================================================================== -->
<template>
  <div>
    <v-toolbar dark color="black">
      <v-toolbar-title>The NewYork Times</v-toolbar-title>

      <v-autocomplete
        v-model="select"
        :loading="loading"
        :items="items"
        @input="content"
        :search-input.sync="search"
        cache-items
        class="mx-3"
        clearable
        flat
        hide-no-data
        hide-details
        label="Search article"
        solo-inverted
      ></v-autocomplete>
    </v-toolbar>
    <v-main>
      <v-container fluid>
        <v-row>
          <v-col class="d-flex" cols="12" sm="3">
            <v-select
              :items="sourceitems"
              v-model="sourcetype"
              @input="content"
              label="Select source"
              outlined
              dense
            ></v-select>
          </v-col>
          <v-col class="d-flex" cols="12" sm="3">
            <v-pagination
              v-model="page"
              :length="20"
              :total-visible="7"
              prev-icon="mdi-menu-left"
              next-icon="mdi-menu-right"
              @input="next"
            ></v-pagination>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" sm="8">
            <v-card max-width="1200" class="mx-auto">
              <v-card-title class="headline">
                Featured news
              </v-card-title>
              <v-container>
                <v-row dense>
                  <v-col
                    cols="12"
                    sm="4"
                    v-for="(item, index) in computedFeaturedNewsObj"
                    :key="index"
                  >
                    <v-card
                      class="mx-auto"
                      max-width="344"
                      style="cursor: pointer;"
                      @click="viewDetails(item)"
                    >
                      <v-img v-bind:src="item.img" height="200px"></v-img>

                      <v-card-title>
                        {{ item.title }}
                      </v-card-title>

                      <v-card-text>
                        {{ item.abstract }}
                      </v-card-text>
                      <v-card-text class="text-right">
                        {{
                          new Date(item.created_date)
                            .toISOString()
                            .substr(0, 10)
                        }}
                      </v-card-text>
                    </v-card>
                  </v-col>
                </v-row>
              </v-container>
            </v-card>
          </v-col>

          <v-col cols="12" sm="4">
            <v-card max-width="600" class="mx-auto">
              <v-card-title class="headline">
                Latest news
              </v-card-title>
              <v-container>
                <v-row dense>
                  <v-col
                    class=""
                    cols="12"
                    v-for="(item, index) in computedlatestNewsObj"
                    :key="index"
                  >
                    <v-card style="cursor: pointer;" @click="viewDetails(item)">
                      <v-card-title>
                        {{ item.title }}
                      </v-card-title>

                      <v-card-text>
                        {{ item.abstract }}
                      </v-card-text>

                      <v-card-text class="text-right">
                        {{
                          new Date(item.created_date)
                            .toISOString()
                            .substr(0, 10)
                        }}
                      </v-card-text>
                    </v-card>
                  </v-col>
                </v-row>
              </v-container>
            </v-card></v-col
          >
        </v-row>
      </v-container>
    </v-main>
  </div>
</template>

<script>
export default {
  name: "App",

  components: {},

  data: () => ({
    show: false,
    limitFeatureNews: 3,
    limitLatestNews: 9,
    latestNews: [],
    FeaturedNews: [],
    sourcetype: "all",
    loading: false,
    items: [],
    search: null,
    page: 1,
    select: null,
    pagVal: 0,
    section: [],
    sourceitems: [
      {
        text: "All",
        value: "all",
      },
      {
        text: "New York Times",
        value: "nyt",
      },
      {
        text: "International New York Times",
        value: "inyt",
      },
    ],
  }),
  watch: {
    search(val) {
      val && val !== this.select && this.querySelections(val);
    },
  },
  created() {
    this.sectionlList();
  },
  computed: {
    computedFeaturedNewsObj() {
      return this.limitFeatureNews
        ? this.FeaturedNews.slice(0, this.limitFeatureNews)
        : this.object;
    },
    computedlatestNewsObj() {
      return this.limitLatestNews
        ? this.latestNews.slice(0, this.limitLatestNews)
        : this.object;
    },
  },

  methods: {
    next(val) {
      this.pagVal = val * 25 - 25;
      this.content();
    },
    viewDetails(val) {
      this.$router
        .push({
          name: "Details",
          query: {
            title: val.title,
          },
        })
        .catch(() => {});
    },
    async content() {
      var selectType = "all";
      if (this.select != null) {
        selectType = this.select;
      }
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

      var requestOptions = {
        method: "GET",
      };

      const rawResponse = await fetch(
        process.env.VUE_APP_BASE_URL +
          "news/v3/content/" +
          this.sourcetype +
          "/" +
          selectType +
          ".json?api-key=" +
          process.env.VUE_APP_API_KEY +
          "&offset=" +
          this.pagVal,
        requestOptions
      );

      const response = await rawResponse.json();
      console.log(response);
      this.latestNews = [];
      this.FeaturedNews = [];
      if (response.results.length) {
        for (let index = 0; index < response.results.length; index++) {
          var Imgurl = "";
          if (response.results.length) {
            if (response.results[index].multimedia != null) {
              Imgurl = response.results[index].multimedia[2].url;
            }
            if (response.results[index].subsection != "") {
              var Row = {
                abstract: response.results[index].abstract,
                created_date: response.results[index].created_date,
                title: response.results[index].title,
                url: response.results[index].url,
                img: Imgurl,
              };
              this.FeaturedNews.push(Row);
            }
            if (response.results[index].subsection == "") {
              var Row = {
                abstract: response.results[index].abstract,
                created_date: response.results[index].created_date,
                title: response.results[index].title,
                url: response.results[index].url,
              };
              this.latestNews.push(Row);
            }
          }
        }
        console.log(this.FeaturedNews);
        console.log(this.latestNews);
      }
    },
    async sectionlList() {
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

      var requestOptions = {
        method: "GET",
      };

      const rawResponse = await fetch(
        process.env.VUE_APP_BASE_URL +
          "news/v3/content/section-list.json?api-key=" +
          process.env.VUE_APP_API_KEY,

        requestOptions
      );

      const response = await rawResponse.json();
      console.log(response);
      if (response.results.length) {
        for (let index = 0; index < response.results.length; index++) {
          var Row = {
            text: response.results[index].display_name,
            value: response.results[index].section,
          };

          this.section.push(Row);
        }
        this.content();
      }
    },
    querySelections(v) {
      this.loading = true;
      setTimeout(() => {
        this.items = this.section.filter((e) => {
          return (
            (e.text || "").toLowerCase().indexOf((v || "").toLowerCase()) > -1
          );
        });
        this.loading = false;
      }, 500);
    },
  },
};
</script>
