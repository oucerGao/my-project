# 程序设计基础实践项目文档

## 程序基本功能简述：

- 计算

  - 求均值
  - 求方差

- 修改尺寸

- 翻转

  - 上下翻转
  - 左右翻转

- 转为灰度图

- 二值化

- 轮廓提取

- 拉普拉斯锐化

- 边缘检测

- 高通滤波

- 高斯模糊

  



## 程序设计过程：

### 第一周

1. 小组成员首先熟悉QT的相关操作以及语法规则
2. 构建自己的图片处理雏形
3. 统一讨论，规定完成的功能

### 第二周

1. 分别编写自己模块的相关程序
2. 书写设计过程

### 第三周

1. 整合全部功能的相应代码，并设计UI
2. 进行相关检测及调试
3. 完成视频录制以及Markdown的上传

## 程序各功能详解及相应展示

### 计算

此处计算分两个模块，一是计算图片RGB的均值，二是计算图片RGB的方差值。

```c++
void MainWindow::on_actionaverage_triggered()//求均值
{
    double * averages = img->average();
    QString displayInfo = tr("均值计算结果:\r\n\r\n");
    QString tempInfo;
    tempInfo = tr("R: %1\r\n").arg(averages[0]);
    displayInfo += tempInfo;
    tempInfo = tr("G: %1\r\n").arg(averages[1]);
    displayInfo += tempInfo;
    tempInfo = tr("B: %1\r\n").arg(averages[2]);
    displayInfo += tempInfo;
    QMessageBox::information(this, tr("result"), displayInfo);
}
```

```c++
void MainWindow::on_actionvariance_triggered()//求方差
{
    double * variances = img->variance();
    QString displayInfo = tr("方差计算结果:\r\n\r\n");
    QString tempInfo;
    tempInfo = tr("R: %1\r\n").arg(variances[0]);
    displayInfo += tempInfo;
    tempInfo = tr("G: %1\r\n").arg(variances[1]);
    displayInfo += tempInfo;
    tempInfo = tr("B: %1\r\n").arg(variances[2]);
    displayInfo += tempInfo;
    QMessageBox::information(this, tr("result"), displayInfo);
}
```

### 修改尺寸

修改尺寸过程中注意放大后矩阵的填充。

```c++
void MainWindow::on_actionresize_triggered()//修改尺寸
{
    checkCopies();
    QDialog dialog(this);
    QFormLayout form(&dialog);
    form.addRow(new QLabel("User input:"));
    // Value1
    QString value1 = QString("Height: ");
    QSpinBox *spinbox1 = new QSpinBox(&dialog);
    spinbox1->setRange(100, 10000);
    form.addRow(value1, spinbox1);
    // Value2
    QString value2 = QString("Width: ");
    QSpinBox *spinbox2 = new QSpinBox(&dialog);
    spinbox2->setRange(100, 10000);
    form.addRow(value2, spinbox2);
    // Add Cancel and OK button
    QDialogButtonBox buttonBox(QDialogButtonBox::Ok | QDialogButtonBox::Cancel,
        Qt::Horizontal, &dialog);
    form.addRow(&buttonBox);
    connect(&buttonBox, &QDialogButtonBox::accepted, &dialog, &QDialog::accept);
    connect(&buttonBox, &QDialogButtonBox::rejected, &dialog, &QDialog::reject);
    // Process when OK button is clicked
    if (dialog.exec() == QDialog::Accepted)
    {
        if(spinbox1->value() <= 0 || spinbox2->value() <= 0)
            return;
        img->resize(spinbox1->value(), spinbox2->value());
        copies.push_back(*img);
        index++;
        showImg();
    }
}
```

### 翻转

自定义交换函数swapPixel()实现图片的翻转。

```c++
void Image::swapPixel(int x1, int y1, int x2, int y2)
{
    double t;
    t = *getPixel(data, x1, y1);
    *getPixel(data, x1, y1) = *getPixel(data, x2, y2);
    *getPixel(data, x2, y2) = t;
    t = *(getPixel(data, x1, y1) + 1);
    *(getPixel(data, x1, y1) + 1) = *(getPixel(data, x2, y2) + 1);
    *(getPixel(data, x2, y2) + 1) = t;
    t = *(getPixel(data, x1, y1) + 2);
    *(getPixel(data, x1, y1) + 2) = *(getPixel(data, x2, y2) + 2);
    *(getPixel(data, x2, y2) + 2) = t;
}
```

### 转为灰度图

自定义函数ToGray()用来修改图片的相关参数。

```c++
void Image::ToGray()
{
    for(int i = 0; i < height; i++)
    {
        for(int j = 0; j < width; j++)
        {
            double t = 0.3 * (*getPixel(data, i, j)) + 0.59 * (*(getPixel(data, i, j) + 1))
                    + 0.11 * (*(getPixel(data, i, j) + 2));
            *getPixel(data, i, j) = t;
            *(getPixel(data, i, j) + 1) = t;
            *(getPixel(data, i, j) + 2) = t;
        }
    }
}
```

### 二值化

通过上述ToGray()函数先转为灰度图，然后设置为阈值进行判断。

```c++
void Image::Binarization()
{
    //转为灰度图
    ToGray();
    //选取128作为全局阈值
    for(int i = 0; i < height; i++)
    {
        for(int j = 0; j < width; j++)
        {
            if(*getPixel(data, i, j) > 128)
            {
                *getPixel(data, i, j) = 255;
                *(getPixel(data, i, j) + 1) = 255;
                *(getPixel(data, i, j) + 2) = 255;
            }
            else
            {
                *getPixel(data, i, j) = 0;
                *(getPixel(data, i, j) + 1) = 0;
                *(getPixel(data, i, j) + 2) = 0;
            }
        }
    }
}
```

### 轮廓提取

在二值化的基础上，判断图片中的边界。

```c++
for(int i = 1; i < height - 1; i++)
    {
        for(int j = 1; j < width - 1; j++)
        {
            if(*getPixel(data, i, j) == 0)
            {
                memset(pixel, 0, 8);
                *getPixel(newImg, i, j) = 0;
                *(getPixel(newImg, i, j) + 1) = 0;
                *(getPixel(newImg, i, j) + 2) = 0;
                pixel[0] = *getPixel(data, i - 1, j - 1);
                pixel[1] = *getPixel(data, i - 1, j);
                pixel[2] = *getPixel(data, i - 1, j + 1);
                pixel[3] = *getPixel(data, i, j - 1);
                pixel[4] = *getPixel(data, i, j + 1);
                pixel[5] = *getPixel(data, i + 1, j - 1);
                pixel[6] = *getPixel(data, i + 1, j);
                pixel[7] = *getPixel(data, i + 1, j + 1);

                if (pixel[0] + pixel[1] + pixel[2] + pixel[3] + pixel[4] + pixel[5] + pixel[6] + pixel[7] == 0)
                {
                    *getPixel(newImg, i, j) = 255;
                    *(getPixel(newImg, i, j) + 1) = 255;
                    *(getPixel(newImg, i, j) + 2) = 255;
                }
            }
        }
    }
```



### 拉普拉斯锐化

设置一个四邻域模板。

```c++
int laplacain[3][3] = {
        {0, -1, 0},
        {-1, 5, -1},
        {0, -1, 0}
    };
```

在此基础上对图片的RGB进行处理。

```c++
int red = 0, green = 0, blue = 0;
    for(int x = 0; x < height; x++)
    {
        for(int y = 0; y < width; y++)
        {
            if(x > 0 && x < height - 1 && y > 0 && y < width - 1)
            {
                red = green = blue = 0;
                for(int i = 0; i < 3; i++)
                {
                    for(int j = 0; j < 3; j++)
                    {
                        red += *getPixel(this->data, x+i-1, y+j-1) * laplacain[i][j];
                        green += *(getPixel(this->data, x+i-1, y+j-1) + 1) * laplacain[i][j];
                        blue += *(getPixel(this->data, x+i-1, y+j-1) + 2) * laplacain[i][j];
                    }
                }
            }
            else
            {
                *getPixel(temp, x, y) = *getPixel(this->data, x, y);
                *(getPixel(temp, x, y) + 1) = *(getPixel(this->data, x, y) + 1);
                *(getPixel(temp, x, y) + 2) = *(getPixel(this->data, x, y) + 2);
                continue;
            }

            if(red < 0) { *getPixel(temp, x, y) = 0; }
            else if(red > 255) { *getPixel(temp, x, y) = 255; }
            else { *getPixel(temp, x, y) = red; }

            if(green < 0) { *(getPixel(temp, x, y) + 1) = 0; }
            else if(green > 255) { *(getPixel(temp, x, y) + 1) = 255; }
            else { *(getPixel(temp, x, y) + 1) = green; }

            if(blue < 0) { *(getPixel(temp, x, y) + 2) = 0; }
            else if(blue > 255) { *(getPixel(temp, x, y) + 2) = 255; }
            else { *(getPixel(temp, x, y) + 2) = blue; }
        }
    }
```



### 边缘检测

边缘检测算子检查每个像素的邻域并对灰度变换率进行量化.

```c++
    int HX[3][3] = {{1, 0, -1}, {2, 0, -2}, {1, 0, -1}};
    int HY[3][3] = {{-1, -2, -1}, {0, 0, 0}, {1, 2, 1}};
    int redX, greenX, blueX;
    int redY, greenY, blueY;
```

```c++
for(int x = 0; x < height; x++)
    {
        for(int y = 0; y < width; y++)
        {
            if(x > 0 && x < height - 1 && y > 0 && y < width - 1)
            {
                redX = greenX = blueX = 0;
                redY = greenY = blueY = 0;
                for(int i = 0; i < 3; i++)
                {
                    for(int j = 0; j < 3; j++)
                    {
                        redX += *getPixel(this->data, x+i-1, y+j-1) * HX[i][j];
                        redY += *getPixel(this->data, x+i-1, y+j-1) * HY[i][j];
                        greenX += *(getPixel(this->data, x+i-1, y+j-1) + 1) * HX[i][j];
                        greenY += *(getPixel(this->data, x+i-1, y+j-1) + 1) * HY[i][j];
                        blueX += *(getPixel(this->data, x+i-1, y+j-1) + 2) * HX[i][j];
                        blueY += *(getPixel(this->data, x+i-1, y+j-1) + 2) * HY[i][j];
                    }
                }
            }
            else
            {
                *getPixel(temp, x, y) = *getPixel(this->data, x, y);
                *(getPixel(temp, x, y) + 1) = *(getPixel(this->data, x, y) + 1);
                *(getPixel(temp, x, y) + 2) = *(getPixel(this->data, x, y) + 2);
                continue;
            }

            int R, G, B;
            R = (int)(sqrt(redX * redX * 1.0 + redY * redY * 1.0));
            G = (int)(sqrt(greenX * greenX * 1.0 + greenY * greenY * 1.0));
            B = (int)(sqrt(blueX * blueX * 1.0 + blueY * blueY * 1.0));

            if (redX < 0 && redY < 0) { *getPixel(temp, x, y)=0; }
            else if (R > 255) { *getPixel(temp, x, y)=255; }
            else { *getPixel(temp, x, y)=R; }

            if (greenX<0 && greenY<0) { *(getPixel(temp, x, y) + 1)=0; }
            else if (G > 255) { *(getPixel(temp, x, y) + 1)=255; }
            else { *(getPixel(temp, x, y) + 1)=G; }

            if (blueX<0 && blueY<0 ) { *(getPixel(temp, x, y) + 2)=0; }
            else if (B > 255) { *(getPixel(temp, x, y) + 2)=255; }
            else { *(getPixel(temp, x, y) + 2)=B; }
        }
    }
```



### 高通滤波

```c++
void Image::High_passFilter()
{
    int H2[3][3] = {
        {-1, -1, -1},
        {-1, 8, -1},
        {-1, -1, -1}
    };
    unsigned char * temp = new unsigned char[height * width * 3];
    int red = 0, green = 0, blue = 0;
    for(int x = 0; x < height; x++)
    {
        for(int y = 0; y < width; y++)
        {
            if(x > 0 && x < height - 1 && y > 0 && y < width - 1)
            {
                red = green = blue = 0;
                for(int i = 0; i < 3; i++)
                {
                    for(int j = 0; j < 3; j++)
                    {
                        red += *getPixel(this->data, x+i-1, y+j-1) * H2[i][j];
                        green += *(getPixel(this->data, x+i-1, y+j-1) + 1) * H2[i][j];
                        blue += *(getPixel(this->data, x+i-1, y+j-1) + 2) * H2[i][j];
                    }
                }
            }
            else
            {
                *getPixel(temp, x, y) = *getPixel(this->data, x, y);
                *(getPixel(temp, x, y) + 1) = *(getPixel(this->data, x, y) + 1);
                *(getPixel(temp, x, y) + 2) = *(getPixel(this->data, x, y) + 2);
                continue;
            }
            if(red < 0) { *getPixel(temp, x, y) = 0; }
            else if(red > 255) { *getPixel(temp, x, y) = 255; }
            else { *getPixel(temp, x, y) = red; }

            if(green < 0) { *(getPixel(temp, x, y) + 1) = 0; }
            else if(green > 255) { *(getPixel(temp, x, y) + 1) = 255; }
            else { *(getPixel(temp, x, y) + 1) = green; }

            if(blue < 0) { *(getPixel(temp, x, y) + 2) = 0; }
            else if(blue > 255) { *(getPixel(temp, x, y) + 2) = 255; }
            else { *(getPixel(temp, x, y) + 2) = blue; }
        }
    }
    delete data;
    data = reinterpret_cast<unsigned char*>(temp);
}
```



### 高斯模糊

通过sigma设置高斯模板的值获取整张图的加权平均值。

```c++
void Image::Gaussian_Blur(double sigma)
{
    //图像副本
    unsigned char * temp = new unsigned char[height * width * 3];
    //获取高斯模板
    double dGaussianTemplate[3][3];
    int center = 1;
    double x = 0, y = 0;
    for(int i = 0; i < 3; i++)
    {
        x = pow(i - center, 2);
        for(int j = 0; j < 3; j++)
        {
            y = pow(j - center, 2);
            double g = exp(-(x + y) / (2 * sigma * sigma));
            g /= 2 * 3.1415926535 * sigma;
            dGaussianTemplate[i][j] = g;
        }
    }
        //归一化处理
    double t = 1 /dGaussianTemplate[0][0];
    for(int i = 0; i < 3; i++)
    {
        for(int j = 0; j < 3; j++)
        {
            dGaussianTemplate[i][j] *= t;
        }
    }
    int *gaussianTemplate = new int[3 * 3];
    unsigned char templateSum = 0;
    for(int i = 0; i < 3; i++)
    {
        for(int j = 0; j < 3; j++)
        {
            gaussianTemplate[i + j * 3] = floor(dGaussianTemplate[i][j]);
            templateSum += floor(dGaussianTemplate[i][j]);
        }
    }

    //卷积
    for(int i = 0; i < height; i++)
    {
        for(int j = 0; j < width; j++)
        {
            //边缘像素不作处理
            if(i == 0 || i == height - 1 || j == 0 || j == width - 1)
            {
                *getPixel(temp, i, j) = *getPixel(this->data, i, j);
                *(getPixel(temp, i, j) + 1) = *(getPixel(this->data, i ,j) + 1);
                *(getPixel(temp, i, j) + 2) = *(getPixel(this->data, i, j) + 2);
                continue;
            }
            //非边缘像素
            else
            {
                //R
                int newRed = 0;
                QVector<int> tempRed;
                for(int m = i - 1; m < i + 2; m++)
                {
                    for(int n = j - 1; n < j + 2; n++)
                    {
                        tempRed.push_back(*getPixel(data, m, n));
                    }
                }
                for(int k = 0; k < 9; k++)
                {
                    newRed += gaussianTemplate[k] * tempRed[k];
                }
                newRed /= templateSum;
                //qDebug() << newRed;
                //G
                int newGreen = 0;
                QVector<int> tempGreen;
                for(int m = i - 1; m < i + 2; m++)
                {
                    for(int n = j - 1; n < j + 2; n++)
                    {
                        tempGreen.push_back(*(getPixel(data, m, n) + 1));
                    }
                }
                for(int k = 0; k < 9; k++)
                {
                    newGreen += gaussianTemplate[k] * tempGreen[k];
                }
                newGreen /= templateSum;
                //qDebug() << newGreen;
                //B
                int newBlue = 0;
                QVector<int> tempBlue;
                for(int m = i - 1; m < i + 2; m++)
                {
                    for(int n = j - 1; n < j + 2; n++)
                    {
                        tempBlue.push_back(*(getPixel(data, m, n) + 2));
                    }
                }
                for(int k = 0; k < 9; k++)
                {
                    newBlue += gaussianTemplate[k] * tempBlue[k];
                }
                newBlue /= templateSum;
                //qDebug() << newBlue;
                //卷积结果写入副本
                *getPixel(temp, i, j) = newRed;
                *(getPixel(temp, i, j) + 1) = newGreen;
                *(getPixel(temp, i, j) + 2) = newBlue;
            }
        }
    }
    delete data;
    data = reinterpret_cast<unsigned char*>(temp);
}
```



## 小组成员分工及贡献

高东强：高斯模糊

孙向东：拉普拉斯锐化、边缘检测、高通滤波、计算，修改代码

邱雨晨：二值化

王煜升：翻转、轮廓提取

何圳奥：整合小组代码，bug检测并修改代码，视频录制

## 程序设计总结

1. 程序设计过程中一开始同学们都是各自编写相应的模块，设计思路没有完全一致，导致后续程序合并出现较大困难，整合过程中大家沟通交流互相帮助下才完成最终成效。完成本次项目后，回想起老师发的结对编程，才体会到其价值性。对于小组分工合作而言，重在合作，重在整体，小组之间思想的不同意会很大程度上影响程序后期的可操作性。
2. 对于没上过亓老师c++同学来说本次程序设计刚开始充满挑战性，但是学习新事物的过程就是从零到一的跨越，不畏惧挑战不怕困难，方能学到新东西掌握新事物。
