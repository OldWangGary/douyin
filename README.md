# douyin

import requests
import json
import time
import os

os.chdir(os.path.dirname(os.path.realpath(__file__)))
def get_data(sec_uid,max_cursor,mode):
    headers = {
        'Connection': 'keep-alive',
        'Accept': 'application/json, text/plain, */*',
        'Agw-Js-Conv': 'str',
        'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Mobile Safari/537.36',
        'Sec-Fetch-Site': 'same-origin',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Dest': 'empty',
        'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
    }
    params = {
        'reflow_source': 'reflow_page',
        'sec_uid': sec_uid,
        'count': '100',
        'max_cursor': max_cursor,
    }
    response = requests.get(f'https://m.douyin.com/web/api/v2/aweme/{mode}/', params=params, headers=headers)
    data = response.json()
    for d in data['aweme_list']:
        output = {}
        output['title'] = d['desc']
        output['VideoUrl'] = d['video']['play_addr']['url_list'][-1]
        output['img'] = d['video']['dynamic_cover']['url_list'][-1]
        output['id'] = d['aweme_id']
        result.append(output)
        print(output['title'])
    return data['max_cursor'],data['has_more']

def run(sec_uid,mode='post'):
    max_cursor = '0'
    for i in range(100):
        x = get_data(sec_uid,max_cursor,mode)
        if x[1]:
            max_cursor = x[0]
            get_data(sec_uid,max_cursor,mode)
        else:
            break

if __name__ == '__main__':
    result = []
    choice = input('''
*******************************************************
    请选择需要下载的类别序号（1/2）：
    1、该账号主页的所有视频；
    2、改账号喜欢列表所有视频。
*******************************************************\n
    ''')
    sec_uid = input('''
*******************************************************
    请选择需账号的sec_uid码：
    例如：主页链接（PC网页打开）
    链接：https://www.douyin.com/user/MS4wLjABAAAAfeHUJALUV_hro9kN7QT5I9pe9DNVDSkiCTiqfK0ziZo?vid=7151405922777107753
    sec_uid码：MS4wLjABAAAAfeHUJALUV_hro9kN7QT5I9pe9DNVDSkiCTiqfK0ziZo
*******************************************************\n''')
    sel = {'1':'post','2':'like'}
    mode = sel[choice]
    path = f'data.json'
    try:        
        run(sec_uid,mode)
        print('完成搜索数量：',len(result))
        with open(path,'w',encoding='utf-8') as f:
            json.dump(result,f,indent=4, ensure_ascii=False)
    except:
        print('完成搜索数量(lost_part)：',len(result))
        with open(path,'w',encoding='utf-8') as f:
            json.dump(result,f,indent=4, ensure_ascii=False)
    time.sleep(3)
