# python-
# -*- coding: utf-8 -*-
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time
import random

# 设置请求头，模拟浏览器行为
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ' +
                  'AppleWebKit/537.36 (KHTML, like Gecko) ' +
                  'Chrome/112.0.0.0 Safari/537.36'
}


def get_movie_data(start=0):
    """
    获取豆瓣电影Top250中从start开始的25部电影数据
    """
    url = f'https://movie.douban.com/top250?start={start}&filter='
    response = requests.get(url, headers=headers)

    if response.status_code != 200:
        print(f"请求失败，状态码：{response.status_code}")
        return []

    soup = BeautifulSoup(response.text, 'html.parser')
    movies = soup.find_all('div', class_='item')

    data = []
    for movie in movies:
        # 排名
        rank = movie.find('em').text.strip()

        # 标题
        title = movie.find('span', class_='title').text.strip()

        # 信息
        info = movie.find('div', class_='bd').p.text.strip().replace('\n', ' ').replace('    ', ' ')

        # 评分
        rating = movie.find('span', class_='rating_num').text.strip()

        # 评价人数
        comment = movie.find('div', class_='star').find_all('span')[-1].text.strip()

        # 简介
        quote = movie.find('p', class_='quote')
        quote_text = quote.find('span').text.strip() if quote else ''

        data.append({
            '排名': rank,
            '标题': title,
            '信息': info,
            '评分': rating,
            '评价人数': comment,
            '简介': quote_text
        })

    return data


def main():
    all_data = []
    for start in range(0, 250, 25):
        print(f"正在获取第 {start // 25 + 1} 页数据...")
        data = get_movie_data(start)
        all_data.extend(data)
        # 随机延时，避免被封
        time.sleep(random.uniform(1, 3))

    # 创建DataFrame
    df = pd.DataFrame(all_data)

    # 导出到Excel
    df.to_excel('douban_top250.xlsx', index=False)
    print("数据已成功导出到douban_top250.xlsx")


if __name__ == '__main__':
    main()
