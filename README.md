# 线性滤波

## 一、任务描述

**对于图像（lena.jpg），使用GDAL编程，分别使用下面的核进行卷积，并观察效果，说明结果是哪一种滤波。**

*lena.jpg*

![lena](/img4/lena.jpg)

![卷积核](/img4/卷积核1.jpg)

![卷积核](/img4/卷积核2.jpg)

## 二、规则要求

（1）滤波器的大小应该是奇数，这样它才有一个中心，例如3X3、5X5，或者7X7。

（2）滤波器所有元素之和应该是1，这是为了保持滤波前后图像的亮度保持不变。

（3）如果滤波矩阵所有元素之和大于 1，那么滤波后的图像就会比原图像亮。反之，如果小于1，那么得到的图像就会变暗。

（4）对于滤波后的结构，可能会出现负数或者大于255的数值，对于这种情况，将它们直接截断至0和255之间即可。

## 三、实验内容

***1、卷积核一：均值模糊***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;
```

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";
	
	//输出图像路径
	char* dstPath = (char*)"result1.jpg";
	//图像内存存储
	GByte* buffTmp;
	
	//图像波段数
	int m, i, j, bandNum;
	
	//二维矩阵
	static double R[256][256];
	static double R1[256][256];
	
	//卷积核
	double a[3][3] = { {0,1,0}, {1,1,1}, {0,1,0} };


	//注册驱动
	GDALAllRegister();
	
	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
	
	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();
	
	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));
	
	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);
	
	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == imgXlen - 1) || (j == 0) || (j == imgYlen - 1))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 1][j - 1] * a[0][0] + R[i - 1][j] * a[0][1] + R[i - 1][j + 1] * a[0][2]
						+ R[i][j - 1] * a[1][0] + R[i][j] * a[1][1] + R[i][j + 1] * a[1][2]
						+ R[i + 1][j - 1] * a[2][0] + R[i + 1][j] * a[2][1] + R[i + 1][j + 1] * a[2][2];
				}
	
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R1[i][j] = R1[i][j] * 0.2;//乘以0.2
				//截断至0和255之间
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);
	
	system("PAUSE");
	return 0;
	}
**#结果如下：均值模糊**

![result1](/img4/result1.jpg)

***2、卷积核二：运动模糊***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";

	//输出图像路径
	char* dstPath = (char*)"result2.jpg";
	//图像内存存储
	GByte* buffTmp;

	//图像波段数
	int m, i, j, bandNum;


	//二维矩阵
	static double R[256][256];
	static double R1[256][256];

	//卷积核
	double a[5][5] = { {1,0,0,0,0}, {0,1,0,0,0}, {0,0,1,0,0}, {0,0,0,1,0}, {0,0,0,0,1} };
	
	//注册驱动
	GDALAllRegister();

	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);

	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();

	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));

	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);

	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == 1) || (i == imgXlen - 1) || (i == imgXlen - 2) || (j == 0) || (j == 1) || (j == imgYlen - 1) ||(j == imgYlen - 2))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 2][j - 2] * a[0][0] + R[i - 2][j - 1] * a[0][1] + R[i - 2][j] * a[0][2] + R[i - 2][j + 1] * a[0][3] + R[i - 2][j + 2] * a[0][4]
						+ R[i - 1][j - 2] * a[1][0] + R[i - 1][j - 1] * a[1][1] + R[i - 1][j] * a[1][2] + R[i - 1][j + 1] * a[1][3] + R[i - 1][j + 2] * a[1][4]
						+ R[i][j - 2] * a[2][0] + R[i][j - 1] * a[2][1] + R[i][j] * a[2][2] + R[i][j + 1] * a[2][3] + R[i][j + 2] * a[2][4]
						+ R[i + 1][j - 2] * a[3][0] + R[i + 1][j - 1] * a[3][1] + R[i + 1][j] * a[3][2] + R[i + 1][j + 1] * a[3][3] + R[i + 1][j + 2] * a[3][4]
						+R[i + 2][j - 2] * a[4][0] + R[i + 2][j - 1] * a[4][1] + R[i + 2][j] * a[4][2] + R[i + 2][j + 1] * a[4][3] + R[i + 2][j + 2] * a[4][4];
				}
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R1[i][j] = R1[i][j] * 0.2;//乘以0.2
				//截断至0和255之间
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);

	system("PAUSE");
	return 0;
}
```

**#结果如下：运动模糊**

![result2](/img4/result2.jpg)

***3、卷积核三：边缘检测***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";

	//输出图像路径
	char* dstPath = (char*)"result3.jpg";
	//图像内存存储
	GByte* buffTmp;

	//图像波段数
	int m, i, j, bandNum;


	//二维矩阵
	static double R[256][256];
	static double R1[256][256];

	//卷积核
	double a[3][3] = { {-1,-1,-1},{-1,8,-1},{-1,-1,-1} };

	//注册驱动
	GDALAllRegister();

	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);

	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();

	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));

	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);

	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == imgXlen - 1) || (j == 0) || (j == imgYlen - 1))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 1][j - 1] * a[0][0] + R[i - 1][j] * a[0][1] + R[i - 1][j + 1] * a[0][2]
						+ R[i][j - 1] * a[1][0] +R[i][j] * a[1][1] + R[i][j + 1] * a[1][2]
						+ R[i + 1][j - 1] * a[2][0] + R[i + 1][j] * a[2][1] + R[i + 1][j + 1] * a[2][2];
					
				}
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);

	system("PAUSE");
	return 0;
}
```

**#结果如下：边缘检测**

![result3](/img4/result3.jpg)

***4、卷积核四：图像锐化***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";

	//输出图像路径
	char* dstPath = (char*)"result4.jpg";
	//图像内存存储
	GByte* buffTmp;

	//图像波段数
	int m, i, j, bandNum;


	//二维矩阵
	static double R[256][256];
	static double R1[256][256];

	//卷积核
	double a[3][3] = { {-1,-1,-1},{-1,9,-1},{-1,-1,-1} };

	//注册驱动
	GDALAllRegister();

	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);

	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();

	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));

	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);

	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == imgXlen - 1) || (j == 0) || (j == imgYlen - 1))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 1][j - 1] * a[0][0] + R[i - 1][j] * a[0][1] + R[i - 1][j + 1] * a[0][2]
						+ R[i][j - 1] * a[1][0] + R[i][j] * a[1][1] + R[i][j + 1] * a[1][2]
						+ R[i + 1][j - 1] * a[2][0] + R[i + 1][j] * a[2][1] + R[i + 1][j + 1] * a[2][2];

				}
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);

	system("PAUSE");
	return 0;
}
```

**#结果如下：图像锐化**

![result4](/img4/result4.jpg)

***5、卷积核五：浮雕***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";

	//输出图像路径
	char* dstPath = (char*)"result5.jpg";
	//图像内存存储
	GByte* buffTmp;

	//图像波段数
	int m, i, j, bandNum;


	//二维矩阵
	static double R[256][256];
	static double R1[256][256];

	//卷积核
	double a[3][3] = { {-1,-1,0},{-1,0,1},{0,1,1} };

	//注册驱动
	GDALAllRegister();

	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);

	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();

	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));

	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);

	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == imgXlen - 1) || (j == 0) || (j == imgYlen - 1))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 1][j - 1] * a[0][0] + R[i - 1][j] * a[0][1] + R[i - 1][j + 1] * a[0][2]
						+ R[i][j - 1] * a[1][0] + R[i][j] * a[1][1] + R[i][j + 1] * a[1][2]
						+ R[i + 1][j - 1] * a[2][0] + R[i + 1][j] * a[2][1] + R[i + 1][j + 1] * a[2][2];

				}
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R1[i][j] += 128;//加上128的偏移
				//截断至0和255之间
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);

	system("PAUSE");
	return 0;
}
```

**#结果如下：浮雕**

![result5](/img4/result5.jpg)

***5、卷积核六：高斯模糊***

**#代码如下：**

```
#include "pch.h"
#include <iostream>
using namespace std;
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")

int main()
{
	//输入图像
	GDALDataset* poSrcDS;

	//输出图像
	GDALDataset* poDstDS;
	//图像的宽度和高度
	int imgXlen, imgYlen;
	//输入图像路径
	char* srcPath = (char*)"lena.jpg";

	//输出图像路径
	char* dstPath = (char*)"result6.jpg";
	//图像内存存储
	GByte* buffTmp;

	//图像波段数
	int m, i, j, bandNum;


	//二维矩阵
	static double R[256][256];
	static double R1[256][256];

	//卷积核
	double a[5][5] = { {0.0120,0.1253,0.2736,0.1253,0.0120},{0.1253,1.3054,2.8514,1.3054,0.1253},{0.2736,2.8514,6.2279,2.8514,0.2736},{0.1253,1.3054,2.8514,1.3054,0.1253},{0.0120,0.1253,0.2736,0.1253,0.0120} };
	
	//注册驱动
	GDALAllRegister();

	//打开图像
	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);

	//获取图像宽度，高度，和波段数量
	imgXlen = poSrcDS->GetRasterXSize();
	imgYlen = poSrcDS->GetRasterYSize();
	bandNum = poSrcDS->GetRasterCount();

	//根据图像的宽度和高度分配内存
	buffTmp = (GByte*)CPLMalloc(imgXlen * imgYlen * sizeof(GByte));

	//创建输出图像
	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);

	for (m = 0; m < bandNum; m++)
	{
		poSrcDS->GetRasterBand(m + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R[i][j] = buffTmp[j * imgXlen + i];
			}
		}
		for (i = 0; i < imgXlen; i++)
		{
			for (j = 0; j < imgYlen; j++)
			{
				if ((i == 0) || (i == 1) || (i == imgXlen - 1) || (i == imgXlen - 2) || (j == 0) || (j == 1) || (j == imgYlen - 1) || (j == imgYlen - 2))
				{
					R1[i][j] = 0;
				}
				else
				{
					R1[i][j] = R[i - 2][j - 2] * a[0][0] + R[i - 2][j - 1] * a[0][1] + R[i - 2][j] * a[0][2] + R[i - 2][j + 1] * a[0][3] + R[i - 2][j + 2] * a[0][4]
						+ R[i - 1][j - 2] * a[1][0] + R[i - 1][j - 1] * a[1][1] + R[i - 1][j] * a[1][2] + R[i - 1][j + 1] * a[1][3] + R[i - 1][j + 2] * a[1][4]
						+ R[i][j - 2] * a[2][0] + R[i][j - 1] * a[2][1] + R[i][j] * a[2][2] + R[i][j + 1] * a[2][3] + R[i][j + 2] * a[2][4]
						+ R[i + 1][j - 2] * a[3][0] + R[i + 1][j - 1] * a[3][1] + R[i + 1][j] * a[3][2] + R[i + 1][j + 1] * a[3][3] + R[i + 1][j + 2] * a[3][4]
						+ R[i + 2][j - 2] * a[4][0] + R[i + 2][j - 1] * a[4][1] + R[i + 2][j] * a[4][2] + R[i + 2][j + 1] * a[4][3] + R[i + 2][j + 2] * a[4][4];
				}
			}
		}
		for (j = 0; j < imgYlen; j++)
		{
			for (i = 0; i < imgXlen; i++)
			{
				R1[i][j] = R1[i][j] / 25 ;//除25
				//截断至0和255之间
				if (R1[i][j] < 0)
				{
					R1[i][j] = 0;
				}
				if (R1[i][j] > 255)
				{
					R1[i][j] = 255;
				}
				buffTmp[j * imgXlen + i] = R1[i][j];
			}
		}
		poDstDS->GetRasterBand(m + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	}


	//清除内存
	CPLFree(buffTmp);
	//关闭dataset
	GDALClose(poDstDS);
	GDALClose(poSrcDS);

	system("PAUSE");
	return 0;
}
```

**#结果如下：高斯模糊**

![result6](/img4/result6.jpg)

## 四、实验总结

1、对于图像的每一个像素点，计算它的邻域像素和滤波器矩阵的对应元素的乘积，然后加起来，作为该像素位置的值。本次实验我采用的是最简单暴力的方法来实现卷积运算。

2、浮雕滤波器可以给图像一种3D阴影的效果。要对结果图像加上128的偏移。

3、与实验三相似，需要对图像的每个通道分别进行滤波，写入图像亦然。

4、滤波之后，需要将像素值截断至0至255之间。

5、忽略边界像素，直接置0。