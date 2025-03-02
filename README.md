# DeepSeek & WhatsApp API

Este projeto integra um modelo de linguagem executado localmente pelo LM Studio com a API do WhatsApp, permitindo responder mensagens automaticamente com contexto conversacional.

## Requisitos

- [LM Studio](https://lmstudio.ai/) instalado
- [Node.js 20+](https://nodejs.org/en) instalado

## Instalação e Configuração

1. Baixe e instale o LM Studio.

2. No LM Studio, baixe o modelo **DeepSeek-r1-distill-qwen-7b**.

3. Execute o terminal e verifique a versão do Node.js:

   ```sh
   node -v
   ```

4. Crie a pasta do projeto e acesse-a:

   ```sh
   # CRIAR
   mkdir botDeepSeek
   # ACESSAR
   cd botDeepSeek
   ```

5. Inicialize o projeto e instale dependências:

   ```sh
   npm init -y
   npm install whatsapp-web.js qrcode-terminal axios
   ```

6. Crie o arquivo `index.js`:

   ```sh
   New-Item index.js -ItemType File   # Windows PowerShell
   touch index.js                      # Linux/macOS
   ```

7. Abra `index.js` e cole o seguinte código:

   ```javascript
   const { Client, LocalAuth } = require('whatsapp-web.js');
   const qrcode = require('qrcode-terminal');
   const axios = require('axios');

   const conversationHistory = {};

   async function processMessage(text, userId) {
       try {
           if (!conversationHistory[userId]) {
               conversationHistory[userId] = [];
           }

           conversationHistory[userId].push({ role: "user", content: text });

           if (conversationHistory[userId].length > 10) {
               conversationHistory[userId].shift();
           }

           const response = await axios.post('http://localhost:1234/api/v0/chat/completions', {
               model: "deepseek-r1-distill-qwen-7b",
               messages: [
                   { role: "system", content: "Responda de forma natural e amigável, mantendo o contexto da conversa." },
                   ...conversationHistory[userId]
               ],
               temperature: 0.7,
               max_tokens: 500
           });

           let reply = response.data.choices[0].message.content;
           reply = reply.replace(/<think>[\s\S]*?<\/think>/g, '').trim();

           conversationHistory[userId].push({ role: "assistant", content: reply });

           console.log(`Resposta gerada para ${userId}: ${reply}`);
           return reply;
       } catch (error) {
           console.error('Erro ao processar mensagem:', error);
           return 'Desculpe, não consegui processar sua mensagem.';
       }
   }

   const client = new Client({
       authStrategy: new LocalAuth()
   });

   client.on('qr', (qr) => {
       qrcode.generate(qr, { small: true });
   });

   client.on('ready', () => {
       console.log('Client is ready!');
   });

   client.on('message', async (message) => {
       console.log(`Mensagem recebida de ${message.from}: ${message.body}`);
       const response = await processMessage(message.body, message.from);
       message.reply(response);
   });

   client.initialize();
   ```

8. Execute o bot:

   ```sh
   node index.js
   ```

9. Escaneie o QR Code com o WhatsApp para conectar o bot.

---



