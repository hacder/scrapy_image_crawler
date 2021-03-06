scrapy_image_crawler
====================

crawl image and monitor

![Image text](https://raw.github.com/donquixote1984/scrapy_image_crawler/master/screenshot.png)

0. architect
	. redis for the q
	. scrapyd for the service deployment and management
	. mysql for the persistence
	. django for the monitor
	. ichartjs for the web page chart
	. jquery for easy javascript
	. lots of css3

1. implemententation
	. The scrapy starts from a extension name counter in extension/counter.py . it start a new thread which used for timely saving the spider status to redis. user can configure the interval. by default it is 10 seconds.
	. The spiders start from a parser in spider class. the parser mainly extract the image links using xpath and exract all the other hyperlink. the crawl depth can be configured. 
	. Then comes to the pipeline, there is 3 pipeline for now, as the order:
		- cache pipeline. cache the images which has been crawled. and stop crawling the duplicated.
		- custom image pipeline used for downloading images and store it to specific folder. the folder is named by the page which the image in. this pipeline will also get the image width and height and byte sizes. for the later db saving.
		- db pipeline save the image/page url and relationship and path in db. 
		- content pipeline.  save the page meta data in disk. each page has a uuid named folder which is actually the page id in database. the folder contains text based title, meta_key, meta_data, content, url info for the full text search.
	. The only util class is in storage folder. just for store the page info in content pipeline.
	. 2 tables in db. The images table in db saves the images url and saved path, update time, size, height,width etc. The page table in db saveds the path url. the page id in db is exactly the folder name saved in shit.

2. usage
	. run the "install.py"
	. start the scrapyd by typing 'scrapyd'
	. start redis, better to config the redis disk sync strategy.
	. start django client by 
		- cd $SPIDER_HOME
		- python client/management.py runserver
	. open browser and enter the console : http://localhost:8000/client
	. start the spider by clicking the button
	. each spider can have multiple instance, this is especially for the distrubute spider which has not beent tested. in one single machine, it is not recommended.
	. all the left is waiting, monitoring, and reporting bugs.

3. configuration
	. default configuration are made before running. 
	. the spider can be run in 2 mode:
		- scrapyd
			~ the spider is started by scrapyd and web console. the spider can be monitored. 

				@the downloaded image and page content will be stored in ~/scrapy_downloads
				@the log can be viewed by the scrapyd console, (for now it can not be viewed by web console, may be next version), but the large log, user can tail the log files in $SPIDER_HOME/logs

			~ the spider is started by typing: scrapy scrawl <spider> in spider folder. this mode do not need to deploy the spider in scrapyd, but can not be monitored by web console

				@the downloaded images and page content will be stored in ./scrapy_downloads
				@the log can be view in $SPIDER_HOME/log

4. customize
	. The only customization thing is add spider. in the spider folder, there is some default spider, just copy them and add your url.
	. Do not forget to re-deploy the spider.

4. deployment
	. The spider can be deployed to scrapyd server and use json-rpc to start/shutdown
	. The json-rpc document is in its offical site.
	. After changing the spider, you must redeploy to the spider to scrapyd to let its new feature take effect. it is the inut:
		- cd $SPIDER_HOME
		- scrapyd-deploy tugouspider -p tugouspider
	. Sometimes the spider can not be deployed due to some error. check the scrapy.cfg file in $SPIDER_HOME

5. getcha
	. The scrapyd folder which locates in $SPIDER_HOME folder does not belong to any version of scrapyd in github. the webservice.py in the folder is modified for special usage,like pid status and job continue.(but not been used yet. maybe next version)
	. Some of the buttons in web ui is unavailable. they are in the right top location of the page. control buttons of the spider. I wish they could be themselves but lack of fu*kin time.
	. Be careful of stopping the spiders. the web ui provides the stop button by clicking the fansy PAUSE button below the chart. it is not recommanded to kill the process cause the information integrated. but you can do it if the web ui is shuting down for quite a long time.
	. Scrapy uses the python twisted framework, the truth is that not everything is synchronized. sometimes the db saving is deferred which means the db image count is n0t corresponding to the image saved in disk. saves later.
	. The scrapy_redis folder in the project is not exactly the one in github. some thing has been changed.

6. todo
	. implement the unavailble buttons in web console.
	. add a input field in web console. use can input one start url and start the shit. do not needs special spiders, one spider with different start url.
	. spider pause/continue, for now it is just start/stop



