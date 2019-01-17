## Scrapy框架的使用

* 新建项目

  ```
  scrapy startproject 项目名
  ```



* 新建Spider爬虫

  ```
  scrapy genspider 爬虫名 需要爬取的网址
  例：
  scrapy genspider images images.so.com
  ```

* 构造请求，构建初始请求(可选),重写start_requests()方法

  > 打开spiders目录下的爬虫文件

  ```python
  """生成50次请求"""
  def start_requests(self):
      data = {'ch': 'beauty', 'listtype': 'new'}
      base_url = 'https://image.so.com/zj?'
      for page in range(1, 51):
          data['sn'] = page * 30
          params = urlencode(data)
          url = base_url + params
          yield Request(url, self.parse)
  ```

* 修改settings.py文件

  ```python
  """自动限速扩展开关"""
  ROBOTSTXT_OBEY = False
  """为True时需要遵守robots.txt的规则进行爬取"""
  ROBOTSTXT_OBEY = False
  ```

* 提取信息

  ```python
  """重写parse方法，返回item对象或一个Request请求"""
  def parse(self, response):
  	
  ```




