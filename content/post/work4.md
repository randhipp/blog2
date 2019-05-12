+++
draft = false
image = ""
showonlyimage = false
date = "2019-05-12T09:30:31+07:00"
title = "Integrasi Laravel dengan Telegram"
writer = "Randhi P. Putra"
categories = [ "code" ]
weight = 4
+++

Hari ini saya ingin mencoba menulis blog kembali setelah lama tidak aktif di dunia blog. Belakangan ini sedang belajar Laravel, jadi tidak ada salahnya saya tuliskan disini supaya tidak lupa. Tulisan ini saya buat berseri ya!
<!--more-->

Saya sedang membuat project experiment, sebuah **Sistem Informasi Rental Kendaraan** yang maunya saya bisa terintegrasi dengan Telegram, baik dari Notification ataupun Login.

Nah yang sudah saya capai saat ini adalah :

1. Integrasi Telegram Widget Login - [Part 1][1]
2. Integrasi Telegram Notification - [Part 2][2]

Disini kenapa saya mau integrasikan Laravel Project dengan telegram karena :

- 'Masih' Free & Open Source
- API boleh digunakan tanpa daftar dan request by email. 
- Whatsapp harus request by email, dan apabila tidak disetujui opsi lain menggunakan 3rd-party API yang berbayar.

>Our API is open, and we welcome developers to create their own Telegram apps. We also have a Bot API, a platform for developers that allows anyone to easily build specialized tools for Telegram, integrate any services, and even accept payments from users around the world. ~ Telegram FAQ


## Telegram Login Widget

Telegram login juga memudahkan proses input data di form register, karena data callback yang diberikan adalah :

- Telegram ID & username
- first & last name
- auth_time
- avatar
- token

Jadi apabila user baru mencoba Login, maka akan langsung diarahkan ke form register yang sudah terisi namanya.




![4]

Validasi lebih aman karena ada hash token, AuthTime, dan Secret Code yang digenerate dari Bot Token.

![3]

>Bonus : Telegram ID juga bisa langsung tersimpan di database, nantinya berfungsi untuk send notification

![5]

Contoh diatas adalah notifikasi yang dikirimkan otomatis oleh bot telegram kepada user yang login menggunakan Telegram Widget.

Bagaimana? tertarik untuk implementasi di website anda? :D


[1]: ../work1
[2]: ../work2
[3]: https://core.telegram.org/file/811140314/17c1/xf4ULBL5tmE.58438/07ff5b2958ed0e7e36 "login-confirmation"
[4]: /img/upload/contoh-login.png
[5]: /img/upload/Notification-success.png