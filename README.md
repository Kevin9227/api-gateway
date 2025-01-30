# api-gateway-express
This is a simple API gateway for nodejs application running on the express framework.

....

## How to use?

Simple clone the project to your local machine and run `npm i` to install the packages. Once done spin up your server by running `npm run dev` - we use nodemon to rerun after file changes.

## The tutorial would be published to FreeCodeCamp's webiste.




Vamos analisar o seu código! Ele é um exemplo de um **gateway API** usando Node.js com Express para lidar com proxy reverso, limites de taxa (rate limiting) e segurança. Aqui estão alguns pontos importantes do código:

### 1. **Pacotes e Dependências**
   - Você está utilizando pacotes importantes como:
     - `express`: para criar o servidor.
     - `cors`: para habilitar CORS (Cross-Origin Resource Sharing).
     - `helmet`: para adicionar cabeçalhos de segurança (bom para proteger contra algumas vulnerabilidades web).
     - `morgan`: para registrar os logs HTTP de cada requisição.
     - `http-proxy-middleware`: para configurar o proxy para microservices.

### 2. **Configuração de Middleware**
   - Você usa o middleware para garantir que o servidor seja seguro, logue as requisições e tenha proteção contra vulnerabilidades:
     - `app.use(cors())`: habilita CORS.
     - `app.use(helmet())`: adiciona cabeçalhos de segurança (como proteção contra clickjacking, XSS, etc).
     - `app.use(morgan("combined"))`: registra todas as requisições no formato `combined`.
     - `app.disable("x-powered-by")`: esconde o cabeçalho `X-Powered-By` que indica que o servidor está rodando com Express (uma boa prática de segurança).

### 3. **Configuração de Rate Limiting**
   - Você tem um controle de rate limit para evitar sobrecarga no servidor e proteger contra abusos de um único IP.
   - A cada minuto (60 segundos), você reseta o contador de requisições para cada IP.
   - **Problema potencial:** Você está armazenando os contadores de requisição em um objeto global, mas **não está lidando com concorrência** em um ambiente com múltiplos servidores. Se o seu aplicativo for escalado horizontalmente, isso pode levar a inconsistências no controle de requisições, pois o contador será local para cada instância do servidor.

### 4. **Rate Limiting & Timeout Middleware**
   - Você define a função `rateLimitAndTimeout`, que verifica se um IP ultrapassou o limite de requisições (20 por minuto) e aplica um tempo limite de 15 segundos para cada requisição.
   - Caso o limite seja ultrapassado, você retorna um erro `429` com a mensagem "Rate limit exceeded".
   - Caso haja um timeout de 15 segundos, o servidor responde com um erro `504` (Gateway Timeout).

   **Melhoria potencial:** Se a sua aplicação aumentar, pode ser interessante configurar um mecanismo de rate limiting distribuído, como o uso de Redis, que pode armazenar os contadores de requisições de forma global, mesmo que o servidor seja escalado.

### 5. **Roteamento e Proxy Reverso**
   - Você configurou um array de serviços com as rotas que devem ser redirecionadas para outros servidores, como `/auth`, `/users`, `/chats`, e `/payment`.
   - Para cada serviço, você está utilizando o `createProxyMiddleware` para redirecionar as requisições para os endpoints correspondentes, mas removendo a parte da URL que se refere ao serviço.
   
   **Exemplo**: Se a rota for `/auth`, a requisição será redirecionada para `https://teste/auth` (removendo o prefixo `/auth` da URL).
   
   A lógica está bem configurada para redirecionar para diferentes microserviços, e o `rateLimitAndTimeout` é aplicado antes de redirecionar para garantir que as regras sejam cumpridas.

### 6. **Tratamento de Erro 404**
   - Você também configurou uma resposta 404 para quando a rota não for encontrada:
   ```js
   app.use((_req, res) => {
     res.status(404).json({
       code: 404,
       status: "Error",
       message: "Route not found.",
       data: null,
     });
   });
   ```
   Isso garante que o usuário receba uma resposta clara quando acessar uma rota inválida.

### 7. **Rota de Teste `/auth`**
   - Você tem uma rota simples `/auth` para testar se o servidor está funcionando corretamente:
   ```js
   app.get('/auth', (req, res) => {
     res.send({ Msg: 'Bem vindo..' });
   });
   ```
   Essa rota parece ser só para fins de teste, o que é bom para verificar o servidor enquanto você desenvolve, mas se não for uma parte essencial, pode ser removida quando estiver pronto para produção.

### **Melhorias Sugeridas**
1. **Persistência de Rate Limiting**: Se você planeja escalar o aplicativo, use algo como Redis para armazenar os contadores de requisições, já que a memória volátil (RAM) será perdida se a aplicação for reiniciada ou escalada para múltiplas instâncias.
   
2. **Gestão de Timeout**: Definir um tempo de espera global para as requisições no proxy pode ser interessante. Embora você tenha um timeout por requisição, às vezes é útil ter uma configuração mais global.
   
3. **Validação e Tratamento de Erros**: Para tornar o código mais robusto, talvez você queira adicionar um tratamento mais detalhado de erros para falhas inesperadas, como se o servidor de destino estiver inativo ou se ocorrer algum erro ao fazer o proxy.

4. **Logging de Erros**: Considerando que você está usando o `morgan` para logs, pode ser interessante adicionar mais detalhamento nos logs para erros (como quando ocorre um 429 ou 504), para facilitar a depuração.

Se você tiver dúvidas ou precisar de mais ajuda com alguma parte específica, é só falar!