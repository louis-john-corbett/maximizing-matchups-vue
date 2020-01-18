<template>

  <!-- ISN'T THIS REALLY JUST GOING TO BE TONS OF BATTER MATCHUPS, BY TEAM THE PITCHER IS FACING?-->

    <div class="pitchers">
        <h1 id="pitchers-header"> Pitcher Matchups </h1>
        <div v-bind:key="pitcher.player_id" v-for="pitcher in pitchers">
          {{pitcher.name_display_first_last}}
        </div>
    </div>
</template>

<script>
export default {
    name: "PitcherMatchups",
    props: ["pitchers"]
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pitchers {
    padding: 10px 10px 10px 10px
}

th{
  font-size: 20px;
  cursor: pointer;
  text-align: left;
}
td{
  font-size: 15px;
  text-align: left;
}
span {
  display:inline;
}
.batterTeam{
  color:steelblue;
  font-size: 12px;
}
.batterPosition{
  font-size: 12px;
}
</style>