<template>
    <div>
        <ul v-for="y in mnm">
            <li v-for="x in y"
                :class="{'isMine': (x['isMine'] && isOver),
                    'isSafe': x['isSafe'] || (isOver&&!x['isMine']),
                    'isMark': x['isMark']}"
                @click="clickItem(x)"
                @contextmenu.prevent="rightClick(x)" >
                {{(x['isSafe'] || isOver) ? (x['numMine'] ? x['numMine'] : '') : ''}}
            </li>
        </ul>
        <div class="win" v-if="numSafe == (num*num - mine)">You Win !</div>
        <div class="over" v-if="isOver">Game Over !</div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            mnm: [],
            isOver: false,
            step: 0,
            initOver: false,
            numSafe: 0
        }
    },
    props: {
        num: Number,
        mine: Number
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
                        isMark: false,
                        x: x,
                        y: y,
                        numMine: 0
                    };
                }
            }
            let tmp_mine = {}, idx;
            for(let i = 0; i < this.mine; i++) {
                do {
                    idx = Math.floor(Math.random() * this.num * this.num);
                } while (idx in tmp_mine);
                tmp_mine[idx] = true;
                tmp_mnm[Math.floor(idx/this.num)][idx%this.num]['isMine'] = true;
            }
            console.log(tmp_mine)
            for(let y = 0; y < this.num; y++) {
                for(let x = 0; x < this.num; x++) {
                    tmp_mnm[y][x].numMine = this.countMine(tmp_mnm, tmp_mnm[y][x]);
                }
            }
            this.mnm = tmp_mnm;
        },
        rightClick(item) {
            let tmp_mnm = this.mnm;
            tmp_mnm[item.y][item.x].isMark = !tmp_mnm[item.y][item.x].isMark;
            this.mnm = tmp_mnm;
        },
        clickItem(item) {
            this.getAroundList(item.x, item.y)
            if (item['isMine']) {
                console.log('GameOver')
                this.isOver = true;
            } else {
                let tmp_mnm = this.mnm;
                tmp_mnm[item.y][item.x].isSafe = true;
                this.numSafe++;
                this.mnm = tmp_mnm;
                if (tmp_mnm[item.y][item.x].numMine == 0) {
                    this.checkAround(tmp_mnm, item.x, item.y);
                }
                this.initOver = true;
            }
        },
        countMine(tmp_mnm, item) {
            let count = 0;
            let arr = this.getAroundList(item.x, item.y);
            arr.forEach(function([x, y]) {
                if (tmp_mnm[y][x]['isMine']) {
                    count++;
                }
            }, this);
            return count;
        },
        getAroundList(x, y) {
            // console.log('被查者:'+y+','+x)
            let list = [];
            if (x > 0) {
                this.exceptList(x-1,y, list);
                if (y > 0) {
                    this.exceptList(x-1, y-1, list);
                }
            }
            if (y > 0) {
                this.exceptList(x, y-1, list);
                if (x+1 < this.num) {
                    this.exceptList(x+1, y-1, list);
                }
            }
            if (x+1 < this.num) {
                this.exceptList(x+1, y, list);
                if (y+1 < this.num) {
                    this.exceptList(x+1, y+1, list);
                }
            }
            if (y+1 < this.num) {
                this.exceptList(x, y+1, list);
                if (x > 0) {
                    this.exceptList(x-1, y+1, list);
                }
            }
            return list;
        },
        exceptList(x, y, list) {
            if (this.initOver) {
                console.log('周围:'+y+','+x+'。安全:'+this.mnm[y][x].isSafe)
                if (!this.mnm[y][x].isSafe) {
                    list.push([x, y]);
                }
            } else {
                list.push([x, y]);
            }
        },
        checkAround(tmp_mnm, x, y) {
            let arr = this.getAroundList(x, y);
            arr.forEach(function([ix, iy]) {
                tmp_mnm[iy][ix].isSafe = true;
                this.numSafe++;
                this.mnm = tmp_mnm;
                if (tmp_mnm[iy][ix]['numMine'] == 0) {
                    this.$nextTick(() => {
                        this.checkAround(tmp_mnm, ix, iy);
                    })
                }
            }, this);
        }
    }
}
</script>
<style>
    .isMark{
        background: #0f0;
    }
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
        text-align: center;
        line-height: 50px;
        font-size: 18px;
    }
</style>
