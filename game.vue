<template>
    <div>
        <ul class="mine-map" v-for="y in mnm">
            <li v-for="x in y"
                :class="{'isMine': (x['isMine'] && isOver),
                    'isSafe': x['isSafe'] || (isOver&&!x['isMine']),
                    'isMark': x['isMark']}"
                @click="clickItem(x)"
                @dblclick="doubleClick(x)"
                @contextmenu.prevent="rightClick(x)" >
                {{(x['isSafe'] || isOver) ? (x['numMine'] ? x['numMine'] : '') : ''}}
            </li>
        </ul>
        <ol class="rule">
            <li>左键翻开，格子上的数字表示周围地雷个数</li>
            <li>右键标记地雷</li>
            <li>双击快速翻开周围格子</li>
        </ol>
        <div class="win" v-if="numSafe >= (num*num - mine)">You Win !</div>
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
            for(let y = 0; y < this.num; y++) {
                for(let x = 0; x < this.num; x++) {
                    tmp_mnm[y][x].numMine = this.countMine(tmp_mnm, tmp_mnm[y][x]);
                }
            }
            this.mnm = tmp_mnm;
        },
        doubleClick(item) {
            let arr = this.getAroundList(item.x, item.y);
            let _this = this;
            let thelist = [], numMark = 0;
            arr.forEach(function ([ix, iy]) {
                if (_this.mnm[iy][ix].isMark) {
                    numMark++;
                } else {
                    if (!_this.mnm[iy][ix].isSafe) {
                        thelist.push([ix,iy]);
                    }
                }
            });
            if (this.mnm[item.y][item.x].numMine == numMark) {
                thelist.forEach(function ([ix, iy]) {
                    if (_this.mnm[iy][ix].isMine) {
                        _this.isOver = true;
                    } else {
                        let tmp_mnm = _this.mnm;
                        if (!tmp_mnm[iy][ix].isSafe) {
                            tmp_mnm[iy][ix].isSafe = true;
                            _this.numSafe++;
                        }
                        _this.mnm = tmp_mnm;
                    }
                });
            }
        },
        rightClick(item) {
            let tmp_mnm = this.mnm;
            tmp_mnm[item.y][item.x].isMark = !tmp_mnm[item.y][item.x].isMark;
            this.mnm = tmp_mnm;
        },
        clickItem(item) {
            this.getAroundList(item.x, item.y)
            if (item['isMine']) {
                this.isOver = true;
            } else {
                let tmp_mnm = this.mnm;
                if (!tmp_mnm[item.y][item.x].isSafe) {
                    tmp_mnm[item.y][item.x].isSafe = true;
                    this.numSafe++;
                }
                this.mnm = tmp_mnm;
                if (tmp_mnm[item.y][item.x].numMine == 0) {
                    this.checkAround(tmp_mnm, item.x, item.y);
                }
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
            let list = [];
            if (x > 0) {
                list.push([x-1,y]);
                if (y > 0) {
                    list.push([x-1, y-1]);
                }
            }
            if (y > 0) {
                list.push([x, y-1]);
                if (x+1 < this.num) {
                    list.push([x+1, y-1]);
                }
            }
            if (x+1 < this.num) {
                list.push([x+1, y]);
                if (y+1 < this.num) {
                    list.push([x+1, y+1]);
                }
            }
            if (y+1 < this.num) {
                list.push([x, y+1]);
                if (x > 0) {
                    list.push([x-1, y+1]);
                }
            }
            return list;
        },
        exceptList(x, y) {
            let list = this.getAroundList(x, y);
            let thelist = [];
            let _this = this;
            list.forEach(function ([ix, iy]) {
                if (!_this.mnm[iy][ix].isSafe) {
                    thelist.push([ix,iy]);
                }
            });
            return thelist;
        },
        checkAround(tmp_mnm, x, y) {
            let arr = this.exceptList(x, y);
            arr.forEach(function([ix, iy]) {
                if (!tmp_mnm[iy][ix].isSafe) {
                    tmp_mnm[iy][ix].isSafe = true;
                    this.numSafe++;
                }
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
    .mine-map li.isMark{
        background: #0f0;
    }
    .mine-map li.isMine{
        background: #f00;
    }
    .mine-map li.isSafe{
        background: #bbb;
    }
    ul.mine-map{
        margin: 0 auto;
        -webkit-padding-start: 0;
        overflow: hidden;
    }
    .mine-map li{
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
