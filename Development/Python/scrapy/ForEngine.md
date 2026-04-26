![Scrapy architecture](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/scrapy_architecture.png)

### Scrapy 框架的工作原理

Scrapy 基于事件驱动的异步框架，通过 Twisted 库处理网络请求和响应。在一个 Scrapy 爬虫项目中，整个流程是通过以下几个模块的配合来完成的：

1. **Spider**：负责爬取网站页面并从中提取数据。
2. **Scheduler**：负责存储待抓取的请求（Request）并按顺序分配给下载器（Downloader）。
3. **Downloader**：下载网络资源并将响应（Response）传递给相应的 spider 进行解析。
4. **Pipeline**：负责对抓取到的数据进行处理、清洗或存储。
5. **Item**：定义抓取的数据结构。
6. **Settings**：配置 Scrapy 项目的各项参数，决定运行方式、调度策略等。

### 各个模块的作用和交互

#### 1. **Spider**

Spider 是 Scrapy 中最重要的模块之一，它负责定义如何从网页中提取数据。Spider 类继承自 Scrapy 的 `scrapy.Spider` 类，需要实现 `parse` 方法（或者其他自定义方法），用于解析响应数据。

- **作用**：发送请求、解析响应、提取数据。
- **交互**：Spider 从 Scheduler 接受请求，收到响应后调用解析方法（如 `parse`），提取数据并生成新的请求或 item。

#### 2. **Scheduler**

Scheduler 负责保存待处理的请求（`Request` 对象）并按队列调度给下载器。它采用先进先出的调度策略，确保每个请求都能按顺序下载。

- **作用**：请求调度管理。
- **交互**：Scrapy 中的请求通过 Spider 生成后被发送给 Scheduler 进行管理，Scheduler 会将请求发送给 Downloader。

#### 3. **Downloader**

Downloader 负责从网络上获取页面内容，它是 Scrapy 中的核心组件，使用了 Twisted 异步库，能够高效地处理大量的请求。

- **作用**：下载网页内容。
- **交互**：Downloader 从 Scheduler 获取请求，下载网页后将结果返回给相应的 Spider。

#### 4. **Item**

Item 是用来存储爬取到的数据的对象。它的作用类似于一个字典，存储爬取数据的各个字段。在 Spider 的 `parse` 方法中，爬取的数据会被封装成 Item 对象，并返回给 Pipeline 进行后续处理。

- **作用**：数据存储。
- **交互**：Spider 中提取的数据会存储到 Item 中，然后传递给 Pipeline 进行后续处理。

#### 5. **Pipeline**

Pipeline 负责处理爬取的数据。它通常用于清洗数据、验证数据、去重、存储到数据库等操作。Scrapy 支持多个管道，可以在设置文件中配置不同的管道处理顺序。

- **作用**：数据处理和存储。
- **交互**：Spider 生成的 Item 对象会传递给 Pipeline 进行处理，可以在 Pipeline 中执行如数据清洗、验证、去重、存储等操作。

#### 6. **Settings**

Settings 是 Scrapy 项目的配置文件，包含了项目的所有配置信息，如 User-Agent、下载延迟、并发数、日志级别等。

- **作用**：配置项目和调度策略。
- **交互**：Settings 文件通过 Scrapy 配置系统提供参数给框架的各个模块，如下载延迟、User-Agent、日志级别等。

### Scrapy 的工作流程

1. **启动爬虫**：用户通过命令行启动一个爬虫，Scrapy 会自动调用 Spider 开始爬取数据。
2. **请求调度**：Spider 中定义的 `start_requests` 方法或者 `parse` 方法中生成的请求会传递给 Scheduler。
3. **下载页面**：Scheduler 将请求传递给 Downloader，Downloader 异步下载网页并将页面内容返回给 Spider。
4. **解析数据**：Spider 根据页面内容调用解析方法（如 `parse`），提取数据并封装成 Item 对象。
5. **数据处理**：Item 对象会通过 Pipeline 进行数据处理，处理包括数据清洗、去重、存储等操作。
6. **存储数据**：处理后的数据可以存储到文件、数据库或其他存储介质中。

### 简略的开发流程

1. **创建 Scrapy 项目**：首先，通过命令行创建一个新的 Scrapy 项目。

   ```bash
   scrapy startproject myproject
   ```

2. **编写 Spider**：在 `spiders` 目录下创建 Spider 类，定义如何请求网页、如何解析页面。

   ```python
   import scrapy
   
   class MySpider(scrapy.Spider):
       name = 'my_spider'
       start_urls = ['http://example.com']
   
       def parse(self, response):
           # 提取数据
           title = response.xpath('//title/text()').get()
           yield {'title': title}
   ```

3. **定义 Item**：在 `items.py` 中定义数据结构。

   ```python
   import scrapy
   
   class MyItem(scrapy.Item):
       title = scrapy.Field()
   ```

4. **编写 Pipeline**：在 `pipelines.py` 中定义数据处理逻辑。

   ```python
   class MyPipeline:
       def process_item(self, item, spider):
           # 数据处理逻辑
           return item
   ```

5. **配置 Settings**：在 `settings.py` 中设置项目参数，如并发数、延迟等。

   ```python
   BOT_NAME = 'myproject'
   DOWNLOAD_DELAY = 2
   ```

6. **运行爬虫**：通过命令行运行爬虫，抓取数据。

   ```bash
   scrapy crawl my_spider
   ```

7. **处理数据**：数据会经过 Spider 提取后，经过 Pipeline 处理，最终存储到指定位置。

