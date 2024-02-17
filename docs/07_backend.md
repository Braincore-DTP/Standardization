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

Sedangkan API adalah kita hanya mendeploy backend dan menjalankannya sebagai API, dalam hal ini, konteksnya adalah kita mereturn `JSON`

Untuk Express, ada arsitektur yang kita perlu sepakati. Buat sebuah proyek Node.js baru dengan menggunakan command `npm init`, dengan starting pointnya di `server.js`, isinya adalah seperti ini

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

[Kembali ke Home](./index.md)
