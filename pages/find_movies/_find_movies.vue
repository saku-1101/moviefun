<template>
<div style="display:flex; justify-content: center;">
  
    <div style="width:100vw;">
    <v-toolbar color="#6A76AB" gradient="to top right, rgba(100,115,201,.7), rgba(25,32,72,.7)" style="width:100vw;">
      <v-toolbar-title class="text-h6 mr-6 hidden-sm-and-down">
        ๐Look for your best๏ผ
      </v-toolbar-title>
      <v-autocomplete
        v-model="selectedMovie"
        :items="items"
        :loading="isLoading"  
        :search-input.sync="search"
        chips
        clearable
        hide-details
        hide-selected
        item-text="name"
        item-value="symbol"
        label="Search for a title ..."
        solo
      >
        <template v-slot:no-data>
          <v-list-item>
            <v-list-item-title>
              ็ฅใใใๆ ็ปใฎใฟใคใใซใๅฅๅใใฆใใ ใใ
            </v-list-item-title>
          </v-list-item>
        </template>
        <template v-slot:item="{ item }">
          <!-- ใชในใใๅบใ -->
          <v-list-item-content>
            <v-list-item-title v-text="item"></v-list-item-title>
          </v-list-item-content>
        </template>
      </v-autocomplete>
      <v-btn 
      depressed
      elevation="2"
      fab
      icon
      outlined
      raised
      rounded
       @click="Search"
       class="ml-5 mr-1"
       style="font-size:1.5rem;">๐</v-btn>
    </v-toolbar>

  <div>
    <template v-if= "(selectedMovie !== '' || selectedMovie !== null) && (searchedMovie !== {})">
    <v-card
      v-for="movie in searchedMovie"
      :key="movie.id"
      class="mx-auto my-12"
      max-width="50vw"
    >
      <div>

      <v-img
        height="250"
        :src=movie.backdrop_path
      ></v-img>

      <v-card-title>{{movie.title}}</v-card-title>

      <v-card-text>
        <v-row
          align="center"
          class="mx-0"
        >
          <v-rating
            :value=movie.vote_average/2
            color="amber"
            dense
            full-increments
            readonly
            size="14"
            length="5"
          ></v-rating>

          <div class="grey--text ms-4">
            {{movie.vote_average/2}}({{movie.vote_count}} reviews)
          </div>
        </v-row>

        <div class="my-4 text-subtitle-1">
          ๅฌ้ๅนดๆๆฅ:{{movie.release_date}}
        </div>

        <div>{{movie.overview}}</div>
      </v-card-text>
      <v-divider class="mx-4"></v-divider>
      <div v-if="movie.casts">
        <v-card-title>{{Casts_message}}</v-card-title>
        <ul class="horizontal-list">
          <li v-for="cast in movie.casts" :key="cast.id">
            <v-card>
              <v-avatar
                  class="ma-3"
                  size="100"
                  tile
                >
                  <v-img :src="cast.profile_path"></v-img>
                </v-avatar>
                <v-card-title class="caption">
                  {{cast.name}} 
                  <br>
                  ({{cast.character}})
                </v-card-title>
            </v-card>
          </li>
        </ul>
      </div>
      <div v-else>
        <v-card-title>{{movie.casts_error}}</v-card-title>
      </div>
      <v-divider class="mx-4"></v-divider>

      <div v-if="movie.streaming">
        <v-card-title>{{Streaming_message}}</v-card-title>
        <v-list
          nav
          dense
        >
          <v-list-item-group
            color="primary"
          >
            <v-list-item
              v-for="(item, i) in movie.streaming"
              :key="i"
            >
                <v-avatar
                  class="ma-3"
                  size="8%"
                  tile
                >
                  <v-img :src="item.logo_path"></v-img>
                </v-avatar>

              <v-list-item-content>
                <v-list-item-title v-text="item.provider_name"></v-list-item-title>
              </v-list-item-content>
            
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </div>
      <div v-else>
        <v-card-title>{{movie.streaming_error}}</v-card-title>
      </div>
      </div>
    </v-card>
  </template>
  </div>
  </div>
  
  

</div>
</template>

<script>
const axios = require('axios')
const home = 'https://api.themoviedb.org/3'
const key = '874b2c31cd678740afe326baa8408862'
const endpoint = '/search/movie'
let query = ''
const language = '&language=ja-JA'
  export default {
    
    data: () => ({
      isLoading: false,
      dialog: false,
      items: [],
      selectedMovie: '',
      search: null,
      tab: null,
      searchedMovie: {},
      streaming: {},
      casts: {},
      Casts_message: "Fascinating casts! ๐๐ซ",
      Streaming_message: "Tonight's available virtual theatre!"
    }),
    watch: {
      model (val) {
        if (val != null) this.tab = 0
        else this.tab = null
      },
      search (val) {
        this.items = []
        console.log(val)
        query = val
        // Items have already been loaded
        if (this.items.length > 0) return

        this.isLoading = true
        let url = home + endpoint + '?api_key=' + key + '&query=' + query + language
        // Lazily load input items
        axios.get(url).then((res) => {
        console.log(res)
            let r = res.data.results
            for (let i = 0; i < r.length; i++) {
              console.log(r[i].title);
              this.items.push(r[i].title)
            }
            console.log(this.items)
      }).catch((error) => {
       console.log(error)
      }).finally(() => (this.isLoading = false))
      }
    },
    methods: {
      Search () {
          const url_id = home + endpoint + '?api_key=' + key + '&query=' + this.selectedMovie + language 
          axios.get(url_id).then((res) => {
              console.log(res)
              this.searchedMovie = res.data.results
              for (let a = 0; a < this.searchedMovie.length; a++) {
                this.searchedMovie[a].backdrop_path = "https://image.tmdb.org/t/p/w500" + this.searchedMovie[a].backdrop_path
              }
              console.log(this.searchedMovie)
              //๐Search Streaming available service in JP
              for (let i = 0; i < this.searchedMovie.length; i++) {
              let streaming_endpoint = "/movie/"+ this.searchedMovie[i].id + "/watch/providers"
              let url_streaming = home + streaming_endpoint + '?api_key=' + key + language 
              axios.get(url_streaming).then((res) => {
                  console.log(res)
                  this.$set(this.searchedMovie[i], 'streaming', res.data.results.JP.rent)
                  for (let j = 0; j < this.searchedMovie[i].streaming.length; j++) {
                    this.searchedMovie[i].streaming[j].logo_path = "https://image.tmdb.org/t/p/w500" + this.searchedMovie[i].streaming[j].logo_path
                  }
                  console.log(this.Streaming_message);
                  }).catch((error) => {
                          console.log(error)
                          this.$set(this.searchedMovie[i], 'streaming_error', "ใใฎๆ ็ปใซ้ขใใในใใชใผใใณใฐใตใผใในใฎใใผใฟใฏใใใพใใ๏ผ(There is no abailable data. Sorry.)")
                  })

              //๐Search Streaming available service in JP
              let cast_endpoint = "/movie/"+ this.searchedMovie[i].id + "/credits"
              let url_casts = home + cast_endpoint + '?api_key=' + key + language 
              axios.get(url_casts).then((res) => {
                  console.log(res)
                  this.$set(this.searchedMovie[i], 'casts', res.data.cast)
                  for (let k = 0; k < this.searchedMovie[i].casts.length; k++) {
                    this.searchedMovie[i].casts[k].profile_path = "https://image.tmdb.org/t/p/w500" + this.searchedMovie[i].casts[k].profile_path
                  }
              //return content // ้ๅๆ้ไฟกใชใฎใงใๅใซใreturn contentใใ่ตฐใฃใฆใใพใใใใ''ใ่กจ็คบใใใใ
              }).catch((error) => {
            console.log(error)
             this.$set(this.searchedMovie[i], 'casts_error', "ใใฎๆ ็ปใซ้ขใใใญใฃในใใฎใใผใฟใฏใใใพใใ๏ผ(There is no abailable data. Sorry.)")
          })
        }
        }).catch((error) => {
          console.log(error)
        })
      }
    }
    // created () { ๅใใ ใ

    // },
    // computed: { computedใงใฏ้ๅๆ้ไฟกใฏใตใใผใใใใฆใใชใ     
    // }
  }
</script>
<style>
.horizontal-list {
    overflow-x: auto;
    white-space: nowrap;
    -webkit-overflow-scrolling: touch;
    padding: 0;
}

.horizontal-list li {
    /* ๆจชในใฏใญใผใซ็จ */
    display: inline-block;
    
}

</style>

<!-- :loadingใชใฉ๏ผใใใใv-bindๅฑๆงใฏใฟใฐใซใใฃใฆ็ฐใชใ๏ผใใใใใชใใกใฌใณในใซใใ 
:loading
:search-input.sync
item-text="name" //itemsใฎใใญในใๅคใฎใใญใใใฃใnameใซ่จญๅฎ
v-slot:ใใ ใฎๅๅ๏ผ can get the data from the child-->
