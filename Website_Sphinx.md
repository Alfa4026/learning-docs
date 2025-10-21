# 📘 Dokumentasi: Membuat Website Dokumentasi Belajar dengan Sphinx + GitHub Pages (Auto Deploy)

## 1. Persiapan

1. Pastikan sudah install **conda** atau **python**.
2. Buat environment khusus (opsional tapi direkomendasikan):

   ```bash
   conda create -n docs python=3.11 -y
   conda activate docs
   ```
3. Install package yang dibutuhkan:

   ```bash
   pip install sphinx myst-parser furo
   ```

---

## 2. Struktur Folder Repo

Buat repo GitHub baru, misalnya `my-docs`.
Clone ke laptop, lalu buat struktur ini:

```
my-docs/
│── conf.py
│── index.md
│── tmux.md
│── python.md
│── federated_learning.md
│── database.md
│── .github/
│    └── workflows/
│         └── deploy.yml
```

---

## 3. Konfigurasi Sphinx

Edit file `conf.py`:

```python
# conf.py
project = "Dokumentasi Belajar"
author = "Alif"
release = "0.1"

extensions = ["myst_parser"]   # biar bisa pakai Markdown
html_theme = "furo"            # tema modern mirip Flower
language = "id"
```

---

## 4. Buat Halaman Utama (`index.md`)

````markdown
# 📚 Dokumentasi Belajar Alfa

Selamat datang di dokumentasi belajar saya.  
Silakan pilih topik di bawah ini:

```{toctree}
:maxdepth: 2

tmux
python
federated_learning
database
````

````

---

## 5. Contoh File Materi

### 🖥️ `tmux.md`
```markdown
# 📘 Dokumentasi Penggunaan tmux di Ubuntu

## 1. Apa itu tmux?
`tmux` adalah terminal multiplexer ...

## 2. Instalasi
```bash
sudo apt update
sudo apt install tmux -y
````

````

### 🐍 `python.md`
```markdown
# 🐍 Dasar Python

## Instalasi
```bash
sudo apt install python3 python3-pip -y
````

## Hello World

```python
print("Hello, World!")
```

````

*(topik lain sama: `federated_learning.md`, `database.md`, dst)*

---

## 6. Workflow GitHub Actions (`.github/workflows/deploy.yml`)
```yaml
name: Build and Deploy Docs

on:
  push:
    branches:
      - main   # otomatis jalan kalau push ke main

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install sphinx myst-parser furo

      - name: Build HTML
        run: |
          sphinx-build -b html . _build/html

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_build/html
````

---

## 7. Cara Update Dokumentasi

1. Edit / tambah file `.md`.
2. Tambahkan ke daftar isi di `index.md`.
3. Simpan → commit → push:

   ```bash
   git add .
   git commit -m "update dokumentasi python"
   git push
   ```

GitHub Actions akan otomatis build & deploy → website langsung update 🎉

---

## 8. Aktifkan GitHub Pages

1. Buka repo di GitHub.
2. **Settings → Pages**.
3. Source: pilih `gh-pages` branch.
4. Website aktif di:

   ```
   https://<username>.github.io/my-docs/
   ```

---

## 🎯 Ringkasan

* **File `.md`** → ditulis langsung di repo (misalnya `python.md`).
* **Push ke main** → GitHub Actions otomatis jalan.
* **Website update otomatis** di GitHub Pages.
