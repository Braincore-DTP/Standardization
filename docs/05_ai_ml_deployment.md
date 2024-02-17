# AI/ML Deployment

Untuk proses deployment model AI/ML, mari kita berfokus ke 3 tools ini saja

- Flask: Sebuah mini framework untuk webserver dengan menggunakan bahasa Python
- FastAPI: Mirip dengan Flask, namun dilengkapi dengan `async` keyword dan `type hinting`-nya Python
- Streamlit: Sebuah framework untuk deployment cepat karena memiliki banyak sekali komponen yang bisa dipanggil melalui fungsi Python

## Flask

Dalam konteks deployment model AI/ML, kita bisa lakukan 2 pendekatan ini di Flask

- Web Monolitik
- API

Web Monolitik artinya kita mendeploy model dalam satu aplikasi Flask yang menyediakan UI tampilan web dalam 1 direktori proyek yang sama

Sedangkan API adalah kita hanya mendeploy model dan menjalankannya sebagai API, dalam hal ini, konteksnya adalah kita mereturn `JSON`

## Flask Web Monolitik

Flask umumnya bisa digunakan untuk membuat sebuah backend sederhana seperti framework atau bahasa pemrograman lainnya, kata kuncinya dalah return jenis `render_template()`

[Simak materinya disini](https://docs.google.com/presentation/d/1T9jcPA82ksyCi6MrxVhsNdjxp6L3QvKIgh1hySyqMUw/edit?usp=sharing)

Mari gunakan virtual environment, pastikan sudah membuat dan masuk ke 1 folder proyek terlebih dahulu

```
$ python -m venv .venv
$ (Windows) .venv\Scripts\activate
$ (UNIX/Linux) source .venv/bin/activate
$ pip install flask gunicorn scikit-learn
$ pip freeze > requirements.txt
```

Gunakan text editor kesayangan kamu, dan buat file `app.py`, kita buat aplikasi web monolitik sederhana dengan menggunakan Flask

```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

Kembali ke terminal, jalankan `flask run`, dan buka di browser alamat localhost di `127.0.0.1:5000` (port default Flask berada di 5000) dan kamu akan mendapatkan tulisan `Hello World!`

## Flask API

Flask bisa juga digunakan sebagai API untuk mendeploy model AI/ML, kunci utamanya adalah di jenis return `jsonify()`

[Simak materinya disini](https://docs.google.com/presentation/d/1L3vGci-Nc40lZbzuSJz4KKgwZ0eEJFZG3AyiwiwdMco/edit?usp=sharing)

Ada aturan untuk mereturn JSON di berbagai bahasa, bentuknya harus seperti ini

```
"status": {
    "code": 200,
    "message": "Success fetching the API",
},
"data": None
```

## Struktur Folder Flask

Ketika mendeploy model AI/ML untuk skala production, coba gunakan Flask Blueprint untuk modularisasi agar memudahkan ketika pengembangan

- Contoh Versi Web Monolitik (COMING SOON)
- [Contoh Versi API](https://github.com/algonacci/flask-docker-template)

[Kembali ke Home](./index.md)
