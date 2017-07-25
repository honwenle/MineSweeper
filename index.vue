<template>
<div>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <game :num="5" :mine="5"></game>
</div>
</template>
<script>
import game from 'game'
export default {
    components: {
        game
    }
}
</script>