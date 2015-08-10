#Hello Node - 基本設定和簡單範例
距離上次摸Node.js [使用Node.js + Express建構一個簡單的微博網站](http://cythilya.blogspot.tw/2014/11/nodejs-express-microblog.html) 又是好長一段時間，這次從基本設定和簡單範例開始記錄。  

跟著[Hello node.js - win7中的nodejs(1) 安裝篇至hello world](http://blog.friendo.com.tw/posts/238208-nodejs)安裝完成後，玩玩暖身題「Hello Node」吧。  

<!-- more -->

##Hello Node 1
######Step 1：在資料夾新增一個「hello.js」檔案。
![Hello Node](https://lh3.googleusercontent.com/sQfK6AlCa_UUdBjlqov7Xj6M0UMSPztRpm_rP8cvFws=w613-h219-no)  

######Step 2：檔案內容如下，寫一行「console.log('hello node');」。
![Hello Node](https://lh3.googleusercontent.com/MIibiFIUc38KSNcMRGCAcisI9nDWFS6Mm-IxO-vPIpU=w672-h293-no)  

######Step 3：到Node.js command prompt輸入「node hello.js」(node 檔案名.副檔名)，即可得到結果。  
![Hello Node](https://lh3.googleusercontent.com/8VcuvPStXepu_yaF0c3rqEWHzeiQ8VCWK7H0EImvbSg=w678-h212-no)  

或是直接在command line中輸入
	
	node -e "console.log('hello node');"

##Hello Node 2
######Step 1：建立一個「`hello2.js`」檔案，內容如下。

	var http = require('http');
	
	http.createServer(function(req, res){
		res.writeHead(200, {'Content-Type': 'text/html'});
		res.write('<h1>Hello Node</h1>');
		res.end('<p>Hello World</p>')
	}).listen(3000);
	
	console.log('HTTP server is listening at port 3000.');

######Step 2：Node.js command prompt輸入「`hello2.js`」，即可得到結果。
![Hello Node](https://lh3.googleusercontent.com/K_YIxNfitMJ8ROALoRyzsvG8FSP9Kqjcsv0drXNSs30=w668-h204-no)

######Step 3：打開瀏覽器，網址輸入「http://localhost:3000」。
![Hello Node](https://lh3.googleusercontent.com/NR39D4gryKBmBDQMXuHoOdhxy2jV5LQ8q-fhm2yqeSg=w403-h245-no)

##Hello Node with EJS
######Step 1：使用以下指令建立基本架構。

	npm uninstall -gd express
	npm install -gd express
	npm install -g express-generator (安裝命令工具)
	express -t ejs helloejs

建立完成以後，資料夾的結構如下。

![Hello Node with EJS](https://lh3.googleusercontent.com/pnYAy-cvqTQGTbE6kPl6kxW1D6GCLDATIIv5sXGuR2M=w113-h135-no)

######Step 2：進入剛建立的資料夾「helloejs」。
	
	cd D:\helloejs

######Step 3：安裝相依檔案。

	npm install

######Step 4：記得看ㄧ下有兩個地方要設定為EJS。
- app.js

		app.set('view engine', 'ejs'); 

-  package.json

		  "dependencies": {
		    "ejs": "^1.0.0",
		    "express": "~4.13.1",
		  }

######Step 5：更改樣版的內容。
layout.ejs

	<!DOCTYPE html>
	<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>Hello Node with EJS</title>
		<meta name="description" content="Hello Node with EJS">
	</head>
	<body>
		<%- body %>
	</body>
	</html>

index.ejs

	<h1>Hello Node with EJS</h1>

######Step 6：執行指令「npm start」，打開瀏覽器並網址輸入「http://localhost:3000」，就可以看到結果了。

![Hello Node with EJS](https://lh3.googleusercontent.com/lu91gh-ipAWoE3FgdkXQ0mbVOO1xfZfVRiiAKQPSHZ8=w318-h148-no)

##範例程式碼
- [Hello Node 1](https://github.com/cythilya/nodejs-example/blob/master/hello.js)
- [Hello Node 2](https://github.com/cythilya/nodejs-example/blob/master/hello2.js)
- [Hello Node with EJS](https://github.com/cythilya/nodejs-example/tree/master/helloejs)

##遇到的問題與解法
######嘗試「express -t ejs helloejs」，但出現錯誤訊息「'express' 不是內部或外部命令，也不是可運行的程序或批處理文件」。
一直以來我都直接用「express -t ejs 專案名稱」來建立基本架構，但一按下Enter鍵後出現錯誤訊息「'express' 不是內部或外部命令，也不是可運行的程序或批處理文件」。這可怎麼辦才好呢？[[Node.js]重新安裝node.js和express的問題](http://souts.pixnet.net/blog/post/57565548-%5Bnode.js%5D%E9%87%8D%E6%96%B0%E5%AE%89%E8%A3%9Dnode.js%E5%92%8Cexpress%E7%9A%84%E5%95%8F%E9%A1%8C)這篇文章提到，Express已經升級到4.0了，將命令工具抽離出來，因此必須要另外安裝命令工具「npm install -g express-generator」，然後才可以使用指令「express -t ejs 專案名稱」來建立基本架構。然後使用「npm install」安裝相依檔案，接著由於安裝了新東西，必須要使用「npm start」重新啟動專案才能看到結果。

---
####推薦閱讀
- [Hello node.js - win7中的nodejs(1) 安裝篇至hello world](http://blog.friendo.com.tw/posts/238208-nodejs)：安裝Node並寫一個Hello World。
- [Hello node.js - win7中的nodejs(2) IDE with Sublime](http://blog.friendo.com.tw/posts/243372-hello-nodejs-win7-nodejs-b-ide-environment-preparation)：Sublime實用工具安裝。
- [使用Node.js + Express建構一個簡單的微博網站](http://cythilya.blogspot.tw/2014/11/nodejs-express-microblog.html)：「Node.js開發指南」閱讀筆記與實作範例。
- [網誌版 - Hello Node - 基本設定和簡單範例](http://cythilya.blogspot.tw/2015/08/hello-node.html)

####參考資料
- [DOS指令/命令提示字元 簡易使用教學](http://readandplay.pixnet.net/blog/post/156379832-dos%E6%8C%87%E4%BB%A4-%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E5%AD%97%E5%85%83-%E7%B0%A1%E6%98%93%E4%BD%BF%E7%94%A8%E6%95%99%E5%AD%B8)：一些DOS指令，例如：切換資料夾。
- [[Node.js]重新安裝node.js和express的問題](http://souts.pixnet.net/blog/post/57565548-%5Bnode.js%5D%E9%87%8D%E6%96%B0%E5%AE%89%E8%A3%9Dnode.js%E5%92%8Cexpress%E7%9A%84%E5%95%8F%E9%A1%8C)：關於「'express' 不是內部或外部命令，也不是可運行的程序或批處理文件」的解法。
- [node . and npm start don't work](http://stackoverflow.com/questions/24334005/node-and-npm-start-dont-work)：安裝命令工具「npm install -g express-generator」後，不用「node app.js」，改用「npm start」來執行專案。

---