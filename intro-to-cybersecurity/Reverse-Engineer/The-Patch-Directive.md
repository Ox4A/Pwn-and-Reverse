非常麻烦的一道题，首先看cimg.c，看到有两个handle用来读data：

```c
while (cimg.header.remaining_directives--)
    {
        uint16_t directive_code;
        read_exact(0, &directive_code, sizeof(directive_code), "ERROR: Failed to read &directive_code!", -1);

        switch (directive_code)
        {
        case 34358:
            handle_34358(&cimg);
            break;
        case 43058:
            handle_43058(&cimg);
            break;
        default:
            fprintf(stderr, "ERROR: invalid directive_code %ux\n", directive_code);
            exit(-1);
        }
    }
```
但是这两个handle的头文件题目不给，那只能拖到IDA里反编译，得到伪代码如下：
handle_43058:

```c
unsigned __int64 __fastcall handle_43058(__int64 a1)
{
  unsigned int v1; // ebx
  unsigned __int8 *v2; // rax
  unsigned __int8 *v3; // rbp
  __int64 v4; // rax
  __int64 v5; // rcx
  int i; // r13d
  int v7; // r14d
  int v8; // eax
  int v9; // ecx
  unsigned int v10; // ebx
  __int64 v11; // rdx
  unsigned __int8 v13; // [rsp+Bh] [rbp-5Dh] BYREF
  unsigned __int8 v14; // [rsp+Ch] [rbp-5Ch] BYREF
  unsigned __int8 v15; // [rsp+Dh] [rbp-5Bh] BYREF
  unsigned __int8 v16; // [rsp+Eh] [rbp-5Ah] BYREF
  __int128 v17; // [rsp+Fh] [rbp-59h] BYREF
  __int64 v18; // [rsp+1Fh] [rbp-49h]
  unsigned __int64 v19; // [rsp+28h] [rbp-40h]

  v19 = __readfsqword(0x28u);
  read_exact(0LL, &v15, 1LL, "ERROR: Failed to read &base_x!", 0xFFFFFFFFLL);
  read_exact(0LL, &v16, 1LL, "ERROR: Failed to read &base_y!", 0xFFFFFFFFLL);
  read_exact(0LL, &v13, 1LL, "ERROR: Failed to read &width!", 0xFFFFFFFFLL);
  read_exact(0LL, &v14, 1LL, "ERROR: Failed to read &height!", 0xFFFFFFFFLL);
  v1 = 4 * v14 * v13;  
  v2 = (unsigned __int8 *)malloc(4LL * v14 * v13);  
  if ( !v2 )
  {
    puts("ERROR: Failed to allocate memory for the image data!");
    goto LABEL_7;
  }
  v3 = v2;
  read_exact(0LL, v2, v1, "ERROR: Failed to read data!", 0xFFFFFFFFLL);
  v4 = 0LL;
  while ( v14 * v13 > (int)v4 )   //检查字符值的循环
  {
    v5 = v3[4 * v4++ + 3]; //v5：检查字符值的临时变量
    if ( (unsigned __int8)(v5 - 32) > 0x5Eu )
    {
      __fprintf_chk(stderr, 1LL, "ERROR: Invalid character 0x%x in the image data!\n", v5);
LABEL_7:
      exit(-1);
    }
  }
  
/*
变量对照表：
v15=base_x
v16=base_y
v13=width=1
v14=height=1
宽，高都=1
*/

/*
.--------------------------.
|                          |
|            ***           |              60*17=1020
|            ***           |
|                          |
|                          |
|                          |
|                          |
.--------------------------.
*/


  for (int i = 0; i<v14; ++i )  //i是列索引
  {
    v7 = 0;   //v7是行索引
    while ( v7<v13 )
    {
      v8 = v7 + v15;  //行索引+base_x   v8：在画布中的第几行
      v9 = v7 + i * v13;  //行索引+列索引*width  v9：
      ++v7;
      v10 = v8 % *(unsigned __int8 *)(a1 + 6) + *(unsigned __int8 *)(a1 + 6) * (i + v16);

      __snprintf_chk(&v17,25LL,1LL,25LL,"\x1B[38;2;%03d;%03d;%03dm%c\x1B[0m",v3[4 * v9],v3[4 * v9 + 1],v3[4 * v9 + 2],v3[4 * v9 + 3]);

      v11 = *(_QWORD *)(a1 + 16) + 24LL * (v10 % *(_DWORD *)(a1 + 12));
      *(_OWORD *)v11 = v17;
      *(_QWORD *)(v11 + 16) = v18;
    }
  }
  return __readfsqword(0x28u) ^ v19;
}
```

以及handle_34358：

```c
unsigned __int64 __fastcall handle_34358(__int64 a1)
{
  int v1; // ebp
  int v2; // edx
  size_t v3; // rbp
  unsigned __int8 *v4; // rax
  unsigned __int8 *v5; // r12
  __int64 v6; // rax
  __int64 v7; // rcx
  int i; // r13d
  int v9; // ebp
  int v10; // r15d
  unsigned __int8 *v11; // rax
  __int64 v12; // kr00_8
  __int64 v13; // rdx
  __int128 v15; // [rsp+1Fh] [rbp-59h] BYREF
  __int64 v16; // [rsp+2Fh] [rbp-49h]
  unsigned __int64 v17; // [rsp+38h] [rbp-40h]

  v1 = *(unsigned __int8 *)(a1 + 6);
  v2 = *(unsigned __int8 *)(a1 + 7);
  v17 = __readfsqword(0x28u);
  v3 = 4LL * v2 * v1;
  v4 = (unsigned __int8 *)malloc(v3);
  if ( !v4 )
  {
    puts("ERROR: Failed to allocate memory for the image data!");
    goto LABEL_7;
  }
  v5 = v4;
  read_exact(0LL, v4, (unsigned int)v3, "ERROR: Failed to read data!", 0xFFFFFFFFLL);
  v6 = 0LL;
  while ( *(unsigned __int8 *)(a1 + 7) * *(unsigned __int8 *)(a1 + 6) > (int)v6 )
  {
    v7 = v5[4 * v6++ + 3];
    if ( (unsigned __int8)(v7 - 32) > 0x5Eu )
    {
      __fprintf_chk(stderr, 1LL, "ERROR: Invalid character 0x%x in the image data!\n", v7);
LABEL_7:
      exit(-1);
    }
  }
  for ( i = 0; *(unsigned __int8 *)(a1 + 7) > i; ++i )
  {
    v9 = 0;
    while ( 1 )
    {
      v10 = *(unsigned __int8 *)(a1 + 6);
      if ( v10 <= v9 )
        break;
      v11 = &v5[4 * i * v10 + 4 * v9];
      __snprintf_chk(&v15, 25LL, 1LL, 25LL, "\x1B[38;2;%03d;%03d;%03dm%c\x1B[0m", *v11, v11[1], v11[2], v11[3]);
      v12 = v9++;
      v13 = *(_QWORD *)(a1 + 16) + 24LL * (((unsigned int)(v12 % v10) + i * v10) % *(_DWORD *)(a1 + 12));
      *(_OWORD *)v13 = v15;
      *(_QWORD *)(v13 + 16) = v16;
    }
  }
  return __readfsqword(0x28u) ^ v17;
}
```

观察cimg.c里的total_data变量，说明程序对输入文件的大小有检查，不能超过1340字节，如果采用handle_34358函数的方法，构造一个完整的文件，大小必然超过；观察到cimg.c在初始化时候会对所有data的ascii字段置0，并且在第224行只对非空格和回车符进行ascii字段校验，如下所示：

```c
if (
            cimg.framebuffer[i].str.c != ' ' &&
            cimg.framebuffer[i].str.c != '\n' &&
            memcmp(cimg.framebuffer[i].data, ((term_pixel_t*)&desired_output)[i].data, sizeof(term_pixel_t))
        )
        {
            won = 0;
        }
```

因此可以构造只含有效字符（非空格和回车）的image.cimg，其余地方借助初始化的填充即可，但这要求知道每个像素点的坐标？

接着分析handle_43058，多了一个读取base_x和base_y的逻辑，恰好有了坐标，因此需要directiveCode需要=43058

但没完，即使这样优化以后还是会超过1340字节，注意到图像输出时有效字符存在连续的情况，因此可以将连续处的有效字符打包写入文件，最终生成文件的C++代码如下所示：
```c++
#include <stdlib.h>
#include <stdint.h>
#include <stdbool.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <time.h>
#include <errno.h>
#include <assert.h>
#include <libgen.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <sys/signal.h>
#include <sys/mman.h>
#include <sys/ioctl.h>
#include <sys/sendfile.h>
#include <sys/prctl.h>
#include <sys/personality.h>
#include <arpa/inet.h>
#include <fstream>
#include <string>
#include <iostream>
using namespace std;
#define Ceil 10
#define Bound 20
void __attribute__((constructor)) disable_buffering()
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 1);
}
char desired_output[] = "\x1b[38;2;049;196;198mc\x1b[0m\x1b[38;2;092;167;123mI\x1b[0m\x1b[38;2;089;007;016mM\x1b[0m\x1b[38;2;244;063;016mG\x1b[0m\x00";
struct header_context
{
    char magic_number[4];
    uint16_t version;
    uint8_t width;
    uint8_t height;
} __attribute__((packed));
typedef struct header_context CimgHead;

struct pixel_t
{
    uint8_t r;
    uint8_t g;
    uint8_t b;
    uint8_t ascii;
};
typedef struct pixel_t PixColor;
typedef struct cimg CimgData;
void Assign(PixColor *p, uint8_t r, uint8_t g, uint8_t b, uint8_t ascii)
{
    (*p).ascii = ascii;
    (*p).r = r;
    (*p).g = g;
    (*p).b = b;
}
PixColor colors[1020];
uint32_t realDirectives = 0;
void LoadColorInfo()
{
    std::ifstream file("data.txt");
    string line;
    int R, G, B;
    uint8_t r;
    uint8_t g;
    uint8_t b;
    uint8_t ascii;
    int idx = 0;
    while (getline(file, line))
    {
        sscanf(line.c_str(), "[38;2;%d;%d;%dm%c", &R, &G, &B, &ascii);
        r = R;
        g = G;
        b = B;
        if (ascii != ' ' && ascii != '\n')
        {
            realDirectives += 1;
        }
        // printf("%d;%d;%d;%d\n",r,g,b,ascii);
        Assign(colors + idx, r, g, b, ascii);
        idx += 1;
    }
    realDirectives -= (59 * 2 + 14 * 2);  //上下左右边界去掉
    realDirectives=42;
    file.close();
}
int main()
{
    int realOps=0;
    char magicNum[4];
    magicNum[0] = 'c';
    magicNum[1] = 'I';
    magicNum[2] = 'M';
    magicNum[3] = 'G';
    uint16_t version = 3;
    uint8_t width = 60;
    uint8_t height = 17;
    uint16_t directiveCode = 43058;
    LoadColorInfo();
    FILE *fp = fopen("image.cimg", "wb");
    fwrite(&magicNum, sizeof(magicNum), 1, fp);
    fwrite(&version, sizeof(version), 1, fp);
    fwrite(&width, sizeof(width), 1, fp);
    fwrite(&height, sizeof(height), 1, fp);
    fwrite(&realDirectives, sizeof(realDirectives), 1, fp);
    int x = 0;
    while (x < width * height)
    {
        if (colors[x].ascii != ' ' && colors[x].ascii != '\n')
        {
            switch (x)
            {
            case 0:
            { // 上边界
                fwrite(&directiveCode, sizeof(directiveCode), 1, fp);
                realOps+=1;
                uint8_t xbp, ybp;
                ybp = x / 60;
                xbp = x % 60;
                fwrite(&xbp, sizeof(xbp), 1, fp);
                fwrite(&ybp, sizeof(ybp), 1, fp);

                uint8_t tempWidth, tempHeight;
                tempWidth = 60;
                tempHeight = 1;
                fwrite(&tempWidth, sizeof(tempWidth), 1, fp);
                fwrite(&tempHeight, sizeof(tempHeight), 1, fp);
                for (int j = 0; j < 60; j++)
                {
                    fwrite(&colors[x + j], sizeof(colors[x + j]), 1, fp);
                }
                x += 60;
                break;
            }
            case 60:
            { // 左边界
                fwrite(&directiveCode, sizeof(directiveCode), 1, fp);
                realOps+=1;
                uint8_t xbp, ybp;
                ybp = x / 60;
                xbp = x % 60;
                fwrite(&xbp, sizeof(xbp), 1, fp);
                fwrite(&ybp, sizeof(ybp), 1, fp);
                uint8_t tempWidth, tempHeight;
                tempWidth = 1;
                tempHeight = 15;
                fwrite(&tempWidth, sizeof(tempWidth), 1, fp);
                fwrite(&tempHeight, sizeof(tempHeight), 1, fp);
                for (int k = 0; k <= 14; k++)
                {
                    fwrite(&colors[x + k * 60], sizeof(colors[x + k * 60]), 1, fp);
                }
                x += 1;
                break;
            }
            case 119:
            { // 右边界
                fwrite(&directiveCode, sizeof(directiveCode), 1, fp);
                realOps+=1;
                uint8_t xbp, ybp;
                ybp = x / 60;
                xbp = x % 60;
                fwrite(&xbp, sizeof(xbp), 1, fp);
                fwrite(&ybp, sizeof(ybp), 1, fp);
                uint8_t tempWidth, tempHeight;
                tempWidth = 1;
                tempHeight = 15;
                fwrite(&tempWidth, sizeof(tempWidth), 1, fp);
                fwrite(&tempHeight, sizeof(tempHeight), 1, fp);
                for (int k = 0; k <= 14; k++)
                {
                    fwrite(&colors[x + k * 60], sizeof(colors[x + k * 60]), 1, fp);
                }
                x += 1;
                break;
            }
            case 960:
            { // 下边界
                fwrite(&directiveCode, sizeof(directiveCode), 1, fp);
                realOps+=1;
                uint8_t xbp, ybp;
                ybp = x / 60;
                xbp = x % 60;
                fwrite(&xbp, sizeof(xbp), 1, fp);
                fwrite(&ybp, sizeof(ybp), 1, fp);
                uint8_t tempWidth, tempHeight;
                tempWidth = 60;
                tempHeight = 1;
                fwrite(&tempWidth, sizeof(tempWidth), 1, fp);
                fwrite(&tempHeight, sizeof(tempHeight), 1, fp);
                for (int j = 0; j < 60; j++)
                {
                    fwrite(&colors[x + j], sizeof(colors[x + j]), 1, fp);
                }
                x += 60;
                break;
            }

            default:
            {
                if (x % 60 != 0 && x % 60 != 59)
                {
                    fwrite(&directiveCode, sizeof(directiveCode), 1, fp);
                    realOps+=1;
                    uint8_t xbp, ybp;
                    ybp = x / 60;
                    xbp = x % 60;
                    fwrite(&xbp, sizeof(xbp), 1, fp);
                    fwrite(&ybp, sizeof(ybp), 1, fp);

                    //试探前方ascii内容
                    int frontInfo=x;
                    //保证不到边界
                    //bool internals=(frontInfo % 60 != 0 && frontInfo % 60 != 59);  
                    while(colors[frontInfo].ascii!=' '&&colors[frontInfo].ascii!='\n') {
                        frontInfo+=1;
                        //internals=(frontInfo % 60 != 0 && frontInfo % 60 != 59);
                    }
                    uint8_t tempWidth, tempHeight;
                    tempWidth = frontInfo-x;
                    tempHeight = 1;
                    fwrite(&tempWidth, sizeof(tempWidth), 1, fp);
                    fwrite(&tempHeight, sizeof(tempHeight), 1, fp);
                    for(int t=x;t<frontInfo;t++) {
                        fwrite(&colors[t], sizeof(colors[t]), 1, fp);
                    }
                    x=frontInfo;
                }
                else x += 1;
            }
            }
        }
        else x += 1;
    }
    fclose(fp);
    if(realDirectives==realOps) cout<<"指令码数量一致"<<endl;
    else cout<<"指令码数量不一致,realOps="<<realOps<<";realDirectives="<<realDirectives<<endl;
}
```
（这里有一个偷了懒的地方，懒得算具体的directiveCode数量是多少，直接先打印出来然后赋值
