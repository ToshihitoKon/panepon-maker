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
          <div 
            v-on:click="swap(idx_r,idx_c)"
            style="display:block; width:30px; height:30px;">
            <img
              v-if="column.src"
              v-bind:src="column.src"
              alt=""
              width=30
              height=30
              style="display:block">
        </div>
        </td>
      </tr>
    </table>
    <button
      v-on:click="deletePanel"
      >delete
    </button>
    <button
      v-on:click="dropPanel"
      >drop
    </button>
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
            upPanel = tmpTable[row-1][column].color
          }
          if (column != 0) {
            leftPanel= tmpRow[column-1].color
          }

          // 抽選候補パネル
          var candidatePanel = panels.filter(v=>{return (v.color != upPanel && v.color != leftPanel)})
          var panelNum = Math.floor(Math.random() * candidatePanel.length)
          // deepcopy
          tmpRow.push(JSON.parse(JSON.stringify(candidatePanel[panelNum])))
        }
        tmpTable.push(tmpRow)
      }
      this.table = tmpTable
    },
    swap: function(row, column){
      if (column == 0) { return }
      var table = this.table
      var tmp = table[row][column]
      table[row][column] = table[row][column-1]
      table[row][column-1] = tmp
      this.table = table
    },
    deletePanel: function(){
      var table = this.table

      for(var row = 0; row < table.length; row++){
        for(var column = 0; column < table[row].length; column++){

          // 縦チェック
          if(row != 0 && row != table.length-1){
            if (table[row-1][column].color == table[row][column].color && table[row][column].color == table[row+1][column].color){
              table[row-1][column].delFlag = true
              table[row][column].delFlag = true
              table[row+1][column].delFlag = true
            }
          }
          // 横チェック
          if(column != 0 && column != table[row].length-1){
            if (table[row][column-1].color == table[row][column].color == table[row][column+1].color){
              table[row][column-1].delFlag = true
              table[row][column].delFlag = true
              table[row][column+1].delFlag = true
            }
          }

        }
      }
      this.table = table.map(r=>{return r.map(c=>{return c.delFlag?{}:c})})
    },
    dropPanel: function(){
      var table = this.table
      // 一つも落下処理がなかったら完了とする
      var isDone = false
      while(!isDone){
        isDone = true
        // 最下段から最上段-1まで走査
        for(var row = table.length-1; 0 < row; row--){
          for(var column = 0; column < table[row].length; column++){
            //上のパネルが空だった場合はスキップ
            if(!Object.keys(table[row-1][column]).length){ continue }
            if(!Object.keys(table[row][column]).length){
             console.log(table[row][column])
             table[row][column] = table[row-1][column]
             table[row-1][column] = {}
             isDone = false
            }
          }
        }
      }
      this.table = table
    }
  }
}
</script>
