<template>
  <div>
    <div>
      <canvas
        :width="display.width"
        :height="display.height"
        ref="paneponDisplay"
        v-on:click="onClick"
        v-on:mousemove="onMousemove"
       ></canvas>
    </div>
    <div>
      <button
        v-on:click="deletePanel"
        >delete
      </button>
      <button
        v-on:click="dropPanel"
        >drop
      </button>
    </div>
    <div>
      <textarea v-model="tabledataTextarea"></textarea>
    </div>
    <div>
      <button
        v-on:click="importTableData"
        >import
      </button>
      <button
        v-on:click="exportTableData"
        >export
      </button>
    </div>
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
    id: 1,
    color: "green",
    image: ImageGreen,
  },
  {
    id: 2,
    color: "purple",
    image: ImagePurple,
  },
  {
    id: 3,
    color: "red",
    image: ImageRed,
  },
  {
    id: 4,
    color: "sky",
    image: ImageSky,
  },
  {
    id: 5,
    color: "yellow",
    image: ImageYellow,
  },
]

export default {
  data: function(){
    return{
      table: [],
      tabledataTextarea: "",
      display: {
        width: 30 * 6,
        height: 30 * 12,
      },
      mouseMoveEventQueue: {},
      mouseClickEventQueue: {},
      currentCursorPos: [
        {
          column: 2,
          row: 6,
        },
        {
          column: 3,
          row: 6,
        },
      ],
    }
  },
  mounted: function() {
    this.displayCtx = this.$refs.paneponDisplay.getContext('2d')
    this.generateTable()
    this.panelPreload().then(this.mainProc)
  },
  methods:{
    mainProc: function(){
      // this.generateTable()
      // 不思議不具合メモ
      // 毎フレームgenerateTableして遊んでたら、開始後10秒で謎盤面が出るようになった
      // fps変えても10秒後、呼び出し回数でもないし謎
      console.log("mainProc")
      
      this.procCursor()
      this.draw()
      setTimeout(this.mainProc, Math.floor((1/1)*1000)) // Nfps
    },

    // 描画系
    draw: function(){
      const ctx = this.displayCtx
      ctx.clearRect(0,0,this.width,this.height)
      this.drawDisplay(ctx)
      this.drawCursor(ctx, 0, 0)
    },
    drawDisplay: function(ctx){
      const table = this.table
      var img = new Image()
      for (var row = 0; row < table.length; row++)  {
        for (var column = 0; column < table[row].length; column++) {
          img.src = table[row][column].image
          ctx.drawImage(img, column*30, row*30, 30, 30)
        }
      }
    },
    drawCursor: function(ctx){
      this.currentCursorPos.map(pos=>{
        ctx.strokeStyle = "#FFF"
        ctx.lineWidth = 2
        ctx.strokeRect(pos.column*30+2, pos.row*30+2, 30-2,30-2)
      })
    },

    // 処理系
    procCursor: function(){
      const clickEv = this.mouseClickEventQueue 
      const moveEv = this.mouseMoveEventQueue
      this.mouseClickEventQueue = {}
      this.mouseMoveEventQueue = {}

      if (!this.isEmpty(clickEv)){
        console.log("click", clickEv)
      }
      if (!this.isEmpty(moveEv)){
        console.log("move", moveEv)
      }
    },
    onClick: function(e){
      this.mouseClickEventQueue = e
    },
    onMousemove: function(e){
      this.mouseMoveEventQueue = e
    },
    loadImage: function(imgSrc){
      return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.onerror = (e) => reject(e);
        img.src = imgSrc;
      })
    },
    panelPreload: function(){
      return Promise.all(
        panels.map(p=>{return this.loadImage(p.image)})
      )
    },
    generateTable: function(){
      const tmpTable = []
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
          const candidatePanel = panels.filter(v=>{return (v.color != upPanel && v.color != leftPanel)})
          const panelNum = Math.floor(Math.random() * candidatePanel.length)
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

          if(this.isEmpty(table[row][column])) {continue}
          // 縦チェック
          if(0 < row && row < table.length -1){
            if (
              !this.isEmpty(table[row-1][column]) &&
              !this.isEmpty(table[row+1][column]) &&
              table[row-1][column].color == table[row][column].color &&
              table[row][column].color == table[row+1][column].color
            ){
              table[row-1][column].delFlag = true
              table[row][column].delFlag = true
              table[row+1][column].delFlag = true
            }
          }
          // 横チェック
          if(0 < column && column < table[row].length-1){
            if (
              !this.isEmpty(table[row][column-1]) &&
              !this.isEmpty(table[row][column+1]) &&
              table[row][column-1].color == table[row][column].color &&
              table[row][column].color == table[row][column+1].color
            ){
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
            if(this.isEmpty(table[row-1][column])) { continue }
            if(this.isEmpty(table[row][column])){
             table[row][column] = table[row-1][column]
             table[row-1][column] = {}
             isDone = false
            }
          }
        }
      }
      this.table = table
    },
    isEmpty: function(obj){
      return !Object.keys(obj).length
    },
    exportTableData: function(){
      const exportTableData = []
      this.table.map(r=>{
        const row = []
        r.map(c=>{
          if(c.id == undefined) {
            row.push(9) 
          }else{
            row.push(c.id)
          }
          console.log(row)
        })
        exportTableData.push(parseInt(row.join('')).toString(36))
      })
      this.tabledataTextarea = exportTableData.join('_')
    },
    importTableData: function(){
      this.table = this.importTableDataDecode(this.tabledataTextarea)
    },
    importTableDataDecode: function(data){
      if (data == "") {
        return
      }
      const tabledata = []
      const splitedData = data.split('_')

      splitedData.map(r=>{
        console.log(r)
        const tmp = []
        const raw = parseInt(r,36).toString().split('')
        raw.map(c=>{
          const panel = panels.find(v=>{
            return v.id == c
          })
          if (panel === undefined){
            tmp.push({})
          }else{
            tmp.push(JSON.parse(JSON.stringify(panel)))
          }
        }) 
        tabledata.push(tmp)
      })
      return tabledata
    }
  }
}
</script>
