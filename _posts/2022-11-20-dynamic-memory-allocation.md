---
title: Alokasi Memori Secara Dinamis
---

Mungkin sebelumnya teman-teman sudah pernah melihat contoh penggunaan memori dinamis<!--more--> pada c++ yang cukup sederhana seperi ini: 

{% highlight cpp %}
#include <iostream>
using namespace std;

int main() 
{
  // alokasi memori dinamis
  int* pointInt = new int;

  // memberikan nilai
  *pointInt = 45;

  // mencetak nilai
  cout << *pointInt << endl;

  // membebaskan memori yang sudah tidak digunakan lagi
  delete pointInt;

  return 0;
}
{% endhighlight %} 

Jika teman-teman sebelumnya sudah pernah melihat contoh seperti di atas, dan kemudian teman-teman bertanya selanjutnya kita mau ngapain lagi. Tulisan ini mungkin akan dapat memberikan sedikit jawaban. 

Sederhananya, kita akan membuat buah ajaib yang dapat memberikan stamina tak terbatas. Buah ajaib yang jika diartikan kedalam bahasa inggris bisa menjadi magic fruit dan bisa juga miracle fruit. Namun kali ini akan saya beri nama "Magic fruit". Pada C++ kita dapat memodelkan buah ajaib itu menjadi sebuah struct seperti berikut:

{% highlight cpp %}
struct MagicFruit
{
    // destructor untuk meng-unload texture buah ajaib apabila sudah tidak terpakai
    ~MagicFruit();

    // mendapatkan rectangle untuk "collision detection" 
    Rectangle GetCollision();

    // untuk me-load texture buah ajaib 
    void LoadTexture(Texture2D texture);

    // untuk menggambar buah ajaib 
    void Draw();

private:
    
    // texture buah ajaib
    Texture2D _texture{};

    // posisi buah ajaib
    const Vector2D _position{ 585.0f, 760.0f };
};
{% endhighlight %}

Sama seperti contoh pertama, kita akan mengalokasi memori secara dinamis dan kemudian menghapusnya apabila sudah tidak di gunakan. Pertama kita buat buah ajaib itu. 

{% highlight cpp %}
.......
.......
.......

// alokasi memori dinamis  
MagicFruit* magicFruit = new MagicFruit;

.......
.......
.......
{% endhighlight %} 

Kemudian kapan kita akan menghapusnya? anggaplah kita mempunyai sebuah class yang salah satu datanya adalah stamina, dan kita akan menjadikan nilai dari stamina itu menjadi 9.0f, tidak berkurang dan tidak bertambah. 

Seperti ini contohnya, dari class itu kita membuat sebuah objek, dan apabila objek itu bertabrakan dengan buah ajaib maka nilai stamina akan menjadi 9.0f, tidak berkurang dan tidak bertambah. Kiranya sepert ini gambaran class-nya:
{% highlight cpp %}
class Player
{
public:
    
    // mendapatkan rectangle untuk "collision detection" 
    Rectangle GetCollision();

    // untuk merubah nilai penanda
    void SetStamina(bool isDragonInside);

private:
    
    // stamina
    float _stamina;

    // penanda apakah sudah bertabrakn dengan buah ajaib
    bool _isDragonInside;
};
{% endhighlight %}

Dari struct MagicFruit dan class Player, kita akan membuat sebuah tabrakan yang dapat merubah data class player seperti berikut:

{% highlight cpp %}
.......
.......
.......

// pertama kita check apakah magic fruit nullptr
if (magicFruit != nullptr)
{
    // apabila tidak terdeteksi tabrakan maka magic fruit akan di gambar
    if (!CheckCollisionRecs(gameObj->player.GetCollision(), magicFruit->GetCollision()))
    {
        magicFruit->Draw();
    }

    // apabila terdeteksi tabrakan maka data class Player akan berubah, dan MagicFruit akan di hapus dan menjadi nullptr
    if (CheckCollisionRecs(gameObj->player.GetCollision(), magicFruit->GetCollision()))
    {
        gameObj->player.SetStamina(1);

        delete magicFruit;

        magicFruit = nullptr;
    }
}

.......
.......
.......
{% endhighlight %}

### Sebelum buah ajaib bertabrakan dengan pemain
<br>
![GAMBAR1]({{ site.url }}/img/posts/dynamic-memory-allocation/1.png)

### Sesudah buah ajaib bertabrakan dengan pemain
<br>
![GAMBAR2]({{ site.url }}/img/posts/dynamic-memory-allocation/2.png)

Mungkin hanya segitu saja yang bisa saya sampaikan, untuk full source code codenya akan saya lampirkan dibawah, dan untuk assetnya dapat teman-teman temukan di github saya. Sekian dan terima kasih !!

## Full Source Code

### 2DWorld.h

{% highlight cpp %}
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

#include <ctime>
#include <memory>
#include <iomanip>
#include <sstream>
#include "Screen.h"
#include "Math2D.h"
#include "GameObject.h"

#define RAYGUI_IMPLEMENTATION
#include "extras/raygui.h"
{% endhighlight %}

### 2DWorld.cpp

{% highlight cpp %}
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

#include "2DWorld.h"

const int screenWidth  = 512;
const int screenHeight = 512;
int framesCounter      = 0;
bool endGame           = 0;

struct Audio
{
    bool  muted         = 0;
    float masterVolume  = 0.2f;
    float currentVolume = 0.2f;
};
struct Timer
{
    void Start(const float lifeTime) 
    { 
        _lifeTime = lifeTime; 
    }
    
    void Update() 
    { 
        if (_lifeTime > 0) _lifeTime -= GetFrameTime(); 
    }

    bool Done() const 
    { 
        return _lifeTime <= 0; 
    }
    
private: 
    float _lifeTime = 0.0f;
};
struct MagicFruit
{
    ~MagicFruit()
    {
        UnloadTexture(_texture);
    }

    Rectangle GetCollision()
    {
        return Rectangle{
            static_cast<float>(_position.x),
            static_cast<float>(_position.y),
            static_cast<float>(_texture.width),
            static_cast<float>(_texture.height)
        };
    }

    void LoadTexture(Texture2D texture)
    {
        _texture = texture;
    }

    void Draw()
    {
        DrawTextureV(_texture, _position, WHITE);
    }

private:
    Texture2D _texture{};
    const Vector2D _position{ 585.0f, 760.0f };
};
MagicFruit* magicFruit = new MagicFruit;

namespace raylib
{
    struct Camera2D : public ::Camera2D
    {
        Camera2D() : ::Camera2D{ offset = {}, target = {}, rotation = 0.0f, zoom = 1.0f } {}

        Camera2D& BeginMode()
        {
            ::BeginMode2D(*this);
            return *this;
        }

        Camera2D& EndMode()
        {
            ::EndMode2D();
            return *this;
        }
    };
}

Timer audioTimer   = {};
Timer teleportText = {};
Audio audio        = {};
Music mainBGM      = {};
Music logoBGM      = {};

static const char* strCameraMode[2]
{
    "Camera Mode: Smooth",
    "Camera Mode: Normal"
};
bool cameraNormalMode = 0;

void SetupWindow();

void InitGame();
void UpdateGame(raylib::Camera2D& camera, std::shared_ptr<GameObject> gameObj);
void EndGame();

void UpdateCamera(raylib::Camera2D& camera, Player& player);

void DrawVolumeBar(const int x = 10);
void DrawMuteCheckBox();
void DrawGamePlayHUD(const raylib::Camera2D& camera, const Player& player);

void UpdateVolumeTimer();

void InitAudio();
void UpdateAudio();
void InitBGM();
void ShutdownAudio();

void DrawGamePlayScreen(std::shared_ptr<GameObject> gameObj);

enum class ApplicationStates
{
    LOGO = 0,
    TITLE,
    GAMEPLAY,
    MENU,
    PAUSE
};
ApplicationStates applicationState = ApplicationStates::LOGO;

class LogoScreen : public Screen
{
public:
    void Draw() override
    {
        DrawHelixNebula();
    }
};
LogoScreen logoScreen;

class TitleScreen : public Screen
{
public:
    void Draw() override
    {
        DrawEarth();

        DrawText("Walking", 15, 20, 40, GREEN);

        DrawCenteredText(164, "Welcome to 2DWorld!", 24, GREEN);
        
        DrawCenteredText(220, "PRESS ENTER to Walking!", 19, GREEN);

        DrawVolumeBar(screenWidth - 220);
    }
};
TitleScreen titleScreen;

class MenuScreen : public Screen
{
public:
    void Draw() override
    {
        DrawDusk();

        float x = (screenWidth / 2) - 50;
        bool isBack = 0;
        bool isExit = 0;

        isBack = GuiButton({ x, 120, 120, 35 }, "Back");
        isExit = GuiButton({ x, 120 + 35, 120, 35 }, "Exit");

        const char* strVolume[6]{ "Muted", "Volume: #", "Volume: # #", "Volume: # # #", "Volume: # # # #", "Volume: # # # # #" };

        const Color volumeColors[4]{ RED, DARKGREEN, LIME, GREEN };

        int volume = (int)(audio.masterVolume * 10);

        if (volume > 5) volume = volume - 1;

        int volumeColor = 2;

        if (audio.masterVolume < 0.1f) volumeColor = 0;
        else if (audio.masterVolume > 0.1f && audio.masterVolume < 0.3f) volumeColor = 1;
        else if (audio.masterVolume > 0.3f && audio.masterVolume < 0.4f) volumeColor = 2;
        else if (audio.masterVolume > 0.4f) volumeColor = 3;

        int volumeBarPosX = (audio.masterVolume < 0.1f) ? (x + 15) : (x - 60);

        DrawText(strVolume[volume], volumeBarPosX, 120 + 140, 28, volumeColors[volumeColor]);

        bool isVolumeUp   = GuiButton({ x + 90, 120 + 200, 80, 30 }, "Volume: Up");
        bool isVolumeDown = GuiButton({ x - 30, 120 + 200, 80, 30 }, "Volume: Down");

        if (isVolumeUp && audio.masterVolume <= 0.5f && !audio.muted)
        {
            audio.masterVolume += 0.1f;
            audio.currentVolume += 0.1f;
        }
        if (isVolumeDown && audio.masterVolume >= 0.1f && !audio.muted)
        {
            audio.masterVolume -= 0.1f;
            audio.currentVolume -= 0.1f;
        }
        if (isVolumeUp && audio.muted || isVolumeDown && audio.muted)
        {
            audio.muted = !audio.muted;
            audio.masterVolume = audio.currentVolume;
        }

        time_t now = time(0);

        char* dt = ctime(&now);

        std::string strDateAndTime = "The local date and time is: ";

        DrawText(strDateAndTime.append(dt).c_str(), 50, 120 + 260, 18, RED);

        DrawFPS(screenWidth - 80, 8);

        DrawMuteCheckBox();

        if (isExit)
        {
            endGame = 1;
        }
        if (isBack)
        {
            SetActiveScreen(nullptr);

            SetWindowTitle("Walking");

            applicationState = ApplicationStates::GAMEPLAY;

            PauseMusicStream(mainBGM);

        }
    }
};
MenuScreen menuScreen;

class PauseScreen : public Screen
{
public:
    void Draw() override
    {
        DrawGalaxy();

        DrawCenteredText(220, "Paused", 60, BLUE);

        DrawVolumeBar();

        bool isMenu = 0;
        bool isBack = 0;

        isMenu = GuiButton({ screenWidth - 165, screenHeight - 160, 120, 35 }, "Menu");
        isBack = GuiButton({ screenWidth - 165, screenHeight - 120, 120, 35 }, "Back");
        
        if (isMenu)
        {
            SetActiveScreen(&menuScreen);

            SetWindowTitle("Menu");

            applicationState = ApplicationStates::MENU;
        }
        if (isBack)
        {
            SetActiveScreen(nullptr);

            SetWindowTitle("Walking");

            applicationState = ApplicationStates::GAMEPLAY;

            PauseMusicStream(mainBGM);
        }
    }
};
PauseScreen pauseScreen;

void SetupWindow()
{
    InitWindow(screenWidth, screenHeight, "Walking");
    
    SetExitKey(0);

    SetTargetFPS(60);

    Image icon = LoadImage("icon/earth_400_400.png");

    ImageFormat(&icon, PIXELFORMAT_UNCOMPRESSED_R8G8B8A8);

    ImageColorReplace(&icon, BLACK, BLANK);

    SetWindowIcon(icon);

    // free the image data
    UnloadImage(icon);
}

void InitGame()
{
    SetupWindow();

    InitAudio();

    InitScreenTexture();

    SetActiveScreen(&logoScreen);

    magicFruit->LoadTexture(LoadTexture("textures/magic_fruit/apple.png"));
}

void UpdateCamera(raylib::Camera2D& camera, Player& player)
{
    Vector2D diff{ player.GetPosition().Subtract(camera.target) };

    float speed{ player.GetSpeed() };

    speed = (speed == 2.0f) ? fmaxf(0.8f * diff.Length(), 70.0f) : fmaxf(0.8f * diff.Length(), 240.0f);

    camera.offset = Vector2D{ screenWidth / 2.0f,  screenHeight / 2.0f };

    if (IsKeyPressed(KEY_C)) cameraNormalMode = !cameraNormalMode;

    if (cameraNormalMode)
    {
        if (diff.Length() > 10.0f)
        {
            camera.target = Vector2D{ camera.target }
                .Add(diff.Scale(0.394f * GetFrameTime() / diff.Length()));
        }
        else
        {
            camera.target = player.GetPosition();
        }
    }

    // don't need cinematic effect when teleporting
    if (diff.Length() > 1000.0f) camera.target = player.GetPosition();
    
    if (diff.Length() > 10.0f)
    {
        camera.target = Vector2D{ camera.target }.Add(diff.Scale(speed * GetFrameTime() / diff.Length()));
    }

    if ((GetMouseWheelMove() > 0.0f) && (camera.zoom < 2.0f))
    {
        camera.zoom += 0.1f;
    }
    if ((GetMouseWheelMove() < 0.0f) && (camera.zoom > 1.0f))
    {
        camera.zoom -= 0.1f;
    }
}

void DrawGamePlayScreen(std::shared_ptr<GameObject> gameObj)
{ 
    gameObj->Draw(GetFrameTime()); 
}

void DrawGamePlayHUD(const raylib::Camera2D& camera, const Player& player)
{
    std::string strFPS{ "FPS: " };
    std::string strStatus{};

    int cameraMode = (cameraNormalMode) ? 1 : 0;

    time_t now = time(0);

    tm* ltm = localtime(&now);

    std::string time = { "Time: " };

    int hour = ltm->tm_hour;

    if (ltm->tm_hour > 12) hour = hour - 12;

    const char* strAmPm = (ltm->tm_hour < 12) ? "AM" : "PM";

    time.append(std::to_string(hour)).append(":").append(std::to_string(ltm->tm_min))
        .append(":").append(std::to_string(ltm->tm_sec)).append(" ").append(strAmPm);

    Vector2D diff{ player.GetPosition().Subtract(camera.target) };

    if (player.GetDirection().Length() == 0) strStatus = "Status: IDLE";
    else strStatus = (player.GetSpeed() == 4.0f) ? "Status: Walk Fast" : "Status: Walk Slow";

    DrawText(player.GetPosition().ToString().c_str(), 10, screenHeight - 50, 24, BLACK);

    std::string strCameraZoom = { "Camera Zoom: " };

    float cameraZoom = camera.zoom;

    std::stringstream stream;
    stream << std::fixed << std::setprecision(1) << cameraZoom;

    strCameraZoom.append(stream.str());

    DrawText(strCameraZoom.c_str(), 10, screenHeight - 124, 18, BLACK);

    DrawText(strCameraMode[cameraMode], 10, screenHeight - 90, 18, BLACK);

    const char* strStamina[7]{ "Stamina: 0", "Stamina: #", "Stamina: # #", "Stamina: # # #", "Stamina: # # # #", "Stamina: # # # # #", "Stamina: # # # # # #" };

    Color staminaColor = GREEN;

    if ((int)player.GetStamina() == 0) staminaColor = RED;
    else if ((int)player.GetStamina() > 0 && (int)player.GetStamina() < 4) staminaColor = DARKGREEN;
    else if ((int)player.GetStamina() > 4) staminaColor = GREEN;

    const char* stamina = (player.GetStamina() == 9.0f) ? "Stamina: Unlimited" : strStamina[(int)player.GetStamina()];

    DrawText(stamina, 10, screenHeight - 150, 18, staminaColor);

    DrawText(strCameraZoom.c_str(), 10, screenHeight - 124, 18, BLACK);

    if (player.IsInvisible()) DrawText("You are invisible", 10, screenHeight - 200, 20, RED);

    if (diff.Length() > 1000.0f) teleportText.Start(4.0f);
    teleportText.Update();

    if (!teleportText.Done()) DrawCenteredText(screenHeight - 270, "You are teleported", 24, RED);

    DrawText(strFPS.append(std::to_string(GetFPS())).c_str(), screenWidth - 140, 10, 18, RED);

    DrawText(time.c_str(), screenWidth - 140, 35, 18, RED);
    
    DrawText(strStatus.c_str(), screenWidth - 165, screenHeight - 40, 19, RED);

    DrawVolumeBar();
}

void DrawVolumeBar(const int x)
{
    const char* strVolume[6]{ "Muted", "Volume: #", "Volume: # #", "Volume: # # #", "Volume: # # # #", "Volume: # # # # #" };

    const Color volumeColors[4]{ RED, DARKGREEN, LIME, GREEN };

    int volume = (int)(audio.masterVolume * 10);

    if (volume > 5) volume = volume - 1;

    if (audio.masterVolume < 0.1f || !audioTimer.Done())
    {
        int volumeColor = 2;

        if (audio.masterVolume < 0.1f) volumeColor = 0;
        else if (audio.masterVolume > 0.1f && audio.masterVolume < 0.3f) volumeColor = 1;
        else if (audio.masterVolume > 0.3f && audio.masterVolume < 0.4f) volumeColor = 2;
        else if (audio.masterVolume > 0.4f) volumeColor = 3;

        DrawText(strVolume[volume], x, 10, 24, volumeColors[volumeColor]);
    }

    DrawMuteCheckBox();
}

void DrawMuteCheckBox()
{
    const char* strMuteCheckBox = (audio.muted) ? "Muted: True" : "Muted: False";

    GuiControlState state = guiState;
    Rectangle textBounds  = { 0 };
    Rectangle bounds      = { screenWidth - 165, screenHeight - 80, 20, 20 };
    textBounds.width      = (float)GetTextWidth(strMuteCheckBox);
    textBounds.height     = (float)GuiGetStyle(DEFAULT, TEXT_SIZE);
    textBounds.x          = bounds.x + bounds.width + GuiGetStyle(CHECKBOX, TEXT_PADDING);
    textBounds.y          = bounds.y + bounds.height / 2 - GuiGetStyle(DEFAULT, TEXT_SIZE) / 2;

    if (GuiGetStyle(CHECKBOX, TEXT_ALIGNMENT) == GUI_TEXT_ALIGN_LEFT) textBounds.x = bounds.x - textBounds.width - GuiGetStyle(CHECKBOX, TEXT_PADDING);

    if ((state != GUI_STATE_DISABLED) && !guiLocked)
    {
        Vector2D mousePoint = GetMousePosition();

        Rectangle totalBounds = {
            (GuiGetStyle(CHECKBOX, TEXT_ALIGNMENT) == GUI_TEXT_ALIGN_LEFT) ? textBounds.x : bounds.x,
            bounds.y,
            bounds.width + textBounds.width + GuiGetStyle(CHECKBOX, TEXT_PADDING),
            bounds.height,
        };

        if (CheckCollisionPointRec(mousePoint, totalBounds))
        {
            if (IsMouseButtonDown(MOUSE_LEFT_BUTTON)) state = GUI_STATE_PRESSED;
            else state = GUI_STATE_FOCUSED;

            if (IsMouseButtonReleased(MOUSE_LEFT_BUTTON)) audio.muted = !audio.muted;
        }
    }

    GuiDrawRectangle(bounds, GuiGetStyle(CHECKBOX, BORDER_WIDTH), Fade(GetColor(GuiGetStyle(CHECKBOX, BORDER + (state * 3))), guiAlpha), BLANK);

    if (audio.muted || audio.masterVolume < 0.1f)
    {
        Rectangle check = { bounds.x + GuiGetStyle(CHECKBOX, BORDER_WIDTH) + GuiGetStyle(CHECKBOX, CHECK_PADDING),
                            bounds.y + GuiGetStyle(CHECKBOX, BORDER_WIDTH) + GuiGetStyle(CHECKBOX, CHECK_PADDING),
                            bounds.width - 2 * (GuiGetStyle(CHECKBOX, BORDER_WIDTH) + GuiGetStyle(CHECKBOX, CHECK_PADDING)),
                            bounds.height - 2 * (GuiGetStyle(CHECKBOX, BORDER_WIDTH) + GuiGetStyle(CHECKBOX, CHECK_PADDING)) };
        GuiDrawRectangle(check, 0, BLANK, Fade(GetColor(GuiGetStyle(CHECKBOX, TEXT + state * 3)), guiAlpha));
    }

    GuiDrawText(strMuteCheckBox, textBounds, 
        (GuiGetStyle(CHECKBOX, TEXT_ALIGNMENT) == GUI_TEXT_ALIGN_RIGHT) ? GUI_TEXT_ALIGN_LEFT : GUI_TEXT_ALIGN_RIGHT, Fade(GetColor(GuiGetStyle(LABEL, TEXT + (state * 3))), guiAlpha));
}

void UpdateVolumeTimer()
{
    if (IsKeyPressed(KEY_M) || IsKeyPressed(KEY_L) || IsKeyPressed(KEY_K)) audioTimer.Start(4.0f);

    audioTimer.Update();
}

void UpdateGame(raylib::Camera2D& camera, std::shared_ptr<GameObject> gameObj)
{
    UpdateAudio();

    UpdateVolumeTimer();

    switch (applicationState)
    {
    case ApplicationStates::LOGO:
    {
        UpdateMusicStream(logoBGM);

        PlayMusicStream(logoBGM);

        framesCounter++;

        if (framesCounter > 180)
        {
            SetActiveScreen(&titleScreen);

            applicationState = ApplicationStates::TITLE;

            StopMusicStream(logoBGM);

            logoBGM.frameCount = 0;

            UnloadMusicStream(logoBGM);
        }

    } break;
    case ApplicationStates::TITLE:
    {
        UpdateMusicStream(mainBGM);

        PlayMusicStream(mainBGM);

        if (IsKeyPressed(KEY_ENTER))
        {
            SetActiveScreen(nullptr);

            applicationState = ApplicationStates::GAMEPLAY;

            PauseMusicStream(mainBGM);
        }

    } break;
    case ApplicationStates::GAMEPLAY:
    {
        if (IsKeyPressed(KEY_P))
        {
            SetActiveScreen(&pauseScreen);

            SetWindowTitle("Paused");

            applicationState = ApplicationStates::PAUSE;
        }
        if (IsKeyPressed(KEY_Q) || IsKeyPressed(KEY_ESCAPE))
        {
            SetActiveScreen(&menuScreen);

            SetWindowTitle("Menu");

            applicationState = ApplicationStates::MENU;
        }

    } break;
    case ApplicationStates::MENU:
    {
        UpdateMusicStream(mainBGM);

        ResumeMusicStream(mainBGM);

        if (IsKeyPressed(KEY_Q) || IsKeyPressed(KEY_ESCAPE))
        {
            SetActiveScreen(nullptr);

            SetWindowTitle("Walking");

            applicationState = ApplicationStates::GAMEPLAY;

            PauseMusicStream(mainBGM);
        }

    } break;
    case ApplicationStates::PAUSE:
    {
        UpdateMusicStream(mainBGM);

        ResumeMusicStream(mainBGM);

        if (IsKeyPressed(KEY_P))
        {
            SetActiveScreen(nullptr);

            SetWindowTitle("Walking");

            applicationState = ApplicationStates::GAMEPLAY;

            PauseMusicStream(mainBGM);
        }

    } break;
    default: break;
    }

    BeginDrawing();

    ClearBackground(SKYBLUE);

    if (applicationState == ApplicationStates::GAMEPLAY)
    {
        UpdateCamera(camera, gameObj->player);

        gameObj->CheckCollision();

        gameObj->PlayWalkSound();

        camera.BeginMode();

        DrawGamePlayScreen(gameObj);
        
        if (magicFruit != nullptr)
        {
            if (!CheckCollisionRecs(gameObj->player.GetCollision(), magicFruit->GetCollision()))
            {
                magicFruit->Draw();
            }

            if (CheckCollisionRecs(gameObj->player.GetCollision(), magicFruit->GetCollision()))
            {
                gameObj->player.SetStamina(1);

                delete magicFruit;

                magicFruit = nullptr;
            }
        }

        camera.EndMode();

        bool isMenu = 0;

        isMenu = GuiButton({ screenWidth - 165, screenHeight - 120, 120, 35 }, "Menu");

        if (isMenu)
        {
            SetActiveScreen(&menuScreen);

            SetWindowTitle("Menu");

            applicationState = ApplicationStates::MENU;
        }

        DrawGamePlayHUD(camera, gameObj->player);
    }

    DrawScreen();

    EndDrawing();
}

void EndGame()
{
    UnloadScreenTexture();

    ShutdownAudio();

    CloseWindow();
}

void InitAudio()
{
    InitAudioDevice();

    InitBGM();
}

void UpdateAudio()
{
    if (IsKeyPressed(KEY_M)) audio.muted = !audio.muted;

    if (IsKeyPressed(KEY_L) && audio.masterVolume <= 0.5f && !audio.muted)
    {
        audio.masterVolume += 0.1f;
        audio.currentVolume += 0.1f;
    }
    if (IsKeyPressed(KEY_K) && audio.masterVolume >= 0.1f && !audio.muted)
    {
        audio.masterVolume -= 0.1f;
        audio.currentVolume -= 0.1f;
    }

    if (audio.muted   && IsKeyPressed(KEY_M) || audio.muted) audio.masterVolume  = 0.0f;
    if (!audio.muted  && IsKeyPressed(KEY_M) || !audio.muted) audio.masterVolume = audio.currentVolume;

    if (IsKeyPressed(KEY_L) && audio.muted || IsKeyPressed(KEY_K) && audio.muted)
    {
        audio.muted = !audio.muted;
        audio.masterVolume = audio.currentVolume;
    }

    SetMasterVolume(audio.masterVolume);
}

void InitBGM()
{
    mainBGM = LoadMusicStream("sounds/epic_cinematic_trailer_elite.ogg");
    logoBGM = LoadMusicStream("sounds/magic_astral_sweep_effect.ogg");

    logoBGM.looping = 0;
    mainBGM.looping = 1;
}

void StopBGM()
{
    if (mainBGM.frameCount > 0)
    {
        StopMusicStream(mainBGM);

        mainBGM.frameCount = 0;

        UnloadMusicStream(mainBGM);
    }
}

void ShutdownAudio()
{
    StopBGM();
    
    CloseAudioDevice();
}

int main()
{
    InitGame();

    std::shared_ptr<GameObject> gameObj{ new GameObject };
    
    raylib::Camera2D camera{};

    while (!endGame)
    {
        endGame = WindowShouldClose();

        UpdateGame(camera, gameObj);
    }

    EndGame();

    return 0;
}
{% endhighlight %}

### BaseAnimation.h

{% highlight cpp %}
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

#pragma once

#include "Math2D.h"

class BaseAnimation
{
public:
    virtual ~BaseAnimation() = default;

protected:
    void Animate(const Vector2D& position, const Texture2D& texture, const float deltaTime, const float size, 
        const float row, float facing = 1.0f, float timer = 0.0f, bool animate = 1, float rotation = 0.0f);

private:
    float _timer{};
    int _frame{};
};
{% endhighlight %}

### BaseAnimation.cpp

{% highlight cpp %}
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

#include "Animation.h"

void BaseAnimation::Animate(const Vector2D& position, const Texture2D& texture, const float deltaTime, 
    const float size, const float row, float facing, float timer, bool animate, float rotation)
{
    static const float updateTime = 0.084f;

    if (animate)
    {
        _timer += deltaTime;
        if (_timer >= updateTime)
        {
            _frame++;
            _timer = timer;
            if (_frame > row) _frame = 1;
        }
    }

    Rectangle source{
        _frame * (float)texture.width / row,
        0.0f, facing * (float)texture.width / row,
        (float)texture.height
    };
    Rectangle dest{
        position.x, position.y,
        size * (float)texture.width / row,
        size * (float)texture.height
    };
    DrawTexturePro(texture, source, dest, Vector2D{}, rotation, WHITE);
}
{% endhighlight %}

### Animal.cpp

{% highlight cpp %}
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

#include <array>
#include "Animation.h"

class Rhino : public BaseAnimation
{
public:
	~Rhino()
	{
		UnloadTexture(_texture);
	}

	void SetPosition(const Vector2D& pos);

	void Draw(const float deltaTime);

	Rectangle GetCollision();

private:
	Texture2D _texture{ LoadTexture("textures/animals/rhino_idle.png") };
	Vector2D _texturePos{};
};

inline void Rhino::SetPosition(const Vector2D& pos)
{
	_texturePos = pos;
}

inline void Rhino::Draw(const float deltaTime)
{
	Animate(_texturePos, _texture, deltaTime, 2.0f, 8.0f);
}

inline Rectangle Rhino::GetCollision()
{
	return Rectangle{
		static_cast<float>(_texturePos.x + 12.0f),
		static_cast<float>(_texturePos.y + 25.0f),
		static_cast<float>(2.0f * _texture.width / 8.0f - 30.0f),
		static_cast<float>(2.0f * _texture.height - 25.0f)
	};
}

class Bat : public BaseAnimation
{
public:
	~Bat()
	{
		UnloadTexture(_texture);
	}

	void SetPosition(const Vector2D& pos);

	void SetFlyRadius(const float flyRadius);

	void Draw(const float deltaTime);

private:
	Texture2D _texture{ LoadTexture("textures/animals/bat_fly.png") };
	Vector2D _speed{ 1.5f, 1.0f };
	Vector2D _texturePos{};
	float _facing{ 1.0f };
	float _flyRadius{};
};

inline void Bat::SetFlyRadius(const float flyRadius)
{
	_flyRadius = flyRadius;
}

inline void Bat::SetPosition(const Vector2D& pos)
{
	_texturePos = pos;
}

inline void Bat::Draw(const float deltaTime)
{
	_texturePos.x += _speed.x;
	_texturePos.y += _speed.y;

	if (_texturePos.x >= _flyRadius || _texturePos.x <= 0)
	{
		_speed.x *= -1.0f;
		_facing *= -1.0f;
	}
	if (_texturePos.y >= _flyRadius || _texturePos.y <= 0) _speed.y *= -1.0f;

	Animate(_texturePos, _texture, deltaTime, 2.0f, 6.0f);
}

class Chicken : public BaseAnimation
{
public:
	~Chicken()
	{
		UnloadTexture(_texture);
	}

	void Draw(const float deltaTime);

private:
	const float _speed{ 1.0f };
	float _facing{ -1.0f };
	bool _animate{ 0 };
	Texture2D _texture{ LoadTexture("textures/animals/chicken_walk.png") };
	Vector2D _texturePos{};
};

inline void Chicken::Draw(const float deltaTime)
{
	int x = 1, y = 0;

	if (_texturePos.x == 320.0f)
	{
		x = 0;
		y = 1;
	}
	if (_texturePos.y == 300.0f)
	{
		x = 2;
		y = 0;
	}
	if (_texturePos.x < 0.0f)
	{
		x = 1;
		y = 2;
	}

	switch (x)
	{
	case 1:
		_texturePos.x += _speed;
		_facing = -1.0f;
		break;
	case 2:
		_texturePos.x -= _speed;
		_facing = 1.0f;
		break;
	default:
		break;
	}

	switch (y)
	{
	case 1:
		_texturePos.y += _speed;
		break;
	case 2:
		_texturePos.y -= _speed;
		break;
	default:
		break;
	}

	if (_texturePos.y != 0.0f) _animate = 1;

	Animate(_texturePos, _texture, deltaTime, 1.2f, 7.0f, _facing, 0.0f, _animate);
}

class Crocodile : public BaseAnimation
{
public:
	~Crocodile()
	{
		UnloadTexture(_texture);
		UnloadTexture(_textureWalk);
		UnloadTexture(_textureHurt);
		UnloadSound(_gettingPunched);
	}

	Vector2D GetPosition()
	{
		return _texturePos;
	}

	void Walk()
	{
		_row = 12.0f;
		_texture = _textureWalk;
		_isWalk = 1;
		_rotate = 0.0f;
	}

	void Hurt()
	{
		_row = 1.0f;
		_texture = _textureHurt;
		_isWalk = 0;

		_rotate = (_speed < 0.0f) ? 10.0f : 355.0f;

		_timer += GetFrameTime();
		if (_timer >= _updateTime)
		{
			_timer = -0.32f;
			PlaySound(_gettingPunched);
		}
	}

	Rectangle GetCollision()
	{
		return Rectangle{
			_texturePos.x, _texturePos.y,
			1.2f * (float)_texture.width / _row,
			1.2f * (float)_texture.height
		};
	}

	void Draw(const float deltaTime)
	{
		if (_isWalk)
		{
			_texturePos.x += _speed;

			if (_texturePos.x >= 1400.0f || _texturePos.x <= 0)
			{
				_speed *= -1.0f;
				_facing *= -1.0f;
			}
		}

		Animate(_texturePos, _texture, deltaTime, 2.0f, _row, _facing, 0.0f, 1, _rotate);
	}

private:
	float _speed{ 1.0f };
	float _row{};
	float _timer{};
	float _facing{ 1.0f };
	float _updateTime{ 0.0834f };
	float _rotate{ 0.0f };
	bool _isWalk{};
	Vector2D _texturePos{};
	Texture2D _texture{ LoadTexture("textures/animals/crocodile_walk.png") };
	Texture2D _textureWalk{ LoadTexture("textures/animals/crocodile_walk.png") };
	Texture2D _textureHurt{ LoadTexture("textures/animals/crocodile_hurt.png") };
	Sound _gettingPunched{ LoadSound("sounds/getting_punched.wav") };
};

class Animals
{
public:
	Animals()
	{
		for (auto& rhino : rhinos)
		{
			Vector2D rhinoPos{ (float)GetRandomValue(2600, 3500), (float)GetRandomValue(10, 450) };

			rhino.SetPosition(rhinoPos);
		}

		for (auto& bat : _bats)
		{
			Vector2D batPos{ (float)GetRandomValue(500, 950), (float)GetRandomValue(50, 155) };

			bat.SetPosition(batPos);

			bat.SetFlyRadius((float)GetRandomValue(1000, 1500));
		}
	}

public:
	Crocodile crocodile;
	Chicken chicken;

	std::array<Rhino, 3> rhinos{};

	void Draw(const float deltaTime);

private:
	std::array<Bat, 16> _bats{};
};

inline void Animals::Draw(const float deltaTime)
{
	for (auto& rhino : rhinos) rhino.Draw(deltaTime);

	for (auto& bat : _bats) bat.Draw(deltaTime);

	crocodile.Draw(deltaTime);

	chicken.Draw(deltaTime);
}
{% endhighlight %}

### GameObject.h

{% highlight cpp %}
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

#pragma once

#include "Map.h"
#include "Player.h"
#include "Prop.cpp"
#include "Animal.cpp"

class GameObject
{
public:
    Map map{};
    Prop prop{};
    Player player{};
    Animals animals{};
    void PlayWalkSound();
    void CheckCollision();
    void Draw(float deltaTime);

private:
    inline bool OnTouch(const Player& player, float targetPosX)
    {
        if (player.GetFacing() == 1.0f && player.GetPosition().x < targetPosX) return 1;
        else if (player.GetFacing() == -1.0f && player.GetPosition().x > targetPosX) return 1;
        
        return 0;
    }
};
{% endhighlight %}

### GameObject.cpp

{% highlight cpp %}
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

#include "GameObject.h"

static const Rectangle invisibleFences[4]
{
    Rectangle{ 980.414f, 280.586f, 150.0f, 560.0f },
    Rectangle{ 980.414f + 280.0f, 280.586f + 180.0f, 150.0f, 450.0f },
    Rectangle{ 40.0f, 252.0f, 250.0f, 0.2f },
    Rectangle{ 40.0f, 900.0f, 350.0f, 0.2f }
};

void GameObject::PlayWalkSound()
{
    if (player.GetPosition().x < -15.0f || player.GetPosition().y < 0.0f ||
        player.GetPosition().x > ((float)map.GetDreamlandSize().width - 15.0f) * map.GetMapScale() &&
        player.GetPosition().x < (float)map.GetDreamlandSize().width + 1500.0f ||
        player.GetPosition().y > (float)map.GetDreamlandSize().height * map.GetMapScale())
    {
        player.OnWater();
    }
    else
    {
        player.OnLand();
    }
}

void GameObject::CheckCollision()
{
    if (CheckCollisionRecs(player.GetCollision(), animals.crocodile.GetCollision()) &&
        player.IsPunch() && OnTouch(player, animals.crocodile.GetPosition().x))
    {
        animals.crocodile.Hurt();
    }
    else
    {
        animals.crocodile.Walk();
    }

    for (auto& rhino : animals.rhinos) if (CheckCollisionRecs(player.GetCollision(), rhino.GetCollision())) player.Stop();

    if (CheckCollisionRecs(player.GetCollision(), map.GetMapLine1()) ||
        CheckCollisionRecs(player.GetCollision(), map.GetMapLine2())) player.Stop();

    for (const auto& invisibleFence : invisibleFences)
    {
        if (CheckCollisionRecs(player.GetCollision(),
            Rectangle{ invisibleFence.x, invisibleFence.y, invisibleFence.width, invisibleFence.height })) player.Stop();
    }

    if (CheckCollisionRecs(player.GetCollision(), prop.naturalObj.GetBigStone1Coll())) player.SetPosition({ 3000.0f, 300.0f });

    if (CheckCollisionRecs(player.GetCollision(), prop.naturalObj.GetBigStone2Coll())) player.SetPosition({ 1600.0f - 80.0f, 700.0f });
}

void GameObject::Draw(float deltaTime)
{
    map.Draw();
    player.Draw();
    prop.Draw();
    animals.Draw(deltaTime);
}
{% endhighlight %}

### Map.h

{% highlight cpp %}
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

#pragma once

#include "Math2D.h"

struct MapSize { int width; int height; };

class Map
{
public:
	~Map()
	{
		UnloadTexture(_dreamland);
		UnloadTexture(_desert);
	}

	inline MapSize	 GetDreamlandSize() const { return { _dreamland.width, _dreamland.height }; /* 768 x 768 */ }
	inline MapSize	 GetDesertSize()    const { return { _desert.width, _desert.height }; /* 768 x 768 */ }
	inline float	 GetMapScale()	    const { return { _mapScale }; }
	inline Vector2D	 GetDreamlandPos()  const { return { _dreamlandPos }; }
	inline Vector2D	 GetDesertPos()	    const { return { _desertPos }; }
	inline Rectangle GetMapLine1()	    const { return { _desertPos.x - 480.0f, 0.0f, 100.0f, (float)_dreamland.height * _mapScale }; }
	inline Rectangle GetMapLine2()      const { return { (float)_dreamland.width * _mapScale * 2.5f, 0.0f, 100.0f, (float)_desert.height * _mapScale }; }

	void Draw();

private:
	const float _mapScale{ 2.0f };
	Texture2D _dreamland{ LoadTexture("textures/maps/dreamland.png") };
	Texture2D _desert{ LoadTexture("textures/maps/desert.png") };
	Vector2D _dreamlandPos{};
	Vector2D _desertPos{ _dreamland.width * _mapScale + 760.0f, 0.0f };
};
{% endhighlight %}

### Map.cpp

{% highlight cpp %}
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

#include "Map.h"

void Map::Draw()
{
	DrawTextureEx(_dreamland, _dreamlandPos, 0.0f, _mapScale, WHITE);
	DrawTextureEx(_desert, _desertPos, 0.0f, _mapScale, WHITE);
}
{% endhighlight %}

### Math2D.h

{% highlight cpp %}
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
{% endhighlight %}

### NaturalObject.cpp

{% highlight cpp %}
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

#include "Math2D.h"

class NaturalObject
{
public:
	~NaturalObject()
	{
		UnloadTexture(_bigStone);
	}

	Rectangle GetBigStone1Coll()
	{
		return Rectangle{
			static_cast<float>(_bigStone1Pos.x + 45.0f),
			static_cast<float>(_bigStone1Pos.y + 35.0f),
			static_cast<float>(_bigStone.width * 0.65f),
			static_cast<float>(_bigStone.height * 0.65f)
		};
	}

	Rectangle GetBigStone2Coll()
	{
		return Rectangle{
			static_cast<float>(_bigStone2Pos.x + 45.0f),
			static_cast<float>(_bigStone2Pos.y + 35.0f),
			static_cast<float>(_bigStone.width * 0.65f),
			static_cast<float>(_bigStone.height * 0.65f)
		};
	}

	void Draw()
	{
		DrawTextureV(_bigStone, _bigStone1Pos, WHITE);
		DrawTextureV(_bigStone, _bigStone2Pos, WHITE);
	}

private:
	Texture2D _bigStone{ LoadTexture("textures/natural_objects/big_stone.png") };
	Vector2D _bigStone1Pos{ 1600.0f, 700.0f };
	Vector2D _bigStone2Pos{ 3400.0f, 600.0f };
};

struct Prop
{
	NaturalObject naturalObj;

	void Draw()
	{
		naturalObj.Draw();
	}
};
{% endhighlight %}

### Player.cpp

{% highlight cpp %}
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

#pragma once

#include <memory>
#include <vector>
#include "Animation.h"

class Player : public BaseAnimation
{
public:
	Player();
	~Player();
	Vector2D GetPosition() const;
	Vector2D GetDirection() const;
	Rectangle GetCollision() const;
	bool IsPunch() const;
	bool IsInvisible() const;
	float GetFacing() const;
	float GetStamina() const;
	float GetSpeed() const;
	void SetPosition(const Vector2D pos);
	void SetStamina(bool isDragonInside);
	void Stop();
	void OnLand();
	void OnWater();
	void Draw();

private:
	bool _isWalk{ 0 }, _isDragonInside{ 0 };
	float _timer{}, _facing{ 1.0f }, _stamina{ 6.0f };
	float Row() const, Timer() const;
	void UpdateTexture();
	int LoadTextureFile(const char* texture);
	const float _updateTime{ 0.084f };
	Vector2D _texturePos{}, _textureLastPos{};
	const Sound _landStep{ LoadSound("sounds/land_step.wav") };
	const Sound _waterStep{ LoadSound("sounds/water_step.wav") };
	std::unique_ptr<std::vector<Texture2D>> _textures{ new std::vector<Texture2D> };
};
{% endhighlight %}

### Player.cpp

{% highlight cpp %}
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

#include "Player.h"

// ---------------- Public Functions ------------------------------------------

Player::Player() : _texturePos{ 40.0f, 140.0f }
{
	_textures->push_back(Texture2D{});

	LoadTextureFile("textures/character/friendly_man_idle.png");
	LoadTextureFile("textures/character/friendly_man_punch.png");
	LoadTextureFile("textures/character/friendly_man_walk.png");
}

Player::~Player()
{
	if (!_textures->empty())
	{
		for (const auto& texture : *_textures)
		{
			UnloadTexture(texture);
		}

		_textures->clear();
	}

	UnloadSound(_landStep);
	UnloadSound(_waterStep);
}

Vector2D Player::GetPosition() const
{
	return _texturePos;
}

Vector2D Player::GetDirection() const
{
	Vector2D direction{};

	if (IsKeyDown(KEY_D) || IsKeyDown(KEY_RIGHT)) direction.x -= 1.0f;
	if (IsKeyDown(KEY_A) || IsKeyDown(KEY_LEFT)) direction.x += 1.0f;
	if (IsKeyDown(KEY_S) || IsKeyDown(KEY_DOWN)) direction.y -= 1.0f;
	if (IsKeyDown(KEY_W) || IsKeyDown(KEY_UP)) direction.y += 1.0f;

	return direction;
}

Rectangle Player::GetCollision() const
{
	const Texture2D& texture = _textures->at(0);

	return Rectangle{
		_texturePos.x, _texturePos.y,
		1.5f * (float)texture.width / Row(),
		1.5f * (float)texture.height
	};
}

float Player::GetFacing() const
{
	return _facing;
}

float Player::GetStamina() const
{
	if (_isDragonInside) return 9.0f;
	else return _stamina;
}

float Player::GetSpeed() const
{
	if (IsKeyDown(KEY_SPACE) && GetDirection().Length() != 0 && _stamina > 0) return 4.0f; // walk fast
	else return 2.0f; // walk slow
}

bool Player::IsPunch() const
{
	if (IsKeyDown(KEY_E) && !_isWalk) return 1;
	else return 0;
}

bool Player::IsInvisible() const
{
	if (IsKeyDown(KEY_LEFT_SHIFT)) return 1;
	else return 0;
}

void Player::SetPosition(const Vector2D pos)
{
	_texturePos = pos;
}

void Player::SetStamina(bool isDragonInside)
{
	_isDragonInside = isDragonInside;
}

void Player::Stop()
{
	_texturePos = _textureLastPos;
}

void Player::OnLand()
{
	if (GetSpeed() == 2.0f) _timer += GetFrameTime() * 0.23f;
	else _timer += GetFrameTime() * 0.33f;

	if (_isWalk && _timer >= _updateTime)
	{
		_timer = 0.0f;
		PlaySound(_landStep);
	}
}

void Player::OnWater()
{
	if (GetSpeed() == 2.0f) _timer += GetFrameTime() * 0.11f;
	else _timer += GetFrameTime() * 0.13f;

	if (_isWalk && _timer >= _updateTime)
	{
		_timer = 0.0f;
		PlaySound(_waterStep);
	}
}

void Player::Draw()
{
	UpdateTexture();

	if (!IsKeyDown(KEY_LEFT_SHIFT))
	{
		Animate(_texturePos, _textures->at(0), GetFrameTime(), 1.5f, Row(), _facing, Timer());
	}
}

// ---------------- Private Functions ------------------------------------------

float Player::Row() const
{
	if (IsPunch() && !_isWalk) return 3.0f;
	else if (_isWalk) return 6.0f;

	return 2.0f;
}

float Player::Timer() const
{
	if (_isWalk) return 0.0f;
	else if (IsPunch() && !_isWalk) return 0.02f;

	return -0.2f;
}

void Player::UpdateTexture()
{
	if (!_isDragonInside)
	{
		if (IsKeyDown(KEY_SPACE) && GetDirection().Length() != 0 && _stamina > 0.0f)
		{
			_stamina -= GetFrameTime();
		}
		else if (!IsKeyDown(KEY_SPACE) && GetDirection().Length() == 0)
		{
			if (_stamina < 6.0f && _stamina > -0.1f)
			{
				_stamina += GetFrameTime();
			}
		}
	}

	_textureLastPos = _texturePos;

	if (GetDirection().Length() != 0)
	{
		_isWalk = 1;

		_texturePos = _texturePos
			.Subtract(GetDirection()
				.Normalize()
				.Scale(GetSpeed()));

		_textures->at(0) = _textures->at(3);

		if (GetDirection().x < 0.0f) _facing = 1.0f;
		if (GetDirection().x > 0.0f) _facing = -1.0f;
	}
	else
	{
		_isWalk = 0;

		_textures->at(0) = _textures->at(1);
	}

	if (IsPunch()) _textures->at(0) = _textures->at(2);
}

int Player::LoadTextureFile(const char* texture)
{
	_textures->push_back(LoadTexture(texture));
	return int(_textures->size() - 1);
}
{% endhighlight %}

### Screen.h

{% highlight cpp %}
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

#pragma once

#include <array>
#include "Math2D.h"

class Screen
{
public:
	virtual void Draw() = 0;
};

void DrawHelixNebula();
void DrawEarth();
void DrawGalaxy();
void DrawDusk();

void SetActiveScreen(Screen* screen);
void DrawScreen();

void InitScreenTexture();
void UnloadScreenTexture();

void DrawCenteredText(int y, const char* text, int fontSize, Color color);
{% endhighlight %}

### Screen.cpp

{% highlight cpp %}
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

#include "Screen.h"

Screen* activeScreen = nullptr;

static const char* strTextureFile[4]
{
	"textures/screens_bg/helix_nebula.png",
	"textures/screens_bg/earth.png",
	"textures/screens_bg/galaxy.png",
	"textures/screens_bg/dusk.png"
};

std::array<Texture2D, 4> screenTextures{};

void InitScreenTexture()
{
	for (auto i = 0; i < 4; i++)
	{
		screenTextures.at(i) = LoadTexture(strTextureFile[i]);
	}
}

void DrawHelixNebula()
{
	DrawTextureV(screenTextures.at(0), {}, WHITE);
}

void DrawEarth()
{
	DrawTextureV(screenTextures.at(1), {}, WHITE);
}

void DrawGalaxy()
{
	DrawTextureV(screenTextures.at(2), {}, WHITE);
}

void DrawDusk()
{
	DrawTextureV(screenTextures.at(3), {}, WHITE);
}

void UnloadScreenTexture()
{
	if (!screenTextures.empty())
	{
		for (const auto& texture : screenTextures)
		{
			UnloadTexture(texture);
		}
	}
}

void SetActiveScreen(Screen* screen)
{
	activeScreen = screen;
}

void DrawScreen()
{
	if (activeScreen != nullptr)
	{
		activeScreen->Draw();
	}
}

void DrawCenteredText(int y, const char* text, int fontSize, Color color)
{
	int textWidth = MeasureText(text, fontSize);
	DrawText(text, GetScreenWidth() / 2 - textWidth / 2, y - fontSize / 2, fontSize, color);
}
{% endhighlight %}