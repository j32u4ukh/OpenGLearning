我們要把所有的渲染(Rendering)操作放到渲染循環中，因為我們想讓這些渲染指令在每次渲染循環叠代的時候都能被執行。

我們使用一個自定義的顏色清空屏幕。在每個新的渲染迭代開始的時候我們總是希望刷新畫面，否則我們仍能看見上一次叠代的渲染結果。

* glClear: 清除指示的緩衝區，它接受一個緩沖位(Buffer Bit)來指定要清空的緩沖，可能的緩沖位有 GL_COLOR_BUFFER_BIT，GL_DEPTH_BUFFER_BIT 和 GL_STENCIL_BUFFER_BIT。
* GL_COLOR_BUFFER_BIT: 指示當前為彩色寫入啟用的緩衝區。Indicates the buffers currently enabled for color writing. 
* GL_DEPTH_BUFFER_BIT: 指示深度緩衝區。Indicates the depth buffer. 
* GL_STENCIL_BUFFER_BIT: 指示模板緩衝區。Indicates the stencil buffer. 

由於現在我們只關心顏色值，所以我們只清空顏色緩沖。

```cpp
// 渲染循環
while(!glfwWindowShouldClose(window))
{
    // 輸入
    processInput(window);

    // 定義 glClear 用來覆寫畫面的顏色
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);

    // 覆寫畫面顏色的緩衝
    glClear(GL_COLOR_BUFFER_BIT);

    // 渲染指令
    ...

    // 檢查並調用事件，交換緩沖
    glfwPollEvents();
    glfwSwapBuffers(window);
}
```

**glClearColor 函數是一個`狀態設置函數`，而 glClear 函數則是一個`狀態使用的函數`，它使用了當前的狀態來獲取應該清除為的顏色。**



