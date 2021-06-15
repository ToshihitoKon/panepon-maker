<template>
  <div>
    <table style="border-spacing: 0px">
      <tr
        v-for="(row, idx_r) in table"
        :key="idx_r">
        <td
          v-for="(column, idx_c) in row"
          :key="idx_r + '.' +  idx_c"
          style="padding: 0px"
        >
          <img
            v-bind:src="column.src"
            alt=""
          v-on:click="swap(idx_r,idx_c)"
            width=30>
        </td>
      </tr>
    </table>
  </div>
</template>


<script>
import ImageGreen from "@/assets/green.png"
import ImagePurple from "@/assets/purple.png"
import ImageRed from "@/assets/red.png"
import ImageSky from "@/assets/sky.png"
import ImageYellow from "@/assets/yellow.png"

const panels = [
  {
    color: "green",
    src: ImageGreen,
  },
  {
    color: "purple",
    src: ImagePurple,
  },
  {
    color: "red",
    src: ImageRed,
  },
  {
    color: "sky",
    src: ImageSky,
  },
  {
    color: "yellow",
    src: ImageYellow,
  },
]

export default {
  data: function(){
    return{
      table: []
    }
  },
  mounted: function() {
    this.generateTable()
  },
  methods:{
    generateTable: function(){
      var tmpTable = []
      var tmpRow

      for (var row = 0; row < 12; row++){
        tmpRow = []
        for (var column = 0; column < 6; column++){
          // 上下左右に隣り合うパネルはNG
          // 上と左を見る
          var upPanel
          var leftPanel
          if (row != 0) {
            upPanel = tmpTable[row-1][column]
          }
          if (column != 0) {
            leftPanel= tmpRow[column-1]
          }

          // 抽選候補パネル
          var candidatePanel = panels.filter(v=>{return (v != upPanel && v != leftPanel)})
          var panelNum = Math.floor(Math.random() * candidatePanel.length)
          tmpRow.push(candidatePanel[panelNum])
        }
        tmpTable.push(tmpRow)
      }
      this.table = tmpTable
    },
    swap: function(row, column){
      console.log(row, column)
      if (column == 0) { return }
      var table = this.table
      var tmp = table[row][column]
      table[row][column] = table[row][column-1]
      table[row][column-1] = tmp
      this.table = table
    }
  }
}
</script>
