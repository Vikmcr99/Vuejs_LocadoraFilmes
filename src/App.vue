<template>
  <div id="app" class="container">
    <h1 v-html="title"></h1>
    <h3 v-if= "horas >= 6 && horas <24" id="aberta">ABERTA!</h3>
    <h3 v-else id="fechada"> FECHADA!</h3>
    <div class="row">
      <div class="col">
        <button type="button" class="btn btn-dark"
          @click="mostrarCarrinho">
          Carrinho: {{quantidadeNoCarrinho}} filmes
          </button>
      </div>
    </div>

    
    <div class="row" v-if="mostrarFilmes">
      <div class="col-4" v-bind:key="filme.id" v-for="filme in filmes">
        <div class="card">
          <img v-bind:src="filme.imagem" class="card-img-top" alt="Imagem do Filme">
          <div class="card-body">
            <h5 class="card-title" v-html="filme.titulo"></h5>
            <p class="card-text" v-html="filme.descricao"></p>
            <span class="mensagem-estoque"
            v-if="filme.estoqueDisponivel - quantidadeNoCarrinhoPorFilme(filme) === 0">
            indisponível!
            </span>
            <span class="mensagem-estoque"
            v-else-if="filme.estoqueDisponivel - quantidadeNoCarrinhoPorFilme(filme) < 5">
              {{filme.estoqueDisponivel - quantidadeNoCarrinhoPorFilme(filme)}} itens no estoque.
            </span>
            <span class="mensagem-estoque"
              v-else>
                Alugue agora!
            </span>
            <p class="card-text">{{filme.valor | formatarPreco("R$:")}}</p>
            <div class="avaliacao">
              <span v-for="n in 5" :key="n"
                v-bind:class="{'avaliacao-active': checarAvaliacao(n, filme)}">
                <img src="./assets/star.png" height="15"/>
              </span>
            </div>
            <a href="#"
             @click="adicionarAoCarrinho(filme)"
             v-if="validarPermissaoParaAdicionarNoCarrinho(filme)" 
             class="btn btn-dark" id="btn">ALUGAR</a>
             <a href="#" v-else class="btn btn-primary disabled">ALUGAR</a>
          </div>
        </div>
      </div>
    </div>
    <div class="row" v-else>
      <h2>DADOS PESSOAIS:</h2>

      <div class="col-12">
        <form>
          <div class="form-group">
            <label for="pedido.primeiroNome"><b>Primeiro Nome:</b></label>
            <input 
            type="text"
            class="form-control"
            id="primeiroNome"
            placeholder="Digite o primeiro nome:"
            v-model.trim.lazy="pedido.primeiroNome"
            >
          </div>
          <div class="form-group">
            <label for="ultimoNome"><b>Ultimo Nome:</b></label>
            <input 
            type="text"
            class="form-control"
            id="ultimoNome"
            placeholder="Digite o ultimo nome:"
            v-model.trim.lazy="pedido.ultimoNome"
            >
          </div>

          <div class="form-group">
           <label for="endereco"><b>Endereço:</b></label>
           <input
             type="text"
             class="form-control"
             id="endereco"
             placeholder="Digite o endereço"
             v-model.trim.lazy="pedido.endereco"
           >
         </div>
         <div class="form-group">
           <label for="cidade"><b>Cidade:</b></label>
           <input
             type="text"
             class="form-control"
             id="cidade"
             placeholder="Digite a cidade"
             v-model.trim.lazy="pedido.cidade"
           >
         </div>
         <div class="form-group">
           <label for="estado"><b>Estado:</b></label>
           <select class="form-control" id="estado" v-model="pedido.estado">
             <option disabled value>Escolha um Estado</option>
             <option 
             v-for="(estado, key) in estados"
             v-bind:value="estado"
             v-bind:key="key">
             {{key}}
             </option>
           </select>
         </div>
         <div class="form-group">
           <label for="cep"><b>CEP:</b></label>
           <input
             type="number"
             class="form-control"
             id="cep"
             placeholder="Digite o CEP"
             v-model.number="pedido.cep"
             >
         </div>

         <div class="form-group form-check">
           <input
             type="checkbox"
             class="form-check-input"
             id="pagoNaEntrega"
             v-bind:true-value="pedido.simNaEntrega"
             v-bind:false-value="pedido.naoNaEntrega"
             v-model="pedido.pagoNaEntrega"
           >
           <label class="form-check-label" for="pagoNaEntrega">Pago na entrega?</label>
         </div>
         <div class="form-group form-check-inline">
           <input
             type="radio"
             class="form-check-input"
             id="manha"
             value="Manhã"
             v-model="pedido.entrega"
           >
           <label class="form-check-label" for="manha">Manhã</label>
         </div>
         <div class="form-group form-check-inline">
           <input
             type="radio"
             class="form-check-input"
             id="tarde"
             value="Tarde"
             v-model="pedido.entrega"
           >
           <label class="form-check-label" for="tarde">Tarde</label>
         </div>
         <div class="form-group form-check-inline">
           <input
             type="radio"
             class="form-check-input"
             id="noite"
             value="Noite"
             v-model="pedido.entrega"
           >
           <label class="form-check-label" for="noite">Noite</label>
         </div>
         
         <div class="form-group">
           <button type="submit" class="btn btn-primary" @click="submitFormulario">
             Finalizar pedido
           </button>
         </div>
       </form>
     </div>
  <div class="col-12">
        <pre>
          Primeiro nome: {{pedido.primeiroNome}}
          Ultimo nome: {{pedido.ultimoNome}}
          Endereço: {{ pedido.endereco }}
          Cidade: {{ pedido.cidade }}
          Estado: {{ pedido.estado }}
          CEP: {{ pedido.cep }}
          Pago na Entrega?: {{pedido.pagoNaEntrega}}
          Entrega: {{pedido.entrega}}
        </pre>
      </div>
    </div>
  </div>
  
</template>

<script>
export default {
  name: 'App',
  data: function(){
    return{
      mostrarFilmes:true,
      title:"<b>LOCADORA DE FILMES<b>",
      horas: new Date().getHours(),
      pedido: {
       primeiroNome: '',
       ultimoNome: '',
       endereco: "",
       cidade: "",
       estado: "",
       cep: "",
       pagoNaEntrega: "Não",
       simNaEntrega:"Sim",
       naoNaEntrega:"Não",
       entrega: "Manhã"
      },

      estados:{
        RJ:"Rio de Janeiro",
        MG:"Minas Gerais",
        SP:"São Paulo",
        ES:"Espirito Santo"
      },
      
      filmes: [
        {id:1, titulo:"<b>SENHOR DOS ANEIS</b>", descricao: "<b>Melhor trilogia</b>", valor: 25, imagem: "https://i0.wp.com/www.jornadageek.com.br/wp-content/uploads/2017/11/O-Senhor-dos-An%C3%A9is.jpg?resize=696%2C398&ssl=1", estoqueDisponivel:3, avaliacao: 5},
        {id:2, titulo:"<b>STAR WARS</b>", descricao: "<b>Melhor saga espacial</b>", valor: 20, imagem: "https://www.jornalopcao.com.br/wp-content/uploads/2019/12/Star-Wars-Epis%C3%B3dio-IX-Ascens%C3%A3o-Skywalker-um-620x363.jpg", estoqueDisponivel: 2, avaliacao: 4},
        {id:3, titulo:"<b>DE VOLTA PARA O FUTURO</b>", descricao: "<b>Melhor filme de viagem no tempo</b>", valor: 30, imagem: "https://cdn.cinepop.com.br/2019/08/back-to-the-future-2015-wallpaper-1.jpg", estoqueDisponivel:4, avaliacao: 5},
        {id:4, titulo:"<b>CAVALEIRO DAS TREVAS</b>", descricao: "<b>Melhor coringa</b>", valor: 25, imagem: "https://timeline.canaltech.com.br/24178.700/batman-o-cavaleiro-das-trevas-e-eleito-melhor-filme-de-super-heroi-da-historia.jpg", estoqueDisponivel: 4, avaliacao: 5},
        {id:5, titulo:"<b>INTERESTELAR</b>", descricao: "<b>Melhor filme sobre viagem espacial</b>", valor: 35, imagem: "https://assets.b9.com.br/wp-content/uploads/2015/03/inter.jpeg", estoqueDisponivel:1, avaliacao: 3},
        {id:6, titulo:"<b>1917</b>", descricao: "<b>Melhor filme de guerra</b>", valor: 40, imagem: "https://www.elfinanciero.com.mx/uploads/2020/01/05/a981f2b7081578284288_standard_desktop_medium_retina.webp", estoqueDisponivel:3, avaliacao: 5}
      ],
      carrinho: []
    };
  },
      methods: {

        checarAvaliacao(n, filme){
          return filme.avaliacao -n >= 0;
        },

        submitFormulario(){
          alert('Pedido finalizado com sucesso!');
        },
        mostrarCarrinho(){
          this.mostrarFilmes = this.mostrarFilmes ? false:true;
        },
        adicionarAoCarrinho: function(filme) {
          this.carrinho.push(filme.id);
        },
        quantidadeNoCarrinhoPorFilme: function(filme) {
          var quantidade = 0;
          for(var i = 0; i < this.carrinho.length; i++){
            if(filme.id == this.carrinho[i]){
              quantidade++;
            }
          }
          return quantidade;
        },
        validarPermissaoParaAdicionarNoCarrinho: function(filme) {
          return filme.estoqueDisponivel > this.quantidadeNoCarrinhoPorFilme(filme);
        }
      
      },

      computed: {
    quantidadeNoCarrinho: function() {
      return this.carrinho.length;
    },
}
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: black;
  margin-top: 30px;
  border-radius: 10px;
  -moz-border-radius: 8px;
  -webkit-border-radius: 8px;
  
  
}

#aberta{
  color:rgb(47, 151, 47)
}

#fechada{
  color: red;
}

#btn{
  color: white;
}

.mensagem-estoque{
  font-weight: bold;
}

span.avaliacao-active{
  background-image: url('./assets/star-fill.png');
  background-repeat: no-repeat;
  background-position-y:3px;
  background-position-x: 1px;
  background-size: 15px;
}

body{
  background-size: 100%;
  background-image: url(./assets/fundo2.png);
}

</style>
