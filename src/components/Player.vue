<template>
    <div class="player">
        <p>
            <span class="name" v-bind:class="position">{{player.name_display_first_last}} ({{player.position}})<button @click="$emit('del-player', player)" class="del">x</button></span>
            <span class="team" v-text="player.team_full"></span>
        </p>
    </div>
</template>>

<script>
export default {
    name: "Player",
    props: ["player"],
    computed: {
        position: function(){
            return this.player.position === 'P' ? 'pitcher' : 'batter';
        }
    }
}
</script>

<style scoped>
    .player{
        border-bottom: 1px #ccc dotted;
    }

    .del {
        background: #ff0000;
        color: #fff;
        border: none;
        padding: 7px 10px;
        border-radius: 50%;
        cursor: pointer;
        float: right;
    }

    .batter{
        color: #25a;
    }

    .pitcher{
        color: #aa3;
    }
</style>