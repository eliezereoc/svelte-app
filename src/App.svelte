<script> 

  let contador = 0;// Incrementada a cada novo audio criado
  let gravando = false;// Alterna em false e true indicando se esta gravando ou não (para html)
  let gravacoes = [];// Recebe os aurdio gravado
  let mediaRecorder = null;// Recebe o strem de audio

  //Solicita ao usuario permissão para utilizar o sudio 
  navigator.mediaDevices.getUserMedia({audio: true})//Permite apenas audio 
    .then(stream => {

      // Para gravar áudio deve-se criar um MediaRecorder passando o stream
      mediaRecorder = new MediaRecorder(stream);

      let chunks = [];
      
      
      mediaRecorder.ondataavailable = function(e) {         
        // A cada evento dataavailable, uma parte do áudio é disponibilizada.
        // Deve-se armazenar os dados do evento para depois juntar todas as
        // partes no arquivo de áudio final.
        chunks.push(e.data);
      }

      //Ao fim de cada gravação, adiciona o adio a lista
      mediaRecorder.onstop = function(e) {

        // No evento stop, é possivel juntar as partes do áudio usando
        // um objeto Blob, especificando o tipo audio/ogg
        const blob = new Blob(chunks, { 'type' : 'audio/ogg; codecs=opus' });

        chunks = [];// Cria um outro array chunks para ser utilizado na próxima gravação

        // Transforma o Blob em uma URL de dados, para usá-la como src de um elemento <audio>.
        const audioURL = URL.createObjectURL(blob);

        //lista de audio, a lista é criada dessa forma para indicar 
        //ao svelt que ocorreu uma alteração e assim atualizar a pagina
        gravacoes = [...gravacoes, {id: ++contador, audioURL: audioURL}];
      }

    });

  function iniciaGravacao() {
      gravando = true;// Indica para o usuario que esta gravando (np html)     
      mediaRecorder.start(); // Esta função deve ser chamada para iniciar a gravação.
  }

  function paraGravacao() {
    gravando = false;// Indica para o usuario que não esta gravando (np html)
    mediaRecorder.stop();// E esta outra para parar a gravação.
  }

  function excluiGravacao(gravacao) {
    //Exclui o audio celecionado 
    gravacoes = gravacoes.filter(gr => gr !== gravacao);
  }
</script>

<main>
  <h1>Gravador de áudio</h1>

  <div class="controles">
    <button disabled={mediaRecorder === null || gravando} on:click={iniciaGravacao}>Gravar</button>
    <button disabled={!gravando} on:click={paraGravacao}>Parar</button>
    {#if gravando}
      <span class="gravando">Gravando</span>
    {:else}
      <span>Aguardando</span>
    {/if}
  </div>

  <div class="gravacoes">
    {#each gravacoes as gravacao}
      <div class="gravacao">
        <span>Gravação {gravacao.id}</span>
        <audio src={gravacao.audioURL} controls>
          Elemento audio não suportado
        </audio>
        <button on:click={() => excluiGravacao(gravacao)}>Excluir</button>
      </div>
    {/each}
  </div>

</main>

<style>
  main {
    background-color: white;
    padding: 1rem;
    max-width: 700px;
    margin: auto;
    text-align: center;
  }
  .gravacao {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  .controles button {
    margin-right: 1rem; 
  }
  .controles {
    margin-bottom: 2rem;
  }
  .gravando {
    color: red;
    font-weight: bold;
    animation: blinker 1.5s linear infinite;
  }

  @keyframes blinker {
    0% {
      color: red;
    }
    50% {
      color: rgba(255, 0, 0, 0.2);
    }
    100% {
      color: red;
    }
  }
</style>