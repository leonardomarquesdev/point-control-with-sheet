# Aplicação de Controle de Ponto com Planilha

## Description
O projeto surgiu de uma necessidade de digitalização de uma folha de ponto em papel para uma planilha. Para realizar essa integração com o GoogleSheets foi utilizado o SheetDB gratuito. Por possuir uma limitação de 500 requisições mensais, foi necessário empregar mecanismos de cache da aplicação usando LocalStorage. Também foi necessário utilizar o Axios para realizar as requisições para a API do SheetDB. 
A solução criada foi oferecer uma área de canvas para que o usuário possa desenhar sua assinatura e salvar na planilha junto com seu nome e período do ponto. O dia e horário são calculados automaticamente. Os dados da assinatura ficam salvos na planilha como data:image/png;base64. 
Pretendo como melhoria criar uma nova área no app para realizar buscas nos funcionários através da data e/ou nome.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
