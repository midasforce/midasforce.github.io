---
title: Menggabungkan 2 Buah Integer Part 3
---

Pada tulisan sebelumnya kita membahas mengenai operator "+" untuk menggabungkan string,<!--more--> dan saya jadi teringat bab pertama buku yang pernah saya baca ini:

![GAMBAR1]({{ site.url }}/img/posts/concatenate-two-integer-3/1.jpg)

Dibuku itu di jelaskan bagaimana melalukan penambahan dan pengurangan vektor menggunakan operator "+" dan "-" seperti ini:

![GAMBAR2]({{ site.url }}/img/posts/concatenate-two-integer-3/2.jpg)

Kemudian muncullah ide untuk menambahkan operator itu pada struct Vector2D milik saya. Sehingga nantinya Vector2D milik saya memiliki kemampuan yang sama seperti std::string, yaitu bisa menggunakan "+" atau append, tapi bedanya kalo Vector2D milik saya menggunakan "+" atau Add. Karena memang ada saatnya kita bisa menentukan mana yang lebih tepat untuk digunakan. Seperti ini contohnya:

![GAMBAR3]({{ site.url }}/img/posts/concatenate-two-integer-3/3.jpg)

Tapi setelah saya pikir-pikir lagi, sepertinya saya harus mengurungkan niat saya itu. Mengingat sampai saat ini saya juga masih belum tahu dua fungsi: Vector2D::Rotate() dan Vector2D::DotProduct() yang saya tulis beberapa bulan lalu mau saya gunakan untuk apa. 

Daftar Pustaka:<br>
[1] Eric Lengyel. Foundations of Game Engine Development Volume 1: Mathematics.
