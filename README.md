day07
1.python 
#编写记账本，要求账户初始10000块
import os
import pickle
from time import strftime
#用于记录收入
def save(fname):
    date = strftime('%Y-%m-%d')
    jin_e = int(input('金额: '))
    shuoming = input('备注: ')
    # 取出所有的记账内容
    with open(fname, 'rb') as fobj:
        data = pickle.load(fobj)
    yu_e = data[-1][-2] + jin_e
    # 构建最新一笔收入
    line = [date, jin_e, 0, yu_e, shuoming]
    data.append(line)
    # 将记账内容写回文件
    with open(fname, 'wb') as fobj:
        pickle.dump(data, fobj)
#用于记录支出
def cost(fname):
    date = strftime('%Y-%m-%d')
    jin_e = int(input('金额: '))
    shuoming = input('备注: ')
    # 取出所有的记账内容
    with open(fname, 'rb') as fobj:
        data = pickle.load(fobj)
    yu_e = data[-1][-2] - jin_e
    # 构建最新一笔支出
    line = [date, 0, jin_e, yu_e, shuoming]
    data.append(line)
    # 将记账内容写回文件
    with open(fname, 'wb') as fobj:
        pickle.dump(data, fobj)
#用于查帐
def query(fname):
    #取出数据
    with open(fname,'rb') as fobj:
        data = pickle.load(fobj)
    #打印表头
    print('%-12s%-8s%-8s%-12s%-20s' % ('date', 'save', 'cost', 'balance', 'comment'))
    for line in data:
        print('%-12s%-8s%-8s%-12s%-20s' % tuple(line))
#用于展示菜单
def show_menu():
    cmds = {'0': save, '1': cost, '2': query}
    prompt ='''(0)收入
(1)开销
(2)查询
(3)退出
请选择(0/1/2/3):'''
    fname = 'account.data'
#判断记账文件是否存在，如果不存在，就初始化它
    chushi = [[strftime('%Y-%m-%d'), 0, 0, 10000, 'init data']]
    if not os.path.exists(fname):
        with open(fname, 'wb') as fobj:
            pickle.dump(chushi, fobj)
    while 1:
        xuan = input(prompt).strip()
        if xuan not in ['0', '1', '2','3']:
            print('无效输入,请重试！')
            continue
        if xuan == '3':
            print('\nBye-bye')
            break
        cmds[xuan](fname)
if __name__ == '__main__':
    show_menu()
2.Python
#给用户出3道100以内加减法运算，答错三次就给出正确答案
from random import randint, choice

def exam():
    '用于出题，让用户作答'
    cmds = {'+': lambda x, y: x + y, '-': lambda x, y: x - y}
    # 随机选出两个数字并降序排列
    nums = [randint(1, 100) for i in range(2)]
    nums.sort(reverse=True)
    # 随机选出加减法
    op = choice('+-')

    # 做出正确答案
    # result = cmds[op](nums[0], nums[1])
    result = cmds[op](*nums)

    # 要求用户作答，并判断
    counter = 0   # 计数器，用于计录用户作答次数
    prompt = '%s %s %s = ' % (nums[0], op, nums[1])
    while counter < 3:
        try:
            answer = int(input(prompt))
        except:   # except后面不指定具体的异常，可以捕获所有异常，不推荐
            print()
            continue

        if answer == result:
            print('不错哟！！！')
            break
        print('不对哟！！！')
        counter += 1
    else:
        print('%s%s' % (prompt, result))

def main():
    '类似于以前的show_menu，用于展示主程序代码'
    while 1:
        exam()
        # 将用户输入的字符串去掉两端空格后，取第一个字符
        try:
            yn = input('Continue(y/n)? ').strip()[0]
        except IndexError:
            continue
        except (KeyboardInterrupt, EOFError):
            yn = 'n'  # 当用户按下ctrl+c或ctrl+d时，算他输入n

        if yn in 'nN':
            print('\nBye-bye')
            break

if __name__ == '__main__':
    main()
    
    
4.Python
#快速排序qsort
from random import randint

def qsort(seq):
    '快速排序'
    # 序列长度小于2，不用排序，直接返回
    if len(seq) < 2:
        return seq

    # 假设第一个值是中间值，比它小的和大的各放到一个列表中
    m = seq[0]
    s = []
    l = []
    for data in seq[1:]:
        if data <= m:
            s.append(data)
        else:
            l.append(data)

    return qsort(s) + [m] + qsort(l)

if __name__ == '__main__':
    nums = [randint(1, 100) for i in range(10)]
    print(nums)
    print(qsort(nums))
    
day08
1.Python
#md5值校验
import hashlib
import sys
利用hashlib模块实现文件的md5值校验
def check_md5(fname):
    m = hashlib.md5()

    with open(fname, 'rb') as fobj:
        while 1:
            data = fobj.read(4096)
            if not data:
                break
            m.update(data)

    return m.hexdigest()

if __name__ == '__main__':
    print(check_md5(sys.argv[1]))


2.Python
#利用tarfile,hashlib等模块进行文件的全量备份和增量备份
# mkdir /tmp/demo
# cp -r /etc/security/ /tmp/demo
# mkdir /tmp/demo/backup
import os
import tarfile
import hashlib
import pickle
from time import strftime

def check_md5(fname):
    m = hashlib.md5()

    with open(fname, 'rb') as fobj:
        while 1:
            data = fobj.read(4096)
            if not data:
                break
            m.update(data)

    return m.hexdigest()

def full_backup(src, dst, md5file):
    # 拼接出备份文件的绝对路径
    fname = '%s_full_%s.tar.gz' % (os.path.basename(src), strftime('%Y%m%d'))
    fname = os.path.join(dst, fname)

    # 打包压缩
    tar = tarfile.open(fname, 'w:gz')
    tar.add(src)
    tar.close()

    # 计算每个文件的md5值，保存到字典中
    md5dict = {}
    for path, folders, files in os.walk(src):
        for file in files:
            k = os.path.join(path, file)
            md5dict[k] = check_md5(k)

    # 将md5字典保存到文件中
    with open(md5file, 'wb') as fobj:
        pickle.dump(md5dict, fobj)

def incr_backup(src, dst, md5file):
    # 拼接出备份文件的绝对路径
    fname = '%s_incr_%s.tar.gz' % (os.path.basename(src), strftime('%Y%m%d'))
    fname = os.path.join(dst, fname)

    # 计算每个文件的md5值，保存到字典中
    md5dict = {}
    for path, folders, files in os.walk(src):
        for file in files:
            k = os.path.join(path, file)
            md5dict[k] = check_md5(k)

    # 取出前一天的md5值
    with open(md5file, 'rb') as fobj:
        old_md5 = pickle.load(fobj)

    # 字典的key是文件名，key在今天有，昨天没有，就是新增的文件, 文件的md5值不一样，就是改动的文件
    # 新增的文件和改动的文件需要备份
    tar = tarfile.open(fname, 'w:gz')
    for k in md5dict:
        if old_md5.get(k) != md5dict[k]:
            tar.add(k)
    tar.close()

    # 更新md5文件
    with open(md5file, 'wb') as fobj:
        pickle.dump(md5dict, fobj)

if __name__ == '__main__':
    src = '/tmp/demo/security'
    dst = '/tmp/demo/backup'
    md5file = '/tmp/demo/backup/md5.data'
    if strftime('%a') == 'Mon':
        full_backup(src, dst, md5file)
    else:
        incr_backup(src, dst, md5file)
3.python
opp 
class Role:
    def __init__(self, name, wuqi, guojia):
        '称作构造器方法，创建实例时，自动调用。通常用于将属性绑定到实例。'
        self.name = name
        self.wuqi = wuqi
        self.guojia = guojia

    def show_me(self):
        # 绑定在实例身上的属性，可以在类中的任意位置使用
        print('我是%s，擅长使用%s' % (self.name, self.wuqi))

    def speak(self, words):
        # 没有绑定到实例的属性，就是函数的局部变量，只能在函数内使用
        print(words)

class ZhanShi(Role):  # 括号中是父类，也叫基类
    # 子类将会继承父类的所有方法
    def __init__(self, name, wuqi, guojia, zuoji):
        # Role.__init__(self, name, wuqi, guojia)  # 调用父类方法
        super(ZhanShi, self).__init__(name, wuqi, guojia)  # 调用父类方法，与上一行一样
        self.zuoji = zuoji

    def attack(self, target):
        print('正在近身与%s肉搏' % target)

class FaShi(Role):
    def attack(self, target):
        print('远程打击%s' % target)

if __name__ == '__main__':
    lb = ZhanShi('吕布', '方天画戟', '汉', '赤兔马')
    # lb.show_me()
    # lb.speak('马中赤兔，人中吕布')
    # lb.attack('张飞')
    # km = FaShi('孔明', '羽扇')
    # km.show_me()
    # km.speak('三分天下有其一')
    # km.attack('司马懿')
