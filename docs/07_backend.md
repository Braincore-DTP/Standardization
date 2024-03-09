# Backend

- Flask
- Express
- Laravel with Livewire
- Golang (Very Serious Project, Performance Critical, User Oriented)

## Flask

Untuk Flask bisa contoh ke [sini](./05_ai_ml_deployment.md)

## Express

Dalam menggunakan Express sebagai backend, ada 2 pendekatan:

- Web Monolitik
- API

Web Monolitik artinya kita mendeploy backend dalam satu aplikasi Express yang menyediakan UI tampilan web dalam 1 direktori proyek yang sama, dengan menggunakan templating engine bernama EJS `(Embedded JavaScript)`

Sedangkan API adalah kita hanya mendeploy backend dan menjalankannya sebagai API, dalam hal ini, konteksnya adalah kita mereturn `JSON`.

### Web Monolitik
*on progress*

</br>

### API
Mari kita mulai dengan API, namun sebelum lebih jauh, ada arsitektur yang kita perlu sepakati terlebih dahulu. Pertama mari kita lihat struktur folder dari proyek yang akan dibuat, yaitu sebagai berikut:

```
/app
├── .github/
│   ├── workflows/
│       ├── node.js.yml
├── db/
│   ├── connect.js
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
/node_modules
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

Bagaimana struktur foldernya hehe, kamu kebingungan? Santuy, mari kita bahas 1 per 1. Jadi struktur folder diatas merupakan struktur folder yang menurut kami cukup baik dan nyaman digunakan untuk kebutuhan pembuatan API menggunakan ExpressJS. Di arsitektur ini kita membuat proyeknya nodeJS nya dengan type 'commonJS' yah, (reminder) jadi saat modularisasi atau pemanggilan file atau librari kita gunakan `const ... = require('nama file/librari')`, dan untuk mengexport variabel/function/class kita gunakan `module.exports = nama variable/function/class`.
</br>
Nah untuk memulai sobat bisa membuatnya secara manual, atau bisa menduplikat template yang sudah kami sediakan ya di repositori github [algonacci](https://github.com/algonacci/express-docker-template). Namun, walau sobat bisa langsung menduplikat, usahakan tetap mengerti dengan arsitektur tersebut. Oleh karena itu, yuk simak penjelasan-penjelasan tentang arsitektur tersebut dibawah:

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
const { isAuthenticated } = require("./middleware/isAuthenticated");

//Router
const indexRouter = require("./routes/index/index.router");

app.use(express.json());
app.use(morgan("dev"));
app.use(cors());

//Set API DOCS with Swagger
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(apiDocs))

//Regis Routes
app.use("/", isAuthenticated, indexRouter);

module.exports = app;
```

##### Routes/
Hal yang cukup berbeda dari kebanyakan arsitektur nodeJS yang lain dengan arsitektu yang kami buat yaitu, terletak pada folder `routes/` ini. Pada folder ini, diisi kan dengan sub folder lagi sesuai kebutuhan/fitur yang ingin dikembangkan, lalu didalam sub folder ini berisikan file `.router.js` untuk mengatur jalurnya, `.controller.js` untuk membuat logika programnya, dan `.test.js` untuk membuat testing dari jalur/feat tersebut. Seperti sobat tahu, biasanya kebanyakan arsitektur nodeJS memisahkan ketiga hal tersebut (router/controller/testing) pada folder folder terpisah. Disini kami menggabungkannya karena memudahkan dalam pengerjaannya baik perorangan maupun team, dimana kita bisa mengerjakannya perfitur, sehingga lebih rapi dan terstruktur.</br>
Dalam pembuatana nama folder dan file nya di namakan dengan nama fitur yang kita buat/kembangkan, misal fitur 'index' maka sub folder dari `routes/` adalah folder `index/` yang berisi 3 file yaitu `index.controller.js`, `index.router.js`, dan `index.test.js`. Berikut adalah contoh kodenya:

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
const cache = require('../../config/cache')
const limit = require("./../../config/rateLimiter")

//Controller
const index = require("./index.controller");

//Routes
router.route("/").get(limit(10) ,index.helloIndex); //Set limiter 10 request/minutes
router.route("/").post(index.helloPost);
router.route("/test").get(compression(), cache('1 minutes'),index.getData); //Set compression and cache

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
Folder `helpers/` lebih kurang sama dengan `utils/` dimana didalam folder ini disimpan file-file fungsi yang akan sering digunakan. Seperti customisasi response, atau pengecekan data di database, atau yang lainnya. Untuk penamaan filenya disesuaikan dengan fungsi yang dibuat, berikut contoh isi kode dari file `isExample.js` dari template yang kami sediakan:
```
const forExample = () => {
  console.log("Anjay pake helper");
};

module.exports = { forExample };
```

note: untuk file `isExample.js` tersebut hanya contoh, sobat bisa ubah dan sesuakan penamaan dan isinya sesuai kebutuhan sobat, atau sobat bisa hapus jika tidak diperlukan.

##### Db/ (Database)
Jika mendengar API pasti tidak jauh dari yang namanya database, umumnya jika kita membuat sebuah API biasanya pasti melakukan CRUD ke database baik SQL (MYSQL/POSTGREE/dll) atau bahkan NOSQL (MongoDB/Firestore/dll). Sehingga kita pasti perlu melakukan pembuatan koneksi ke database tersebut. Nah, disinilah kita gunakan folder `db/` untuk menyimpan file yang mengandung kode untuk melakukan koneksi ke database. Berikut contoh kode file koneksi ke firestore (noSql):
```
var admin = require("firebase-admin");

admin.initializeApp({
    credential: admin.credential.cert(JSON.parse(process.env.SERVICEACCOUNTKEY))
});

const db = admin.firestore()

module.exports = db
```

##### Models/
Jika kita melakukan koneksi atau aktifitas CRUD ke database maka kita juga tidak jauh dari yang namanya model. Apa itu model? model merupakan tempat kita membuat sebuah inisialisasi atau mendisign tabel/schema(SQL) di proyek kita. Biasanya saat menerapkan konsep model seperti ini kita menggunakan yang namanya ORM (Object Relation Mapping) misalnya seperti sequelize/prisma/typeorm/lainnya. Nah, di folder `models/` ini lah tempat menyimpan file-file model tersebut.

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
note: Kode di atas melakukan pengecekan terhadap kode kita, di versi nodeJS 14.x, 16.x, dan 18.x

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
Kemudian kita daftarkan dan aktifkan seperti akhir potongan kode sebelumnya `app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(apiDocs))` sobat bisa mengganti path untuk dokumentasinya pada parameter pertama, untuk template yang kami sediakan, kami gunakan `/api-docs` untuk melihat dokumentasi dari API-nya.

[Kembali ke Home](./index.md)
