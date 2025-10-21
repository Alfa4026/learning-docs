# 📘 Dokumentasi Penggunaan `tmux` di Ubuntu

## 1. Apa itu `tmux`?

`tmux` (*terminal multiplexer*) adalah aplikasi yang memungkinkan kita:

* Membuka **banyak terminal dalam satu jendela**.
* Menyimpan **session** meskipun koneksi SSH terputus.
* Berpindah antar panel/jendela dengan cepat.
* Menjalankan proses di background.

Sangat berguna saat bekerja dengan server/SSH, terutama untuk eksperimen atau task yang berjalan lama.

---

## 2. Instalasi

```bash
sudo apt update
sudo apt install tmux -y
```

Cek versi:

```bash
tmux -V
```

---

## 3. Membuat dan Mengelola Session

### Membuat session baru

```bash
tmux new -s nama_session
```

### Melihat daftar session

```bash
tmux ls
```

### Masuk ke session

```bash
tmux attach -t nama_session
```

### Keluar dari session (tanpa menghentikan proses)

Tekan:

```
Ctrl + b  lalu d
```

(`d` = detach)

### Menghapus session

```bash
tmux kill-session -t nama_session
```

---

## 4. Manajemen Window (Tab di dalam session)

* **Buat window baru**

  ```
  Ctrl + b  c
  ```

* **Pindah antar window**

  ```
  Ctrl + b  n   → next window
  Ctrl + b  p   → previous window
  Ctrl + b  nomor_window
  ```

* **Ganti nama window**

  ```
  Ctrl + b  , 
  ```

* **Tutup window**

  ```
  exit
  ```

---

## 5. Manajemen Pane (Split layar)

* **Bagi vertikal**

  ```
  Ctrl + b  %
  ```

* **Bagi horizontal**

  ```
  Ctrl + b  "
  ```

* **Pindah antar pane**

  ```
  Ctrl + b  arah_panah
  ```

* **Atur ukuran pane**

  ```
  Ctrl + b  :resize-pane -D 5
  Ctrl + b  :resize-pane -U 5
  Ctrl + b  :resize-pane -L 5
  Ctrl + b  :resize-pane -R 5
  ```

* **Tutup pane**

  ```
  exit
  ```

---

## 6. Shortcut Berguna

| Kombinasi    | Fungsi                          |
| ------------ | ------------------------------- |
| `Ctrl+b ?`   | Lihat daftar semua shortcut     |
| `Ctrl+b d`   | Detach (keluar tanpa mematikan) |
| `Ctrl+b %`   | Split vertikal                  |
| `Ctrl+b "`   | Split horizontal                |
| `Ctrl+b o`   | Pindah ke pane lain             |
| `Ctrl+b x`   | Tutup pane                      |
| `Ctrl+b ,`   | Rename window                   |
| `Ctrl+b c`   | Buat window baru                |
| `Ctrl+b n/p` | Next/prev window                |

---

## 7. Konfigurasi Tambahan (Opsional)

Edit file konfigurasi:

```bash
nano ~/.tmux.conf
```

Contoh konfigurasi:

```conf
# Gunakan Ctrl+a sebagai prefix (lebih enak, mirip GNU screen)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Warna status bar
set -g status-bg colour235
set -g status-fg white
set -g status-left-length 30
set -g status-right-length 60
```

Reload konfigurasi:

```bash
tmux source-file ~/.tmux.conf
```

---

## 8. Studi Kasus (SSH & Long Running Task)

Misal kamu jalankan program di server (training ML):

```bash
tmux new -s training
python train.py
```

Jika SSH terputus, proses **tetap berjalan**.
Masuk lagi:

```bash
ssh user@server
tmux attach -t training
```

---

## 9. Tips Tambahan

* Gunakan `tmux ls` untuk memeriksa session yang masih berjalan.
* Biasakan **detach (Ctrl+b d)** sebelum keluar dari server.
* Bisa menjalankan **beberapa session** untuk project berbeda.

