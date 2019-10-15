# Aula 2 

A aula 2 é voltada para a criação do backend da aplicação com node.js

---

## Backend

"backend é tudo que o usuario não echerga - jênio" 😒

1. adicionou express como roteador 
2. ele usa um template bem default de aplicação com:
  ```js
  const express = require('express');
  const app = express()

  app.get('/', (req,res) => {
    return res.send('Hello world');
  });

  app.listen(3333);
  ```
3. ele modifica o codigo para retornar json usando ```res.json()```
4. ele adiciona nodemon como dependencia de desenvolvimento
5. ele lista os tipos de requisições HTTP : GET, POST, PUT, DELETE
6. instalado o insomnia para fazer as requisições ✨
7. ele ensina como accessar os parametros da querry (info passada na URL) string usando:
  ```js
  //o req.query tem acesso a todos os paramaetros query

  app.get('/users',(req,res)=>{
    return res.json({ message: req.query.idade })
  })

  //ex:
  //URL: localhost:3030/users?idade=20
  //resposta: {idade:20}
  ```
  os query params são muito usados para filtros
8. ele explica como obter route params (parametros passados direto na rota):
  ```js
  //o req.params tem acesso aos parametros de route especificados com :

  app.get('/users/:id',(req,res)=>{
    return res.json({ id: req.params.id })
  })

  //ex:
  //URL: localhost:3030/users/4
  //resposta: {id:4}
  ```
  route params são usados em deletes ou put para definir qual id do objeto a ser deletado ou atualizado
9. ele explica como acessar o corpo da requisição (corpo que contem o grosso das informações do request):
  ```js
  //é necessario utilizar o express.json() middleware para que o express saiba como reagir a um body com json 
  app.use(express.json())

  app.get('/users',(req,res)=>{
    return res.json(req.body)
  })

  // o req.body vai guardar todo o body que foi passado no pedido HTTP
  ```
  o body é utilizado na criação ou edição, quando é necessario passar as informações do novo objeto
10. ele explica como separar as rotas para outro arquivo com uma forma bem padrão:

    - no outro arquivo:
      
    ```js
    const express = require('express');

    const routes = express.Router();

    routes.post('/users',(req,res)=>{
      return res.json(req.body)
    })

    module.exports = routes;
    ```

    - no arquivo principal:
      
    ```js
    const routes = require('./routes');

    app.use(routes);
    ```

    !!! lembrar que os ```app.use``` devem ser sequenciais por isso o ```app.use(express.json())``` deve vir antes pois é usado pelas rotas !!!
11. ele cria uma conta e cria um banco de dados no mongo db atlas:
    1. criar novo usuario no atlas e novo cluster em https://cloud.mongodb.com
    2. adicionar novo usuario ao database na aba de database access
    3. adicionar acesso de qualquer IP na lista de whitelist na aba Network Access
    4. selecionar connect > connect your application para receber uma string de conexão
12. ele adiciona o acesso ao banco de dados na aplicação:
    1. adiciona o pacote mongoose
    2. usa conexão padrão:
      ```js
      const mongoose = require('mongoose');

      //app = express();

      mongoose.connect('<string de conexão>',{
        useNewUrlParser:true,
        useUnifiedTopology:true
      })
      ```
13. ele separa a aplicação em M(model), V(view), C(controllers).
14. cria um Schema/Model para o usuario tendo email:
    ```js
    const mongoose = require('mongoose');

    const UserSchema = new mongoose.Schema({
      email: String
    })

    module.exports = mongoose.model('User', UserSchema);
    ```
15. adicionou uma SessionController para administrar o login/logout do usuario com e criou a lógica de adicionar um usuario:
    1. store:
      ```js
      async store(req,res) {
        const {email} = req.body;
        let user = await User.findOne({email});
        if(!user) {
          user = await User.create({email});
        }
        return res.json(user);
      }
      ```
16. criou um model para um Spot:
    ```js
    const mongoose = require('mongoose');

    constSpotSchema = new mongoose.Schema({
      thumbnail: String,
      company: String,
      price: Number,
      techs: [String],
      user: {
        type: mongoose.Schema.Types.ObjectId,
        ref:'User'
      }
    })

    module.exports = mongoose.model('User',SpotSchema);
    ```
17. adicionou o multer para upload de imagens
    1.  criou arquivo de configurações:
      ```js
      const multer = require('multer');
      const path = require('path')

      module.exports = {
        storage: multer.diskStorage({
          destination: path.resolve(__dirname,'..','..','uploads'),
          filename: (req,file,cb) => {
            const ext = path.extname(file.originalname);
            const name = path.basename(file.originalname,ext);
            cb(null,`${name}-${Date.now()}${ext}`)
          }
        })
      }
      ```
    2. criou a função store no controller do spot:
      ```js
      async store(req,res) {
        const { filename } = req.file;
        const { company, techs, price } = req.body;
        const {user_id} = req.headers;

        const user = await User.findById(user_id);

        if(!user){
          return res.status(400).json({
            error:'User does not exist'
          })
        }

        const spot = await Spot.create({
          user:user_id,
          thumbnail:filename,
          company,
          techs: techs.split(',').map(tech => tech.trim()),
          price
        })

        return res.json(spot);
      }
      ```
    3. adicionou criação as rotas:
      ```js
      const multer = require('multer');
      const uploadConfig = require('./config/upload');
      const SpotControlller = require('./controllers/SpotController');

      const upload = multer(uploadConfig);

      routes.post('/spots',upload.single('thumbnail'),SpotControlller.store)
      ```
18. adicionou um GET para spots
    1. adicionou no controller  
      ```js
      async index(req,res){
        const {tech} = req.query;

        const spots = await Spot.find({techs:tech});

        return res.json(spots);
      },
      ```
    2. adicionou no routes
      ```js 
      routes.get('/spots',SpotControlller.index)
      ```
19. adicionar uma listagem dos spots criados por um mesmo usuario
    1.  adicionou o controller
      ```js
      async show(req,res){
        const { user_id } = req.headers;

        const spots = await Spot.find({user:user_id})

        return res.json(spots);
      }
      ```
    2. adicionou as rotas
      ```js
      routes.get('/dashboard', DashboardController.show)
      ```
20. criou o model do booking
  ```js
  const mongoose = require('mongoose');

  const BookingSchema = new mongoose.Schema({
    date: String,
    approved: Boolean,
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    Spot: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Spot'
    }
  })

  module.exports = mongoose.model('Booking', BookingSchema);
  ```
21. adicionou um POST para bookings   
    1.  adicionou o controller:
      ```js
      async store(req,res) {
        const { user_id } = req.headers;
        const { spot_id } = req.params;
        const { date } = req.body;

        const booking = await Booking.create({
          user: user_id,
          spot: spot_id,
          date,
        });

        return res.json(booking);
      }  
      ```
    2. adicionou as rotas: 
      ```js
      routes.post('/spots/:spot_id/bookings',BookingController.store)
      ```
    3. adicionou um populate para a resposta
      ```js 
      await booking.populate('spot').populate('user').execPopulate();
      ```
      
---

**Notas minhas:**

- eu gostei muito da forma como ele explicou os 3 acessos as informações do request para depois passar para o código.
- realmente a aula é bem proveitosa em 1:30 pórem deixa passar um grande número de boas praticas e principalmente validações e testes.
- senti falta de alguma forma basica de autenticação

---

**Neat picking (detalhes "errados"):**

- no 11:12 ele diz que ```res.send()``` retorna um texto, res.send vai aitomaticamente identificar que tipo de informação esta sendo passada e retornar ela, ele pode retornar: JSON, imagem, array, número,...