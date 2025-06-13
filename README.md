<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Sistema de mensagens</title>
  <style>
    * {
      box-sizing: border-box;
    }
    html, body {
      margin: 0;
      min-height: 100vh;
      background: #0f0f1a;
      font-family: Arial, Helvetica, sans-serif;
      color: #e0e0ff;
      overflow-x: hidden;
      font-size: 16px;
      height: 100%;
    }

    .container {
  display: flex;
  gap: 30px;
  padding: 30px;
  flex-wrap: nowrap;
  width: 100%;
  height: 100vh;
}

.corpo {
  flex-shrink: 0;
  width: 300px;
  background: #1b1b2f;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(144, 134, 235, 0.425);
  display: flex;
  flex-direction: column;
  gap: 15px;
}



.chat {
  flex-grow: 1;
  height: 100%;
  background: #1b1b2f;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(144, 134, 235, 0.425);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}


    .chat-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }

    .chat-header h3 {
      margin: 0;
      color: #e0e0ffa2;
      font-size: 1.2rem;
    }

    .chat-header .count {
      font-size: 0.9em;
      color: #9c9b9b;
    }

    .chat-body {
      flex: 1;
      overflow-y: auto;
      padding-right: 5px;
    }

    ul {
      list-style: none;
      margin: 0;
      padding: 0;
    }

    li {
      background: #292940;
      margin-bottom: 5px;
      padding: 10px;
      border-radius: 5px;
      position: relative;
      font-size: 0.9rem;
      line-height: 1.4;
      color: #e0e0ff;
      border-left: 3px solid rgba(8,123,255,0.65);
    }

    .timestamp {
      position: absolute;
      right: 8px;
      bottom: 4px;
      font-size: 0.6rem;
      color: #9c9b9b;
    }

    .controls, .botoes {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-top: 10px;
    }

    .input_userr, .input_mensagem {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      background: #2c2c3a;
      color: #fff;
      border: 1px solid #e0e0ff;
      border-radius: 5px;
      font-size: 1rem;
    }

    button {
      padding: 10px;
      width: 100%;
      background: rgb(28,56,117);
      color: #fff;
      border: none;
      border-radius: 5px;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.3s;
      font-size: 1rem;
    }

    button:hover {
      background: rgba(8,123,255,0.65);
    }

    h2 {
      text-align: center;
      color: #e0e0ffa2;
      font-size: 1.4rem;
    }

    /* Tablet e menores */
    @media (max-width: 992px) {
      .container {
        flex-direction: column;
        gap: 20px;
        padding: 20px;
      }
      .chat {
        height: 450px;
      }
      .corpo {
        width: 100%;
      }
    }

    /* Celulares */
    @media (max-width: 600px) {
      body {
        font-size: 15px;
      }

      .container {
        padding: 15px;
      }

      .chat, .corpo {
        padding: 20px;
      }

      h2, .chat-header h3 {
        font-size: 1.2rem;
      }

      button {
        font-size: 0.95rem;
      }

      .input_userr, .input_mensagem {
        font-size: 0.95rem;
      }
    }

    /* Celulares muito pequenos */
    @media (max-width: 400px) {
      body {
        font-size: 14px;
      }

      .chat {
        height: 400px;
      }

      .chat-header h3 {
        font-size: 1rem;
      }

      .timestamp {
        font-size: 0.55rem;
      }

      button {
        padding: 8px;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <!-- Painel lateral -->
    <div class="corpo">
      <h2>Chat dos Chats</h2>
      <p id="usuarios_online">Usuários online: ...</p>
      <div class="controls">
        <input id="input_userr" class="input_userr" placeholder="Digite seu nome..." />
      </div>
      <div class="botoes">
        <button onclick="confirmRemoveLast()">Apagar Última mensagem</button>
        <button onclick="confirmRemoveAll()">Limpar Tudo</button>
        <button onclick="recarregar()">Atualizar</button>
      </div>
    </div>

    <!-- Área de chat -->
    <div class="chat">
      <div class="chat-header">
        <h3>Mensagens Enviadas</h3>
        <span class="count" id="msg_count">0</span>
      </div>
      <div class="chat-body">
        <ul id="lista_mensagens"></ul>
      </div>
      <div class="controls">
        <input id="input_mensagem" class="input_mensagem" placeholder="Digite sua mensagem..." />
        <button onclick="enviar_mensagem()">Enviar</button>
      </div>
    </div>
  </div>

  <script>
    function mostrar_usuarios_online() {
      fetch("http://vianetminas.com.br/mensagens/quantos_online.php")
        .then(res => res.json())
        .then(data => {
          document.getElementById("usuarios_online").textContent =
            "Usuários online: " + data.total;
        });
    }
    setInterval(mostrar_usuarios_online, 5000);

    let mensagens = [];

    function atualizar_tela() {
      const ul = document.getElementById("lista_mensagens");
      ul.innerHTML = "";
      mensagens.forEach(({ text, time }, i) => {
        const li = document.createElement("li");
        li.textContent = `${i+1}. ${text}`;
        const ts = document.createElement("div");
        ts.className = "timestamp";
        ts.textContent = time;
        li.appendChild(ts);
        ul.appendChild(li);
      });
      document.getElementById("msg_count").textContent = mensagens.length;
      ul.scrollTop = ul.scrollHeight;
    }

    function enviar_mensagem() {
      const inpMsg = document.getElementById("input_mensagem");
      const inpUser = document.getElementById("input_userr");
      const texto = inpMsg.value.trim();
      const usuario = inpUser.value.trim();
      if (!usuario || !texto) {
        alert("Nome e mensagem obrigatórios");
        return;
      }
      if (usuario.length > 100 || texto.length > 100) {
        alert("Máximo 100 caracteres");
        return;
      }
      const fullText = `${usuario}: ${texto}`;
      const agora = new Date().toLocaleTimeString();
      mensagens.push({ text: fullText, time: agora });
      atualizar_tela();

      fetch("http://vianetminas.com.br/mensagens/enviar_mensagem.php", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: "mensagem=" + encodeURIComponent(fullText)
      }).then(() => recarregar());

      inpMsg.value = "";
    }

    function recarregar() {
      fetch("http://vianetminas.com.br/mensagens/listar_mensagens.php")
        .then(res => res.json())
        .then(data => {
          mensagens = data.map(obj => ({
            text: obj.conteudo,
            time: new Date().toLocaleTimeString()
          }));
          atualizar_tela();
        });
    }
    setInterval(recarregar, 3000);
    window.onload = recarregar;

    function confirmRemoveLast() {
      if (confirm("Apagar última mensagem?")) {
        fetch("http://vianetminas.com.br/mensagens/remover_ultima.php")
          .then(() => {
            mensagens.pop();
            atualizar_tela();
          });
      }
    }

    function confirmRemoveAll() {
      if (confirm("Apagar todas as mensagens?")) {
        mensagens = [];
        atualizar_tela();
        fetch("http://vianetminas.com.br/mensagens/limpar_mensagens.php");
      }
    }

    document.getElementById("input_mensagem")
      .addEventListener("keydown", e => {
        if (e.key === "Enter") {
          e.preventDefault();
          enviar_mensagem();
        }
      });
  </script>
</body>
</html>
