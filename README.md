# YUV2RGB(C++、OpenGL GLSL)
主要包括 `YUV420P to RGB`,`YUV422P to RGB`,`YUV444P to RGB`.

YUV与RGB互转，在不同色彩空间上（ BT601，BT709，BT2020，RP177）的转换公式不同，可以参考*[博客](https://www.cnblogs.com/luoyinjie/p/7219319.html)*。

本文主要给出代码示例思路。


# YUV转RGB实现（C++ OpenGL GLSL）

**原理**：也叫做YCbCr，其中“Y”表示明亮度（Luminance）,“U”和“V”分别表示色度（Chrominance）和浓度（Chroma）。

![Aaron Swartz](https://github.com/CSsaan/YUV2RGB-with-OpenGL/raw/main/IMG/yuv.jpg)
<p style="text-align: center;">数据格式</p>


![Aaron Swartz](https://github.com/CSsaan/YUV2RGB-with-OpenGL/raw/main/IMG/YUV2RGB.png)
<p style="text-align: center;">转换公式</p>


# 1.YUV通道分离
## 1.	YUV420P：
YUV420P转换:
````C++
void split_yuv420(char *inputPath, int width, int height) {
    FILE *fp_yuv = fopen(inputPath, "rb+");
    FILE *fp_y = fopen("output_420_y.y", "wb+");
    FILE *fp_u = fopen("output_420_u.y", "wb+");
    FILE *fp_v = fopen("output_420_v.y", "wb+");
    unsigned char *data = (unsigned char *) malloc(width * height * 3 / 2);
    fread(data, 1, width * height * 3 / 2, fp_yuv);
    //Y
    fwrite(data, 1, width * height, fp_y);
    //U
    fwrite(data + width * height, 1, width * height / 4, fp_u);
    //V
    fwrite(data + width * height * 5 / 4, 1, width * height / 4, fp_v);
    //释放资源
    free(data);
    fclose(fp_yuv);
    fclose(fp_y);
    fclose(fp_u);
    fclose(fp_v);
}
````

OpenGL加载YUV420P的Y、U、V纹理：
````GLSL
unsigned char *y_plane = yuv,
    *u_plane = yuv + img_w * img_h,
    *v_plane = yuv + (img_w * img_h) + (img_w * img_h) / 4;
````


## 2.	YUV422P：
YUV422P转换：
````C++
void split_yuv422(char *inputPath, int width, int height) {
    FILE *fp_yuv = fopen(inputPath, "rb+");
    FILE *fp_y = fopen("output_422_y.y", "wb+");
    FILE *fp_u = fopen("output_422_u.y", "wb+");
    FILE *fp_v = fopen("output_422_v.y", "wb+");
    unsigned char *data = (unsigned char *) malloc(width * height * 2);
    fread(data, 1, width * height * 2, fp_yuv);
    //Y
    fwrite(data, 1, width * height, fp_y);
    //U
    fwrite(data + width * height, 1, width * height / 2, fp_u);
    //V
    fwrite(data + width * height * 3 / 2, 1, width * height / 2, fp_v);
    //释放资源
    free(data);
    fclose(fp_yuv);
    fclose(fp_y);
    fclose(fp_u);
    fclose(fp_v);
}
````

OpenGL加载YUV422P的Y、U、V纹理：
````GLSL
unsigned char *y_plane = yuv,
    *u_plane = yuv + img_w * img_h,
    *v_plane = yuv + (img_w * img_h) + (img_w * img_h) / 2;
````


## 3.	YUV444P：
YUV444P转换：
````C++
void split_yuv444(char *inputPath, int width, int height) {
    FILE *fp_yuv = fopen(inputPath, "rb+");
    FILE *fp_y = fopen("output_444_y.y", "wb+");
    FILE *fp_u = fopen("output_444_u.y", "wb+");
    FILE *fp_v = fopen("output_444_v.y", "wb+");
    unsigned char *data = (unsigned char *) malloc(width * height * 3);
    fread(data, 1, width * height * 3, fp_yuv);
    //Y
    fwrite(data, 1, width * height, fp_y);
    //U
    fwrite(data + width * height, 1, width * height, fp_u);
    //V
    fwrite(data + width * height * 2, 1, width * height, fp_v);
    //释放资源
    free(data);
    fclose(fp_yuv);
    fclose(fp_y);
    fclose(fp_u);
    fclose(fp_v);
}
````

OpenGL加载YUV422P的Y、U、V纹理：
````GLSL
unsigned char *y_plane = yuv,
    *u_plane = yuv + img_w * img_h,
    *v_plane = yuv + (img_w * img_h) + (img_w * img_h);
````

# 2.OpenGL将YUV纹理转RGB

在GLSL中计算如下，构建mat3矩阵时按照列构建：
````GLSL
vec4 YUV2RGB(vec2 Coord)
{
    vec3 rgb = mat3(1.0, 1.0, 1.0,
                    0, -0.337, 1.732,
                    1.370, -0.698, 0.0)
                * vec3(texture(y_samp, Coord).r,
                        texture(u_samp, Coord).r - 0.5,
                        texture(v_samp, Coord).r - 0.5);
    return vec4(rgb, 1.0);
````



