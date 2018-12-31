一.相关概念


线性滤波可以说是图像处理最基本的方法，它可以允许我们对图像进行处理，产生很多不同的效果。做法非常简单，首先，有一个二维的滤波器矩阵（有个高大上的名字叫做卷积核，熟悉深度学习和卷积神经网络的同学肯定会比较熟悉，哈哈哈）和一个待处理的二维图像。
然后，对于图像中每一个像素点，计算它的邻域像素和滤波器矩阵对应元素的乘积，然后加起来，作为该像素位置的值，这样就完成了滤波过程。

对图像和滤波矩阵进行逐个元素相乘再求和的操作就相当于将一个二维的函数移动到另一个二维函数的所有位置，这个操作就叫卷积（Convolution）。
Convolution可以说是图像处理最基本的操作，但却非常有用。这个操作有两个非常关键的特点：第一，它是线性的；第二，具有平移不变性（shift-invariant）。线性的指这个操作是我们用每个像素的邻域的线性组合来代替这个像素。平移不变性表示我们在图像的每个位置都执行相同的操作。这两个特点使得这个操作非常简单，因为线性操作是最简单的，然后在所有地方都做同样的操作就更简单了。
图像卷积需要4个嵌套循环，所以它并不快，除非我们使用非常小的卷积核。这里一般使用3X3，或者5X5。而且，对于滤波器，也有一定的规则要求：
（1）滤波器的大小应该是奇数，这样它才有一个中心，例如3X3、5X5，或者7X7。
（2）滤波器所有元素之和应该是1，这是为了保持滤波前后图像的亮度保持不变。
（3）如果滤波矩阵所有元素之和大于 1，那么滤波后的图像就会比原图像亮。反之，如果小于1，那么得到的图像就会变暗。
（4）对于滤波后的结构，可能会出现负数或者大于255的数值，对于这种情况，我们将他们直接截断至0和255之间即可。


二.实现过程

#include <iostream>

using namespace std;

#include "./gdal/gdal_priv.h"

#pragma comment(lib, "gdal_i.lib")



//卷积核1

int fun1()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "firstgril.tif";



	GByte* buffTmpr;//图像红色通道内存存储

	GByte* buffTmpg;//图像绿色色通道内存存储

	GByte* buffTmpb;//图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();

	

	//根据图像的高度和宽度分配内存

	buffTmpr = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));

	buffTmpg = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));

	buffTmpb = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Byte, 0, 0);



	//三通道卷积

	for (int i = 1; i < imgXlen - 1; i++)

	{

		for (int j = 1; j < imgYlen - 1; j++)

		{

			buffTmpr[i * imgYlen + j] = (buffTmpr[(i - 1) * imgYlen + j] + buffTmpr[i * imgYlen + j - 1] + buffTmpr[i * imgYlen + j] + buffTmpr[i * imgYlen + j + 1] + buffTmpr[(i + 1) * imgYlen + j]) / 5;

			buffTmpg[i * imgYlen + j] = (buffTmpg[(i - 1) * imgYlen + j] + buffTmpg[i * imgYlen + j - 1] + buffTmpg[i * imgYlen + j] + buffTmpg[i * imgYlen + j + 1] + buffTmpg[(i + 1) * imgYlen + j]) / 5;

			buffTmpb[i * imgYlen + j] = (buffTmpb[(i - 1) * imgYlen + j] + buffTmpb[i * imgYlen + j - 1] + buffTmpb[i * imgYlen + j] + buffTmpb[i * imgYlen + j + 1] + buffTmpb[(i + 1) * imgYlen + j]) / 5;

		}

	}



	//第一行，最末行设置为0

	for (int i = 0; i < imgXlen; i++)

	{

		buffTmpr[i] = (GByte)0;

		buffTmpg[i] = (GByte)0;

		buffTmpb[i] = (GByte)0;



		buffTmpr[(imgYlen - 1) * imgXlen + i] = (GByte)0;

		buffTmpg[(imgYlen - 1) * imgXlen + i] = (GByte)0;

		buffTmpb[(imgYlen - 1) * imgXlen + i] = (GByte)0;

	}



	//第一列，最后一列设置为0

	for (int i = 0; i < imgYlen; i++)

	{

		buffTmpr[i * imgXlen] = (GByte)0;

		buffTmpg[i * imgXlen] = (GByte)0;

		buffTmpb[i * imgXlen] = (GByte)0;



		buffTmpr[i * imgXlen + imgXlen - 1] = (GByte)0;

		buffTmpg[i * imgXlen + imgXlen - 1] = (GByte)0;

		buffTmpb[i * imgXlen + imgXlen - 1] = (GByte)0;

	}



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Byte, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);



	system("PAUSE");

	return 0;

}



//卷积核2

int fun2()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "secondGril.tif";



	GByte* buffTmpr;//图像红色通道内存存储

	GByte* buffTmpg;//图像绿色色通道内存存储

	GByte* buffTmpb;//图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();



	//根据图像的高度和宽度分配内存

	buffTmpr = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));

	buffTmpg = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));

	buffTmpb = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Byte, 0, 0);



	//三通道卷积

	for (int i = 2; i < imgYlen - 2; i++)

	{

		for (int j = 2; j < imgXlen - 2; j++)

		{

			buffTmpr[i * imgXlen + j] = (buffTmpr[(i - 2) * imgXlen + j - 2] + buffTmpr[(i - 1) * imgXlen + j - 1] + buffTmpr[i * imgXlen + j] + buffTmpr[(i + 1) * imgXlen + j + 1] + buffTmpr[(i + 2) * imgXlen + j + 2]) / 5;

			buffTmpg[i * imgXlen + j] = (buffTmpg[(i - 2) * imgXlen + j - 2] + buffTmpg[(i - 1) * imgXlen + j - 1] + buffTmpg[i * imgXlen + j] + buffTmpg[(i + 1) * imgXlen + j + 1] + buffTmpg[(i + 2) * imgXlen + j + 2]) / 5;

			buffTmpb[i * imgXlen + j] = (buffTmpb[(i - 2) * imgXlen + j - 2] + buffTmpb[(i - 1) * imgXlen + j - 1] + buffTmpb[i * imgXlen + j] + buffTmpb[(i + 1) * imgXlen + j + 1] + buffTmpb[(i + 2) * imgXlen + j + 2]) / 5;

		}

	}



	//第一、二行，最末行、倒数第二行设置为0

	for (int i = 0; i < imgXlen; i++)

	{

		buffTmpr[i] = (GByte)0;

		buffTmpg[i] = (GByte)0;

		buffTmpb[i] = (GByte)0;

		buffTmpr[imgXlen + i] = (GByte)0;

		buffTmpg[imgXlen + i] = (GByte)0;

		buffTmpb[imgXlen + i] = (GByte)0;



		buffTmpr[(imgYlen - 1) * imgXlen + i] = (GByte)0;

		buffTmpg[(imgYlen - 1) * imgXlen + i] = (GByte)0;

		buffTmpb[(imgYlen - 1) * imgXlen + i] = (GByte)0;

		buffTmpr[(imgYlen - 2) * imgXlen + i] = (GByte)0;

		buffTmpg[(imgYlen - 2) * imgXlen + i] = (GByte)0;

		buffTmpb[(imgYlen - 2) * imgXlen + i] = (GByte)0;

	}



	//第一列、第二列，最后一列、倒数第二列设置为0

	for (int i = 0; i < imgYlen; i++)

	{

		buffTmpr[i * imgXlen] = (GByte)0;

		buffTmpg[i * imgXlen] = (GByte)0;

		buffTmpb[i * imgXlen] = (GByte)0;

		buffTmpr[1 + i * imgXlen] = (GByte)0;

		buffTmpg[1 + i * imgXlen] = (GByte)0;

		buffTmpb[1 + i * imgXlen] = (GByte)0;



		buffTmpr[i * imgXlen + imgXlen - 1] = (GByte)0;

		buffTmpg[i * imgXlen + imgXlen - 1] = (GByte)0;

		buffTmpb[i * imgXlen + imgXlen - 1] = (GByte)0;

		buffTmpr[i * imgXlen + imgXlen - 2] = (GByte)0;

		buffTmpg[i * imgXlen + imgXlen - 2] = (GByte)0;

		buffTmpb[i * imgXlen + imgXlen - 2] = (GByte)0;

	}



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Byte, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Byte, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);



	system("PAUSE");

	return 0;

}



//卷积核3

int fun3()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "thirdGril.tif";



	float* buffTmpr;//原图像红色通道内存存储

	float* buffTmpg;//原图像绿色色通道内存存储

	float* buffTmpb;//原图像蓝色通道内存存储



	float* buffTmprn;//处理后图像红色通道内存存储

	float* buffTmpgn;//处理后图像绿色通道内存存储

	float* buffTmpbn;//处理后图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();



	//根据图像的高度和宽度分配内存

	buffTmpr = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpg = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpb = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmprn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpgn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpbn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Float32, 0, 0);







	/*三通道卷积

	for (int i = 1; i < imgYlen - 1; i++)

	{

		for (int j = 1; j < imgXlen - 1; j++)

		{

			buffTmprn[i * imgXlen + j] = (buffTmpr[i * imgXlen + j] * 8 - buffTmpr[(i - 1) * imgXlen + j - 1] - buffTmpr[(i - 1) * imgXlen + j] - buffTmpr[(i - 1) * imgXlen + j + 1] - buffTmpr[i * imgXlen + j - 1] - buffTmpr[i * imgXlen + j + 1] - buffTmpr[(i + 1) * imgXlen + j - 1] - buffTmpr[(i + 1) * imgXlen + j] - buffTmpr[(i + 1) * imgXlen + j + 1])%256;

			buffTmpgn[i * imgXlen + j] = (buffTmpg[i * imgXlen + j] * 8 - buffTmpg[(i - 1) * imgXlen + j - 1] - buffTmpg[(i - 1) * imgXlen + j] - buffTmpg[(i - 1) * imgXlen + j + 1] - buffTmpg[i * imgXlen + j - 1] - buffTmpg[i * imgXlen + j + 1] - buffTmpg[(i + 1) * imgXlen + j - 1] - buffTmpg[(i + 1) * imgXlen + j] - buffTmpg[(i + 1) * imgXlen + j + 1])%256;

			buffTmpbn[i * imgXlen + j] = (buffTmpb[i * imgXlen + j] * 8 - buffTmpb[(i - 1) * imgXlen + j - 1] - buffTmpb[(i - 1) * imgXlen + j] - buffTmpb[(i - 1) * imgXlen + j + 1] - buffTmpb[i * imgXlen + j - 1] - buffTmpb[i * imgXlen + j + 1] - buffTmpb[(i + 1) * imgXlen + j - 1] - buffTmpb[(i + 1) * imgXlen + j] - buffTmpb[(i + 1) * imgXlen + j + 1])%256;

		}

	}*/

	//测试

	for (int j = 1; j < imgYlen - 1; j++)

	{

		for (int i = 1; i < imgXlen - 1; i++)

		{

			buffTmprn[j*imgXlen + i] = (-buffTmpr[(j - 1)*imgXlen + i - 1]

				- buffTmpr[(j - 1)*imgXlen + i]

				- buffTmpr[(j - 1)*imgXlen + i + 1]

				- buffTmpr[j*imgXlen + i - 1]

				+ buffTmpr[j*imgXlen + i] * 8

				- buffTmpr[j*imgXlen + i + 1]

				- buffTmpr[(j + 1)*imgXlen + i - 1]

				- buffTmpr[(j + 1)*imgXlen + i]

				- buffTmpr[(j + 1)*imgXlen + i + 1]);



			buffTmpgn[j*imgXlen + i] = (-buffTmpg[(j - 1)*imgXlen + i - 1]

				- buffTmpg[(j - 1)*imgXlen + i]

				- buffTmpg[(j - 1)*imgXlen + i + 1]

				- buffTmpg[j*imgXlen + i - 1]

				+ buffTmpg[j*imgXlen + i] * 8

				- buffTmpg[j*imgXlen + i + 1]

				- buffTmpg[(j + 1)*imgXlen + i - 1]

				- buffTmpg[(j + 1)*imgXlen + i]

				- buffTmpg[(j + 1)*imgXlen + i + 1]);



			buffTmpbn[j*imgXlen + i] = (-buffTmpb[(j - 1)*imgXlen + i - 1]

				- buffTmpb[(j - 1)*imgXlen + i]

				- buffTmpb[(j - 1)*imgXlen + i + 1]

				- buffTmpb[j*imgXlen + i - 1]

				+ buffTmpb[j*imgXlen + i] * 8

				- buffTmpb[j*imgXlen + i + 1]

				- buffTmpb[(j + 1)*imgXlen + i - 1]

				- buffTmpb[(j + 1)*imgXlen + i]

				- buffTmpb[(j + 1)*imgXlen + i + 1]);

		}

	}



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmprn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpgn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpbn, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	CPLFree(buffTmprn);

	CPLFree(buffTmpgn);

	CPLFree(buffTmpbn);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);



	system("PAUSE");

	return 0;

}



//卷积核4

int fun4()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "forthGril.tif";



	float* buffTmpr;//原图像红色通道内存存储

	float* buffTmpg;//原图像绿色色通道内存存储

	float* buffTmpb;//原图像蓝色通道内存存储



	float* buffTmprn;//处理后图像红色通道内存存储

	float* buffTmpgn;//处理后图像绿色通道内存存储

	float* buffTmpbn;//处理后图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();



	//根据图像的高度和宽度分配内存

	buffTmpr = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpg = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpb = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmprn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpgn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpbn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Float32, 0, 0);







	//三通道卷积

	for (int i = 1; i < imgYlen - 1; i++)

	{

		for (int j = 1; j < imgXlen - 1; j++)

		{

			buffTmprn[i * imgXlen + j] = (buffTmpr[i * imgXlen + j] * 9 -

				buffTmpr[(i - 1) * imgXlen + j - 1] - 

				buffTmpr[(i - 1) * imgXlen + j] - 

				buffTmpr[(i - 1) * imgXlen + j + 1] - 

				buffTmpr[i * imgXlen + j - 1] - 

				buffTmpr[i * imgXlen + j + 1] - 

				buffTmpr[(i + 1) * imgXlen + j - 1] - 

				buffTmpr[(i + 1) * imgXlen + j] - 

				buffTmpr[(i + 1) * imgXlen + j + 1]);

			buffTmpgn[i * imgXlen + j] = (buffTmpg[i * imgXlen + j] * 9 - buffTmpg[(i - 1) * imgXlen + j - 1] - buffTmpg[(i - 1) * imgXlen + j] - buffTmpg[(i - 1) * imgXlen + j + 1] - buffTmpg[i * imgXlen + j - 1] - buffTmpg[i * imgXlen + j + 1] - buffTmpg[(i + 1) * imgXlen + j - 1] - buffTmpg[(i + 1) * imgXlen + j] - buffTmpg[(i + 1) * imgXlen + j + 1]);

			buffTmpbn[i * imgXlen + j] = (buffTmpb[i * imgXlen + j] * 9 - buffTmpb[(i - 1) * imgXlen + j - 1] - buffTmpb[(i - 1) * imgXlen + j] - buffTmpb[(i - 1) * imgXlen + j + 1] - buffTmpb[i * imgXlen + j - 1] - buffTmpb[i * imgXlen + j + 1] - buffTmpb[(i + 1) * imgXlen + j - 1] - buffTmpb[(i + 1) * imgXlen + j] - buffTmpb[(i + 1) * imgXlen + j + 1]);

		}

	}

	

	/*

	//第一行，最末行设置为0

	for (int i = 0; i < imgXlen; i++)

	{

		buffTmprn[i] = 0;

		buffTmpgn[i] = 0;

		buffTmpbn[i] = 0;



		buffTmprn[(imgYlen - 1) * imgXlen + i] = 0;

		buffTmpgn[(imgYlen - 1) * imgXlen + i] = 0;

		buffTmpbn[(imgYlen - 1) * imgXlen + i] = 0;

	}



	//第一列，最后一列设置为0

	for (int i = 0; i < imgYlen; i++)

	{

		buffTmprn[i * imgXlen] = 0;

		buffTmpgn[i * imgXlen] = 0;

		buffTmpbn[i * imgXlen] = 0;



		buffTmprn[i * imgXlen + imgXlen - 1] = 0;

		buffTmpgn[i * imgXlen + imgXlen - 1] = 0;

		buffTmpbn[i * imgXlen + imgXlen - 1] = 0;

	} */

	



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmprn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpgn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpbn, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	CPLFree(buffTmprn);

	CPLFree(buffTmpgn);

	CPLFree(buffTmpbn);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);



	system("PAUSE");

	return 0;

}



//卷积核5

int fun5()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "fifthGril.tif";



	float* buffTmpr;//原图像红色通道内存存储

	float* buffTmpg;//原图像绿色色通道内存存储

	float* buffTmpb;//原图像蓝色通道内存存储



	float* buffTmprn;//处理后图像红色通道内存存储

	float* buffTmpgn;//处理后图像绿色通道内存存储

	float* buffTmpbn;//处理后图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();



	//根据图像的高度和宽度分配内存

	buffTmpr = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpg = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpb = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmprn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpgn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpbn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//三通道卷积

	for (int i = 1; i < imgYlen - 1; i++)

	{

		for (int j = 1; j < imgXlen - 1; j++)

		{

			buffTmprn[j*imgXlen + i] = (-buffTmpr[(j - 1)*imgXlen + i - 1]

				- buffTmpr[(j - 1)*imgXlen + i]

				- buffTmpr[j*imgXlen + i - 1]

				+ buffTmpr[j*imgXlen + i + 1]

				+ buffTmpr[(j + 1)*imgXlen + i]

				+ buffTmpr[(j + 1)*imgXlen + i + 1]);

			buffTmpgn[j*imgXlen + i] = (-buffTmpg[(j - 1)*imgXlen + i - 1]

				- buffTmpg[(j - 1)*imgXlen + i]

				- buffTmpg[j*imgXlen + i - 1]

				+ buffTmpg[j*imgXlen + i + 1]

				+ buffTmpg[(j + 1)*imgXlen + i]

				+ buffTmpg[(j + 1)*imgXlen + i + 1]);

			buffTmpbn[j*imgXlen + i] = (-buffTmpb[(j - 1)*imgXlen + i - 1]

				- buffTmpb[(j - 1)*imgXlen + i]

				- buffTmpb[j*imgXlen + i - 1]

				+ buffTmpb[j*imgXlen + i + 1]

				+ buffTmpb[(j + 1)*imgXlen + i]

				+ buffTmpb[(j + 1)*imgXlen + i + 1]);

		}

	}



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmprn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpgn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpbn, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	CPLFree(buffTmprn);

	CPLFree(buffTmpgn);

	CPLFree(buffTmpbn);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);



	system("PAUSE");

	return 0;

}



//卷积核6

int fun6()

{

	//输入图像

	GDALDataset* poSrcDS;

	//输出图像

	GDALDataset* poDstDS;

	//图像的宽度和高度

	int imgXlen, imgYlen;

	//输入图像路径

	char* srcPath = "girl.jpg";

	//输出图像路径

	char* dstPath = "sixthGril.tif";



	float* buffTmpr;//原图像红色通道内存存储

	float* buffTmpg;//原图像绿色色通道内存存储

	float* buffTmpb;//原图像蓝色通道内存存储



	float* buffTmprn;//处理后图像红色通道内存存储

	float* buffTmpgn;//处理后图像绿色通道内存存储

	float* buffTmpbn;//处理后图像蓝色通道内存存储

	//图像波段数

	int bandNum;



	//注册驱动

	GDALAllRegister();



	//打开图像

	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);



	//获取图像宽度，高度和波段数量

	imgXlen = poSrcDS->GetRasterXSize();

	imgYlen = poSrcDS->GetRasterYSize();

	bandNum = poSrcDS->GetRasterCount();



	//根据图像的高度和宽度分配内存

	buffTmpr = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpg = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpb = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmprn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpgn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	buffTmpbn = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));



	//图像三通道信息读入对应存储

	poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpr, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpg, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poSrcDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmpb, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//三通道卷积

	for (int i = 1; i < imgYlen - 1; i++)

	{

		for (int j = 1; j < imgXlen - 1; j++)

		{

			buffTmprn[j*imgXlen + i] = (0.0120*buffTmpr[(j - 2)*imgXlen + i - 2] +

				0.1253*buffTmpr[(j - 2)*imgXlen + i - 1] +

				0.2736*buffTmpr[(j - 2)*imgXlen + i] +

				0.1253*buffTmpr[(j - 2)*imgXlen + i + 1] +

				0.0120*buffTmpr[(j - 2)*imgXlen + i + 2] +

				0.1253*buffTmpr[(j - 1)*imgXlen + i - 2] +

				1.3054*buffTmpr[(j - 1)*imgXlen + i - 1] +

				2.8514*buffTmpr[(j - 1)*imgXlen + i] +

				1.3054*buffTmpr[(j - 1)*imgXlen + i + 1] +

				0.1253*buffTmpr[(j - 1)*imgXlen + i + 2] +

				0.2763*buffTmpr[j*imgXlen + i - 2] +

				2.8514*buffTmpr[j*imgXlen + i - 1] +

				6.2279*buffTmpr[j*imgXlen + i] +

				2.8514*buffTmpr[j*imgXlen + i + 1] +

				0.2763*buffTmpr[j*imgXlen + i + 2] +

				0.1253*buffTmpr[(j + 1)*imgXlen + i - 2] +

				1.3054*buffTmpr[(j + 1)*imgXlen + i - 1] +

				2.8514*buffTmpr[(j + 1)*imgXlen + i] +

				1.3054*buffTmpr[(j + 1)*imgXlen + i + 1] +

				0.1253*buffTmpr[(j + 1)*imgXlen + i + 2] +

				0.0120*buffTmpr[(j + 2)*imgXlen + i - 2] +

				0.1253*buffTmpr[(j + 2)*imgXlen + i - 1] +

				0.2736*buffTmpr[(j + 2)*imgXlen + i] +

				0.1253*buffTmpr[(j + 2)*imgXlen + i + 1] +

				0.0120*buffTmpr[(j + 2)*imgXlen + i + 2]) / 25.0f;

			buffTmpgn[j*imgXlen + i] = (0.0120*buffTmpg[(j - 2)*imgXlen + i - 2] +

				0.1253*buffTmpg[(j - 2)*imgXlen + i - 1] +

				0.2736*buffTmpg[(j - 2)*imgXlen + i] +

				0.1253*buffTmpg[(j - 2)*imgXlen + i + 1] +

				0.0120*buffTmpg[(j - 2)*imgXlen + i + 2] +

				0.1253*buffTmpg[(j - 1)*imgXlen + i - 2] +

				1.3054*buffTmpg[(j - 1)*imgXlen + i - 1] +

				2.8514*buffTmpg[(j - 1)*imgXlen + i] +

				1.3054*buffTmpg[(j - 1)*imgXlen + i + 1] +

				0.1253*buffTmpg[(j - 1)*imgXlen + i + 2] +

				0.2763*buffTmpg[j*imgXlen + i - 2] +

				2.8514*buffTmpg[j*imgXlen + i - 1] +

				6.2279*buffTmpg[j*imgXlen + i] +

				2.8514*buffTmpg[j*imgXlen + i + 1] +

				0.2763*buffTmpg[j*imgXlen + i + 2] +

				0.1253*buffTmpg[(j + 1)*imgXlen + i - 2] +

				1.3054*buffTmpg[(j + 1)*imgXlen + i - 1] +

				2.8514*buffTmpg[(j + 1)*imgXlen + i] +

				1.3054*buffTmpg[(j + 1)*imgXlen + i + 1] +

				0.1253*buffTmpg[(j + 1)*imgXlen + i + 2] +

				0.0120*buffTmpg[(j + 2)*imgXlen + i - 2] +

				0.1253*buffTmpg[(j + 2)*imgXlen + i - 1] +

				0.2736*buffTmpg[(j + 2)*imgXlen + i] +

				0.1253*buffTmpg[(j + 2)*imgXlen + i + 1] +

				0.0120*buffTmpg[(j + 2)*imgXlen + i + 2]) / 25.0f;

			buffTmpbn[j*imgXlen + i] = (0.0120*buffTmpb[(j - 2)*imgXlen + i - 2] +

				0.1253*buffTmpb[(j - 2)*imgXlen + i - 1] +

				0.2736*buffTmpb[(j - 2)*imgXlen + i] +

				0.1253*buffTmpb[(j - 2)*imgXlen + i + 1] +

				0.0120*buffTmpb[(j - 2)*imgXlen + i + 2] +

				0.1253*buffTmpb[(j - 1)*imgXlen + i - 2] +

				1.3054*buffTmpb[(j - 1)*imgXlen + i - 1] +

				2.8514*buffTmpb[(j - 1)*imgXlen + i] +

				1.3054*buffTmpb[(j - 1)*imgXlen + i + 1] +

				0.1253*buffTmpb[(j - 1)*imgXlen + i + 2] +

				0.2763*buffTmpb[j*imgXlen + i - 2] +

				2.8514*buffTmpb[j*imgXlen + i - 1] +

				6.2279*buffTmpb[j*imgXlen + i] +

				2.8514*buffTmpb[j*imgXlen + i + 1] +

				0.2763*buffTmpb[j*imgXlen + i + 2] +

				0.1253*buffTmpb[(j + 1)*imgXlen + i - 2] +

				1.3054*buffTmpb[(j + 1)*imgXlen + i - 1] +

				2.8514*buffTmpb[(j + 1)*imgXlen + i] +

				1.3054*buffTmpb[(j + 1)*imgXlen + i + 1] +

				0.1253*buffTmpb[(j + 1)*imgXlen + i + 2] +

				0.0120*buffTmpb[(j + 2)*imgXlen + i - 2] +

				0.1253*buffTmpb[(j + 2)*imgXlen + i - 1] +

				0.2736*buffTmpb[(j + 2)*imgXlen + i] +

				0.1253*buffTmpb[(j + 2)*imgXlen + i + 1] +

				0.0120*buffTmpb[(j + 2)*imgXlen + i + 2]) / 25.0f;

		}

	}



	//创建输出图像

	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);



	//输出图像

	poDstDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmprn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpgn, imgXlen, imgYlen, GDT_Float32, 0, 0);

	poDstDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmpbn, imgXlen, imgYlen, GDT_Float32, 0, 0);



	//清除内存

	CPLFree(buffTmpr);

	CPLFree(buffTmpg);

	CPLFree(buffTmpb);

	CPLFree(buffTmprn);

	CPLFree(buffTmpgn);

	CPLFree(buffTmpbn);

	//关闭dataset

	GDALClose(poDstDS);

	GDALClose(poSrcDS);

	return 0;

}



int main()

{

	//fun1();

	//fun2();

	//fun3();

	//fun4();

	fun5();

	//fun6();

	return 0;

}



三.实验感想


1.在使用滤波器时，要注意+128像素


2.对于边界值的计算，有4种主流方法，


（1）第一种就是想象边界是无限长的图像的一部分，除了我们给定值的部分，其他部分的像素值都是0。


（2）第二种方法也是想象边界是无限图像的一部分。但没有指定的部分是用图像边界的值进行拓展。


（3）第三种情况就是认为图像是周期性的。也就是边界不断的重复。周期就是边界的长度。


（4）不管其他地方了。我们觉得I之外的情况是没有定义的，所以没办法使用这些没有定义的值，所以要使用图像I没有定义的值的像素都没办法计算.
我们所使用的是将边界值都取0的方法
