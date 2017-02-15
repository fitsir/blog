---
title: A Python Serial Tool--串口调试
categories: Python
toc: true
date: 2016-10-01 19:06:39
tags:
    - Python
    - Serial
    - PyQt5
---

# 串口调试助手
开发环境:
- Python 3.5.2
- PyQt5 

<!--more-->

```python
# -*- coding: utf-8 -*-
import binascii
import re
import sys

from PyQt5.Qt import *
from PyQt5.QtGui import QIcon, QFont
from PyQt5.QtSerialPort import QSerialPort, QSerialPortInfo
from PyQt5.QtWidgets import *
class PyQt_Serial(QWidget):
    def __init__(self, parent=None): #初始化
    def CreateItems(self): # 创建元素
    def CreateLayout(self): # 创建布局
    def CreateSignalSlot(self): # 信号与槽
    def on_refreshCom(self): # 刷新串口信息
    def on_setEncoding(self): #设置编码
    def on_openSerial(self): # 打开串口
    def on_closeSerial(self): # 关闭串口
    def on_timerSendChecked(self): #定时发送check槽函数
    def on_stopShowing(self): # 停止显示
    def on_clearCouter(self): # 清除计数
    def on_updateTimer(self): # 界面刷新
    def on_sendData(self): # 串口发送数据
    def on_receiveData(self): # 串口接收数据
    def on_hexShowingChecked(self): # 十六进制显示check槽函数
    def on_hexSendingChecked(self): # 十六进制发送check槽函数
    def on_saveReceivedData(self): # 保存接收数据
    def on_aboutButton(self): # 关于
    def on_aboutPyQtButton(self): # 关于QT

if __name__ == '__main__':
    app = QApplication(sys.argv)
    win = PyQt_Serial()
    win.show()
    app.exec_()
    app.exit()
```

## 类初始化
初始化包括以下几个步骤:
1. 初始化父类
2. 创建元素
3. 创建界面布局
4. 创建信号与槽
5. 配置界面标题, 图标, 字体
6. 初始化全局变量
7. 打开刷新界面定时器

```python
    def __init__(self, parent=None):
        super().__init__(parent)
        self.CreateItems()
        self.CreateLayout()
        self.CreateSignalSlot()

        self.setWindowTitle('串口调试助手 V0.4 by fitsir')
        self.setWindowIcon(QIcon('./icon.ico'))
        self.setFont(QFont('宋体', 9))

        self.sendCount = 0 # 发送计数
        self.receiveCount = 0 # 接收计数
        self.encoding = 'utf-8' # 默认编码
        self.updateTimer.start(100) # 界面刷新定时器开启,100ms
```
## 刷新串口
在界面不关闭的情况下,通过点击刷新按钮,更新当前串口信息,即,可在界面不关闭的情况下插拔串口
```python
    def on_refreshCom(self):
        self.comNameCombo.clear()
        com = QSerialPort()
        for info in QSerialPortInfo.availablePorts():
            com.setPort(info)
            if com.open(QSerialPort.ReadWrite):
                self.comNameCombo.addItem(info.portName())
                com.close()
```

## 设置编码
常用编码为`utf-8`和`gbk`,通过单选来设置,默认位`utf-8`
```python
    def on_setEncoding(self):
        if self.encodingGroup.checkedId() == 0:
            self.encoding = 'utf-8'
        else:
            self.encoding = 'gbk'
```

## 打开串口
1. 获取系统界面选择的串口号和波特率,通过`self.com.open(QSerialPort.ReadWrite) `语句打开.
2. 如果定时设置定时发送, 则打开定时发送定时器
3. 显示串口状态

```python
    def on_openSerial(self):
        comName = self.comNameCombo.currentText()
        comBaud = int(self.baudCombo.currentText())
        self.com.setPortName(comName)

        try:
            if self.com.open(QSerialPort.ReadWrite) == False:
                QMessageBox.critical(self, '严重错误', '串口打开失败')
                return
        except:
            QMessageBox.critical(self, '严重错误', '串口打开失败')
            return

        self.com.setBaudRate(comBaud)
        if self.timerSendCheck.isChecked():
            self.sendTimer.start(int(self.timerPeriodEdit.text()))

        self.openButton.setEnabled(False)
        self.closeButton.setEnabled(True)
        self.comNameCombo.setEnabled(False)
        self.baudCombo.setEnabled(False)
        self.sendButton.setEnabled(True)
        self.refreshComButton.setEnabled(False)
        self.comStatus.setText('串口状态：%s  打开   波特率 %s' % (comName, comBaud))
```

## 关闭串口
1. 关闭串口
2. 关闭定时发送定时器

```python
    def on_closeSerial(self):
        self.com.close()
        self.openButton.setEnabled(True)
        self.closeButton.setEnabled(False)
        self.comNameCombo.setEnabled(True)
        self.baudCombo.setEnabled(True)
        self.sendButton.setEnabled(False)
        self.comStatus.setText('串口状态：%s  关闭' % self.com.portName())
        if self.sendTimer.isActive():
            self.sendTimer.stop()
```

## 定时发送check槽函数
在串口打开的情况下修改定时发送定时器

```python
    def on_timerSendChecked(self):
        if self.com.isOpen():
            if self.timerSendCheck.isChecked():
                self.sendTimer.start(int(self.timerPeriodEdit.text()))
            else:
                self.sendTimer.stop()
        return
```
## 发送函数
1. 将输入框中字符转为字符串
2. 如果选中十六进制发送
    - 去掉所有的空格
    - 字符个数必须为偶数个
    - 字符只能包含`0-9a-zA-Z`
    - 使用`binascii.a2b_hex(s)`语句将其转化为十六进制的二进制字符串,例如:
        - `'aabb'`转化为`b'\xaa\xbb'`
3. 如果不是十六进制发送,则将发送内容按照选定的编码进行编码再发送
4. 更新发送计数器
 
```python
    def on_sendData(self):
        txData = self.inputEdit.toPlainText()
        if len(txData) == 0:
            return
        if self.hexSendingCheck.isChecked():

            s = txData.replace(' ', '')
            if len(s) % 2 == 1:  # 如果16进制不是偶数个字符,去掉最后一个
                QMessageBox.critical(self, '错误', '十六进制数不是偶数个')
                return
            pattern = re.compile('[^0-9a-fA-F]')
            r = pattern.findall(s)
            if len(r) != 0:
                QMessageBox.critical(self, '错误', '包含非十六进制数')
                return
            try:
                hexData = binascii.a2b_hex(s)
            except:
                QMessageBox.critical(self, '错误', '转换编码错误')
                return

            try:
                n = self.com.write(hexData)
            except:
                QMessageBox.critical(self, '异常', '十六进制发送错误')
                return
        else:
            n = self.com.write(txData.encode(self.encoding, "ignore"))
        self.sendCount += n
```

## 接收函数
1. Qt串口接收的数据为`QByteArray`格式, 先将其转为python的`bytes`,
2. 如果接收字符数量大于0, 更新接收计数器
3. 如果没有停止显示, 且没有设置十六进制显示,则按照设置的编码将接收数据进行解码并显示
4. 如果没有停止显示,且设置十六进制显示,则
    - 使用`binascii.b2a_hex(receivedData)`函数将十六进制的二进制字符串转为普通二进制字符串,再使用`decode`按照`ascii`进行解码,例如
        - `b'\xaa\xbb'`通过`binascii.b2a_hex`函数转化为`b'aabb'`
        - `b'aabb'`使用`decode('ascii')`解码为`'aabb'`
    - 使用正则表达式将字符串每两个字符后插入一个空格
    
```python
    def on_receiveData(self):
        try:
            '''将串口接收到的QByteArray格式数据转为bytes,并用gkb或utf8解码'''
            receivedData = bytes(self.com.readAll())
        except:
            QMessageBox.critical(self, '严重错误', '串口接收数据错误')
        if len(receivedData) > 0:
            self.receiveCount += len(receivedData)
            if self.stopShowingButton.text() == '停止显示':
                if self.hexShowingCheck.isChecked() == False:
                    receivedData = receivedData.decode(self.encoding, 'ignore')
                    self.receivedDataEdit.insertPlainText(receivedData)
                else:
                    data = binascii.b2a_hex(receivedData).decode('ascii')
                    pattern = re.compile('.{2,2}')
                    hexStr = ' '.join(pattern.findall(data)) + ' '
                    self.receivedDataEdit.insertPlainText(hexStr.upper())
```

## 十六进制发送check槽函数
选择十六进制发送后,将输入框中的字符转为答谢字符

```python
    def on_hexSendingChecked(self):
        if self.hexSendingCheck.isChecked():
            data = self.inputEdit.toPlainText().upper()
            self.inputEdit.clear()
            self.inputEdit.insertPlainText(data)
```

## 保存接收数据
使用`Qt`的`QFileDialog`标准文件对话框, 将接收到的数据保存至选择的文件

```python
    def on_saveReceivedData(self):
        fileName, fileType = QFileDialog.getSaveFileName(
            self, '保存数据', 'data', "文本文档(*.txt);;所有文件(*.*)")
        print('Save file', fileName, fileType)

        writer = QTextDocumentWriter(fileName)
        writer.write(self.receivedDataEdit.document())
```

## 关于对话框
使用`Qt`的`QMessageBox`生成版本信息
```python
    def on_aboutButton(self):
        QMessageBox.about(self, ABOUT_TITLE, ABOUT_STRING)

    def on_aboutPyQtButton(self):
        QMessageBox.aboutQt(self)
```  
    
# 文件打包
文件保存为`myPySerial.py`,使用`pyinstaller`来打包为单个exe文件,命令说明如下:
 - -F: 打包为单一文件
 - -w: 使用窗口,无控制台
 - -p: 搜索路径


```bash
pyinstaller -F -w myPySerial.py -p D:\Program\Python35;D:\Program\Python35\Lib\site-packages\PyQt5\Qt\bin
```

运行之后,在当面路径下生成`dist`文件夹, 其下有`myPySerial.exe`文件,运行界面如下:
![](http://oedcidkmi.bkt.clouddn.com/myPySerial.jpg)

# 最终文件
Github Repository:[myPySerial](https://github.com/fitsir/myPySerial)
代码:[myserial_GUI.py](https://github.com/fitsir/myPySerial/blob/master/myPySerial.py)

