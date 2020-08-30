* OpenGL: 圖形API的正式規範，用於定義每個函數的佈局和輸出。a formal specification of a graphics API that defines the layout and output of each function.
* GLAD: 一個擴展加載庫，可以為我們加載並設置所有OpenGL的函數指針，以便我們可以使用所有（現代）OpenGL的函數。an extension loading library that loads and sets all OpenGL's function pointers for us so we can use all (modern) OpenGL's functions.
* GLM: 專為OpenGL量身定制的數學庫。a mathematics library tailored for OpenGL.
* stb_image: 圖像加載庫。image loading library.

* Viewport: 我們渲染到的2D窗口區域。the 2D window region where we render to.
* Graphics Pipeline: 頂點在屏幕上顯示為一個或多個像素前必須遍歷的整個過程。the entire process vertices have to walk through before ending up as one or more pixels on the screen.
* Shader: 在圖形卡上運行的小程式。圖形流水線的多個階段可以使用用戶製作的著色器來替換現有功能。a small program that runs on the graphics card. Several stages of the graphics pipeline can use user-made shaders to replace existing functionality.
* Vertex: 代表單個點的數據集合。a collection of data that represent a single point.
* Normalized Device Coordinates(NDC): 在剪輯坐標上執行透視劃分後，頂點最終位於的坐標系。 NDC 中介於-1.0和1.0之間的所有頂點位置都不會被丟棄或修剪，並且最終可見。the coordinate system your vertices end up in after perspective division is performed on clip coordinates. All vertex positions in NDC between -1.0 and 1.0 will not be discarded or clipped and end up visible.
* Vertex Buffer Object: 一個緩衝區對象，該對像在GPU上分配內存並在其中存儲所有頂點數據供圖形卡使用。a buffer object that allocates memory on the GPU and stores all the vertex data there for the graphics card to use.
* Vertex Array Object: 存儲緩衝區和頂點屬性狀態信息。stores buffer and vertex attribute state information.
* Element Buffer Object: 一個在GPU上存儲索引以進行索引繪製的緩衝區對象。a buffer object that stores indices on the GPU for indexed drawing.
* Uniform: 全局的GLSL變量的一種特殊類型（著色器程序中的每個著色器都可以訪問該統一變量），並且只需要設置一次。a special type of GLSL variable that is global (each shader in a shader program can access this uniform variable) and only has to be set once.
* Texture: 著色器中使用的一種特殊類型的圖像，通常將其包裹在對象周圍，從而使對象的錯覺極其細緻。a special type of image used in shaders and usually wrapped around objects, giving the illusion an object is extremely detailed.
* Texture Wrapping: 定義一種模式，該模式指定當紋理坐標超出範圍（0，1）時OpenGL如何採樣紋理。defines the mode that specifies how OpenGL should sample textures when texture coordinates are outside the range: (0, 1).
* Texture Filtering: 定義模式，該模式指定當有多個紋理像素（紋理像素）可供選擇時OpenGL如何採樣紋理。通常在放大紋理時發生。defines the mode that specifies how OpenGL should sample the texture when there are several texels (texture pixels) to choose from. This usually occurs when a texture is magnified.
* Mipmaps: 存儲較小版本的紋理，其中根據距查看器的距離選擇適當大小的版本。stored smaller versions of a texture where the appropriate sized version is chosen based on the distance to the viewer.
* Texture Units: 通過綁定多個紋理（每個紋理到一個不同的紋理單元），在單個著色器程序上允許多個紋理。allows for multiple textures on a single shader program by binding multiple textures, each to a different texture unit.
* Vector: 在任何維度上定義方向和/或位置的數學實體。a mathematical entity that defines directions and/or positions in any dimension.
* Matrix: 具有有用轉換屬性的數學表達式的矩形數組。a rectangular array of mathematical expressions with useful transformation properties.
* Local Space: 對像開始的空間。相對於對象原點的所有坐標。the space an object begins in. All coordinates relative to an object's origin.
* World Space: 所有相對於全球原點的坐標。all coordinates relative to a global origin.
* View Space: 從相機的角度來看的所有坐標。all coordinates as viewed from a camera's perspective.
* Clip Space: 從相機的角度觀看但應用了投影的所有坐標。這是頂點坐標應作為頂點著色器輸出的最終空間。其餘部分由OpenGL執行（剪切/透視分割）。all coordinates as viewed from the camera's perspective but with projection applied. This is the space the vertex coordinates should end up in, as output of the vertex shader. OpenGL does the rest (clipping/perspective division).
* Screen Space: 從屏幕上查看的所有坐標。坐標範圍從0到屏幕的寬度/高度。all coordinates as viewed from the screen. Coordinates range from 0 to screen width/height.
* LookAt: 一種特殊類型的視圖矩陣，可創建一個坐標系統，在該坐標系統中，所有坐標都將以用戶從給定位置查看給定目標的方式旋轉和平移。a special type of view matrix that creates a coordinate system where all coordinates are rotated and translated in such a way that the user is looking at a given target from a given position.
* Euler Angles: 定義為偏航，俯仰和橫滾，使我們能夠從這3個值形成任何3D方向矢量。defined as yaw, pitch and roll that allow us to form any 3D direction vector from these 3 values.
