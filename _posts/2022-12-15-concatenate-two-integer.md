---
title: Menggabungkan 2 Buah Integer
---

Jadi begini teman-teman, pada waktu itu saya ingin menggabungkan 2 buah integer.<!--more--> Kemudian saya cari caranya di internet. Lalu ketemulah artikel ini:

![GAMBAR1]({{ site.url }}/img/posts/concatenate-two-integer/1.jpg)

![GAMBAR2]({{ site.url }}/img/posts/concatenate-two-integer/2.jpg)

Kemudian cara tersebut saya terapkan pada projek saya, dan setelah beberapa hari saya mulai berpikir, kenapa saya harus membuat 3 buah std::string. Padahal 1 aja juga bisa. Beginilah kalo cara yang punya saya sendiri:

{% highlight cpp %}
#include <iostream>
#include <string>
using namespace std;
 
// Function to concatenate
// two integers into one
int concat(int a, int b)
{
    string str;

    str.append(to_string(a))
        .append(to_string(b));

    return stoi(str);
}
 
int main()
{
    int a = 23;
    int b = 43;
 
    cout << concat(a, b) << endl;
 
    return 0;
}
{% endhighlight %} 

Dan ini perbedaan nya jika diterapkan pada projek saya:

Cara dari geeksforgeeks.org:

{% highlight cpp %}
/*
MIT License

Copyright (c) 2022 Wildan R. (@wildan9)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
*/

#pragma once

#include <cmath>
#include <string>
#include "raylib.h"

struct Vector2D : public Vector2
{
    Vector2D(const Vector2& vec) : Vector2{ vec.x, vec.y } {}
    Vector2D(float x, float y) : Vector2{ x, y } {}
    Vector2D(float x) : Vector2{ x, 0.0f } {}
    Vector2D() : Vector2{ 0.0f, 0.0f } {}
    float Length() const;
    float DotProduct(const Vector2D& vec) const;
    Vector2D Add(const Vector2D& vec) const;
    Vector2D Add(const Vector2& vec) const;
    Vector2D Subtract(const Vector2D& vec) const;
    Vector2D Subtract(const Vector2& vec) const;
    Vector2D Scale(float scale) const;
    Vector2D Normalize() const;
    Vector2D Rotate(float angle) const;
    std::string ToString() const;
    int ConcatXY();
};

inline float Vector2D::Length() const
{
    return { sqrtf((x * x) + (y * y)) };
}

inline float Vector2D::DotProduct(const Vector2D& vec) const
{
    return { x * vec.x + y * vec.y };
}

inline Vector2D Vector2D::Add(const Vector2D& vec) const
{
    return { x + vec.x, y + vec.y };
}

inline Vector2D Vector2D::Add(const Vector2& vec) const
{
    return { x + vec.x, y + vec.y };
}

inline Vector2D Vector2D::Subtract(const Vector2D& vec) const
{
    return { x - vec.x, y - vec.y };
}

inline Vector2D Vector2D::Subtract(const Vector2& vec) const
{
    return { x - vec.x, y - vec.y };
}

inline Vector2D Vector2D::Scale(float scale) const
{
    return { x * scale, y * scale };
}

inline Vector2D Vector2D::Normalize() const
{
    Vector2D result{};

    float length{ sqrtf((x * x) + (y * y)) };

    if (length > 0)
    {
        float ilength = 1.0f / length;
        result.x = x * ilength;
        result.y = y * ilength;
    }

    return result;
}

inline Vector2D Vector2D::Rotate(float angle) const
{
    Vector2D result{};

    float cosres{ cosf(angle) };
    float sinres{ sinf(angle) };

    result.x = x * cosres - y * sinres;
    result.y = x * sinres + y * cosres;

    return result;
}

inline std::string Vector2D::ToString() const
{
    std::string str{};

    str.append("x: ")
        .append(std::to_string((int)x))
        .append("  y: ")
        .append(std::to_string((int)y));

    return str;
}

inline int Vector2D::ConcatXY()
{
    std::string s1 = std::to_string((int)x);
    std::string s2 = std::to_string((int)y);

    std::string s = s1 + s2;

    int s3 = stoi(s);

    return s3;
}
{% endhighlight %} 

Cara dari saya sendiri:

{% highlight cpp %}
/*
MIT License

Copyright (c) 2022 Wildan R. (@wildan9)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
*/

#pragma once

#include <cmath>
#include <string>
#include "raylib.h"

struct Vector2D : public Vector2
{
    Vector2D(const Vector2& vec) : Vector2{ vec.x, vec.y } {}
    Vector2D(float x, float y) : Vector2{ x, y } {}
    Vector2D(float x) : Vector2{ x, 0.0f } {}
    Vector2D() : Vector2{ 0.0f, 0.0f } {}
    float Length() const;
    float DotProduct(const Vector2D& vec) const;
    Vector2D Add(const Vector2D& vec) const;
    Vector2D Add(const Vector2& vec) const;
    Vector2D Subtract(const Vector2D& vec) const;
    Vector2D Subtract(const Vector2& vec) const;
    Vector2D Scale(float scale) const;
    Vector2D Normalize() const;
    Vector2D Rotate(float angle) const;
    std::string ToString() const;
    int ConcatXY() const;
};

inline float Vector2D::Length() const
{
    return { sqrtf((x * x) + (y * y)) };
}

inline float Vector2D::DotProduct(const Vector2D& vec) const
{
    return { x * vec.x + y * vec.y };
}

inline Vector2D Vector2D::Add(const Vector2D& vec) const
{
    return { x + vec.x, y + vec.y };
}

inline Vector2D Vector2D::Add(const Vector2& vec) const
{
    return { x + vec.x, y + vec.y };
}

inline Vector2D Vector2D::Subtract(const Vector2D& vec) const
{
    return { x - vec.x, y - vec.y };
}

inline Vector2D Vector2D::Subtract(const Vector2& vec) const
{
    return { x - vec.x, y - vec.y };
}

inline Vector2D Vector2D::Scale(float scale) const
{
    return { x * scale, y * scale };
}

inline Vector2D Vector2D::Normalize() const
{
    Vector2D result{};

    float length{ sqrtf((x * x) + (y * y)) };

    if (length > 0)
    {
        float ilength = 1.0f / length;
        result.x = x * ilength;
        result.y = y * ilength;
    }

    return result;
}

inline Vector2D Vector2D::Rotate(float angle) const
{
    Vector2D result{};

    float cosres{ cosf(angle) };
    float sinres{ sinf(angle) };

    result.x = x * cosres - y * sinres;
    result.y = x * sinres + y * cosres;

    return result;
}

inline std::string Vector2D::ToString() const
{
    std::string str{};

    str.append("x: ")
        .append(std::to_string((int)x))
        .append("  y: ")
        .append(std::to_string((int)y));

    return str;
}

inline int Vector2D::ConcatXY() const
{
    std::string str{};

    str.append(std::to_string((int)x))
        .append(std::to_string((int)y));

    return stoi(str);
}
{% endhighlight %} 

Bagaimana kalo menurut teman-teman? tapi karena disini fitur komennya tidak aktif, nampaknya semua ini cukup untuk dijadikan renungan kita bersama. 

Referensi:<br>
[https://www.geeksforgeeks.org/how-to-concatenate-two-integer-values-into-one/](https://www.geeksforgeeks.org/how-to-concatenate-two-integer-values-into-one/){:target="\_blank"}