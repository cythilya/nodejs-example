#Node - 使用模版引擎 Handlebars
##使用Handlebars作為模版引擎
app.js設定如下：

	var handlebars = require('express3-handlebars').create({defaultLayout: 'main'}); //引用Handlebars模組
	app.engine('handlebars', handlebars.engine); //使用Handlebars作為模版引擎
	app.set('view engine', 'handlebars'); //設定辨識用的副檔名，假設沒有設定副檔名，之後render view都要使用「檔名.副檔名」

##Layout
Layout 一般會放在 views > layouts 資料夾裡面。主要Layout的檔案為「main.handlebars」。

- 兩個大括號「{{ }}」用來套版，代入資料。 
- 三個大括號「{{{ }}}」用來關閉HTML轉義功能。
- 使用`{{{ body }}}`將view載入。

語法可參考官方網站 [Handlebars.js: Minimal Templating on Steroids](http://handlebarsjs.com/expressions.html)。

	<!doctype html>
	<html>
		<head>
			<title> {{ title }} </title>
			<meta charset="utf-8">
		</head>
		<body>
			{{{ body }}}
		</body>
	</html>

##View
使用「index.handlebars」。我們在首頁(index.handlebars)載入這段partial程式碼，程式碼如下。

	<!-- index.handlebars -->
	{{> weather }}

##Partial
Partial View 一般會放在 views > partials 資料夾裡面。建立「weather.handlebars」程式碼如下，並將資料代入，資料範例如下Mock Data。

	<!-- weather.handlebars -->
	<div class="weatherWidget">
		{{#each partials.weather.locations}}
			<div class="location">
				<h3>{{name}}</h3>
				<a href="{{forcastUrl}}">
					<img src="{{iconUrl}}" alt="{{weather}}">
					{{weather}}, {{temp}}
				</a>
			</div>
		{{/each}}
	</div>

###Mock Data

	function getWeatherData(){
		return {
			locations: [
				{
					name: 'Portland',
					forecastUrl: 'http://sample.com.tw',
					iconUrl: 'http://dummyimage.com/50x50/000/fff',
					weather: 'Overcast',
					temp: '54.1 F (12.3 C)'
				},
				{
					name: 'Bend',
					forecastUrl: 'http://sample.com.tw',
					iconUrl: 'http://dummyimage.com/50x50/000/fff',
					weather: 'Partly Cloudly',
					temp: '54.1 F (12.3 C)'
				}
			]
		}
	};

##Router

	router.get('/', function(req, res, next) {
		if(!res.locals.partials){
			res.locals.partials = {};
		}
		res.locals.partials.weather = getWeatherData(); //get mock data
		res.render('index', { title: 'This is index page.' }); //get index page
	});

##Demo
重新啟動專案，瀏覽器網址列輸入 `localhost:3000` 就會看到以下畫面：

![Node.js - Handlebars](https://lh3.googleusercontent.com/dVTXQtrUsNEAlgSrIe9im7cO2mzYqX21vApj07BTMj4=w467-h194-no)

---
####Reference
- [Handlebars.js: Minimal Templating on Steroids](http://handlebarsjs.com/expressions.html)