### 第1关：基本测试

题目要求:根据S-DES算法编写和调试程序，提供GUI解密支持用户交互。输入可以是8bit的数据和10bit的密钥，输出是8bit的密文。

代码

#### src_func.py 核心代码函数实现

```python
# 定义置换表和S盒
P10 = [3,5,2,7,4,10,1,9,8,6]
P8 = [6,3,7,4,8,5,10,9]
IP = [2,6,3,1,4,8,5,7]
IP_inverse = [4,1,3,5,7,2,8,6]
E_P = [4,1,2,3,2,3,4,1]
P4 = [2,4,3,1]
S0 = [[1,0,3,2],[3,2,1,0],[0,2,1,3],[3,1,0,2]]
S1 = [[0,1,2,3],[2,0,1,3],[3,0,1,2],[2,1,0,3]]

# 置换函数
def permute(k, arr, n):
    per = []
    for i in range(0,n):
        per.append(k[arr[i]-1])  # 根据arr中的位置信息进行置换
    return per

# 左移函数
def shift(l, n):
    s = l[n:] + l[:n]  # 左移n位
    return s

# 密钥生成函数
def keygen(key):
    key = permute(key, P10, 10)  # 第一次置换
    key = shift(key, 1)  # 左移
    key1 = permute(key, P8, 8)  # 第二次置换得到K1
    key = shift(key, 2)  # 再次左移
    key2 = permute(key, P8, 8)  # 第三次置换得到K2
    return key1, key2

# 轮函数F
def F(R, key):
    t = permute(R, E_P, 8)  # 扩展置换
    t = [t[i] ^ key[i] for i in range(8)]  # 与子密钥异或
    t = t[:4], t[4:]  # 分为两半
    row = int(str(t[0][0])+str(t[0][3]),2)  # 计算S盒的行
    col = int(''.join([str(x) for x in t[0][1:3]]),2)  # 计算S盒的列
    val = bin(S0[row][col])[2:].zfill(2)  # 从S盒中获取值
    row = int(str(t[1][0])+str(t[1][3]),2)
    col = int(''.join([str(x) for x in t[1][1:3]]),2)
    val += bin(S1[row][col])[2:].zfill(2)
    val = [int(x) for x in str(val)]
    val = permute(val, P4, 4)  # P4置换
    return val



# 加密函数
def encrypt(pt, keys):
    pt = permute(pt, IP, 8)  # 初始置换
    L, R = pt[:4], pt[4:]  # 分为左右两半
    temp = R
    # 计算轮函数F的结果，然后再与左半部分异或
    f = F(temp, keys[0])
    L = [L[i] ^ f[i] for i in range(4)]
    # 交换左右两部分
    L, R = R, L
    temp = R
    # 计算轮函数F的结果，然后再与左半部分异或
    f = F(temp, keys[1])
    L = [L[i] ^ f[i] for i in range(4)]
    pt = permute(L+R, IP_inverse, 8)  # 进行逆初始置换
    return pt

# 解密函数
def decrypt(pt, keys):
    pt = permute(pt, IP, 8)  # 初始置换
    L, R = pt[:4], pt[4:]  # 分为左右两半
    temp = R
    # 计算轮函数F的结果，然后再与左半部分异或
    f = F(temp, keys[1])
    L = [L[i] ^ f[i] for i in range(4)]
    # 交换左右两部分
    L, R = R, L
    temp = R
    # 计算轮函数F的结果，然后再与左半部分异或
    f = F(temp, keys[0])
    L = [L[i] ^ f[i] for i in range(4)]
    pt = permute(L+R, IP_inverse, 8)  # 进行逆初始置换
    return pt

def ascii_to_bin(text):
    return ''.join(format(ord(c), '08b') for c in text)

def bin_to_ascii(binary):
    return ''.join(chr(int(binary[i:i+8], 2)) for i in range(0, len(binary), 8))


if __name__ == '__main__':

    # 测试
    key = '1010000010'
    pt = '01110010'
    key = [int(i) for i in key]
    pt = [int(i) for i in pt]
    keys = keygen(key)  # 生成子密钥K1和K2
    cipher = encrypt(pt, keys)  # 加密
    print('加密结果: ', ''.join([str(i) for i in cipher]))
    plain = decrypt(cipher, keys)  # 解密
    print('解密结果: ', ''.join([str(i) for i in plain]))

```

#### gui.py  图形化界面代码实现

```python
# -*- coding: utf-8 -*-

from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QTextEdit, QLineEdit, QLabel, QComboBox
from src_func import *

class MyApp(QWidget):
    def __init__(self):
        super().__init__()

        self.input_type = QComboBox()
        self.input_type.addItem("二进制")
        self.input_type.addItem("ASCII")

        self.encrypt_text = QLineEdit()
        self.encrypt_key = QLineEdit()
        self.decrypt_text = QLineEdit()
        self.decrypt_key = QLineEdit()
        self.result_box = QTextEdit()

        self.init_ui()

    def init_ui(self):
        layout = QVBoxLayout()

        layout.addWidget(QLabel("输入类型:"))
        layout.addWidget(self.input_type)

        layout.addWidget(QLabel("加密信息:"))
        layout.addWidget(self.encrypt_text)
        layout.addWidget(QLabel("密钥:"))
        layout.addWidget(self.encrypt_key)
        encrypt_button = QPushButton("加密")
        encrypt_button.clicked.connect(self.encrypt_clicked)
        layout.addWidget(encrypt_button)

        layout.addWidget(QLabel("解密信息:"))
        layout.addWidget(self.decrypt_text)
        layout.addWidget(QLabel("密钥:"))
        layout.addWidget(self.decrypt_key)
        decrypt_button = QPushButton('解密')
        decrypt_button.clicked.connect(self.decrypt_clicked)
        layout.addWidget(decrypt_button)

        layout.addWidget(QLabel("结果:"))
        layout.addWidget(self.result_box)

        self.setLayout(layout)
        self.setWindowTitle('S-DES Encryptor/Decryptor')
        self.show()

    def encrypt_clicked(self):
        pt = self.encrypt_text.text()
        key = self.encrypt_key.text()

        if not key or len(key) != 10 or not all(c in '01' for c in key):
            self.result_box.setPlainText('错误: 密钥必须是10位0或1组成的字符串')
            return

        keys = keygen([int(i) for i in key])

        if self.input_type.currentText() == "二进制":
            if not pt or len(pt) != 8 or not all(c in '01' for c in pt):
                self.result_box.setPlainText('错误: 明文必须是8位0或1组成的字符串')
                return
            pt = [int(i) for i in pt]
            cipher = encrypt(pt, keys)
            self.result_box.setPlainText('加密结果: ' + ''.join([str(i) for i in cipher]))
        else:
            cipher_text = ""
            for char in pt:
                bin_char = [int(i) for i in format(ord(char), '08b')]
                cipher = encrypt(bin_char, keys)
                cipher_text += chr(int(''.join([str(i) for i in cipher]), 2))
            self.result_box.setPlainText('加密结果: ' + cipher_text)

    def decrypt_clicked(self):
        ct = self.decrypt_text.text()
        key = self.decrypt_key.text()

        if not key or len(key) != 10 or not all(c in '01' for c in key):
            self.result_box.setPlainText('错误: 密钥必须是10位0或1组成的字符串')
            return

        keys = keygen([int(i) for i in key])

        if self.input_type.currentText() == "二进制":
            if not ct or len(ct) != 8 or not all(c in '01' for c in ct):
                self.result_box.setPlainText('错误: 密文必须是8位0或1组成的字符串')
                return
            ct = [int(i) for i in ct]
            plain = decrypt(ct, keys)
            self.result_box.setPlainText('解密结果: ' + ''.join([str(i) for i in plain]))
        else:
            plain_text = ""
            for char in ct:
                bin_char = [int(i) for i in format(ord(char), '08b')]
                plain = decrypt(bin_char, keys)
                plain_text += chr(int(''.join([str(i) for i in plain]), 2))
            self.result_box.setPlainText('解密结果: ' + plain_text)


if __name__ == '__main__':
    import sys
    app = QApplication(sys.argv)
    ex = MyApp()
    sys.exit(app.exec_())

```

#### brute_force.py 暴力破解代码实现

```python
# -*- coding: utf-8 -*-
import time
from src_func import keygen , encrypt
def brute_force(plain_text, cipher_text):
    start_time = time.time()
    for i in range(1024):  # 遍历所有可能的密钥
        key = format(i, '010b')  # 将i转换为10位二进制字符串
        keys = keygen([int(bit) for bit in key])  # 生成子密钥
        cipher = encrypt([int(bit) for bit in plain_text], keys)  # 加密
        if ''.join(map(str, cipher)) == cipher_text:  # 如果加密结果和已知的密文匹配
            print(f"Found the key: {key}, time elapsed: {time.time() - start_time} seconds")
            return key
    print(f"No key found, time elapsed: {time.time() - start_time} seconds")
    return None

if __name__ == '__main__':
    plain_text = '01110010'
    cipher_text = '01110111'
    brute_force(plain_text, cipher_text)

```





#### 测试

8bit的数据加解密功能



#### 测试记录

密钥1010000010

信息01110010

加密结果: 11110110

解密结果: 01110010

![image-20231007172433889](./pic_typora/image-20231007172433889.png)

![image-20231007172444898](./pic_typora/image-20231007172444898.png)

通过测试



### 第2关：交叉测试



![image-20231007204257063](./pic_typora/image-20231007204257063.png)

交叉测试结果与上一关中的结果相同



### 第3关：扩展功能

加密信息:qwe

密钥:1110011110

加密结果: É	

解密信息:É	

解密结果: qwe

![image-20231007202157282](./pic_typora/image-20231007202157282.png)

![image-20231007202238617](./pic_typora/image-20231007202238617.png)

通过测试

### 第4关：暴力破解

![image-20231007202442982](./pic_typora/image-20231007202442982.png)

用时

0.0009996891021728516 seconds

### 第5关：封闭测试

在理想情况下，一个好的加密算法应该是一个双射函数，也就是说，对于明文空间中的每一个元素，都存在一个唯一的密钥使其映射到密文空间中的一个元素，反之亦然。这意味着，对于给定的一对明文和密文，应该只存在一个密钥能够将明文加密为密文。然而，在实际的加密算法中，可能存在多个密钥能够将同一明文加密为同一密文，尤其是在加密算法的设计或实现存在缺陷，或者当密钥空间小于明文空间时。

对于S-DES加密算法，由于它的密钥空间（1024）大于明文空间（256，因为明文是8位二进制），所以理论上是可能存在多个密钥将同一明文加密为同一密文的情况。然而，由于S-DES的设计，它试图使每一个密钥都产生不同的映射，所以在实践中，找到这样的密钥对可能会很困难。

如果你在暴力破解过程中找到了多个密钥，那么可能是因为S-DES的实现存在问题，或者是由于输入数据的特定性质。例如，如果明文是全0或全1，那么可能存在多个密钥能够将其加密为同一密文。

总的来说，对于任意给定的明文，找到多个使其加密为同一密文的密钥是可能的，但不常见。这种情况通常被视为加密算法的弱点，因为它可能被用于密码分析攻击。
