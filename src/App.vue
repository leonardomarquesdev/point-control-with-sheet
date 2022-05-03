<template>
  <div id="app">
    <h1>Controle de Ponto por Assinatura</h1>
    <div class="signature-input">
      <label for="firstName">Selecione o seu <strong>nome:</strong> </label>
      <select
        class="input-select"
        name="firstName"
        id="firstName"
        v-model="selectedName"
      >
        <option
          v-for="(name, index) in deployeeNames"
          :key="index"
          :value="name"
        >
          {{ name }}
        </option>
      </select>
    </div>

    <div class="signature-input">
      <label for="period"
        >Selecione qual <strong>horário</strong> está assinando:
      </label>
      <select
        class="input-select"
        name="period"
        id="period"
        v-model="periodSelected"
      >
        <option value="entrada">Entrada</option>
        <option value="saida">Saída</option>
      </select>
    </div>

    <CanvasComponent
      :selectedName="selectedName"
      :periodSelected="periodSelected"
    />

    <img :src="imageData" alt="" />
  </div>
</template>

<script>
import axios from "axios";
import CanvasComponent from "./components/CanvasComponent.vue";

export default {
  name: "App",
  components: {
    CanvasComponent,
  },
  data() {
    return {
      deployeeNames: [],
      selectedName: null,
      periodSelected: null,
    };
  },
  mounted() {
    let deployees = JSON.parse(localStorage["deployees"]);

    if (deployees) {
      console.log("Já existia func salvo !")
      deployees.forEach((element) => {
        if (element.nome) this.deployeeNames.push(element.nome);
      });
      return 
    }

    this.getDeployeesName();
  },
  methods: {
    getDeployeesName() {
      console.log("NÃO existia func salvo !")
      axios
        .get("https://sheetdb.io/api/v1/rreazbkpzzvpn?sheet=nomesDaSemana", {
          auth: {
            username: "mzh7sk6c",
            password: "vv4n1yy4fj02sv5ulffw",
          },
        })
        .then((response) => {
          let deployees = response.data;
          localStorage["deployees"] = JSON.stringify(deployees);
          deployees.forEach((element) => {
            if (element.nome) this.deployeeNames.push(element.nome);
          });
          //removendo duplicados, caso tenha
          this.deployeeNames = [...new Set(this.deployeeNames)];
        });
    },
  },
};
</script>

<style>
body {
  padding: 2em;
  font-family: sans-serif;
  background-color: rgb(248, 223, 141);
}

#app {
  text-align: center;
  margin-top: 10px;
}
.signature-input {
  margin-bottom: 7px;
}

.signature-input select {
  margin-left: 10px;
  width: 100px;
  height: 30px;
}
</style>

