<template>
  <div id="app">
    <h1>Canvas Demo</h1>
    <div>
      <label for="firstName">Selecione o seu nome: </label>
      <select name="firstName" id="firstName" v-model="selectedName">
        <option v-for="(name, index) in deployeeNames" :key="index" :value="name">{{name}}</option>
      </select>
    </div>    

    <div>
      <label for="period">Selecione qual horário está assinando: </label>
    <select name="period" id="period" v-model="periodSelected">
      <option value="entrada">Entrada</option>
      <option value="saida">Saída</option>
    </select>
    </div>

    <canvas ref="canvas" id="drawing-pad"></canvas>

    <div>
        <button @click='resetCanvas' >Limpar</button>
        <button @click='saveSignature' >Salvar assinatura</button>  
    </div>

    <img :src="imageData" alt="" />
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      canvas: null,
      context: null,
      isDrawing: false,
      startX: 0,
      startY: 0,
      points: [],
      deployeeNames: [],
      selectedName: null,
      imageData: null,
      periodSelected: null
    }
  },
  mounted(){
      var vm = this
      vm.canvas = vm.$refs.canvas
      vm.context = vm.canvas.getContext("2d");

      vm.canvas.addEventListener('mousedown', vm.mousedown);
      vm.canvas.addEventListener('mousemove', vm.mousemove);
      document.addEventListener('mouseup', vm.mouseup);

      this.getDeployeesName();
  },
  methods:{
    mousedown(){
      var vm = this
      vm.isDrawing = true;
    },
    mousemove(e){
      let vm = this

      if (!vm.isDrawing) return;

      let mousePos = this.getMousePos(vm.canvas, e);       
      vm.context.lineWidth = 1;
      vm.context.lineCap = 'round';
      vm.context.strokeStyle = "rgba(0,0,0,1)";
      vm.context.lineTo(mousePos.x, mousePos.y);
      vm.context.stroke();

      vm.startX = mousePos.x;
      vm.startY = mousePos.y;

      vm.points.push({
          x: mousePos.x,
          y: mousePos.y
      });
      
    },
    mouseup(){
      var vm = this
      vm.isDrawing = false;
      if (vm.points.lenght > 0){
          localStorage['points'] = JSON.stringify(vm.points);
      }
    },
    resetCanvas(){
      var vm = this
      vm.points = []; //reset points array
      vm.context.clearRect(0,0, vm.canvas.width, vm.canvas.height);
    },
    saveSignature(){
      let vm = this
      let dataURL = vm.canvas.toDataURL();
      
      this.imageData = dataURL;
      let today = new Date(Date.now()); 
      let todayFormatted = this.formatDate(today);
      let time = today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();

      axios.post(`https://sheetdb.io/api/v1/rreazbkpzzvpn`,
      {
        "data": {
          "nome": this.selectedName,
          "dia": todayFormatted,          
          "assinatura_entrada": this.periodSelected.toLowerCase() === 'entrada' ? dataURL : '',
          "horario_entrada" : this.periodSelected.toLowerCase() === 'entrada' ? time : '',
          "assinatura_saida": this.periodSelected.toLowerCase() === 'saida' ? dataURL : '',
          "horario_saida" : this.periodSelected.toLowerCase() === 'saida' ? time : '',
        }
      }, 
      {
        "auth": {
          "username": "mzh7sk6c",
          "password": "vv4n1yy4fj02sv5ulffw"
        }
      })
      .then(response => {
        console.log(response);
      });      
    },
    getMousePos(canvas, evt) {
      var rect = canvas.getBoundingClientRect();
      return {
          x: (evt.clientX - rect.left) / (rect.right - rect.left) * canvas.width,
          y: (evt.clientY - rect.top) / (rect.bottom - rect.top) * canvas.height
      };
    },
    getDeployeesName(){
      axios.get('https://sheetdb.io/api/v1/rreazbkpzzvpn?sheet=nomesDaSemana',        
      {
        "auth": {
          "username": "mzh7sk6c",
          "password": "vv4n1yy4fj02sv5ulffw"
        }
      })
      .then(response => {
        console.log(response.data);
        let deployees = response.data;
        deployees.forEach(element => {
          if (element.nome)
            this.deployeeNames.push(element.nome);
        });

        //removendo duplicados, caso tenha
        this.deployeeNames = [...new Set(this.deployeeNames)];
      });
    },
    formatDate(date){
      let day = date.getDate();
      if (day < 10)
        day = '0'+day;
      let month = date.getMonth() + 1;
      if (month < 10)
        month = '0'+month;
      return day + "/" + month + "/" + date.getFullYear();
    }    
  }
}
</script>

<style>
  body{
    padding: 2em;
  } 

  #app {  
    text-align: center;
    margin-top: 60px;
  }  

  canvas{
    width: 90%;
    height: 300px;
    border: 1px solid red;
    cursor: crosshair;
  }
</style>

