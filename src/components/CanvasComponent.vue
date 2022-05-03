<template>
  <div>
    <canvas ref="canvas" id="drawing-pad"></canvas>
    <div>
      <button @click="resetCanvas">Limpar</button>
      <button @click="saveSignature">Salvar assinatura</button>
    </div>
    <div>
      <h3 :class="`${this.msgColor}`">{{ msgSave }}</h3>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "CanvasComponent",
  props: {
    selectedName: null,
    periodSelected: null,
  },
  data() {
    return {
      canvas: null,
      context: null,
      isDrawing: false,
      startX: 0,
      startY: 0,
      points: [],
      msgSave: "",
      msgColor: ""
    };
  },
  mounted() {
    var vm = this;
    vm.canvas = vm.$refs.canvas;
    vm.context = vm.canvas.getContext("2d");

    vm.canvas.addEventListener("mousedown", vm.mousedown);
    vm.canvas.addEventListener("mousemove", vm.mousemove);
    document.addEventListener("mouseup", vm.mouseup);
  },
  methods: {
    mousedown() {
      var vm = this;
      vm.isDrawing = true;
      vm.context.beginPath();
    },
    mousemove(e) {
      let vm = this;

      if (!vm.isDrawing) return;

      let mousePos = this.getMousePos(vm.canvas, e);
      vm.context.lineWidth = 1;
      vm.context.lineCap = "round";
      vm.context.strokeStyle = "rgba(0,0,0,1)";
      vm.context.lineTo(mousePos.x, mousePos.y);
      vm.context.stroke();

      vm.startX = mousePos.x;
      vm.startY = mousePos.y;

      vm.points.push({
        x: mousePos.x,
        y: mousePos.y,
      });
    },
    mouseup() {
      var vm = this;
      vm.isDrawing = false;
      if (vm.points.lenght > 0) {
        localStorage["points"] = JSON.stringify(vm.points);
      }
    },
    getMousePos(canvas, evt) {
      var rect = canvas.getBoundingClientRect();
      return {
        x:
          ((evt.clientX - rect.left) / (rect.right - rect.left)) * canvas.width,
        y:
          ((evt.clientY - rect.top) / (rect.bottom - rect.top)) * canvas.height,
      };
    },
    resetCanvas() {
      var vm = this;
      vm.points.lenght = 0; //reset points array
      vm.context.clearRect(0, 0, vm.canvas.width, vm.canvas.height);
    },
    saveSignature() {

      if(!this.selectedName||!this.periodSelected){
          this.msgSave = 'Por favor, selecione o nome e/ou período';
          this.msgColor = 'error';
          return
      }

      let vm = this;
      let dataURL = vm.canvas.toDataURL();

      this.imageData = dataURL;
      let today = new Date(Date.now());
      let todayFormatted = this.formatDate(today);
      let time =
        today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();

      axios
        .post(
          `https://sheetdb.io/api/v1/rreazbkpzzvpn`,
          {
            data: {
              nome: this.selectedName,
              dia: todayFormatted,
              assinatura_entrada:
                this.periodSelected.toLowerCase() === "entrada" ? dataURL : "",
              horario_entrada:
                this.periodSelected.toLowerCase() === "entrada" ? time : "",
              assinatura_saida:
                this.periodSelected.toLowerCase() === "saida" ? dataURL : "",
              horario_saida:
                this.periodSelected.toLowerCase() === "saida" ? time : "",
            },
          },
          {
            auth: {
              username: "mzh7sk6c",
              password: "vv4n1yy4fj02sv5ulffw",
            },
          }
        )
        .then((response) => {
          if (response.status === 201) {
            this.msgSave = "Assinatura salva com sucesso!";
            this.msgColor = 'success';
            this.resetCanvas();
          } else {
            this.msgSave =
              "Tivemos um erro ao salvar a assinatura, tente novamente";
              this.msgColor = 'error';
          }
        });
    },
    formatDate(date) {
      let day = date.getDate();
      if (day < 10) day = "0" + day;
      let month = date.getMonth() + 1;
      if (month < 10) month = "0" + month;
      return day + "/" + month + "/" + date.getFullYear();
    },
  },
};
</script>

<style scoped>
canvas {
  width: 90%;
  height: 300px;
  border: 1px solid red;
  cursor: crosshair;
  background-color: white;
}
button {
  margin: 5px 20px 0;
  background-color:rgb(0, 0, 114);
  color: rgb(255, 255, 255);
  padding: 7px;
  cursor: pointer;
}
.success{
    color: rgb(5, 129, 5);
}
.error{
    color: rgb(204, 17, 4);
}
</style>