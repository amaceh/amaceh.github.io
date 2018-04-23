---
layout: post
title: Trik Instalasi Hadoop 3.1 dalam Mode Cluster
categories:
- big-data
---

Kali ini saya akan membagikan "sedikit tips" dalam instalasi Hadoop 3.1 dalam mode cluster, saya rasa tips ini berguna bagi kamu yang sedang mencoba instalasi dengan beberapa laptop, karena saya dan teman-teman saya menemui beberapa kendala saat instalasi ini sendiri dan perlu waktu beberapa hari bagi kami menemukan solusi ~~mungkin karena kami masih terlalu noob~~.

---
## Kebutuhan
- Package binary Hadoop terbaru di [situs resminya](http://hadoop.apache.org).
- Beberapa laptop yang digunakan sebagai cluster, yang sudah terinstall JDK.
- Jaringan komputer yang menghubungkan semua laptop, saya sendiri menggunakan wifi yang ada dikampus atau kafe, tapi lebih baik jika punya jaringan menggunakan kabel (lebih cepat).

## Instalasi
- Beberapa situs di internet sudah menyediakan cara instalasi yang cukup lengkap, beberapa referensi yang bisa saya berikan antara lain [Referensi satu](http://michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/), dan [Referensi dua](https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/). kamu juga bisa cari referensi lain di google, masalah versi tinggal dilakukan penyesuaian karena tidak terlalu berbeda (hingga saat blog ini ditulis pada versi 3.1). ~~Blog ini saya tulis bukan untuk nambahin referensi internet mengenai cara yang sama yang udah banyak XD~~.
- Satu hal yang bisa saya beri tahu, config harusnya hanya dilakukan di namenode, kemudian dicopy menggunakan scp(ssh) di semua node.
- Jika kamu mau lihat config hadoop saya di `$HADOOP_HOME/etc/hadoop`, kamu bisa lihat di [Repository saya](https://github.com/amaceh/Big-Data).

## Daftar Kendala dan Solusi
1. Muncul Error native library hadoop tidak bisa ditemukan, ketika semua perintah hadoop dijalankan.
	- native hadoop ini penting karena yang saya rasakan dengan native library ini job jadi lebih sedikit errornya (mungkin ada hubungannya dengan performa).
	- error ini muncul karena native lib pathnya somehow tidak menunjuk ke folder `$HADOOP_HOME/lib/native`
	- solusinya tambahkan line dibawah berikut ke `~/.bashrc` (home dari user hadoop).
```bash
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
	export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
	export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
```

    *  jika masih ada error, tambahkan `log4j.logger.org.apache.hadoop.util.NativeCodeLoader=DEBUG` pada `$HADOOP_HOME/etc/hadoop` dan lihat kemana sebenarnya hadoop mencari native lib.
    * jika hadoop malah mencari ke folder `$HADOOP_HOME/lib/` , pindahkan seluruh isi folder `$HADOOP_HOME/lib/native` ke `$HADOOP_HOME/lib/`.
2. Saat cluster di jalankan, saat di cek dengan JPS, semua service di namenode dan datanode ada (namenode, secondary namenode, resource manager, datanode, dll). Namun saat di cek dengan perintah `hdfs dfsadmin -report` datanode ada datanode yang tidak muncul, atau malah tidak muncul sama sekali.
	- Cek Koneksi Jaringan, ping satu sama lain dari namenode ke datanode yang bermasalah dan sebaliknya.
	- Jika masih belum terdeteksi hentikan hadoop dengan `stop-all.sh`. lalu jalankan perintah dibawah ini.
```bash
	sudo rm -rf /usr/local/hadoop_store/hdfs/datanode/*
	sudo rm -rf /usr/local/hadoop_store/hdfs/namenode/*
	sudo rm -rf /app/hadoop/tmp/*
```
3. Saat cluster di jalankan, saat di cek dengan JPS, semua service di namenode dan datanode ada (namenode, secondary namenode, resource manager, datanode, dll). Saat di cek dengan perintah `hdfs dfsadmin -report`, datanode ada. Tetapi ketika dicek dengan perintah `yarn node -list`, tidak ada node yang muncul atau hanya datanode yang merupakan namenode saja yang muncul. Lalu ketika mapreduce dijalankan job accepted tapi hang dan tidak jalan sama sekali.
	- sekali lagi cek koneksi jaringan
	- jika masih error, ini ada hubungannya dengan DNS Reverse dan Forward, jadi DNS yang digunakan resource manager menggunakan hostname dari `/etc/hostname` sementara yang kita daftarkan di file worker adalah hostname dari `/etc/hosts`. solusinya daftarkan kedua hosts di `/etc/hosts` contohnya berikut ini.
```bash
# Hadoop Multi Node IP Config
192.168.43.222  master-dulz.id.upi
192.168.43.235  slave-opal.id.upi
192.168.43.222  master-dulz.id
192.168.43.235  slave-opal.id
```

4. Kendala lain yang belum saya temukan.

---

Nah segitu saja tips dan trik yang bisa saya bagikan, semoga tips ini berguna buat kamu yang sedang belajar konfigurasi hadoop cluster.

---
