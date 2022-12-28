---
title: Menggabungkan 2 Buah Integer Part 2
---

Pada tulisan sebelumnya, ada sebuah pertanyaan <!--more--> dan ini jawabannya:

{% highlight cpp %}

.......
.......
.......

inline int Vector2D::ConcatXY() const
{
    return stoi(std::to_string((int)x) + std::to_string((int)y));
}
{% endhighlight %}

Tapi pada saat saya coba "+" untuk menampilkan waktu tidak bisa.

Jika menggunakan "+" waktunya tidak muncul.

Kodenya jika menggunakan "+":
{% highlight cpp %}

.......
.......
.......

std::string time { "Time: " };

time + std::to_string(hour) + ":" + std::to_string(ltm->tm_min) + ":" + std::to_string(ltm->tm_sec) + " " + strAmPm;

.......
.......
.......
{% endhighlight %}

![GAMBAR1]({{ site.url }}/img/posts/concatenate-two-integer-2/1.jpg)

Jika menggunakan std::string::append() waktunya akan muncul.

Kodenya jika menggunakan std::string::append():
{% highlight cpp %}

.......
.......
.......

std::string time { "Time: " };

time.append(std::to_string(hour)).append(":").append(std::to_string(ltm->tm_min))
            .append(":").append(std::to_string(ltm->tm_sec)).append(" ").append(strAmPm);

.......
.......
.......
{% endhighlight %}

![GAMBAR2]({{ site.url }}/img/posts/concatenate-two-integer-2/2.jpg)

