# Modul 9

> a. How much data your publisher program will send to the message broker in one run?

Dalam satu kali eksekusi fungsi `main()`, publisher akan mengirim tepat 5 pesan ke message broker, karena memang terdapat lima pemanggilan `publish_event` dalam kode, setiap pesan berupa instance `UserCreatedEventMessage` yang berisi dua field `user_id` dan `user_name` yang keduanya bertipe String, setelah diserialisasi dengan Borsh, biasanya menghasilkan payload sekitar 30-50 byte per pesan. Dengan demikian total data payload yang dikirim berkisar 150-250 byte, belum termasuk overhead protokol AMQP dan header lainnya.


> b. The url of: “amqp://guest:guest@localhost:5672” is the same as in the subscriber program, what does it mean?

URL koneksi dari `amqp://guest:guest@localhost:5672` sama di program publisher maupun subscriber menandakan bahwa keduanya terhubung ke instansi RabbitMQ atau broker AMQP lain yang kompatibel yang sama, berjalan di mesin lokal atau localhost pada port 5672 menggunakan kredensial default guest/guest, sehingga pesan yang dikirim publisher bisa diambil dan diproses oleh subscriber melalui broker yang sama.

## Running RabbitMQ

![running rabbitMQ](images/running-publisher.png)


## Sending and Processing Event

![sending event](images/sending-event.png)

## Monitoring Chart based on Publisher
![monitoring chart](images/monitoring-chart-1.png)
![monitoring chart](images/monitoring-chart-2.png)

Spike pada grafik RabbitMQ terjadi karena setiap kali menjalankan program publisher, ia langsung mengirim lima pesan secara beruntun ke broker, sehingga dalam sekejap laju publish (publish rate) melonjak tajam. Setelah itu, subscriber baru mengambil dan memproses pesan-pesan tersebut—terlihat sebagai kenaikan laju consume (consume rate) hingga grafik kembali ke level normal ketika antrean kosong.


## Bonus

### Sending and Processing Event with Cloud

![bonus sending](images/bonus-sending-event.png)

### Monitoring Chart based on Publisher with Cloud

![monitoring cloud](images/bonus-monitoring.png)

Ketika saya menjalankan cargo run dengan menggunakan cloud, ada sedikit perbedaan pada spike yaitu spike pada cloud terlihat lebih landai dibandingkan dengan localhost. Hal tersebut dikarenakan koneksi ke broker cloud (CloudAMQP) menambahkan lapisan TLS dan latensi jaringan. Setiap frame AMQPS perlu dienkripsi/dekripsi dan melewati internet. Maka waktu round-trip untuk setiap publish dan ack menjadi lebih panjang daripada di localhost; akibatnya, ketika publisher mengirim batch pesan, lonjakan pada grafik terdistribusi selama periode yang lebih panjang, dan setelah itu penurunan antrian juga terjadi lebih perlahan dibanding saat broker ada di mesin lokal dengan latensi hampir nol.
