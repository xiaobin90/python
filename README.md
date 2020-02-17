# python day02
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
