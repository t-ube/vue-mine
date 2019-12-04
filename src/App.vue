<template>
  <div id="app">
    <h1>Simple Vue Mine</h1>
    <p v-if="gameover" :class="{failed: true}">Game Over</p>
    <p v-else-if="complete" :class="{success: true}">Clear</p>
    <p v-else>Please click green cell</p>
    <div id="map">
      <div v-for="cell in grid_size*grid_size"
       v-on:click="dig_cell(cell-1)"
       v-bind:key="cell.id"
       :class="{
         cell: true,
         bomb: is_bomb[cell - 1] === 1 && gameover,
         near: surround_bomb[cell - 1] > 0,
         dig: is_dig[cell - 1] == 1,
         flag: is_dig[cell - 1] == 2
         }">
         <span>{{surround_bomb[cell - 1]}}</span>
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
      surround_bomb: [],
      is_bomb: [],
      is_dig: [],
      gameover: false,
      tool: 'dig',
      complete: false
    }
  },
  // 初期化
  created() {
      for(let n = 0; n <(this.grid_size*this.grid_size); n++){
        this.surround_bomb[n] = 0;
        this.is_bomb[n] = 0;
        this.is_dig[n] = 0;
      }
      this.randomize_bomb_index();
      this.set_surrounding_bombs_count();
  },
  methods: {
    select_tool(tool){
      this.tool = tool;
    },
    check_complete() {
      var result = this.is_dig.filter(function(value) {
          return value == 0;
      })
      return (result.length < 1);
    },
    dig_cell(index) {
      if(this.tool == 'dig'){
        if(this.is_bomb_index(index)){
          this.bomb(index);
        }else{
          this.$set(this.is_dig, index, 1);
        }
      }else{
        this.$set(this.is_dig, index, 2);
      }
      this.complete = this.check_complete();
    },
    bomb(){
      this.gameover = true;
    },
    randomize_bomb_index() {
      let bomb_index = Math.floor(Math.random() * this.grid_size * this.grid_size);
      let bomb_index2 = Math.floor(Math.random() * this.grid_size * this.grid_size);
      for(let n = 0; n <(this.grid_size*this.grid_size); n++){
        if(bomb_index == n || bomb_index2 == n){
          this.$set(this.is_bomb, n, 1);
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
      return (this.is_bomb[index] == 1);
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
      for(let n = 0; n <(this.grid_size*this.grid_size); n++){
        let counts = this.count_surrounding_bombs(n);
        if(counts > 0){
          this.$set(this.surround_bomb, n, counts);
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
