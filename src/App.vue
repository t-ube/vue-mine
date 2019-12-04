<template>
  <div id="app">
    <h1>Simple Vue Mine</h1>
    <p v-if="gameover" :class="{failed: true}">Game Over</p>
    <p v-else-if="complete" :class="{success: true}">Clear</p>
    <p v-else>Please click green cell</p>
    <div id="map">
      <div v-for="i in cellcount"
       v-on:click="dig_cell(i-1)"
       v-bind:key="i.id"
       :class="{
         cell: true,
         bomb: cell_state[i - 1].is_bomb == true && gameover,
         near: cell_state[i - 1].surround_bomb > 0,
         dig: cell_state[i - 1].is_dig == 1,
         flag: cell_state[i - 1].is_dig == 2
         }">
         <span>{{cell_state[i - 1].surround_bomb}}</span>
      </div>
    </div>
    <button type="button" name="button1" v-on:click="select_tool('dig')">Dig</button>
    <button type="button" name="button2" v-on:click="select_tool('flag')">Flag</button>
  </div>
</template>

<script>
export default {
  name: 'app',
  components: {
  },
  data () {
    return {
      grid_size: 6,
      cell_state: [],
      gameover: false,
      tool: 'dig',
      complete: false
    }
  },
  computed: {
    cellcount() {
      return (this.grid_size*this.grid_size);
    }
  },
  // 初期化
  created() {
      for(let n = 0; n < this.cellcount; n++){
        let cell_temp = {surround_bomb: 0, is_bomb: false, is_dig: 0};
        this.cell_state[n] = cell_temp;
      }
      this.randomize_bomb_index();
      this.set_surrounding_bombs_count();
  },
  methods: {
    select_tool(tool){
      this.tool = tool;
    },
    check_complete() {
      var result = this.cell_state.filter(function(value) {
          return value.is_dig == 0;
      })
      return (result.length < 1);
    },
    dig_cell(index) {
      if(this.tool == 'dig'){
        if(this.is_bomb_index(index)){
          this.bomb(index);
        }else{
          let cell = this.cell_state[index];
          cell.is_dig = 1;
          this.$set(this.cell_state, index, cell);
        }
      }else{
        let cell = this.cell_state[index];
        cell.is_dig = 2;
        this.$set(this.cell_state, index, cell);
      }
      this.complete = this.check_complete();
    },
    bomb(){
      this.gameover = true;
    },
    randomize_bomb_index() {
      let bomb_index = Math.floor(Math.random() * this.cellcount);
      let bomb_index2 = Math.floor(Math.random() * this.cellcount);
      for(let n = 0; n < this.cellcount; n++){
        if(bomb_index == n || bomb_index2 == n){
          let cell = this.cell_state[n];
          cell.is_bomb = true;
          this.$set(this.cell_state, n, cell);
        }
      }
    },
    get_cell_x(index){
      return (Math.floor(index%this.grid_size));
    },
    get_cell_y(index){
      return (Math.floor(index/this.grid_size));
    },
    get_index(x,y){
      if(x<0 || y<0) return -1;
      else if(x>=this.grid_size || y>=this.grid_size) return -1;
      return ((y*this.grid_size)+x);
    },
    is_bomb_index(index){
      if(index < 0) return false;
      return (this.cell_state[index].is_bomb == true);
    },
    is_bomb_cell(x,y){
      return (this.is_bomb_index(this.get_index(x,y)));
    },
    count_surrounding_bombs(index) {
      let x = this.get_cell_x(index);
      let y = this.get_cell_y(index);
      let counts = 0;
      if(this.is_bomb_cell(x-1,y-1)) counts += 1;
      if(this.is_bomb_cell(x,y-1))   counts += 1;
      if(this.is_bomb_cell(x+1,y-1)) counts += 1;
      if(this.is_bomb_cell(x-1,y))   counts += 1;
      if(this.is_bomb_cell(x+1,y))   counts += 1;
      if(this.is_bomb_cell(x-1,y+1)) counts += 1;
      if(this.is_bomb_cell(x,y+1))   counts += 1;
      if(this.is_bomb_cell(x+1,y+1)) counts += 1;
      return counts;
    },
    set_surrounding_bombs_count () {
      for(let n = 0; n < this.cellcount; n++){
        let counts = this.count_surrounding_bombs(n);
        if(counts > 0){
          this.cell_state[n] = Object.assign({}, this.cell_state[n], {
            surround_bomb: counts
          })
        }
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#map {
  --grid-size: 6;
  display: grid;
  grid-template-columns: repeat(var(--grid-size), 30px);
  grid-template-rows: repeat(var(--grid-size), 30px);
  justify-content: center;
  align-content: end;
  padding: auto;
  margin: auto;
}

.cell {
  border: 1px solid gray;
  background: green;
}

.cell span {
  display: none;
}

.dig {
  background-color: brown;
}

.dig.near span {
  display: block;
  line-height: 30px;
}

.dig.near {
  background-color: #FF0;
}

.bomb {
  background-color: #000;
}

.flag {
  background-color: #F00;
}

button {
  margin: 10px;
  height: 30px;
  width: 60px;
}

.success {
  color: lightgreen;
  font-size: 32px;
}

.failed {
  color: red;
  font-size: 32px;
}

</style>
