#Node - 使用express-partials製作Partial View 

![Node - 使用express-partials製作Partial View](https://lh3.googleusercontent.com/HZa36KzQIAGUaO84ApRCAfmKVOERkrO0azJLOpbKvlE=w800-h494-no)  

由於Express EJS 版本3去除了部份的middleware，不再支援layout.ejs，因此若要製作Partial View，可使用 [express-partials](https://github.com/publicclass/express-partials)。  

<!-- more -->

步驟如下：  

######Step 1：安裝[express-partials](https://github.com/publicclass/express-partials)。
	
	npm install express-partials

######Step 2：安裝後到app.js做設定。

- `var partials = require('express-partials');`
- 在 `app.set('view engine', 'ejs');` 後加上 `app.use(partials());`

######Step 3：在View中引用Partial View，語法如下：

	<% include _partial.ejs %>

######Step 4：注意路徑設定，例如在layout.ejs引用_meta.ejs這個partial view，而放partial view的地方是在「views > partials」，因此 _meta.ejs 要這樣被參照：

	<% include ../views/partials/_meta.ejs %>

就用這個方法把畫面切乾淨吧！

---
####Reference
- [express-partials 說明文件](https://github.com/publicclass/express-partials)：Express 3.x Layout & Partial support.