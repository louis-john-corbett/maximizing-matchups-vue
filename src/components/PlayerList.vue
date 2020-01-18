<template>
    <div class="players">
        <h1 id="players-header"> Players </h1>
        <div v-bind:key="player.player_id" v-for="player in players">
            <Player v-bind:player="player" v-on:del-player="$emit('del-player', player.player_id)"/>
        </div>
    </div>
</template>

<script>
import Player from './Player.vue'

export default {
    name: "PlayerList",
    components: {
        Player
    },
    props: ["players"]
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.players {
    padding: 10px 10px 10px 10px
}
#players-header{
    border-bottom: #ccc dotted;
    padding: 10px 10px 10px 10px
}
.fa-times {
  cursor: pointer;
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
  box-shadow: 0 0 10px rgba(0,0,0, 0.25);
  z-index: 1000;
}
li {
  padding: 10px 16px;
  border-bottom: 1px solid #ccc;
  cursor: pointer;
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
span {
  display: block;
  color: #2c3e50;
}
.name {
  font-weight: 700;
  font-size: 18px;
}
.team {
  font-style: italic;
}
</style>