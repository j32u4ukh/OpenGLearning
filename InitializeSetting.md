## 實例化 GLFW 窗口

```cpp
/* 初始化套件，若錯誤則直接返回 */
if (!glfwInit()) {
    return -1;
}

// 設置 Major version
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);

// 設置 Minor version
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);

// 使用核心模式(Core-profile)
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
```

調用 glfwInit 函數來初始化 GLFW，然後我們可以使用 glfwWindowHint 函數來配置 GLFW。
glfwWindowHint 函數的第一個參數代表選項的名稱，我們可以從很多以 GLFW_ 開頭的枚舉值中選擇；第二個參數接受一個整型，用來設置這個選項的值。

我們將主版本號(Major)和次版本號(Minor)都設為 3。
我們同樣明確告訴 GLFW 我們使用的是核心模式(Core-profile)。
明確告訴 GLFW 我們需要使用核心模式意味著我們只能使用 OpenGL 功能的一個子集（我們已不再需要的向後兼容特性）。

### 創建一個窗口物件

這個窗口對象存放了所有和窗口相關的數據，而且會被 GLFW 的其他函數頻繁地用到。

```cpp
/* glfwCreateWindow: 建立 GLFW 視窗物件
GLFWwindow* glfwCreateWindow(
    int             width,  視窗寬度
    int             height, 視窗高度
    const char*  	title,  視窗標題
    GLFWmonitor*    monitor,
    GLFWwindow*  	share
)

最後兩個參數我們暫時忽略。
*/
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);

// 利用 window 在當前的執行續當中建立 Context
glfwMakeContextCurrent(window);
```

[context](https://www.khronos.org/opengl/wiki/OpenGL_Context): 儲存各種狀態機，管理著 OpenGL 的所有東西。

一個 process 可以創建多個 OpenGL context，每個 context 相當於一個可被觀看的視窗。

每個 context 管理著多個 OpenGL 物件，基本上 context 們的物件之間彼此獨立，但有些可共享，有些不行。 

### GLAD

GLAD 是用來管理 OpenGL 的函數指針的，所以在調用任何 OpenGL 的函數之前我們需要初始化 GLAD。

glfwCreateWindow 等函式為 glfw 開頭，不是 OpenGL 的函數，因此前句沒有問題。
GLFW 是一個專門針對 OpenGL的 C 語言庫，它提供了一些渲染物體所需的最低限度的接口。
它允許用戶創建 OpenGL 上下文，定義窗口參數以及處理用戶輸入。

我們給 GLAD 傳入了用來加載系統相關的 OpenGL 函數指針地址的函式。
GLFW給我們的是 glfwGetProcAddress，它根據我們編譯的系統定義了正確的函數。

```cpp
// glfwGetProcAddress: 返回用來加載系統相關的 OpenGL 函數指針地址的函式
// gladLoadGLLoader: 利用上述函式，初始化 GLAD
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
    return -1;
}
```

### Viewport 視野

我們必須告訴 OpenGL 渲染視窗的尺寸大小，即視野(Viewport)，這樣 OpenGL 才只能知道怎樣根據窗口大小顯示數據和坐標。
我們可以通過調用 glViewport 函數來設置窗口的維度(Dimension)：

```cpp
int width = 800;
int height = 600;
glViewport(0, 0, width, height);
```

glViewport 函數前兩個參數控制視窗左下角的位置。第三個和第四個參數控制渲染窗口的寬度和高度（像素）。

我們實際上也可以將視口的維度設置為比 GLFW 的維度小，這樣子之後所有的 OpenGL 渲染將會在一個更小的 Viewport 中顯示，這樣子的話我們也可以將一些其它元素顯示在 OpenGL 的 Viewport之外。
例如，UI 控制面板等。

OpenGL 幕後使用 glViewport 中定義的位置和寬高進行 2D 坐標的轉換，將 OpenGL 中的位置坐標轉換為你的屏幕坐標。

例如，OpenGL 中的坐標 (-0.5, 0.5) 有可能（最終）被映射為屏幕中的坐標 (200,450)。
注意，處理過的 OpenGL 坐標範圍只為 -1 到 1，因此我們事實上將 (-1 到 1)範圍內的坐標映射到 (0, 800) 和 (0, 600)。

然而，當用戶改變窗口的大小的時候，視口也應該被調整。
我們可以對窗口注冊一個回調函數(Callback Function)，它會在每次窗口大小被調整的時候被調用。
這個回調函數如下：

```cpp
void framebufferSizeCallback(GLFWwindow* window, int width, int height)
{
    /*
    void glViewport(
        GLint   x,
        GLint   y,
        GLsizei width,
        GLsizei height
    );

    x, y
        Specify the lower left corner of the viewport rectangle, in pixels.
        The initial value is (0,0).

    width, height
        Specify the width and height of the viewport. When a GL context is first attached to
        a window, width and height are set to the dimensions of that window.
    */

    // 根據傳入的長寬，調整視窗大小
    glViewport(0, 0, width, height);
}
```

注冊這個函數，告訴 GLFW 我們希望每當窗口調整大小的時候調用這個函數：

```cpp
glfwSetFramebufferSizeCallback(window, framebufferSizeCallback);
```

### 渲染循環(Render Loop)

我們希望程序在我們主動關閉它之前不斷繪制圖像並能夠接受用戶輸入。
因此，我們需要在程序中添加一個 while 循環，我們可以把它稱之為渲染循環(Render Loop)，它能在我們讓 GLFW 退出前一直保持運行。

```cpp
// 檢查 GLFW 是否被要求退出
while(!glfwWindowShouldClose(window))
{
    // 前後緩衝內容交換，為平滑畫面刷新，避免被發現畫面在更新，詳參考"雙緩沖(Double Buffer)"
    glfwSwapBuffers(window);

    // 雖然事件的回應"將定義在其他函式中"，但若無 glfwPollEvents，這些事件根本無法產生，視窗的移動與縮放也無法        
    glfwPollEvents();    
}
```

* glfwWindowShouldClose 函數在我們每次循環的開始前檢查一次 GLFW 是否被要求退出，如果是的話該函數返回 true 然後渲染循環便結束了，之後為我們就可以關閉應用程序了。

* glfwPollEvents 函數檢查有沒有觸发什麽事件（比如鍵盤輸入、鼠標移動等）、更新窗口狀態，並調用對應的回調函數（可以通過回調方法手動設置）。

* glfwSwapBuffers 函數會交換顏色緩沖（它是一個儲存著 GLFW 窗口每一個像素顏色值的大緩沖），它在這一叠代中被用來繪制，並且將會作為輸出顯示在屏幕上。


**雙緩沖(Double Buffer)**

應用程序使用單緩沖繪圖時可能會存在圖像閃爍的問題。 

這是因為生成的圖像不是一下子被繪制出來的，而是按照從左到右，由上而下逐像素地繪制而成的。最終圖像不是在瞬間顯示給用戶，而是通過一步一步生成的，這會導致渲染的結果很不真實。

為了規避這些問題，我們應用雙緩沖渲染窗口應用程序。前緩沖保存著最終輸出的圖像，它會在屏幕上顯示；而所有的的渲染指令都會在後緩沖上繪制。

當所有的渲染指令執行完畢後，我們利用 glfwSwapBuffers 交換(Swap)前緩沖和後緩沖，這樣圖像就立即呈顯出來，之前提到的不真實感就消除了。

### 正確釋放/刪除之前的分配的所有資源

調用 glfwTerminate 函數來完成。

```cpp
glfwTerminate();
```

### 輸入

我們同樣也希望能夠在 GLFW 中實現一些輸入控制，這可以通過使用 GLFW 的幾個輸入函數來完成。
我們將會使用 GLFW 的 glfwGetKey 函數，它需要一個窗口以及一個按鍵作為輸入。
這個函數將會返回這個按鍵是否正在被按下。

我們將創建一個 processInput 函數來讓所有的輸入代碼保持整潔，此為我們自己定義的函式。
在渲染循環(Render Loop)的每一個迭代中調用 processInput，來檢測特定的鍵是否被按下，並在每一幀做出處理。

```cpp
void processInput(GLFWwindow *window)
{
    // 檢查是否按下 ESC，如果沒有按下，glfwGetKey 將會返回 GLFW_RELEASE
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS){
        // 設置 glfwWindowShouldClose(window) 為 true，前面提到的渲染循環(Render Loop)則會結束
        glfwSetWindowShouldClose(window, true);
    } 
}

while (!glfwWindowShouldClose(window))
{
    processInput(window);
    ...
}
```