<template>
    <div>
        <ul v-for="y in mnm">
            <li v-for="x in y"
                :class="{'isMine': (x['isMine'] && isOver), 'isSafe': x['isSafe'] || (isOver&&!x['isMine'])}"
                @click="clickItem(x)">
                {{(x['isSafe'] || isOver) ? (x['numMine'] ? x['numMine'] : '') : ''}}
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
                        y: y,
                        numMine: 0
                    };
                }
            }
            for(let i = 0; i < this.num; i++) {
                let idx = Math.round(Math.random() * this.num * this.num);
                console.log(Math.floor(idx/this.num),idx%this.num)
                tmp_mnm[Math.floor(idx/this.num)][idx%this.num]['isMine'] = true;
            }
            for(let y = 0; y < this.num; y++) {
                for(let x = 0; x < this.num; x++) {
                    tmp_mnm[y][x].numMine = this.countMine(tmp_mnm, tmp_mnm[y][x]);
                }
            }
            this.mnm = tmp_mnm;
        },
        clickItem(item) {
            if (item['isMine']) {
                console.log('GameOver')
                this.isOver = true;
            } else {
                let tmp_mnm = this.mnm;
                tmp_mnm[item.y][item.x].isSafe = true;
                if (tmp_mnm[item.y][item.x].numMine == 0) {
                    this.checkAround();
                }
                this.mnm = tmp_mnm;
            }
        },
        countMine(tmp_mnm, item) {
            let count = 0,
                x = item.x,
                y = item.y;
            if (y-1 >= 0 && x-1 >= 0 && tmp_mnm[y-1][x-1]['isMine']) {
                count++;
            }
            if (y-1 >= 0 && tmp_mnm[y-1][x]['isMine']) {
                count++;
            }
            if (y-1 >= 0 && x+1 < this.num && tmp_mnm[y-1][x+1]['isMine']) {
                count++;
            }
            if (x+1 < this.num && tmp_mnm[y][x+1]['isMine']) {
                count++;
            }
            if (y+1 < this.num && x+1 < this.num && tmp_mnm[y+1][x+1]['isMine']) {
                count++;
            }
            if (y+1 < this.num && tmp_mnm[y+1][x]['isMine']) {
                count++;
            }
            if (y+1 < this.num && x-1 >= 0 && tmp_mnm[y+1][x-1]['isMine']) {
                count++;
            }
            if (x-1 >= 0 && tmp_mnm[y][x-1]['isMine']) {
                count++;
            }
            return count;
        },
        getAroundList() {
            // TODO: 获取周边坐标列表
        },
        checkAround() {
            // TODO: 检测周围 如果数值=0，递归checkAround
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