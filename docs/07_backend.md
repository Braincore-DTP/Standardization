# Backend

- Flask
- Express
- Laravel with Livewire
- Golang (Very Serious Project, Performance Critical, User Oriented)

## Flask

Untuk Flask bisa contoh ke [sini](./05_ai_ml_deployment.md)

## Express

Dalam menggunakan Express sebagai backend, ada 2 pendekatan:

- [API](https://github.com/algonacci/express-docker-template)
- [Web Monolitik](https://github.com/algonacci/monolithic-express-docker-template)

API aritnya kita hanya mendeploy backend dan menjalankannya sebagai API, dalam hal ini, konteksnya adalah kita mereturn `JSON`.

Sedangkan Web Monolitik, kita mendeploy backend dalam satu aplikasi Express yang menyediakan UI tampilan web dalam 1 direktori proyek yang sama, dengan menggunakan templating engine.

### API
Pertama kita mulai dengan API, namun sebelum lebih jauh, ada arsitektur yang kita perlu sepakati terlebih dahulu. Mari kita lihat struktur folder dari template yang telah kami sediakan, yaitu sebagai berikut:

```
/
├── .github/
│   ├── workflows/
│       ├── node.js.yml
├── configs/
│   ├── database.js
├── helpers/
│   ├── forExample.js
│   ├── hashing.js
│   ├── response.js
├── middleware/
│   ├── cache.js
│   ├── isAuthenticated.js
│   ├── rateLimiter.js
├── models/
│   ├── models.js
├── routes/
│   ├── index/
│       ├── index.controller.js
│       ├── index.router.js
│       ├── index.test.js
/.dockerignore
/.env
/.gitignore
/api-docs.yaml
/app.js
/Dockerfile
/LICENSE
/package-lock.json
/package.json
/README.md
/server.js
```

#### Ringkasan Struktur Proyek
Bagaimana struktur foldernya hehe, kamu kebingungan? Santuy, mari kita bahas 1 per 1. Jadi struktur folder diatas merupakan struktur folder yang menurut kami cukup baik dan nyaman digunakan untuk kebutuhan pembuatan API menggunakan ExpressJS. Di arsitektur ini kita membuat proyeknya nodeJS nya dengan type 'commonJS' yah, (reminder) jadi saat modularisasi atau pemanggilan file atau librari kita gunakan `const ... = require('nama file/library')`, dan untuk mengexport variabel/function/class kita gunakan `module.exports = nama variable/function/class`.
</br>
Nah untuk memulai sobat bisa membuatnya secara manual, atau bisa menduplikat template yang sudah kami sediakan ya di repositori github [algonacci-api-express-template](https://github.com/algonacci/express-docker-template). Namun, walau sobat bisa langsung menduplikat, usahakan tetap mengerti dengan arsitektur tersebut. Oleh karena itu, yuk simak penjelasan-penjelasan tentang arsitektur tersebut dibawah:

##### Starting Point (server.js)
Pertama, jika sobat memulainya secara manual, buatlah sebuah proyek Node.js baru dengan menggunakan command `npm init`, dengan starting pointnya di `server.js`, lalu isi dengan code seperti berikut:

```
require("dotenv").config();
const http = require("http");
const app = require("./app");
const PORT = process.env.PORT || 8000;

const server = http.createServer(app);

const start = async () => {
  try {
    server.listen(PORT, () => {
      console.log(`🚀 [SERVER] is running on port http://localhost:${PORT}`);
    });
  } catch (error) {
    console.log(error);
  }
};

start();
```

Pada kode tersebut sobat bisa lihat ada sebuat variabel yang di panggil dari sebuah file, yaitu app. Setelah kita membuat starting point (`server.js`), selanjutnya root folder kita buat lagi 1 file dengan nama `app.js`. Di file ini kita akan melakukan inisialisasi serta pemanggilan-pemanggilan router agar terlihat rapi. Dengan kode seperti berikut:

```
const express = require("express");
const morgan = require("morgan");
const cors = require("cors");
const app = express();
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs')
const apiDocs = YAML.load('./api-docs.yaml')

//Middleware
const notFoundHandler = require("./middleware/errors");

//Router
const indexRouter = require("./routes/index/index.router");

app.use(express.json());
app.use(morgan("dev"));
app.use(cors());

//API DOCS with Swagger
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(apiDocs))

//Routes
app.use("/", indexRouter);

// Routes Not found
app.use(notFoundHandler);

module.exports = app;
```

##### Routes/
Hal yang cukup berbeda dari kebanyakan arsitektur nodeJS yang lain dengan arsitektu yang kami buat yaitu, terletak pada folder `routes/` ini. Pada folder ini, diisi kan dengan sub folder lagi sesuai kebutuhan/fitur yang ingin dikembangkan, lalu didalam sub folder ini berisikan file `.router.js` untuk mengatur jalurnya, `.controller.js` untuk membuat logika programnya, dan `.test.js` untuk membuat testing dari jalur/fitur tersebut. Seperti sobat tahu, biasanya kebanyakan arsitektur nodeJS memisahkan ketiga hal tersebut (router/controller/testing) pada folder-folder terpisah. Disini kami menggabungkannya karena memudahkan dalam pengerjaannya baik perorangan maupun team, dimana kita bisa mengerjakannya perfitur, sehingga lebih rapi dan terstruktur.
```
│
├── routes/
│   ├── index/
│       ├── index.controller.js
│       ├── index.router.js
│       ├── index.test.js
```
</br>
Dalam pembuatan nama folder dan file nya di namakan dengan nama fitur yang kita buat/kembangkan, misal fitur 'index' maka sub folder dari `routes/` adalah folder `index/` yang berisi 3 file yaitu `index.controller.js`, `index.router.js`, dan `index.test.js`. Berikut adalah contoh kodenya:

###### *index.controller.js*
```
const { forExample } = require("../../helpers/forExample");
const response = require("./../../helpers/response")

const helloIndex = (req, res) => {
  forExample();

  response.res200("Success fetching the API!", null, res)
};

const helloPost = (req, res) => {
  const inputData = req.body;
  console.log(inputData);
  if (Object.keys(inputData).length !== 0) {
    response.res201("Success create data", inputData, res)
  } else {
    response.res400(res)
  }
};

const getData = async (req, res) => {
  try {
    let result = 'Testing '.repeat(10000)

    response.resCustom(200, "Success fetching data", result, res)

  } catch (error) {
    console.log(error.message)
    response.res500(res)
  }
};

module.exports = { helloIndex, helloPost, getData };
```

###### *index.router.js*
```
const express = require("express");
const router = express.Router();
const compression = require("compression");

//Middleware
const { isAuthenticated } = require("./../../middleware/isAuthenticated");
const cache = require('../../middleware/cache');
const limit = require("./../../middleware/rateLimiter");

//Controller
const index = require("./index.controller");

//Routes
router.route("/").get(isAuthenticated, limit(10) ,index.helloIndex); //Set limiter 10 request/minutes
router.route("/").post(isAuthenticated, index.helloPost);
router.route("/test").get(isAuthenticated, compression(), cache('1 minutes'),index.getData); //Set compression and cache

module.exports = router;
```

###### *index.test.js*
```
const { response } = require("express");
const request = require("supertest");
const app = require("../../app");

const token = 'Ceritanya Token JWT';

describe("Test GET /", () => {
  test("It should respond with 200 success", async () => {
    const response = await request(app).get("/")
    .set("Authorization", `Bearer ${token}`)
    .expect("Content-Type", /json/);

    expect(response.status).toBe(200);
    expect(response.body).toEqual({
      status: {
        code: 200,
        message: "Success fetching the API!",
      },
      data: null,
    });
  });
});

...dst
```

##### Middleware/
Folder middleware ini sama seperti arsitektur nodejs lainnya, disini kita menyimpan file-file yang berfungsi sebagai middleware atau file yang berisi fungsi untuk menengahi antara router dan controller. Biasanya berisikan pengecekan atau validasi seperti authentikasi/authorisasi, cache, rate limiter atau yang lainnya. Pada template yang kami buat kami menyediakan middleware yang umum digunakan yakni file `isAuthenticate.js`. Berikut contoh kodenya:
```
const jwt = require("jsonwebtoken")
const response = require("./../helpers/response")

const isAuthenticated = (req, res, next) => {
  const { authorization } = req.headers;
  
  if (!authorization) {
    console.log("Token is not exist")
    return response.res401(res);

  }

  const token = authorization.split(" ")[1] || authorization;

  try {
    // // Uncomment to use Basic verify jwt token
    // const secret = process.env.JWT_KEY;
    // const jwtDecode = jwt.verify(token, secret);

    // //create user data to use in server, adjust with your data when you create token first time
    // const user = {
    //   id: jwtDecode.id,
    //   email: jwtDecode.email,
    //   name: jwtDecode.name,
    //   role: jwtDecode.role,
    //   token,
    // };

    // req.user = user; //set data to use in the next

    console.log("Anjay Authenticate")
    next();
  } catch (error) {
    console.log(error.message);
    return response.res401(res);
  }
};

module.exports = { isAuthenticated };
```

##### Helpers/
Folder `helpers/` lebih kurang sama dengan `utils/` dimana didalam folder ini disimpan file-file fungsi yang akan sering digunakan (reusable function). Seperti customisasi response, atau pengecekan data di database, atau yang lainnya. Untuk penamaan filenya di sesuaikan dengan fungsi yang dibuat, berikut contoh isi kode dari file `isExample.js` dari template yang kami sediakan:
```
const forExample = () => {
  console.log("Anjay pake helper");
};

module.exports = { forExample };
```

*note: untuk file `isExample.js` tersebut hanya contoh, sobat bisa ubah dan sesuaikan penamaan dan isinya sesuai kebutuhan sobat, atau sobat bisa hapus jika tidak diperlukan.*

##### Configs/
Disetiap proyek kita pasti sering melakukan sebuah configurasi baik langsung di file yang membutuhkan atau pun tidak. Nah disini kita menggunakan folder `configs/` ini untuk menyimpan semua file configurasi yang di butuhkan di sebuah proyek/aplikasi, seperti misalnya database. Jika mendengar API pasti tidak jauh dari yang namanya database, umumnya jika kita membuat sebuah API biasanya pasti melakukan CRUD ke database baik SQL (MYSQL/POSTGREE/dll) atau bahkan NOSQL (MongoDB/Firestore/dll). Sehingga kita pasti perlu melakukan pembuatan koneksi ke database tersebut. Nah, disinilah kita gunakan folder `configs/` untuk menyimpan konfigurasi untuk melakukan koneksi ke database. Berikut contoh kode file koneksi ke firestore (noSql):
```
let admin = require("firebase-admin");

admin.initializeApp({
    credential: admin.credential.cert(JSON.parse(process.env.SERVICEACCOUNTKEY))
});

const db = admin.firestore()

module.exports = db
```

##### Models/
Jika kita melakukan koneksi atau aktifitas CRUD ke database maka kita juga tidak jauh dari yang namanya model. Apa itu model? model merupakan tempat kita membuat sebuah inisialisasi atau mendisign tabel/schema(SQL) di proyek/aplikasi kita. Biasanya saat menerapkan konsep model seperti ini kita menggunakan yang namanya ORM (Object Relation Mapping) misalnya seperti sequelize/prisma/typeorm/lainnya. Nah, di folder `models/` ini lah tempat menyimpan file-file model tersebut.

##### .github/Workflows
Terakhir ada folder `.github/` yang berisi subfolder `Workflows/`. Apakah ini? nah, disini merupakan folder tempat kita menyimpan alur kerja atau simplenya file automasi yang berisi kode langkah-langkah pengerjaan saat terjadi komunikasi ke github kita. Misalnya, seperti saat terjadi push/pull/lainnya ke repo github kita apa yang akan kita lakukan. Untuk template yang kami sediakan, kami sudah membuat file automasi untuk melakukan pengecekan semua kode-kode dari file `.test.js` yang kita buat, guna memastikan semua fitur yang kita buat dan sudah test berjalan dengan benar, baik di local maupun saat setelah di push ke github.</br>
Berikut contoh kode dari file automasi `node.js.yaml` yang kami sediakan di template:
```
name: Express Docker Template Application

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    env:
      CI: true
      # MONGODB_URI: mongodb://localhost/nasa
    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/
        # mongodb-version: ["4.4"]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm install
      - run: npm test

```
*note: Kode di atas melakukan pengecekan terhadap kode kita, di versi nodeJS 14.x, 16.x, dan 18.x*

#### API Documentation
Jika kita membuat sebuah proyek API, pastinya kita akan membuat sebuah dokumentasi dari API tersebut betul? Nah, untuk membuat sebuah dokumentasi API ada banyak tools yang tersedia, seperti Postman/Swagger/ReDoc/lainnya. Disini kita akan menggunakan swagger sebagai tools untuk membuat dokumentasinya, mengapa agar kita bisa langsung membuatnya di dalam proyek kita, tanpa harus menginstall aplikasi ketiga atau membuat di website toolsnya tersebut. Di root folder proyek kita, sobat bisa lihat di file `app.js` ada potongan kode seperti berikut:
```
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs')
const apiDocs = YAML.load('./api-docs.yaml')

...

//Set API DOCS with Swagger
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(apiDocs))

...
```
Yap, seperti potongan kode di atas, jika kita ingin menggunakan swagger langsung di proyek kita, kita perlu menginstall 2 librari yakni, `swagger-ui-express` dan `yaml.js`. Disini kita menggunakan `swagger-ui-express` yang membuatkan otomatis tampilan dokumentasi swaggernya dan `yamljs` untuk membaca kode dokumentasi yang kita buat menggunakan file `.yaml` seperti `./api-docs.yaml` yang kami siapkan di template. Disini kita harus membuat terlebih dahulu dokumentasinya di file `api-docs.yaml` serperti contoh:
```
openapi: '3.0.2'

info:
  title: API Documentation
  description: Optional multiline or single-line description in [CommonMark](http://commonmark.org/help/) or HTML.
  version: '1.0.0'

servers:
  - url: "http://localhost:8080"
    description: Optional server description, e.g. Main (development) server

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []

paths:
  /:
    get:
      summary: Returns a list of users.
      description: Optional extended description in CommonMark or HTML.
      tags: 
        - 'Index'
      security:
        - bearerAuth: []
      responses:
        '200':
          description: Response success for index routes.
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: object
                    properties:
                      code:
                        type: integer
                        format: int64
                        example: 200
                      message:
                        type: string
                        example: "Success fetching the API!"
                  data:
                    type: string
                    example: null
        '401':
          description: Unauthorized user.
        default:
          description: Unexpected error

```
Kemudian kita daftarkan dan aktifkan seperti akhir potongan kode sebelumnya `app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(apiDocs))` sobat bisa mengganti path untuk dokumentasinya pada parameter pertama, untuk template yang kami sediakan, kami gunakan route `/api-docs` untuk melihat dokumentasi dari API-nya.

[Kembali ke Atas](./07_backend.md)

</br>

### Web Monolitik
Selanjutnya untuk Web Monolitik juga ada arsitektur yang kita perlu sepakati terlebih dahulu. Arsitektunya lebih kurang mirip dengan arsitektur API di atas, hanya ada sedikit penambahan folder/file. Mari langsung kita lihat struktur folder dari template yang sudah kami sediakan, yaitu sebagai berikut:
```
/
├── public/
│   ├── css/
│   │   ├── library/
│   │   │   ├── bootstrap.min.css
│   │   │   ├── bootstrap.min.css.map
│   │   │   ├── tailwind.css
│   │   ├── style/
│   │   │   ├── main.css
│   ├── font/
│   │   ├── Nunito-Medium.ttf
│   │   ├── Poppins-Regular.ttf
│   ├── images/
│   │   ├── icons
│   │   │   ├── favicon.ico
│   │   ├── braincore.png
│   │   ├── sorry.gif
│   ├── js/
│   │   ├── library/
│   │   │   ├── alpine.js
│   │   │   ├── bootstrap.bundle.min.js
│   │   │   ├── bootstrap.js
│   │   │   ├── htmx.js
│   │   │   ├── jquery.js
│   │   ├── scripts/
│   │   │   ├── main.js
├── server/
│   ├── configs/
│   │   ├── database.js
│   ├── helpers/
│   │   ├── hasing.js
│   │   ├── isExample.js
│   ├── middlewares/
│   │   ├── cache.js
│   │   ├── isAuthenticated.js
│   │   ├── errors.js
│   │   ├── rateLimiter.js
│   ├── models/
│   │   ├── model.js
│   ├── routes/
│   │   ├── index/
│   │   │   ├── index.controller.js
│   │   │   ├── index.router.js
│   ├── app.js
├── views/
│   ├── components/
│   │   ├── button.njk
│   ├── includes/
│   │   ├── footer.njk
│   │   ├── navbar.njk
│   ├── layouts/
│   │   ├── masterLayout.njk
│   ├── pages/
│   │   ├── errors/
│   │   │   ├── 400.njk
│   │   │   ├── 404.njk
│   │   │   ├── 500.njk
│   │   ├── index/
│   │   │   ├── index.njk
/.env
/.gitignore
/.prettierrc
/Dockerfile
/LICENSE
/package-lock.json
/package.json
/README.md
/server.js
/tailwind.config.js
```

#### Ringkasan Struktur Proyek
Bagaimana struktur foldernya hehe, kamu kebingungan? Santuy, memang terlihat sedikit lebih panjang dari struktur API namun lebih kurang sama saja, hanya ada beberapa penambahan seperti folder `views`/`public/`, mari kita bahas 1 per 1. Jadi struktur folder diatas merupakan struktur folder yang menurut kami cukup baik dan nyaman digunakan untuk kebutuhan minimal pembuatan Web Monolitik menggunakan ExpressJS. Di arsitektur ini kita juga membuat proyek nodeJS-nya dengan type 'commonJS' yah.
</br>
Nah untuk memulai sobat bisa membuatnya secara manual, atau bisa menduplikat template yang sudah kami sediakan ya di repositori github [algonacci-monolitic-express-template](https://github.com/algonacci/monolithic-flask-docker-template). Namun, walau sobat bisa langsung menduplikat, usahakan tetap mengerti dengan arsitektur tersebut. Oleh karena itu, yuk simak penjelasan-penjelasan tentang arsitektur tersebut dibawah:

##### Starting Point (server.js)
Pertama, jika sobat memulainya secara manual, buatlah sebuah proyek Node.js baru dengan menggunakan command `npm init`, dengan starting pointnya di `server.js`. Untuk kodenya sendiri sama dengan yang versi 'API' hanya sumber file app.js yang berbeda yaitu dari `"./server/app"`.

##### Public/
Pada arsitektur ini ada folder `public/` yang berisikan file-file yang bisa di akses secara public oleh user, serperti file style css, gambar/images, font, dan file javascripts. Pada folder `css/` dan `js/`, berisi subfolder `library/` yang berisi librari css/js yang akan di gunakan. Kemudian ada sub folder `styles/` dari folder `css/` yang berisi file `main.css` yang dimana itu digunakan untuk menyimpan styling kita, secara default `main.css` menyimpan hasil dari styling library tailwind. Sobat juga bisa menambahkan file styling lain di subfolder ini, jika ada template layout yang menggunakan styling berbeda.
Kemudian subfolder `scripts/` dari folder `js/` juga menggunakan konsep yang sama dengan subfolder `styles/` sebelumnya, namun bedanya disini kita menyimpan file yang berisi kode javascript yang kita gunakan untuk memanipulasi halaman web kita.
Dengan cara seperti ini menurut kami lebih rapi, efektif/efisien dan memudahkan dalam pengembangan.

*noted: jika ada librari default yang kami sediakan tidak sobat butuhkan, sobat bisa menghapusnya dan sesuaikan dengan librari yang sobat gunakan. Jika sobat menggunakan styling tailwind, kami sudah configurasi secara default dan jangan lupa menjalankannya diterminal (`npm run css`) saat melakukan development.*

##### Server/
Pada folder `server/` disini kami mengelompokkan semua folder/file yang digunakan untuk kebutuhan server. Nah semua folder yang ada di arsitektur API kami simpan di dalam folder `server/` ini, file `app.js` yang ada di root folder pada arsitektur API kami juga simpan disini. Sehingga lebih rapi dan memudahkan dalam pengembangan.
Untuk isinya dan penggunaannya lebih kurang sama dengan yang ada di arsitektur API. Hanya sedikit konfigurasi dasar pada app.js yang berbeda, yaitu seperti berikut:
```
const path = require("path");
const express = require("express");
const morgan = require("morgan");
const nunjucks = require("nunjucks");

const app = express();
app.use(morgan("dev"));

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, "../public")));
nunjucks.configure("views", {
  autoescape: true,
  express: app,
});
app.set("view engine", "njk");

// Routes
const indexRouter = require("./routes/index/index.router");

app.use("/", indexRouter);

// Routes Not Found
const notFoundHandler = require("./middlewares/errors");
app.use(notFoundHandler);

module.exports = app;
```

##### Views/
Semua tampilan UI dari web kita ada di dalam folder ini `views/`. Seperti penjelasan awal, web monolitik ini memungkinkan aplikasi express kita bisa memiliki UI website. Untuk menggunakan atau membuat tampilan webnya kita perlu menggunakan template engine. Template engine express ada cukup banyak, yaitu ejs, pug/jade, handlebars, dan nunjucks. Dari keempat template engine tersebut, kami menggunakan Nunjucks pada arsitektur ini, karena setelah riset dan mencoba beberapa template engine, kami merasa bahwa Nunjucks lebih nyaman digunakan dan memiliki fitur yang banyak. Untuk dokumentasi lengkapnya sobat bisa akses di laman resmi [Nunjucks](https://mozilla.github.io/nunjucks/templating.html). Mari kita bahas 1 per 1 fungsi folder-foldernya:

##### 1. Components/
Folder components berfungsi seperti namanya, yaitu sebuah reusable code atau component yang bisa kita gunakan berulang di dalam halaman kita. Dengan nunjucks kita bisa membuat sebuah component statis(tetap). Bahkan component yang dinamis (menggunakan props) seperti di react kita bisa buat juga walau tidak sama persis. Namun untuk folder `components/` ini khusus untuk component yang dinamis. Untuk menggunakannya kita perlu membuat sebuah bagian code tag html yang akan sering kita gunakan, lalu dibungkus dengan fungsi micro seperti, contoh berikut:

```
{% macro button(text, id='test-btn') %}
<button id="{{ id }}" onclick="return alert('Hi Kids')" class="bg-braincore py-1 px-2 rounded-md text-white text-sm hover:bg-" >
    {{text}}
</button>
{% endmacro %}
```
Kemudian kita panggil componentnya seperti ini:
```
{% import "components/button.njk" as btn %}

{{ btn.button("Say Hi", "btn-hi") }}
```

##### 2. Includes/
Folder `includes/` mirip dengan folder `components/`, akan tetapi folder dikhususkan untuk component yang statik. Kita hanya perlu membuat sebuah bagian code tag html yang akan sering kita gunakan, lalu langsung panggil componentnya seperti contoh berikut:
```
{% include "components/navbar.njk" %}
```

##### 3. Layouts/
Folder `Layouts/` berisikan template layouting yang kita tetapkan secara default, jika sobat pernah bermain dengan template engine jinja pada flask(python) maka sobat tidak akan asing dengan layouting di Nunjucks ini. Konsepnya sama persis dengan template layout di jinja pada flask(python). Contoh dari template default yang kami sediakan yaitu seperti berikut:

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8"/>
        <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
        <title>Monolithic Express Docker Template</title>
        <link rel="icon" type="image/x-icon" href="/images/icons/favicon.ico" />
        <link rel="stylesheet" href="/css/styles/main.css"/>
        {# <link rel="stylesheet" href="/css/library/bootstrap.min.css"/> #}
        
        {# <script src="/js/library/bootstrap.js"></script> #}
        <script src="/js/library/alpine.js" defer></script>
        <script src="/js/library/htmx.js"></script>
        <script src="/js/library/jquery.js"></script>
    </head>
    <body class="font-nunito flex flex-col h-screen justify-between">
        {% include "components/navbar.njk" %}
        {% block content %}{% endblock content %}
        {% include "components/footer.njk" %}
        <script src="/js/scripts/main.js"></script>
    </body>
</html>
```
Dengan template seperti diatas kita cukup membuatnya di satu file saja dan bisa kita gunakan sebanyak yang kita butuhkan. Sobat bisa sesuaikan dengan kebutuhan sobat dari default template yang kami sediakan. Untuk menggunakan template layout tersebut, sobat bisa fokus pada baris yang berisi kode 'block' yakni `{% block content %}{% endblock content %}`, dengan menambahkan kode tersebut di template kita dan di file content yang ada di folder `pages/` kita sudah bisa menggunakan templatenya.

##### 4. Pages/
Terakhir folder `pages/` berisi subfolder setiap fitur yang akan kita tampilkan di web. Misal pada default yang kami sediakan, ada subfolder `errors/` yang berisi file halaman(konten) error `400.njk`, `404.njk`, dan `500.njk`. Dimana nanti sobat cukup mengisi filenya dengan content yang sesuai, tidak perlu menulis ulang dari awal `<html>`...`</html>` karena kita menggunakan template layout yang kita buat sebelumnya. Jadi, jika sobat punya sebuah fitur tertentu maka sobat bisa membuat subfolder dengan nama fitur tersebut. Contoh default isi file dari yang kami sediakan, seperti berikut:

###### index.njk
```
{% extends "layouts/masterLayout.njk" %}
{% import "components/button.njk" as btn %}

{% block content %}
    {# start content #}
    <div class="mt-7 p-3 flex-auto" x-data="{name: 'Anjay Template'}">
        <div class="flex flex-col items-center">
            <h1 class="text-xl">{{ title }} 🥰</h1>
            <p class="text-xl">{{ message }}</p>
            <img src="https://i.pinimg.com/originals/e5/d8/ee/e5d8ee323f9526725d6ad355eaed7b05.gif" alt="image" width="400px">
            <p class="text-xl" x-text="name"></p>
        </div>
        <div class="text-center mt-3">
            {# this is components with props ala nunjucks, but firts import the components in the top #}
            {{ btn.button("Say Hi", "btn-hi") }}
        </div>
    </div>
    {# end content #}

    {# start call/include the special scripts that this file uses #}
    {# <script src="/js/scripts/index.js"></script> #}
    {# end call/include custom script #}
{% endblock content %}


```
Pada kode diatas sobat bisa lihat untuk memanggil template layout yang kita buat sebelumnya, sobat perlu memanggil layoutnya dengan fungsi extends disertai path dari lokasi file-nya, seperti contoh di atas `{% extends "layouts/masterLayout.njk" %}`. Lalu dilanjutkan dengan fungsi block yang kita juga deklarasikan di template layout, namun bedanya di dalam block inilah kita menulis kode dari konten yang ingin kita isikan seperti contoh di atas.

*noted: sobat bisa baca comment yang kami sudah tulis juga di default kodenya, dan sebelum mengakhiri blocknya (endblock) sobat bisa menambahkan customisasi script(js) jika ada script khusus untuk halaman/file tersebut.*

😄 Happy coding 😄 <br/><br/>
[Kembali ke Home](./index.md)