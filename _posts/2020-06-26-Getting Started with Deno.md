---
layout: post
title: Mulai Menggunakan Deno
---

Apakah Deno itu? Mengapa akhir-akhir ini mulai terdengar nama itu? Di post ini kita akan mulai melakukan setup Deno dan mencoba membuat server Deno sederhana.

# Latar
Deno adalah runtime JavaScript yang dibuat oleh Ryan Dahl. Ryan adalah developer original NodeJS yang sudah sering terdengar saat ini.

Di tahun 2018, Ryan menyampaikan hal-hal yang ia sesali di NodeJS, sekaligus memperkenalkan runtime baru untuk mengatasi masalah-masalah Nodejs. Video lengkapnya dapat dilihat di: [Youtube](https://www.youtube.com/watch?v=M3BM9TB-8yA)

Fakta-fakta Deno dan NodeJS saat artikel ini ditulis (Juni 2020):
* Menurut Ryan, Deno bukanlah pengganti NodeJS, karena lingkungan development NodeJS sudah termasuk besar dan matang. Dia memperkirakan baik Deno maupun NodeJS akan tetap dipakai dan sama-sama terkenal di masa yang akan datang
* Deno memang sudah masuk versi 1.x (APInya termasuk stabil dan minim perubahan nama/cara kerja), namun jumlah library pihak ketiga yang ada saat ini masih sangat sedikit dibandingkan NodeJS
* Deno saat ini tidak dapat memakai library NodeJS (NPM modules). Namun diperkirakan kompatibilitas akan ada di masa yang akan datang

Kelebihan Deno:
* Runtime yang aman (ada permission-permission yang harus diberikan secara explisit sebelum menjalankan program)
* Mendukung TypeScript secara native
* Dapat dibuat (compile output) menjadi sebuah bundle JavaScript (mirip React). Sehingga sangat memudahkan proses deployment. Salah satu fitur yang akan ditambahkan di masa depan adalah compile executable (mirip `nexe` untuk NodeJS)
* Sistem library decentralized. Tidak seperti NodeJS yang mengharuskan kita meng-install `npm`, Deno memperbolehkan kita melakukan import library melalui URL (seperti browser)

# Instalasi
Runtime Deno hanya berupa satu buah file executable, tidak seperti runtime server-server lainnya. Hal ini memudahkan proses instalasi dan setup environment untuk server.

Step instalasi Windows:
1. Buka `PowerShell` di windows
2. Jalankan command `iwr https://deno.land/x/install/install.ps1 -useb | iex`
    ![Install Deno](/assets/getting_started_with_deno/install_deno.png)

3. Jalankan `deno -V` untuk mengecek apakah proses instalasi sukses
    ![Check Deno Version](/assets/getting_started_with_deno/check_version.png)

Untik sistem operasi lain, dapat dilihat langsung di [Web resmi Deno](https://deno.land/#installation).

## Membuat server sederhana
Untuk memulai mencoba Deno, buatlah sebuah file bernama `server.ts`, lalu isi code sebagai berikut:
```ts
import { serve } from "https://deno.land/std@0.55.0/http/server.ts";
const s = serve({ port: 8000 });

console.log("Server running on http://localhost:8000");

for await (const req of s) {
  console.log("Found request");
  req.respond({ body: "Hello World\n" });
}
```

Selanjutnya coba run dengan cara:
1. Buka terminal/command prompt/powershell
2. Masuk ke folder project deno
3. Panggil `deno run server.ts`

Seharusnya kita akan menemukan error `Uncaught PermissionDenied: network access`.
![Permission Denied](/assets/getting_started_with_deno/network_permission_denied.png)

Seperti yang telah dijelaskan diatas, Deno memiliki sistem-sistem permission untuk menjaga keamanan server. Error ini terjadi karena program ini ingin membuka akses server (network), namun kita belum memberi permission terkait.

Sekarang, run lagi program dengan menambahkan permission: `deno run --allow-net server.ts`. Seharusnya akan muncul tulisan `Server running on http://localhost:8000` di terminal.

Sekarang kita bisa membuka browser kita, lalu mengunjungi `http://localhost:8000`.
![Hello world](/assets/getting_started_with_deno/hello_world.png)

Selamat! Anda telah membuat program Deno pertama anda.

# Bonus: Tool yang cocok development untuk Deno: VisualStudio Code
Salah satu tool yang cocok untuk development Deno adalah VisualStudio Code sering disebut juga `VSCode`. VSCode termasuk "Code Editor" bukan IDE, namun dia juga dilengkapi fitur debugging dan memiliki support TypeScript secara native, sehingga sangat membantu proses development.

Step setup VSCode untuk Deno:
1. Install VSCode: [Website VSCode](https://code.visualstudio.com/)
2. Buka Tab `Extension`
3. Cari "deno" (`denoland.vscode-deno`) lalu install
    ![Extension deno](/assets/getting_started_with_deno/deno_vscode.png)

Cara melakukan debugging Deno di VSCode:
1. Buka folder project Deno
2. Buat config `.vscode/launch.json`
    ```json
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Deno",
                "type": "node",
                "request": "launch",
                "cwd": "${workspaceFolder}",
                "runtimeExecutable": "deno",
                "runtimeArgs": ["run", "--inspect-brk", "-A", "server.ts"],
                "outputCapture": "std",
                "port": 9229
            }
        ]
    }
    ```
    Pastikan `server.ts` sesuai dengan nama file Deno yang kita buat
3. Run debug di VSCode. Kita dapat juga menambahkan `breakpoint` untuk melakukan debugging
    ![Debug deno](/assets/getting_started_with_deno/debug.png)


