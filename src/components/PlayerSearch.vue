<template>
  <div class="Typeahead">
    <i class="fa fa-spinner fa-spin" v-if="loading"></i>
    <template v-else>
      <i class="fa fa-search" v-show="isEmpty"></i>
      <i class="fa fa-times" v-show="isDirty" @click="reset"></i>
    </template>

    <input
      type="text"
      class="Typeahead__input"
      placeholder="Search for player(s) by name"
      autocomplete="off"
      v-model="query"
      @keydown.down="down"
      @keydown.up="up"
      @keydown.enter="hit"
      @keydown.esc="reset"
      @blur="reset"
      @input="update"
    />

    <!-- the list -->
    <ul v-show="hasItems">
      <li class="noselect"
        v-bind:key="item.player_id"
        v-for="(item, $item) in items"
        :class="activeClass($item)"
        @mousedown="hit"
        @mousemove="setActive($item)"
      >
        <span class="name">{{item.name_display_first_last}} ({{item.position}})</span>
        <span class="team" v-text="item.team_full"></span>
      </li>
    </ul>
  </div>
</template>

<script>
import VueTypeahead from 'vue-typeahead'
import { util } from 'vue'

export default {
  extends: VueTypeahead,
  props: ["players"],

  //// Player by ID - JSON
  //// const playerByIdUrl = 'https://statsapi.mlb.com/api/v1/people/';

  data () {
    return {
      // The source url
      // (required)
      src: "http://lookup-service-prod.mlb.com/json/named.search_player_all.bam?sport_code='mlb'&active_sw='Y'",

      // The data that would be sent by request
      // (optional)
      data: {},

      // Limit the number of items which is shown at the list
      // (optional)
      limit: 5,

      // The minimum character length needed before triggering
      // (optional)
      minChars: 2,

      // Highlight the first item in the list
      // (optional)
      selectFirst: false,

      // Override the default value (`q`) of query parameter name
      // Use a falsy value for RESTful query
      // (optional)
      queryParamName: 'name_part'
    }
  },

  methods: {
    // The callback function which is triggered when the user hits on an item
    // (required)
    onHit (item) {
      item.player_id = +item.player_id;
      item.team_id = +item.team_id;
      this.$emit('add-player', item)
      this.update()
    },

    // The callback function which is triggered when the response data are received
    // (optional)
    prepareResponseData (searchResults) {
      this.players.forEach(savedPlayer => searchResults = searchResults.filter(searchResult => parseInt(searchResult.player_id, 10) !== savedPlayer.player_id));
      return searchResults;
    },

    filterOutCurrentPlayers (data) {
      this.players.forEach(savedPlayer => this.data = this.data.filter(searchResult => parseInt(searchResult.player_id, 10) !== savedPlayer.player_id));
      return data;
    },

    update () {
      this.cancel()

      if (!this.query) {
        return this.reset()
      }

      if (this.minChars && this.query.length < this.minChars) {
        this.items = []
        return
      }

      this.loading = true

      this.fetch().then((response) => {
        if (response && this.query) {

          let data = response.data.search_player_all.queryResults.row ? (response.data.search_player_all.queryResults.totalSize == '1' ? [ response.data.search_player_all.queryResults.row ] : Array.from(response.data.search_player_all.queryResults.row)) : []
          data = this.prepareResponseData ? this.prepareResponseData(data) : data
          this.items = this.limit ? data.slice(0, this.limit) : data
          this.current = -1
          this.loading = false

          if (this.selectFirst) {
            this.down()
          }
        }
      })
    },

    fetch () {
      if (!this.$http) {
        return util.warn('You need to provide a HTTP client', this)
      }

      if (!this.src) {
        return util.warn('You need to set the `src` property', this)
      }

      const src = this.queryParamName
        ? this.src
        : this.src + `'${this.query}'`

      const params = this.queryParamName
        ? Object.assign({ [this.queryParamName]: `'${this.query}%'` }, this.data)
        : this.data

      let cancel = new Promise((resolve) => this.cancel = resolve)
      let request = this.$http.get(src, { params })

      return Promise.race([cancel, request])
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.Typeahead {
  position: relative;
  padding: 0px 0px 30px;
}
.Typeahead__input {
  width: 100%;
  font-size: 14px;
  color: #2c3e50;
  line-height: 1.42857143;
  box-shadow: inset 0 1px 4px rgba(0, 0, 0, 0.4);
  -webkit-transition: border-color ease-in-out 0.15s,
    -webkit-box-shadow ease-in-out 0.15s;
  transition: border-color ease-in-out 0.15s, box-shadow ease-in-out 0.15s;
  font-weight: 300;
  padding: 12px 26px;
  border: none;
  border-radius: 22px;
  letter-spacing: 1px;
  box-sizing: border-box;
}
.Typeahead__input:focus {
  border-color: #4fc08d;
  outline: 0;
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px #4fc08d;
}
i {
  float: right;
  position: relative;
  top: 30px;
  right: 29px;
  opacity: 0.4;
}
ul {
  position: absolute;
  padding: 0;
  margin-top: 8px;
  min-width: 100%;
  background-color: #fff;
  list-style: none;
  border-radius: 4px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
  z-index: 1000;
  opacity: 0.85;
}
li {
  padding: 10px 16px;
  border-bottom: 1px solid #ccc;
}
li:first-child {
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
}
li:last-child {
  border-bottom-left-radius: 4px;
  border-bottom-right-radius: 4px;
  border-bottom: 0;
}
.noselect {
  -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
     -khtml-user-select: none; /* Konqueror HTML */
       -moz-user-select: none; /* Old versions of Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
            user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome, Opera and Firefox */
}
span {
  display: block;
  color: #2c3e50;
}
.active {
  background-color: #3aa373;
}
.active span {
  color: white;
}
.name {
  font-weight: 700;
  font-size: 18px;
}
.team {
  font-style: italic;
}
</style>
