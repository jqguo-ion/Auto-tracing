# Auto-tracing
# auto tracing 7.29
# Author: Jiaqi Guo

from PIL import Image 
import numpy as np 
import nrrd           #nrrd
import os
import time
import sys            #swc
import h5py  #导入工具包
import random

FILE_PATH = "20180123_438NiE_fish79_ZCM_ROI2_YYX_YYX.swc"
if os.path.exists(FILE_PATH):
    # 把文件对象给一个变量，这样后续才能操作这个文件对象，这里默认是以只读方式打开
    FILE_HANDLER = open(FILE_PATH, encoding="utf-8")
    # 读取
    """
    read(size) 不加SIZE可以是整个文件读取，加上SIZE可以一部分一部分的读取，对于大文件要使用SIZE分段读取，对于小文件可以整体读取
    readline() 一次读取一行
    readlines() 一次读取全部文件到一个列表，适合读取配置文件，因为配置文件通常都是一行一行的，对于配置文件这种非常适合因为它比较小而且
    配置文件都是一行一行的
    """
    data = FILE_HANDLER.readline()
    NeuronPosition = np.zeros((1620,3))           
    for i in range(1620):
        data = FILE_HANDLER.readline()
        #print(FILE_HANDLER.fileno())
        #time.sleep(600)
        LineContent=data.split()            #提取数据
        #print(Tempnumber)
        #print(int(LineContent[0]))          #报告行数
        NeuronPosition[i,0]=int(float(LineContent[2])*1024/499.71)
        NeuronPosition[i,1]=int(float(LineContent[3])*2970/1449.36)
        NeuronPosition[i,2]=int(float(LineContent[4]))
        #烦死了一时半会不知道怎么构建整形数组，先拿浮点数组顶一下，后面要记得改成整形！
        #print(NeuronPosition[i,1])
        for j in range(9):
            data = FILE_HANDLER.readline()
    # 关闭
    FILE_HANDLER.close()
else:
    print("文件：", FILE_PATH, "不存在。")

    
# nrrd图片读取
# nrrd图片使用nrrd包gitHub中的data数据
nrrd_filename = '20180123_438NiE_fish79_ZCM_R.nrrd'
nrrd_data, nrrd_options = nrrd.read(nrrd_filename)
ExtraImage = np.zeros((1144,3090,326))
#print(nrrd_data[2969,1023,261])

for i in range(1024):
    for j in range(2970):
        for k in range(262):
            ExtraImage[i+60,j+60,k+32]=float(nrrd_data[i,j,k])       #坐标变换，扩展图像
    print(i)
#print(nrrd_data)           #输出数组
#print(nrrd_options)
#print(np.amax(nrrd_data))  #求最大、最小值
#print(np.amin(nrrd_data))
#print(nrrd_data[1000,2000,200]) #输出单个值
#for i in [0,1,2,3,4,5,6,7,8]:
#   print(nrrd_data[481,1914,int(i)])
TempArray = np.zeros((61,61,5))        #像素块大小
for i in range(1620):
    for j in range(61):
        for k in range(61):
            for l in range(5):
                TempArray[j,k,l] = ExtraImage[int(NeuronPosition[i,0])-30+60+j,int(NeuronPosition[i,1])-30+60+k,int(NeuronPosition[i,2])-2+32+l]
    f = h5py.File('TrainingData-6-'+str(i)+'.h5','w')   #创建一个h5文件，文件指针是f
    f['data'] = TempArray                 #将数据写入文件的主键data下面
    f['labels'] = 1  
    for m in range(2):
        r1=random.randint(10,30)*(random.randint(1,2)*2-3)
        r2=random.randint(10,30)*(random.randint(1,2)*2-3)
        r3=random.randint(10,30)*(random.randint(1,2)*2-3)#可以考虑在这里加上一个判断是否在神经元上的步骤，但是其实没啥必要哈哈哈哈
        for j in range(61):
            for k in range(61):
                for l in range(5):
                    TempArray[j,k,l] = ExtraImage[int(NeuronPosition[i,0])+r1-30+60+j,int(NeuronPosition[i,1])+r2-30+60+k,int(NeuronPosition[i,2])+r3-2+32+l]
        f = h5py.File('TrainingData-6-'+str(i)+'-'+str(m)+'.h5','w')   #创建一个h5文件，文件指针是f
        f['data'] = TempArray                 #将数据写入文件的主键data下面
        f['labels'] = 0

        
        
    
        

    

                
    

