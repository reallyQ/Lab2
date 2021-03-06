.. Mr.Lan Lab2 documentation master file, created by
   sphinx-quickstart on Wed Dec 15 22:35:26 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to Mr.Lan Lab2's documentation!
=======================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

**小组成员**

夏金明 - 201832110145 - 736001469@qq.com

邱政淞 - 201932110132 - 1712590429@qq.com

方世玉 - 201932110117 - 874524395@qq.com



**项目GitHub地址**: `BluePrint`_.

.. _BluePrint: https://github.com/reallyQ/Lab2


**项目演示视频**: `Bilibili`_.

.. _Bilibili: https://www.bilibili.com/video/BV1LQ4y1v7Pg/


Abstract
=================================================

Use blueprints to organize a web application.


Introduction
=================================================

Download the source code of Photo String .

Make the following blueprints: upload bp, show bp, search bp, and api bp.

Register the above blueprints to the web application.

The upload bp blueprint allows uploading a new photo. The associated route is /upload.

The show bp blueprint allows displaying all photos and their descriptions in chronological order. The
associated route is /show.

The search bp blueprint allows filtering photos according to their descriptions. The associated route
is /search/query-string. Only the photos whose descriptions match query-string will be returned
as the search result.

The api bp blueprint allows us to get all photo information in JSON format from command-line.
HTTPie is a useful API testing tool. The associated route is /api/json. The returned json string
must contain photo ID, date of upload, photo size (in KB) and photo description for each photo.


Methods and materials
=================================================

**blueprint**

把不同功能的module分开。可以让应用模块化，针对大型应用。
蓝图的基本概念：在蓝图被注册到应用之后，所要执行的操作的集合。当分配请求时， Flask 会把蓝图和视图函数关联起来，并生成两个端点之前的 URL 。
比如只有一个run.py。有些功能需要多人分开来写，有些功能会有交错的可能，代码位置也不会在一起，这样可以用蓝图来开关一些模块（功能）和宏定义类似，但不是可插拔的应用而是一套可以注册在应用中的操作，并且可以注册多次。或者简单滴需要降低耦合，提高模块复用率。比如开发环境和应用环境的不同，可以用蓝图来切换环境。
蓝图的缺点是一旦应用被创建后，只有销毁整个应用对象才能注销蓝图。

首先在子文件夹init文件中定义蓝图，比如app/auth.py中
::
   from flask import Blueprint
   auth= Blueprint('auth',__name__)
   # 蓝图包含的内容


然后在工厂方法create_app()中注册蓝图，比如app/init.py中：
::
   from auth import auth as auth_blueprint
   app.register_blueprint(auth_blueprint,url_prefix='/auth',statil_folder='static')



code
=================================================



**Lab.py**
::
      from flask import Flask, request,Blueprint
      from UseSqlite import InsertQuery
      from datetime import datetime
      from upload_bp import upload_bp
      from show_bp import show_bp
      from search_bp import search_bp
      from api_bp import api_bp

      app=Flask(__name__)

      app.register_blueprint(upload_bp)
      app.register_blueprint(show_bp)
      app.register_blueprint(search_bp)
      app.register_blueprint(api_bp)


      @app.route('/')
      def main():
            page='''
      <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
      <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js" />
      <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
      <!DOCTYPE  html>
      <html lang="en">
      <body> 
         <div class="container">
            <center>
            <h1>Welcome To Use PhotoString</h1>
            <a href='/upload'><button type="button" class="btn btn-prinary btn-lg" >upload</button></a>
            <a href='/show'><button type="button" class="btn btn-prinary btn-lg">show</button></a>
            <a href='/search'><button type="button" class="btn btn-prinary btn-lg">search</button></a>
            <a href='/api'><button type="button" class="btn btn-prinary btn-lg" >api</button></a>
            </center>
            </div>

         </body>
      </html>'''
            return page
         
      if __name__=='__main__':
         print(app.url_map)
         app.run(debug=True) 




**upload_bp.py**
::
      from flask import Blueprint,render_template,request
      from datetime import datetime
      from UseSqlite import InsertQuery

      upload_bp= Blueprint('upload_bp',__name__)
      

      @upload_bp.route("/upload/",methods=['POST','GET'])
      def print():
         if request.method=='POST':
            uploaded_file=request.files['file']
            time_str=datetime.now().strftime('%Y%m%d%H%M%S')
            new_filename=time_str+'.jpg'
            uploaded_file.save('./static/upload/'+new_filename)
            time_info=datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            description=request.form['description']
            path='./static/upload/'+new_filename
            iq=InsertQuery('./static/RiskDB.db')
            iq.instructions("INSERT INTO photo Values('%s','%s','%s','%s')"%(time_info,description,path,new_filename))
            iq.do()
            return '<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" /><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js" /><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" /><center><h2>You have uploaded %s.</h2> <button type="button" class="btn btn-prinary btn-lg" ><a href="/">Return</button></a></center>.'%(uploaded_file.filename)
         else:
            page ='''<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                     <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js" />
                     <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                     <center><a href='/'><button type="button" class="btn btn-prinary btn-lg" >return</button></a>
                     <form action="/upload/" method="post" enctype="multipart/form-data">
                     <input type="file"name="file"><input name="description"><input type="submit"value="Upload"></form></center>'''
            return page


**show_dp.py**
::
      from flask import Blueprint,render_template,request
      from UseSqlite import  RiskQuery
      from PIL import Image

      show_bp= Blueprint('show_bp',__name__)
      
      def make_html_paragraph(s):
         if s.strip()=='':
            return ''
         lst=s.split(',')
         picture_path=lst[2].strip()
         picture_name=lst[3].strip()
         im = Image.open(picture_path)
         im.thumbnail((400, 300))
         im.save('./static/figure/'+picture_name,'jpeg')
         result='<p>'
         result+='<i>%s</i><br/>'%(lst[0])
         result+='<i>%s</i><br/>'%(lst[1])
         result+='<a href="%s"><img src="../static/figure/%s"alt="风景图"></a>'%('.'+picture_path,picture_name)
         return result+'</p>'

      def get_database_photos():
         rq=RiskQuery('./static/RiskDB.db')
         rq.instructions("SELECT * FROM photo ORDER By time desc")
         rq.do()
         record='<center><h3>My past photo</h3></center>'
         for r in rq.format_results().split('\n\n'):
            record+='<center>%s</center>'%(make_html_paragraph(r))
         return record+'</table>\n'

      @show_bp.route('/show/') 
      def show():
         
         page ='''<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                  <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js" />
                  <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                  <center><a href='/'><button type="button" class="btn btn-prinary btn-lg" >return</button></a></center>'''
         page +=get_database_photos()
         return page


**search_bp.py**
::
      from flask import Blueprint,render_template,request
      from UseSqlite import  RiskQuery
      from PIL import Image

      search_bp= Blueprint('search_bp',__name__)

      def make_html_photos(s):
         if s.strip()=='':
            return ''
         lst=s.split(',')
         picture_path=lst[2].strip()
         picture_name=lst[3].strip()
         im = Image.open(picture_path)
         im.thumbnail((400, 300))
         path = '.' + picture_path
         result='<p>'
         result+='<i>%s</i><br/>'%(lst[0])
         result+='<i>%s</i><br/>'%(lst[1])
         result+='<a href="%s"><img src="./static/figure/%s"alt="风景图"></a>'%(path,picture_name)
         return result+'</p>'

      def get_info_photos(info):
         rq = RiskQuery('./static/RiskDB.db')
         rq.instructions("SELECT * FROM photo where description = '%s' " % info)
         rq.do()
         record = '<p>search result</p>'
         for r in rq.format_results().split('\n\n'):
            record += '%s' % (make_html_photos(r))
         return record + '</table>\n'

      @search_bp.route("/search",methods=['POST','GET'])
      def search():
         page ='''<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                     <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js" />
                     <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css" />
                     <center><a href='/'><button type="button" class="btn btn-prinary btn-lg" >return</button></a>
                     <form action="/search/query-string"method="post"enctype="multipart/form-data">
                     <input name="description"><input type="submit"value="search"></form></center>'''
         return page

      @search_bp.route('/search/query-string', methods=['POST', 'GET'])
      def query_string():
         if request.method == 'POST':
            description = request.form['description']
            page = get_info_photos(description)

         return page


**api_bp.py**
::
      from flask import Blueprint,render_template,request
      import json
      import os.path
      from UseSqlite import  RiskQuery

      api_bp= Blueprint('api_bp',__name__)
      
      @api_bp.route("/api")
      def api():
         rq = RiskQuery('./static/RiskDB.db')
         rq.instructions("SELECT * FROM photo ORDER By time desc")
         rq.do()
         lst = []  # 存储输出的图片信息的数组
         page = ''
         i = 1  # 手动给图片添加ID
         for r in rq.format_results().split('\n\n'):
            photo = r.split(',')
            picture_time = photo[0]  # 获取上传时间
            picture_description = photo[1]  # 获取图片描述
            picture_path = photo[2].strip()  # 获取图片存储路径
            photo_size = str(format((os.path.getsize(picture_path) / 1024), '.2f')) + 'KB'  # 获取图片文件大小
            lst = [{'ID': i, 'upload_date': picture_time, 'description': picture_description, 'photo_size': photo_size}]
            lst2 = json.dumps(lst[0], sort_keys=True, indent=4, separators=(',', ':'))
            page += '</p>%s' % lst2
            i += 1
         return page

**UseSqlite.py**
::
      # Reference: Dusty Phillips.  Python 3 Objected-oriented Programming Second Edition. Pages 326-328.
      # Copyright (C) 2019 Hui Lan

      import sqlite3

      class Sqlite3Template:
          def __init__(self, db_fname):
              self.db_fname = db_fname

          def connect(self, db_fname):
              self.conn = sqlite3.connect(self.db_fname)

          def instructions(self, query_statement):
              raise NotImplementedError()

          def operate(self):
              self.results = self.conn.execute(self.query) # self.query is to be given in the child classes
              self.conn.commit()

          def format_results(self):
              raise NotImplementedError()

          def do(self):
              self.connect(self.db_fname)
              self.instructions(self.query)
              self.operate()


      class InsertQuery(Sqlite3Template):
          def instructions(self, query):
              self.query = query


      class RiskQuery(Sqlite3Template):
          def instructions(self, query):
              self.query = query

          def format_results(self):
              output = []
              for row in self.results.fetchall():
                  output.append(', '.join([str(i) for i in row]))
              return '\n\n'.join(output)


      if __name__ == '__main__':

          #iq = InsertQuery('RiskDB.db')
          #iq.instructions("INSERT INTO inspection Values ('FoodSupplies', 'RI2019051301', '2019-05-13', '{}')")
          #iq.do()
          #iq.instructions("INSERT INTO inspection Values ('CarSupplies', 'RI2019051302', '2019-05-13', '{[{\"risk_name\":\"elevator\"}]}')")
          #iq.do()

          rq = RiskQuery('RiskDB.db')
          rq.instructions("SELECT * FROM inspection WHERE inspection_serial_number LIKE 'RI20190513%'")
          rq.do()
          print(rq.format_results())



References
=================================================

[1] Flask 中文文档 (2.0.2) https://dormousehole.readthedocs.io/en/latest/

[2] started-with-sphinx https://docs.readthedocs.io/en/stable/intro/getting-started-with-sphinx.html

[3] Read the docs 环境搭建  https://blog.csdn.net/u010386121/article/details/83274964?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163957890116780274182682%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163957890116780274182682&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-9-83274964.pc_search_insert_es_download&utm_term=read+the+docs

