# 实验代码：
```c++

`// lesson4.cpp: 定义控制台应用程序的入口点。`
`//`

`#include "stdafx.h"`
`#include <iostream>`

`using namespace std;`
`#include "./gdal/gdal_priv.h"`
`#pragma comment(lib, "gdal_i.lib")`

`int main()`
`{`
	`GDALDataset* poSrcDS;`
	`GDALDataset* poDstDS;`
	`char* srcPath = "lena.jpg";`
	`char* dstPath = "resullt_1.tif";`
	`GByte* buffTmp, *buffTmp1;`
	`int imgXlen, imgYlen, bandNum;`
	`int i, j, k, m, n;`
	`double sum = 0.0;`



/*double matrix[5][5] = {1.0,0.0,0.0,0.0,0.0,
					   0.0,1.0,0.0,0.0,0.0,
					   0.0,0.0,1.0,0.0,0.0,
					   0.0,0.0,0.0,1.0,0.0,
					   0.0,0.0,0.0,0.0,1.0};*/
/*double matrix[5][5] = {0.0120,0.1253,0.2736,0.1253,0.0120,
                       0.1253,1.3054,2.8514,1.3054,0.1253,
                       0.2736,2.8514,6.2279,2.8514,0.2736,
                       0.1253,1.3054,2.8514,1.3054,0.1253,
                       0.0120,0.1253,0.2736,0.1253,0.0120};*/

GDALAllRegister();
poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
imgXlen = poSrcDS->GetRasterXSize();
imgYlen = poSrcDS->GetRasterYSize();
bandNum = poSrcDS->GetRasterCount();
buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
buffTmp1 = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen,bandNum, GDT_Byte, NULL);

for (i = 0; i < bandNum; i++)
{
	poSrcDS->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	for (j = 1; j < imgYlen; j++)
	{
		for (k = 1; k < imgXlen - 1; k++)
		{
			//卷积核1--均值模糊
			//sum = (buffTmp[(j - 1)*imgXlen + k] + buffTmp[j*imgXlen + k - 1] + buffTmp[j*imgXlen + k] + buffTmp[j*imgXlen + k + 1] + buffTmp[(j + 1)*imgXlen + k])*0.2;
			//卷积核3--边缘检测
			//sum = buffTmp[(j - 1)*imgXlen + (k - 1)] * -1 + buffTmp[(j - 1)*imgXlen + k] * -1 + buffTmp[(j - 1)*imgXlen + k + 1] * -1 + buffTmp[j*imgXlen + k - 1] * -1 + buffTmp[j*imgXlen + k] * 8 + buffTmp[j*imgXlen + k + 1] * -1 + buffTmp[(j + 1)*imgXlen + k - 1] * -1 + buffTmp[(j + 1)*imgXlen + k] * -1 + buffTmp[(j + 1)*imgXlen + k + 1] * -1;
			//卷积核4--图像锐化
			//sum = buffTmp[(j - 1)*imgXlen + (k - 1)] * -1 + buffTmp[(j - 1)*imgXlen + k] * -1 + buffTmp[(j - 1)*imgXlen + k + 1] * -1 + buffTmp[j*imgXlen + k - 1] * -1 + buffTmp[j*imgXlen + k] * 9 + buffTmp[j*imgXlen + k + 1] * -1 + buffTmp[(j + 1)*imgXlen + k - 1] * -1 + buffTmp[(j + 1)*imgXlen + k] * -1 + buffTmp[(j + 1)*imgXlen + k + 1] * -1;
			//卷积核5--浮雕效果
			sum = buffTmp[(j - 1)*imgXlen + (k - 1)] * -1 + buffTmp[(j - 1)*imgXlen + k] * -1 + buffTmp[(j - 1)*imgXlen + k + 1] * 0 + buffTmp[j*imgXlen + k - 1] * -1 + buffTmp[j*imgXlen + k] * 0 + buffTmp[j*imgXlen + k + 1] * 1 + buffTmp[(j + 1)*imgXlen + k - 1] *0 + buffTmp[(j + 1)*imgXlen + k] * 1 + buffTmp[(j + 1)*imgXlen + k + 1] * 1;
			sum += 128;
			//卷积核2--运动模糊
			/*for (m = j - 2; m <= j + 2; m++)
			{
				for (n = k - 2; n <= k + 2; n++)
				{
					sum += buffTmp[m*imgXlen + n] * matrix[m - j + 2][n - k + 2];
				}
			}
			sum = sum * 0.2;*/
			//卷积核6--高斯模糊
			/*for (m = j - 2; m <= j + 2; m++)
			{
				for (n = k - 2; n <= k + 2; n++)
				{
					sum += buffTmp[m*imgXlen + n] * matrix[m - j + 2][n - k + 2];
				}
			}
			sum = sum*(1.0/25.0);*/

			//对于负数或者大于255的值，直接截断
			if (sum < 0)
			{
				buffTmp1[j*imgXlen + k] = 0;
				
			}
			else if (sum > 255)
			{
				buffTmp1[j*imgXlen + k] = 255;
			}
			else
			{
				buffTmp1[j*imgXlen + k] = (GByte)sum;
			}



				

		}
	}
	//将边缘值置为0
	for (k = 0; k < imgXlen; k++)
	{
		buffTmp1[k] = (GByte)0;
		buffTmp1[(imgYlen - 1)*imgXlen + k] = (GByte)0;
	}
	for (k = 0; k < imgYlen; k++)
	{
		buffTmp[k*imgXlen] = (GByte)0;
		buffTmp1[k*imgXlen + imgXlen - 1] = (GByte)0;
	}
	//写入文件
	poDstDS->GetRasterBand(i + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp1, imgXlen, imgYlen, GDT_Byte, 0, 0);
}

CPLFree(buffTmp);
CPLFree(buffTmp1);
GDALClose(poDstDS);
GDALClose(poSrcDS);
system("PAUSE");

return 0;

}
```

# 实验截图：

1. 卷积核1--均值模糊

   ![](http://t1.aixinxi.net/o_1crbt7gvmc6a1cepkac1rbvp0ua.png-j.jpg)

2. 卷积核2--运动模糊

   ![](http://t1.aixinxi.net/o_1crbtd98b1eq1rs359nebvm3ua.png-j.jpg)

3. 卷积核3--边缘检测

   ![](http://t1.aixinxi.net/o_1crbtftm91grf8t1s3lmdj1u1a.png-j.jpg)

4. 卷积核4--图像锐化

   ![](http://t1.aixinxi.net/o_1crbth0kkek71uiq1tt7cud19r2a.png-j.jpg)

5. 卷积核5--浮雕效果

   ![](http://t1.aixinxi.net/o_1crbtht63b091rgjuac49n1vssa.png-j.jpg)

6. 卷积核6--高斯模糊

![](http://t1.aixinxi.net/o_1crbtipcshjhstt1mb94tf7n5a.png-j.jpg)



# 实验感想：

浮雕效果时，要给每个像素加上128的偏移量。
