---
layout: post
title: Menggunakan Oak di webserver Deno
---

Seperti NodeJS, terdapat library-library webserver yang memudahkan proses pembuatan backend server di Deno. Pada post kali ini, kita akan mempelajari framework Oak, salah satu framework middleware Deno dengan `Star` terbanyak saat ini.

# Latar
Oak dibuat dengan inspirasi dari Koa (salah satu webserver NodeJS yang cukup terkenal). Meskipun Oak sendiri disebut oleh para pembuatnya sebagai "Middleware framework for Deno", secara implementasi dan penggunaan Oak dapat dianggap sebagai webserver.

## Konsep
Terdapat dua konsep utama pengaturan route di Oak, yaitu `Application` dan `Router`. Perlu diketahui, kedua benda ini memiliki perbedaan dari `Application` dan `Router` dari ExpressJS, karena Oak tidak mendukung nested router.

`Application` adalah komponen utama Oak dan digunakan untuk memulai server kita, sedangkan `Router` umumnya digunakan untuk menentukan rute API server. Sebuah `Router` harus digunakan oleh `Application` agar dapat bekerja (metode `use`).

# Cara Memakai Oak
Sekarang mari memulai dengan membuat project baru, buat file `server.ts`.

```ts
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

app.use(async (ctx, next) => {
    ctx.response.body = "Hello World!";
});

console.log("Server is running on localhost:8080");
await app.listen({ port: 8080 });
```

Dari code diatas, kita memberikan instruksi pada `app` untuk selalu mengembalikan response "Hello World" untuk semua request (tidak peduli rute apapun). Mari jalankan project kita menggunakan VSCode atau run `deno run --allow-net ./server.ts`, coba kunjungi alamat `localhost:8080` di browser, "Hello World" akan muncul sebagai response.

![Hasil Awal](/assets/menggunakan_webserver_oak_di_deno/first.png)

Sampai disini hasil project Oak ini tidak berbeda jauh dari `http` native Deno (artikel [Mulai menggunakan Deno](https://msvcode.com/Getting-Started-with-Deno/)]).

## Membuat rute-rute di Oak
Kita ingin memberikan response yang berbeda-beda untuk setiap rute. Karena itu, mari kita lanjutkan dengan menambahkan rute-rute lain, tambah code ini pada `server.ts` (sesudah `app = new Application()`):
```ts
const router = new Router({prefix:"/trivia"});

router.get("/fox", ({response}, next)=>{
    response.body="The fox is hungry";
});

router.get("/owl", ({response}, next)=>{
    response.body="The owl is tired";
});

app.use(router.routes(), router.allowedMethods());
```

Pada code diatas, kita menggunakan `Router` untuk mengatur response pada rute-rute spesifik (`/fox` `/owl`). Kita dapat juga menambahkan `prefix` pada router yang berguna untuk pengelompokan rute. Di browser, kita harus mengunjungi `prefix+url` untuk mendapatkan response yang sesuai.

Sekarang jika kita coba run `server.ts`, kita dapat menerima response:
- Rute `/trivia/fox`: The fox is hungry
- Rute `/trivia/owl`: The owl is tired
- Rute lain: Hello World!

Dari response yang kita terima, dapat kita simpulkan bahwa `app.use Hello World` dapat kita gunakan sebagai cara untuk mengatasi rute yang tidak ada (seperti mengembalikan `Error 404`).

## Membuat middleware tambahan pada Application
Selain memberikan response pada rute-rute tertentu, terkadang kita juga ingin melakukan pendataan pada request yang masuk (mendata kecepatan, menulis `log` request yang masuk). Karena itu, mari tambahkan beberapa middleware lain yang digunakan di semua rute. Middleware ini harus diletakkan sebelum `app.use(router.routes(), router.allowedMethods());`.

```ts
...

// Logger
app.use(async (ctx, next) => {
  await next();
  const rt = ctx.response.headers.get("X-Response-Time");
  console.log(`${ctx.request.method} ${ctx.request.url} - ${rt}`);
});

// Timing
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.response.headers.set("X-Response-Time", `${ms}ms`);
});

app.use(router.routes(), router.allowedMethods());

...
```

Untuk dapat melanjutkan proses dari satu middleware ke middleware lainnya, kita harus memanggil `next`. Jika tidak dipanggil, maka proses request akan terhenti di dalam middleware terkait dan response akan diberikan (seperti middleware "Hello World"). Dengan memanggil `next` kita mengoper proses request yang diterima pada middleware itu ke middleware berikutnya (contohnya middleware rute `/trivia/fox`).

Mari coba jalankan `server.ts` dan kunjungi `/trivia/fox`. Kita dapat melihat pada `terminal` hasil "logging" yang kita lakukan.

![Hasil Awal](/assets/menggunakan_webserver_oak_di_deno/middleware-2.png)

Kita dapat juga melihat response header pada browser untuk melihat hasil "timing".

![Hasil Awal](/assets/menggunakan_webserver_oak_di_deno/middleware-1.png)

Untuk dapat mengecek urutan pemanggilan middleware, mari tambahkan `console.log` pada middleware-middleware berikut:
```ts
// fox route
router.get("/fox", ({ response }, next) => {
  console.log("fox");
  response.body = "The fox is hungry";
});

// Logger
app.use(async (ctx, next) => {
  console.log("log start");
  await next();
  const rt = ctx.response.headers.get("X-Response-Time");
  console.log(`log end: ${ctx.request.method} ${ctx.request.url} - ${rt}`);
});

// Timing
app.use(async (ctx, next) => {
  console.log("time start");
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.response.headers.set("X-Response-Time", `${ms}ms`);
  console.log("time end");
});
```

Mari kita coba jalankan lagi dan kunjungi di browser. Urutan panggil dapat kita cek di terminal kita.

![Hasil Awal](/assets/menggunakan_webserver_oak_di_deno/order.png)

Dari hasil diatas dapat kita ambil kesimpulan:
- `log start` muncul pertama karena di code kita karena kita menggunakan (`use`) middleware paling awal
- `time start` dipanggil berikutnya karena kita menggunakannya sesudah logger
- Middleware logger dan timing menunggu hasil proses middleware berikutnya oleh karena panggilan `await next()`. Karena itu `fox` dipanggil berikutnya
- Sesusai `fox` dipanggil, kita melanjutkan proses middleware secara `Stack`, karena itu `time end` terpanggil dulu sebelum `log end`

# Kesimpulan
Sementara ini dulu pelajaran mula-mula menggunakan Oak di Deno, code lengkap yang digunakan di artikel ini adalah sebagai berikut:

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";

const app = new Application();

const router = new Router({ prefix: "/trivia" });

router.get("/fox", ({ response }, next) => {
  console.log("fox");
  response.body = "The fox is hungry";
});

router.get("/owl", ({ response }, next) => {
  response.body = "The owl is tired";
});

// Logger
app.use(async (ctx, next) => {
  console.log("log start");
  await next();
  const rt = ctx.response.headers.get("X-Response-Time");
  console.log(`log end: ${ctx.request.method} ${ctx.request.url} - ${rt}`);
});

// Timing
app.use(async (ctx, next) => {
  console.log("time start");
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.response.headers.set("X-Response-Time", `${ms}ms`);
  console.log("time end");
});

app.use(router.routes(), router.allowedMethods());

app.use(async (ctx, next) => {
  ctx.response.body = "Hello World!";
});

console.log("Server is running on localhost:8080");
await app.listen({ port: 8080 });
```



