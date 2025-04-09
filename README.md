# Jenkins Docker Agent Setup

## 🧩 Tujuan
Menjalankan Jenkins agent berbasis Docker dengan akses ke Docker host (`/var/run/docker.sock`) dan berhasil mengeksekusi job pipeline.

---

## 🛠️ Langkah-langkah

### 1. Buat Dockerfile untuk Jenkins Agent
```Dockerfile
FROM jenkins/inbound-agent:latest
USER root
RUN apt-get update && apt-get install -y docker.io
```

### 2. Build Docker Image
```bash
docker build -t jenkins-agent-docker .
```

### 3. Jalankan Container Agent
```bash
docker run -d \
  --name jenkins-agent-1 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-agent-docker \
  -url https://jenkins.naufal.net/ <SECRET> <AGENT_NAME>
```
> Catatan: Gantilah `<SECRET>` dan `<AGENT_NAME>` sesuai yang diberikan oleh Jenkins master saat membuat node.

### 4. Pastikan Jenkins Master Dapat Di-resolve dari Container
Jika container tidak bisa resolve DNS seperti `jenkins.naufal.net`, pastikan:
- DNS lokal bisa diakses dari container
- Atau tambahkan IP-Hostname di `/etc/hosts` host maupun di dalam image.

### 5. Buat Node Baru di Jenkins
- Masuk ke `Manage Jenkins` → `Nodes`
- Tambah node baru dengan:
  - `Remote root directory`: `/home/jenkins`
  - `Launch method`: `Launch agent by connecting it to the master`
  - Tentukan secret dan nama node

---

## ✅ Uji Pipeline
Contoh Jenkinsfile:
```groovy
pipeline {
  agent { label 'docker' }
  stages {
    stage('Test Docker Agent') {
      steps {
        sh 'echo Hello from Docker agent'
        sh 'docker ps'
      }
    }
  }
}
```

### Output:
```bash
+ echo Hello from Docker agent
Hello from Docker agent
+ docker ps
<daftar container muncul>
```

---

## 📌 Troubleshooting

### Masalah: `permission denied` saat akses Docker
- Pastikan container dijalankan sebagai `root`
- `jenkins/inbound-agent` default-nya bukan root
- Solusi: build image sendiri dan set `USER root`

### Masalah: DNS tidak resolve
- Solusi: tambahkan entry di `/etc/hosts` atau gunakan IP langsung

### Masalah: conflict nama container
- Gunakan perintah:
```bash
docker rm -f jenkins-agent-1
```

---

## 📄 Status Akhir: ✅ **Berhasil**
Pipeline berhasil jalan dan agent Docker bisa akses Docker host.

---

## ✍️ Disiapkan oleh
`naufal` - DevOps Intern @ LNSW

