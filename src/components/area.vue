<template>
  <div class="area">
    <button v-on:click="resetGame">Новая игра</button><span v-show="glPlayerEnd > 0">{{ playerEndStr }}</span>
    <div class="srow" v-for="(row, rowInd) in matrixObj" :key="row">
      <img v-for="(col, colInd) in row" :key="col" :src="getImgFromRowCol(rowInd,colInd)" 
        @click.left="leftMouse(rowInd,colInd)" @click.right="rightMouse(rowInd,colInd, $event)" :style="getBgColorFromRowCol(rowInd,colInd)" />
    </div>
  </div>
</template>
<script>
const LEN_AREA = 3;
export default {
  name: 'areacomp',
  data() {
    return {
      matrixObj: {
        row0: {c0:0,c1:0,c2:0},
        row1: {c0:0,c1:0,c2:0},
        row2: {c0:0,c1:0,c2:0},
      },
      styleObj: {
        row0: {c0:'white',c1:'white',c2:'white'},
        row1: {c0:'white',c1:'white',c2:'white'},
        row2: {c0:'white',c1:'white',c2:'white'},
      },
      resKindObj: {//Вид завершения и координаты строки/столбца
        kind: 0, //kind = 1 - проверка строк; row - строка с совпадением, kind = 2 - Проверка столбцов; col - столбец с совпадением
        //kind = 3 - Проверка диагонали лево верх, право низ; col, row - начальное совпадение,
        //kind = 4 - Проверка диагонали право верх, лево низ; col, row - начальное совпадение
        col:'', row:''
      },
      tempCount: 0,
      countEmpty: 0,
      glPlayerEnd: 0,//0 - Не конец, 1 - выиграл крест, 2 - выиграл 0, 3 - ничья
      svgZ: require('@/assets/zero1.svg'),
      svgC: require('@/assets/cross1.svg'),
      svgE: require('@/assets/empty1.svg'),
    }
  },
  computed: {
    playerEndStr() {
      let ret = '';
      switch (this.glPlayerEnd) {
        case 0:
          ret = '';
        break;
        case 1:
          ret = 'Выиграл крест';
        break;
        case 2:
          ret = 'Выиграл 0';
        break;
        case 3:
          ret = 'Победила дружба!';
        break;
      }
      return ret;
    }
  },
  methods: {
    getImgFromRowCol(row,col) {
      let val = this.matrixObj[row][col];
      let ret = this.svgE;
      switch (val) {
        case 0: ret = this.svgE;//empty
          break;
        case 1: ret = this.svgC;//cross
          break;
        case 2: ret = this.svgZ;//zero
          break;
      }
      return ret;
    },
    getBgColorFromRowCol(row,col) {
      return 'background-color:'+this.styleObj[row][col];
    },
    resetGame() {
      this.matrixObj.row0 = {c0:0,c1:0,c2:0};
      this.matrixObj.row1 = {c0:0,c1:0,c2:0};
      this.matrixObj.row2 = {c0:0,c1:0,c2:0};
      this.glPlayerEnd = 0;
      this.resKindObj.kind = 0;
      this.resKindObj.col = ''; this.resKindObj.row = ''; 
      this.styleObj.row0 = {c0:'white',c1:'white',c2:'white'};
      this.styleObj.row1 = {c0:'white',c1:'white',c2:'white'};
      this.styleObj.row2 = {c0:'white',c1:'white',c2:'white'};
    },

    checkRows(playerEnd) {
      //Проверка строк
      for (let row in this.matrixObj) {//Идём по строкам
        let start1 = this.matrixObj[row].c0;
        this.tempCount = 0;
        this.resKindObj.row = row;//Строка с совпадением
        for (let cell in this.matrixObj[row]) {//Идём по ячейкам
          if (start1 > 0 && start1 == this.matrixObj[row][cell]) {
            this.tempCount = this.tempCount + 1;
          }
          if (this.matrixObj[row][cell] == 0) {//Если остались пустые ячейки
            this.countEmpty = this.countEmpty + 1;
          }
        }
        if (this.tempCount > (LEN_AREA - 1)) {
          playerEnd = start1;
          this.resKindObj.kind = 1;//проверка строк
          break;
        }
      }
      return playerEnd;
    },
    checkCols(playerEnd) {
      //Проверка столбцов
      if (playerEnd == 0) {
        this.tempCount = 0;
        for (let i=0;i<LEN_AREA;i++) {//Идём по столбцам
          let colName = 'c'+i;
          let start1 = this.matrixObj.row0[colName];
          this.tempCount = 0;
          this.resKindObj.col = colName;//Столбец с совпадением
          for (let j=0;j<LEN_AREA;j++) {//Идём по ячейкам по вертикали
            let rowName = 'row'+j;
            if (start1 > 0 && start1 == this.matrixObj[rowName][colName]) {
              this.tempCount = this.tempCount + 1;
            }
          }
          if (this.tempCount > (LEN_AREA - 1)) {
            playerEnd = start1;
            this.resKindObj.kind = 2;//Проверка столбцов
            break;
          }
        }
      }
      return playerEnd;
    },
    checkDiag1(playerEnd) {
      //Проверка диагоналей
      if (playerEnd == 0) {
        this.tempCount = 0;
        let start1 = this.matrixObj.row0.c0;
        this.resKindObj.row = 'row0';
        this.resKindObj.col = 'c0';
        for (let i=0;i<LEN_AREA;i++) {
          let colName = 'c'+i;
          let rowName = 'row'+i;
          if (start1 > 0 && start1 == this.matrixObj[rowName][colName]) {
            this.tempCount = this.tempCount + 1;
          }
        }
        if (this.tempCount > (LEN_AREA - 1)) {
          playerEnd = start1;
          this.resKindObj.kind = 3;//Проверка 1й диагонали
        }
      }
      return playerEnd;
    },
    checkDiag2(playerEnd) {
      if (playerEnd == 0) {
        this.tempCount = 0;
        let start1 = this.matrixObj.row0.c2;
        let k1 = LEN_AREA - 1;
        this.resKindObj.row = 'row0';
        this.resKindObj.col = 'c2';
        for (let i=0;i<LEN_AREA;i++) {
          let colName = 'c'+k1;
          let rowName = 'row'+i;
          if (start1 > 0 && start1 == this.matrixObj[rowName][colName]) {
            this.tempCount = this.tempCount + 1;
          }
          k1 = k1 - 1;
        }
        if (this.tempCount > (LEN_AREA - 1)) {
          playerEnd = start1;
          this.resKindObj.kind = 4;//Проверка 2й диагонали
        }
      }
      return playerEnd;
    },

    checkGameState() {
      let playerEnd = 0;//0 - Не конец, 1 - выиграл крест, 2 - выиграл 0, 3 - ничья
      
      this.tempCount = 0;
      this.countEmpty = 0;
      
      playerEnd = this.checkRows(playerEnd);
      playerEnd = this.checkCols(playerEnd);
      playerEnd = this.checkDiag1(playerEnd);
      playerEnd = this.checkDiag2(playerEnd);
      
      //Проверка на пустоту
      if (playerEnd == 0 && this.countEmpty == 0) {
        playerEnd = 3;//Ничья
      }
      return playerEnd;
    },
    setExellentGame() {
      //kind = 1 - проверка строк; row - строка с совпадением, kind = 2 - Проверка столбцов; col - столбец с совпадением
        //kind = 3 - Проверка диагонали лево верх, право низ; col, row - начальное совпадение,
        //kind = 4 - Проверка диагонали право верх, лево низ; col, row - начальное совпадение
      let row = this.resKindObj.row;
      let col = this.resKindObj.col;
      let k1 = LEN_AREA - 1;
      switch (this.resKindObj.kind) {
        case 1:
          for (let i=0;i<LEN_AREA;i++) {
            this.styleObj[row]['c'+i] = 'coral';
          }
        break;
        case 2:
          for (let i=0;i<LEN_AREA;i++) {
            this.styleObj['row'+i][col] = 'coral';
          }
        break;
        case 3:
          for (let i=0;i<LEN_AREA;i++) {
          let colName = 'c'+i;
          let rowName = 'row'+i;
            this.styleObj[rowName][colName] = 'coral';
          }
        break;
        case 4:
          for (let i=0;i<LEN_AREA;i++) {
          let colName = 'c'+k1;
          let rowName = 'row'+i;
            this.styleObj[rowName][colName] = 'coral';
          k1 = k1 - 1;
          }
        break;

      }
    },
    leftMouse(rowInd,colInd) {
      let val = this.matrixObj[rowInd][colInd];
      if (val == 0) {
        this.matrixObj[rowInd][colInd] = 1;//cross
      } 
      //Проверка на совпадения. Горизонтали, вертикали, диагонали
      this.glPlayerEnd = this.checkGameState();
      if (this.glPlayerEnd > 0 && this.glPlayerEnd < 3) {
        this.setExellentGame();
      }
    },
    rightMouse(rowInd,colInd, event) {
      if (event) {
        event.preventDefault();
      }
      let val = this.matrixObj[rowInd][colInd];
      if (val == 0) {
        this.matrixObj[rowInd][colInd] = 2;//zreo
      } 
      //Проверка на совпадения. Горизонтали, вертикали, диагонали
      this.glPlayerEnd = this.checkGameState();
      if (this.glPlayerEnd > 0 && this.glPlayerEnd < 3) {
        this.setExellentGame();
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only 
  <u>{{ row.c0 }}, {{ ind }}</u>  @contextmenu.native="handler"
  display: flex; flex-direction: row; justify-content: space-between; flex-wrap: wrap;
-->
<style scoped>
  .area {
    width: 600px; height: 650px; margin-left: auto; margin-right: auto;
    border-top: 2px solid silver; border-left: 2px solid silver;
    border-right: 2px solid gray; border-bottom: 2px solid gray;
    
  }
  .srow {
    width: 100%; height: 200px; margin: 0px 0px 0px 0px; padding: 0px 0px 0px 0px;
    display: flex; flex-direction: row; justify-content: space-between;
  }
  .extellent {
    background-color:coral;
  }
</style>
