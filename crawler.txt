import urllib.request

import pymysql
from bs4 import BeautifulSoup
from jupyter_server.auth import passwd

start = 0
for i in range(0,11):

     connection = pymysql.connect(host='localhost',
                             user='root',
                             password='0319',
                             database='spider2202_douban',
                             cursorclass=pymysql.cursors.DictCursor)
     h={"user-agent":"Mozilla/5.0(Windows NT 10.0;win64;x64)"
                "AppleWebKit/537.36(KHTML,like Gecko)Chrome/128.0.0.0 Safari/537.36"}  #键值对
     req= urllib.request.Request(f'https://movie.douban.com/top250?start={start}&filter=',headers=h)
     r=urllib.request.urlopen(req) #传Request对象

#使用常规获取
#print(r.read().decode())

#使用bs4库获取item
     soup=BeautifulSoup(r.read().decode(),'html.parser')

     items =soup.find_all("div",class_="item") #获取第一页每个电影的div,div中的item

#print(items)
     with connection:
          for item in items:
              pic_div =item.find("div",class_="pic")
              img= pic_div.a.img
              name = img['alt']  #取出img里的alt,电影名
              url = img['src']   #网址
    #print(img['alt'])
   #print(img['src'])
              hd = item.find("div", class_="hd")  # zong
              English = hd.find_all("span")  # 找出hd下的所以span标签
              rename = English[1].text
              other = hd.find("span", class_="other").text

              bd = item.find("div", class_="bd")  # zong
              director = item.find("div", class_="bd").p.text.strip()  # 导演

              star = bd.find("div", class_="star")
              grade = star.find("span", class_="rating_num").text  # 评分

              comments = star.find_all("span")
              comment = comments[3].text  # 索引从0开始
              summary = bd.find("span", class_="inq")  # 简介

              if summary is not None:
                  summary = bd.find("span", class_="inq").text
              else:
                  summary = bd.find("span", class_="inq")

          # 存储数据
              with connection.cursor() as cursor:  # 游标
                  sql = ("INSERT INTO `movie_douban`(`movie_name`,`movie_url`,`movie_rename`,`movie_director`,"
                         "`movie_grade`,`movie_comment`,`movie_summary`,`movie_other`) VALUES (%s,%s,%s,%s,%s,%s,%s,%s)")
                  cursor.execute(sql, (name, url, rename, director, grade, comment, summary,other))
          connection.commit()

     start = start + 25  # 每个页面的start加25
     '''
      https://movie.douban.com/top250?start=25&filter=
      https://movie.douban.com/top250?start=50&filter=
      https://movie.douban.com/top250?start=75&filter=
      '''

