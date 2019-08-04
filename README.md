# Scrapy-TiebaImage
使用requests + BeautifulSoup爬取百度贴吧图片数据
## demo 说明
```python
from bs4 import BeautifulSoup

import requests

import os

# http://tieba.baidu.com/f?kw=%E7%BE%8E%E9%A3%9F&ie=utf-8&pn=50

class TiebaSpider:

    def __init__(self):

        #初始化路由
        self.base_url = "http://tieba.baidu.com/f?"

        #构造请求头
        self.headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko"}


    def send_request(self, url):

        #发送请求
        html = requests.get(url, headers=self.headers)

        return html.content


    def extract_tiezi_url(self, tieba_html):
        """
        获取帖子路由
        :param tieba_html: html
        :return: 路由
        """
        #构造对象
        html_obj = BeautifulSoup(tieba_html,'lxml')

        #class选择器解析
        html_be = html_obj.find_all('a',class_='j_th_tit')


        urls = []

        for each in html_be:

            #添加每个帖子路由
            urls.append(each.attrs['href'])

        #print(urls)

        #构造完整路由返回
        return ["http://tieba.baidu.com" + path for path in urls]


    def extract_img_url(self, tiezi_html):
        """
        获取图片url
        :param tiezi_html: 帖子html
        :return: 图片路由
        """
        #构造对象
        html_obj = BeautifulSoup(tiezi_html, 'lxml')

        #通过选择器提取图片节点
        html_be = html_obj.find_all('img', class_='BDE_Image')

        urls = []

        for each in html_be:

            #添加图片路由
            urls.append(each.attrs['src'])

        #返回图片路由
        return urls


    def change_path(self):
        """
        更改路径
        :return:
        """
        path1 = "/home/python/Pictures"

        if not os.path.exists(path1):

            os.mkdir(path1)

        else:

            print("目录已存在")

        os.chdir(path1)


    def write_file(self, file_content, file_name):

        """保存图片"""

        with open(file_name, "wb") as f:

            print("正在保存 %s..." % file_name)

            f.write(file_content)


    def start(self, tieba_name, page_count):

        # 拼接url
        for page in range(1, page_count + 1):

            query_str = "kw=%s&ie=utf-8&pn=%d" % (tieba_name, (page-1)*50)

            tieba_url = self.base_url + query_str

            # 请求贴吧url
            tieba_html = self.send_request(tieba_url).decode("utf-8")

            # 提取帖子url
            tiezi_urls = self.extract_tiezi_url(tieba_html)

            # 请求帖子url
            for tiezi_url in tiezi_urls:

                tiezi_html = self.send_request(tiezi_url).decode("utf-8")

                # 提取图片url
                img_urls = self.extract_img_url(tiezi_html)

                # 请求图片url
                for img_url in img_urls:

                    img_bytes = self.send_request(img_url)

                    # 保存文件
                    self.write_file(img_bytes, img_url[-10:])




if __name__ == '__main__':

    tieba_n = input("请输入贴吧名:")

    page_c = int(input("请输入爬取页数:"))

    spider = TiebaSpider()

    spider.change_path()

    spider.start(tieba_n, page_c)
```
