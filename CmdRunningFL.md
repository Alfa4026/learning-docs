## 📌 Checklist running FL

1. Jalankan **server superlink**:

   ```bash
   flower-superlink --insecure
   ```
2. Jalankan **client 1 (node2)**:

   ```bash
   flower-supernode --insecure --superlink 192.168.1.100:9092 \
     --node-config "partition-id=0 num-partitions=2"
   ```
3. Jalankan **client 2 (node3)**:

   ```bash
   flower-supernode --insecure --superlink 192.168.1.100:9092 \
     --node-config "partition-id=1 num-partitions=2"
   ```
4. Terakhir, dari **server root project**:

   ```bash
   cd ~/Documents/fl-eksperimen4
   flwr run . local-deployment --stream
   ```

