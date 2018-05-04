<style lang="less">
@import "../../less/common/variables.less";

// --- 大号字体(16px)宽度
.width-loop(@n, @i: 10) when (@i =< @n) {
  .htWidth@{i} {
    width: (@i * 1px);
    min-width: (@i * 1px);
    max-width: (@i * 1px);
  }
  .width-loop(@n, (@i + 10));
}
// --- 使用示例：
// .width-loop(170, 50);

// --- 标准字体(14px)宽度
.width-loop-95(@n, @i: 50) when (@i =< @n) {
  .htWidth@{i} {
    width: ceil(@i * 0.95px);
    min-width: ceil(@i * 0.95px);
    max-width: ceil(@i * 0.95px);
  }
  .width-loop-95(@n, (@i + 10));
}
.width-loop(180, 40);
.fontSize14 {
  .width-loop-95(180, 40);
}

table {
  line-height: 29px;
  width: 100%;
  font-size: 14px;
  border-collapse: collapse;
  thead {
    background: @bg-color__table_head;
    color: @color__table_head;
  }
  tbody {
    color: @color__table_body;
    //   tr {
    //     &:nth-child(even) {
    //       background-color: @table-rows-even;
    //     }
    //     &:nth-child(odd) {
    //       background-color: @table-rows-odd;
    //     }
    //   }
  }
  tr {
    &:first-child {
      th,
      td {
        border-top-width: 0;
      }
    }
  }
  th,
  td {
    border: 1px solid @boder-color__table;
    border-left-width: 0;
    border-right-width: 0;
    padding: 0 3px;
    text-align: left;
  }
  div {
    width: fit-content;
  }
}

.ht-table-container {
  height: 100%;
  overflow: hidden;
  position: relative;
  table {
    table-layout: fixed;
  }
  td {
    white-space: nowrap;
    div {
      max-width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
}
.ht_table-fixed-head {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 10;
  //   table {
  //     border-right: 1px solid @color-boder-1;
  //   }
}
.ht_table-fixed-head,
.ht_table-head {
  td {
    line-height: 28px;
    font-size: 12px;
  }
}
.ht_table-fixed-body,
.ht_table-body {
  background: #fff;
}
.ht_table-fixed-body {
  position: absolute;
  left: 0;
  top: 29px;
  z-index: 9;
  height: e("calc(100vh - 38px)");
  overflow: hidden;
}
.ht_table-body {
  overflow: auto;
  position: relative;
  width: 100%;
  height: e("calc(100vh - 30px)");
}
.ht_main-table {
  //   position: absolute;
}
[rowspan="2"] {
  height: 59px;
}
</style>
// ┌─────────────────────────┬──────────────────────────────────┐
// │   ht_table-fixed-head   │           ht_table-head          │
// ├─────────────────────────┼──────────────────────────────────┤
// │                         │                                  │
// │                         │                                  │
// │   ht_table-fixed-body   │           ht_table-body          │
// │                         │                                  │
// │                         │                                  │
// └─────────────────────────┴──────────────────────────────────┘
<template>
    <div class="ht-table-container" v-if="_ht_config && _ht_config.length">
        <div class="ht_table-fixed-head" v-if="fixedColCount">
            <table :style="{
                width: fixedTableWidth + 'px'
            }">
                <thead>
                    <tr v-for="(item, index) in _head_config" :key="index">
                        <td v-for="(n, i) in item" :key="i" v-if="n.colIndex < 3" :colspan="n.col" :rowspan="n.row" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" v-html="n.title" :style="{textAlign: n.align || ''}"></td>
                    </tr>
                </thead>
            </table>
        </div>
        <div class="ht_table-fixed-body" v-if="fixedColCount">
            <table class="ht_main-table" :style="{marginTop: -fixedBodyTop + 'px', width: fixedTableWidth + 'px'}">
                <tbody>
                    <tr v-for="(item, i) in tableData" :key="i" v-if="i >= startIndex && i < endIndex" :class="'row_'+i" @click="clickTableRow($event, item)">
                        <!-- <td class="threeDots" v-for="(n, index) in _ht_config" :key="index" v-if="index < 3" v-html="n.formatter(item[n.field],item,  i)" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" :style="{textAlign: n.align || ''}"></td> -->
                        <td class="threeDots" v-for="(n, index) in _ht_config" :key="index" v-if="index < fixedColCount" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" @click="clickTableCell($event, item)" :style="{textAlign: n.align || ''}">
                            <slot :name="n.field" :rowData="item">
                                <!-- 这里写入备用内容 -->
                            </slot>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="ht_table-head">
            <table :style="{marginLeft: headMarginLeft + fixedTableWidth + 'px', width: 'calc(100% - ' + fixedTableWidth + 'px)'}">
                <thead>
                    <tr v-for="(item, index) in _head_config" :key="index">
                        <td v-for="(n, i) in item" :key="i" :colspan="n.col" :rowspan="n.row" v-if="i >= fixedColCount" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" v-html="n.title" :style="{textAlign: n.align || ''}">

                        </td>
                    </tr>
                </thead>
            </table>
        </div>
        <div class="ht_table-body" @scroll="handleScroll">
            <table class="ht_main-table" :style="{
                marginLeft: fixedTableWidth + 'px',
                marginTop: startMargin +'px', 
                marginBottom: endMargin + 'px',
                width: 'calc(100% - ' + fixedTableWidth + 'px)'
                }">
                <tbody>
                    <!-- <tr v-for="(item, i) in tableData" :key="i" v-show="i >= startIndex && i < endIndex" :class="'row_'+i" @click="clickTableRow($event, item)"> -->
                    <tr v-for="(item, i) in tableData" :key="i" :class="'row_'+i" @click="clickTableRow($event, item)">
                        <!-- <td class="threeDots" v-for="(n, index) in _ht_config" :key="index" v-html="n.formatter(item[n.field],item,  i)" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" @click="clickTableCell($event, item)" :style="{textAlign: n.align || ''}"> -->
                        <td class="threeDots" v-for="(n, index) in _ht_config" :key="index" v-if="index >= fixedColCount" :class="[n.class || '', n.width? 'htWidth'+ n.width: '']" @click="clickTableCell($event, item)" :style="{textAlign: n.align || ''}">
                            <slot :name="n.field" :rowData="item">
                                <!-- 这里写入备用内容 -->
                            </slot>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>
<script>
import tool from './tool.js'
export default {
    data() {
        return {
            // --- 表头配置， 二维数组
            _head_config: [],
            // --- 表身配置， 一维数组
            _ht_config: [],

            startIndex: -1,
            endIndex: 0,
            startMargin: 0,
            endMargin: 0,
            headMarginLeft: 0,
            fixedBodyTop: 0,
            fixedTableWidth: 0
        }
    },
    props: {
        tableData: {
            type: Array,
            default: () => { return [] }
        },
        height: {
            type: Number,
            default: 640
        },
        config: {
            type: Object,
            default: () => { return {} }
        },
        fields: {
            type: Array,
            default: () => { return [] }
        },
        // --- 左侧固定列数
        fixedColCount: {
            type: Number,
            default: 0
        }
    },
    created() {
        this.generateConfig()
        this.endIndex = 40
    },
    mounted() {
    },
    watch: {
        tableData: function (newData) {
            this.startIndex = 0
            this.endIndex = newData.length
            this.handleScroll()
        },
        fields(val) {
            // console.log('this._head_config:', this._head_config)
            this.generateConfig()
        },
        endIndex(val) {
            console.log('\nthis.startIndex: ', this.startIndex, ' this.endIndex: ', this.endIndex)
        }
    },

    methods: {
        generateConfig() {
            this.generateBodyConfig()
            this.generateHeadConfig()
        },
        generateBodyConfig() {
            this._ht_config = this.diGuiConfig([].concat(this.fields), this.config)
            console.log('this._ht_config:', this._ht_config)
            if (this.fixedColCount) {
                let fixedTableWidthTemp = 0
                for (var i = 0; i < this.fixedColCount; i++) {
                    fixedTableWidthTemp += this._ht_config[i].width || 0
                }
                this.fixedTableWidth = fixedTableWidthTemp
            }
        },
        generateHeadConfig() {
            var temp = []
            this.fields.forEach(val => {
                temp.push(this.config[val])
            })
            this._head_config = tool.init(temp)
            console.log('\nthis._head_config:', this._head_config)
            console.log('\nthis.startIndex: ', this.startIndex, ' this.endIndex: ', this.endIndex)
        },
        diGuiConfig(fields, configSample) {
            var result = []
            fields.forEach(val => {
                if (configSample[val].hasOwnProperty('son')) {
                    // result = result.concat(configSample[val].son)
                    Array.prototype.push.apply(result, configSample[val].son)
                } else {
                    result.push(this.config[val])
                }
            })
            return result
        },
        handleScroll() {
            var el = document.querySelector('.ht_table-body')
            this.headMarginLeft = -el.scrollLeft
            var containerHeight = el.offsetHeight - 2 * 30,
                containerOffsetTop = el.scrollTop,
                DL = this.tableData.length
            this.endIndex = DL
            console.log(containerOffsetTop)
            this.fixedBodyTop = containerOffsetTop
            return;

            var heightCount = 30 * this.tableData.length;
            if (containerHeight > 30 * DL) {

            } else if (heightCount - this.height >= 200) {
                var tempStart = Math.floor((containerOffsetTop - 100) / 30),
                    tempEnd = Math.floor((containerOffsetTop + containerHeight + 100) / 30)
                this.startIndex = tempStart >= 0 ? tempStart : 0
                this.endIndex = tempEnd < DL ? tempEnd : DL
            } else {
                this.startIndex = 0
                this.endIndex = 0
            }
            this.startMargin = this.startIndex * 30

            if (this.endIndex) {
                this.endMargin = (this.tableData.length - this.endIndex) * 30
            }

            this.fixedBodyTop = containerOffsetTop - this.startIndex * 30
            // console.log(containerHeight, ' ', containerOffsetTop, ' startIndex: ', this.startIndex, ' endIndex: ', this.endIndex, ' startMargin: ', this.startMargin, ' endMargin: ', this.endMargin)
        },
        clickTableCell(e, data) {
            this.$emit('clickTableCell', e, data)
        },
        clickTableRow(e, data) {
            this.$emit('clickTableRow', e, data)
        },
    }
}
</script>
