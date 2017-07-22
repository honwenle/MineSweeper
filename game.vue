<template>
    <div>
        <ul v-for="y in mnm">
            <li v-for="x in y"
                :class="{'isMine': (x['isMine'] && isOver), 'isSafe': x['isSafe']}"
                @click="clickItem(x)">
            </li>
        </ul>
    </div>
</template>
<script>
export default {
    data() {
        return {
            mnm: [],
            isOver: false,
            step: 0
        }
    },
    props: {
        num: Number
    },
    mounted() {
        this.initGame();
    },
    methods: {
        initGame() {
            let tmp_mnm = [];
            for(let y = 0; y < this.num; y++) {
                tmp_mnm[y] = [];
                for(let x = 0; x < this.num; x++) {
                    tmp_mnm[y][x] = {
                        isMine: false,
                        isSafe: false,
                        x: x,
                        y: y
                    };
                }
            }
            for(let i = 0; i < this.num; i++) {
                let idx = Math.round(Math.random() * (this.num * this.num + 1));
                console.log(Math.floor(idx/this.num),idx%this.num)
                tmp_mnm[Math.floor(idx/this.num)][idx%this.num]['isMine'] = true;
            }
            this.mnm = tmp_mnm;
        },
        clickItem(item) {
            if (item['isMine']) {
                console.log('GameOver')
                this.isOver = true;
            } else {
                // TODO: 计算显示
                let tmp_mnm = this.mnm;
                tmp_mnm[item.y][item.x].isSafe = true;
                this.mnm = tmp_mnm;
            }
        }
    }
}
</script>
<style>
    .isMine{
        background: #f00;
    }
    .isSafe{
        background: #bbb;
    }
    ul{
        margin: 0 auto;
        -webkit-padding-start: 0;
        overflow: hidden;
    }
    li{
        list-style: none;
        border: 1px solid #eee;
        width: 50px;
        height: 50px;
        background: #666;
        float: left;
    }
</style>