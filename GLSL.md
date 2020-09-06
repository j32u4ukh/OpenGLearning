## 頂點著色器

每個輸入變數也叫頂點屬性(Vertex Attribute)，GLSL 以 layout(location = 0) 的方式來宣告變數位置，關鍵字 in 
說明它是屬於輸入變數。但輸出變數其實也不需要定義位置，只需給予關鍵字 out 即可，所以說 
layout(location = 0) 是宣告輸入變數的一部分也沒錯。

我們能聲明的頂點屬性數量是有上限的，它一般由硬體來決定。OpenGL 確保至少有 16 個包含 4 分量(vec4)的頂點屬性可用，
但是有些硬件或許允許更多的頂點屬性，可以查詢 GL_MAX_VERTEX_ATTRIBS 來獲取具體的上限。
通常情況下它至少會返回 16 個，大部分情況下是夠用了。

```cpp
int nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes << std::endl;
```

## 數據類型

和其他程式語言一樣，GLSL 有數據類型可以來指定變量的種類。GLSL 中包含 C 等其它語言大部分的默認基礎數據類型：
int、float、double、uint 和 bool。GLSL 也有兩種容器類型，分別是向量(Vector)和矩陣(Matrix)。

### 向量

GLSL 中的向量是一個可以包含有 1、2、3或者 4 個分量的容器，分量的類型可以是前面默認基礎類型的任意一個。
它們可以是下面的形式（n 代表分量的數量）：

* vecn: 包含 n 個 float 分量的默認向量
* bvecn: 包含 n 個 bool 分量的向量
* ivecn: 包含 n 個 int 分量的向量
* uvecn: 包含 n 個 unsigned int 分量的向量
* dvecn: 包含 n 個 double 分量的向量

大多數時候我們使用 vecn，因為 float 足夠滿足大多數要求了。

一個向量的分量可以通過 vec.x 這種方式獲取，這里 x 是指這個向量的第一個分量。
你可以分別使用 .x、.y、.z 和 .w 來獲取它們的第 1、2、3、4 個分量。
GLSL 也允許你對顏色使用 rgba，或是對紋理坐標使用 stpq 訪問相同的分量。

向量這一數據類型也允許一些有趣而靈活的分量選擇方式，叫做重組(Swizzling)。重組允許這樣的語法：

```cpp
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

你可以使用上面 4 個字母任意組合來創建一個和原來向量一樣長的（同類型）新向量，只要原來向量有那些分量即可；
然而，在一個 vec2 向量中去獲取.z元素是不被允許的。

我們也可以把一個向量作為一個參數傳給不同的向量構造函數，以減少需求參數的數量：

```cpp
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

## 輸入與輸出

雖然著色器是各自獨立的小程序，但是它們都是一個整體的一部分，
出於這樣的原因，我們希望每個著色器都有輸入和輸出，這樣才能進行數據交流和傳遞。
GLSL 定義了 in 和 out 關鍵字專門來實現這個目的。

每個著色器使用這兩個關鍵字設定輸入和輸出，只要一個輸出變量與下一個著色器階段的輸入匹配，它就會傳遞下去。
但在頂點和片段著色器中會有點不同。

頂點著色器應該接收的是一種特殊形式的輸入，否則就會效率低下。
頂點著色器的輸入特殊在，它從頂點數據中直接接收輸入。
為了定義頂點數據該如何管理，我們使用 location 這一元數據指定輸入變量，這樣我們才可以在 CPU 上配置頂點屬性。

layout (location = 0)。頂點著色器需要為它的輸入提供一個額外的 layout 標識，這樣我們才能把它鏈接到頂點數據。

你也可以忽略 layout (location = 0) 標識符，通過在OpenGL代碼中使用 glGetAttribLocation 查詢屬性位置值(Location)，
但在著色器中設置它們更容易理解而且節省你（和 OpenGL）的工作量。

另一個例外是片段著色器，它需要一個 vec4 顏色輸出變量，因為片段著色器需要生成一個最終輸出的顏色。
如果你在片段著色器沒有定義輸出顏色，OpenGL 會把你的物體渲染為黑色（或白色）。

所以，如果我們打算從一個著色器向另一個著色器發送數據，我們必須在發送方著色器中聲明一個輸出，在接收方著色器中聲明一個類似的輸入。
當類型和名字都一樣的時候，OpenGL 就會把兩個變量鏈接到一起，它們之間就能發送數據了（這是在鏈接程序對象時完成的）。

```GLSL
// 頂點著色器

#version 330 core
layout (location = 0) in vec3 pos; // 位置變量的屬性位置值為 0

out vec4 vertex_color; // 為片段著色器指定一個顏色輸出

void main()
{
    gl_Position = vec4(pos, 1.0); // 注意我們如何把一個 vec3 作為 vec4 的構造器的參數
    vertex_color = vec4(0.5, 0.0, 0.0, 1.0); // 把輸出變量設置為暗紅色
}

// 片段著色器

#version 330 core
out vec4 frag_color;

in vec4 vertex_color; // 從頂點著色器傳來的輸入變量（名稱相同、類型相同）

void main()
{
    frag_color = vertex_color;
}
```

你可以看到我們在頂點著色器中聲明了一個 vertex_color 變量作為 vec4 輸出，並在片段著色器中聲明了一個類型相同的 vertex_color。
由於它們名字相同且類型相同，片段著色器中的 vertex_color 就和頂點著色器中的 vertex_color 鏈接了。

## Uniform

Uniform 是一種從 CPU 中的應用向 GPU 中的著色器發送數據的方式，但 uniform 和頂點屬性有些不同。

首先，uniform 是全局的(Global)。全局意味著 uniform 變量必須在每個著色器程序對象中都是獨一無二的，
而且它可以被著色器程序的任意著色器在任意階段訪問。

第二，無論你把 uniform 值設置成什麽，uniform 會一直保存它們的數據，直到它們被重置或更新。

我們可以在一個著色器中添加 uniform 關鍵字至類型和變量名前來聲明一個 GLSL 的 uniform。
從此處開始我們就可以在著色器中使用新聲明的 uniform 了。

```GLSL
#version 330 core
out vec4 frag_color;

uniform vec4 our_color; // 在 OpenGL 程式碼中設定這個變量

void main()
{
    frag_color = our_color;
}
```

我們在片段著色器中聲明了一個 uniform vec4 的 our_color，並把片段著色器的輸出顏色設置為 uniform 值的內容。
因為 uniform 是全局變量，我們可以在任何著色器中定義它們，而無需通過頂點著色器作為中介。

頂點著色器中不需要這個 uniform，所以我們不用在那里定義它。

如果你聲明了一個 uniform 卻在 GLSL 代碼中沒用過，編譯器會靜默移除這個變量，導致最後編譯出的版本中並不會包含它，
這可能導致幾個非常麻煩的錯誤，記住這點！

這個 uniform 現在還是空的；我們還沒有給它添加任何數據，所以下面我們就做這件事。
我們首先需要找到著色器中 uniform 屬性的索引/位置值。當我們得到 uniform 的索引/位置值後，我們就可以更新它的值了。

這次我們不去給像素傳遞單獨一個顏色，而是讓它隨著時間改變顏色：

```cpp
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shader_program, "our_color");
glUseProgram(shader_program);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

首先我們通過 glfwGetTime() 獲取運行的秒數。然後我們使用 sin 函數讓顏色在 0.0 到 1.0 之間改變，最後將結果儲存到 greenValue 里。

接著，我們用 glGetUniformLocation 查詢 uniform our_color 的位置值。

我們為查詢函數提供著色器程序和 uniform 的名字（這是我們希望獲得的位置值的來源）。

如果 glGetUniformLocation 返回 -1 就代表沒有找到這個位置值。

最後，我們可以通過 glUniform4f 函數設置uniform值。注意，查詢 uniform 地址不要求你之前使用過著色器程序，
但是更新一個 uniform 之前你必須先使用程序（調用 glUseProgram)，因為它是在當前激活的著色器程序中設置 uniform的。

因為OpenGL在其核心是一個C庫，所以它不支持類型重載，在函數參數不同的時候就要為其定義新的函數；glUniform 是一個典型例子。
這個函數有一個特定的後綴，標識設定的 uniform 的類型。可能的後綴有：

後綴: 含義
f: 函數需要一個 float 作為它的值
i: 函數需要一個 int 作為它的值
ui: 函數需要一個 unsigned int 作為它的值
3f:	函數需要 3 個 float 作為它的值
fv:	函數需要一個 float 向量/數組作為它的值

每當你打算配置一個 OpenGL 的選項時就可以簡單地根據這些規則選擇適合你的數據類型的重載函數。
在我們的例子里，我們希望分別設定 uniform 的 4 個 float 值，所以我們通過 glUniform4f 傳遞我們的數據(注意，我們也可以使用 fv 版本)。

現在你知道如何設置 uniform 變量的值了，我們可以使用它們來渲染了。
如果我們打算讓顏色慢慢變化，我們就要在遊戲循環的每一次迭代中（所以他會逐幀改變）更新這個 uniform，否則三角形就不會改變顏色。

下面我們就計算 greenValue 然後每個渲染迭代都更新這個 uniform：

```cpp
while(!glfwWindowShouldClose(window))
{
    // 輸入
    processInput(window);

    // 渲染
    // 清除顏色緩沖
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    // 記得激活著色器
    glUseProgram(shader_program);

    // 更新 uniform 顏色
    float timeValue = glfwGetTime();
    float greenValue = sin(timeValue) / 2.0f + 0.5f;
    int vertexColorLocation = glGetUniformLocation(shader_program, "our_color");
    glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

    // 繪制三角形
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);

    // 交換緩沖並查詢 IO 事件
    glfwSwapBuffers(window);
    glfwPollEvents();
}
```

## 更多屬性！

除了位置數據，這次還要將把顏色數據添加進 vertices 數組中。
顏色數據為 3 個 float 值，我們將把三角形的三個角分別指定為紅色、綠色和藍色：

```cpp
float vertices[] = {
    // 位置              // 顏色
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 頂部
};
```

由於現在有更多的數據要發送到頂點著色器，我們有必要去調整一下頂點著色器，使它能夠接收顏色值作為一個頂點屬性輸入。

需要注意的是我們用 layout 標識符來把 color 屬性的位置值設置為 1：

```GLSL
#version 330 core
layout (location = 0) in vec3 pos;   // 位置變量的屬性位置值為 0 
layout (location = 1) in vec3 color; // 顏色變量的屬性位置值為 1

out vec3 our_color; // 向片段著色器輸出一個顏色

void main()
{
    gl_Position = vec4(pos, 1.0);
    our_color = color; // 將ourColor設置為我們從頂點數據那里得到的輸入顏色
}
```

由於我們不再使用 uniform 來傳遞片段的顏色了，現在使用 our_color 輸出變量，我們必須再修改一下片段著色器：

```GLSL
#version 330 core
out vec4 frag_color;  
in vec3 our_color;

void main()
{
    frag_color = vec4(our_color, 1.0);
}
```

因為我們添加了另一個頂點屬性，並且更新了 VBO 的內存，我們就必須重新配置頂點屬性指針。更新後的 VBO 內存中的數據現在看起來像這樣：

![新 VBO 配置](repo/image/vertex_attribute_pointer_interleaved.png)

知道了現在使用的布局，我們就可以使用 glVertexAttribPointer 函數更新頂點格式，

```cpp
// 位置屬性
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

// 顏色屬性
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
glEnableVertexAttribArray(1);
```

由於我們現在有了兩個頂點屬性，我們不得不重新計算步長值(第 5 個參數)。

為獲得數據隊列中下一個屬性值（比如位置向量的下個 x 分量）我們必須向右移動 6 個 float，其中 3 個是位置值，另外 3 個是顏色值。
這使我們的步長值為 6 乘以 float 的字節數（= 24 字節）。

同樣，這次我們必須指定一個偏移量。對於每個頂點來說，位置頂點屬性在前，所以它的偏移量是 0。
顏色屬性緊隨位置數據之後，所以偏移量就是 3 * sizeof(float)，用字節來計算就是 12 字節。


雖然我們只提供了 3 個顏色，但繪製出的三角形卻會是從各個角向另外兩個角形成漸層的顏色。
這是在片段著色器中進行的所謂片段插值(Fragment Interpolation)的結果。

當渲染一個三角形時，光柵化(Rasterization)階段通常會造成比原指定頂點更多的片段。
光柵會根據每個片段在三角形形狀上所處相對位置決定這些片段的位置。
基於這些位置，它會插值(Interpolate)所有片段著色器的輸入變量。

比如說，我們有一個線段，上面的端點是綠色的，下面的端點是藍色的。
如果一個片段著色器在線段的 70% 的位置運行，它的顏色輸入屬性就會是一個綠色和藍色的線性結合；更精確地說就是 30% 藍 + 70% 綠。

這正是在這個三角形中发生了什麽。我們有3個頂點，和相應的3個顏色，從這個三角形的像素來看它可能包含50000左右的片段，
片段著色器為這些像素進行插值顏色。如果你仔細看這些顏色就應該能明白了：紅首先變成到紫再變為藍色。
片段插值會被應用到片段著色器的所有輸入屬性上。

## 我們自己的著色器類

編寫、編譯、管理著色器是件麻煩事。
在著色器主題的最後，我們會寫一個類來讓我們的生活輕松一點，它可以從硬盤讀取著色器，然後編譯並鏈接它們，
並對它們進行錯誤檢測，這就變得很好用了。這也會讓你了解該如何封裝目前所學的知識到一個抽象對象中。

我們會把著色器類全部放在 header 文件里，主要是為了學習用途，當然也方便移植。我們先來添加必要的 include，並定義類結構：

```cpp
#ifndef SHADER_H
#define SHADER_H

#include <glad/glad.h>; // 包含glad來獲取所有的必須OpenGL頭文件

#include <string>
#include <fstream>
#include <sstream>
#include <iostream>


class Shader
{
public:
    // 程序ID
    unsigned int ID;

    // 構造器讀取並構建著色器
    Shader(const GLchar* vertexPath, const GLchar* fragmentPath);
    // 使用/激活程序
    void use();
    // uniform工具函數
    void setBool(const std::string &name, bool value) const;  
    void setInt(const std::string &name, int value) const;   
    void setFloat(const std::string &name, float value) const;
};

#endif
```

```
在上面，我們在 header 文件頂部使用了幾個預處理指令(Preprocessor Directives)。
這些預處理指令會告知你的編譯器只在它沒被包含過的情況下才包含和編譯這個頭文件，即使多個文件都包含了這個著色器頭文件。
它是用來防止鏈接沖突的。
```

著色器類儲存了著色器程序的ID。
它的構造器需要頂點和片段著色器源代碼的文件路徑，這樣我們就可以把原始碼的文字文件儲存在硬碟上了。

除此之外，為了讓我們的生活更輕松一點，還加入了一些工具函數：use 用來激活著色器程序，
所有的 set… 函數能夠查詢一個 unform 的位置值並設置它的值。

### 從文件讀取

我們使用 C++ 文件流讀取著色器內容，儲存到幾個 string 對象里：

```cpp
Shader(const char* vertexPath, const char* fragmentPath)
{
    // 1. 從文件路徑中獲取頂點/片段著色器
    std::string vertexCode;
    std::string fragmentCode;
    std::ifstream vShaderFile;
    std::ifstream fShaderFile;

    // 保證ifstream對象可以拋出異常：
    vShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);
    fShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);

    try 
    {
        // 打開文件
        vShaderFile.open(vertexPath);
        fShaderFile.open(fragmentPath);
        std::stringstream vShaderStream, fShaderStream;

        // 讀取文件的緩沖內容到數據流中
        vShaderStream << vShaderFile.rdbuf();
        fShaderStream << fShaderFile.rdbuf(); 
      
        // 關閉文件處理器
        vShaderFile.close();
        fShaderFile.close();

        // 轉換數據流到string
        vertexCode   = vShaderStream.str();
        fragmentCode = fShaderStream.str();     
    }
    catch(std::ifstream::failure e)
    {
        std::cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << std::endl;
    }
    const char* vShaderCode = vertexCode.c_str();
    const char* fShaderCode = fragmentCode.c_str();

    [...]
```

下一步，我們需要編譯和鏈接著色器。
注意，我們也將檢查編譯/鏈接是否失敗，如果失敗則打印編譯時錯誤，調試的時候這些錯誤輸出會及其重要（你總會需要這些錯誤日誌的）：

```cpp
// 2. 編譯著色器
unsigned int vertex, fragment;
int success;
char infoLog[512];

// 頂點著色器
vertex = glCreateShader(GL_VERTEX_SHADER);
glShaderSource(vertex, 1, &vShaderCode, NULL);
glCompileShader(vertex);

// 打印編譯錯誤（如果有的話）
glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
if(!success)
{
    glGetShaderInfoLog(vertex, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
};

// 片段著色器也類似
[...]

// 著色器程序
ID = glCreateProgram();
glAttachShader(ID, vertex);
glAttachShader(ID, fragment);
glLinkProgram(ID);

// 打印連接錯誤（如果有的話）
glGetProgramiv(ID, GL_LINK_STATUS, &success);
if(!success)
{
    glGetProgramInfoLog(ID, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
}

// 刪除著色器，它們已經鏈接到我們的程序中了，已經不再需要了
glDeleteShader(vertex);
glDeleteShader(fragment);
```

use 函數非常簡單：

```cpp
void use() 
{ 
    glUseProgram(ID);
}
```

uniform 的 setter 函數也很類似：

```cpp
void setBool(const std::string &name, bool value) const
{
    glUniform1i(glGetUniformLocation(ID, name.c_str()), (int)value); 
}

void setInt(const std::string &name, int value) const
{ 
    glUniform1i(glGetUniformLocation(ID, name.c_str()), value); 
}

void setFloat(const std::string &name, float value) const
{ 
    glUniform1f(glGetUniformLocation(ID, name.c_str()), value); 
} 
```

現在我們就寫完了一個完整的著色器類。使用這個著色器類很簡單；只要創建一個著色器對象，從那一點開始我們就可以開始使用了：

```cpp
Shader ourShader("path/to/shaders/shader.vs", "path/to/shaders/shader.fs");
...
while(...)
{
    ourShader.use();
    ourShader.setFloat("someUniform", 1.0f);
    DrawStuff();
}
```

