#Node - 使用平台MongoLab連接MongoDB
在連接MongoDB的時候遇到了一些小問題，將這些問題與解法記錄在這篇文章中。  

使用平台MongoLab。

<!-- more -->

##Downgrade to 2.6
一開始我就遇到了「MongoError: auth failed」的問題，查了[stack overflow](http://stackoverflow.com/questions/30924859/unable-to-connect-to-mongolab-getting-mongoerror-auth-failed)才知道，在**MongoLab**所選擇的方案中，支援MongoDB的版本是2.4~2.6，因此只好降級，降級完就OK了。

##權限不足
建好Shema和準備好一筆資料，執行npm start後，出現錯誤訊息：  

	MongoError: not authorized for insert on eshopper.products

![Node - 連接MongoDB](https://lh3.googleusercontent.com/cStPXVeagWOwc6k2ldd19cpCq0QtdsFbfTVe07x2njU=w502-h127-no)  

觀察我的connect string許久...

`mongodb://<dbuser>:<dbpassword>@ds035583.mongolab.com:35583/<dbname>`

原來是我的connect string設錯了！帳號、密碼要這樣產生：點 "Add database user" 後會出現一個popup。　　

![Node - 連接MongoDB](https://lh3.googleusercontent.com/PIoMgv9ydiFk9JaWaLkPKadJ5JyWozri1ZNnpNbV18bbPJwwrvu4wpedE2r3hnCJ2KUhiY2q5gxs2EjrKQX1StKbbtswdfup47SlXjKmrGS266aE97H1ha6L9bsGwdyYiri3sJ9s9U-OCR7wAOgyg1EgLtnifqIGmbECHuoADsOgodsczriZT6c6yg0dOtNyrGsvCos2T6VC0o-PGFt4EqBmi1ua4X9tQ3flWMh4Rznlyr1dLgSiw74bW10KtjkCtwxsy8psGbq-1sJ36YdLLWz-9sKAWj00cj4HT44ZXDhfdEng1fYntZD4HJOVdu6_x0TEjfTcP7C0mErvsARcX3tSotYsUVYeOpZndy5i-ZcOIe6eMl0O76S56uttN4xw04Z1-PsucSWNpKyreD8OiJ96eLMQXTwmoW9SGsx79nMQmIBL_E0THGgWJHqs4YtKZ2zgH60nPFTlcHsrMI-k80u3hGkJQdaCDJhStFv1ylbhmPTG8d2vuH5ZG4wc6lzJmPfg-4Wj1ZmGslgBvibKxbY=w800-h440-no)　　

在這個popup中填入username(帳號)、password(密碼)。然後分別代入connect string的`<dbuser>`和`<dbpassword>`。

![Node - 連接MongoDB](https://lh3.googleusercontent.com/90KxnXJT-qlOiFWSC29gUe_cN3yGBdTtpty71dzi32NFe4CEZKSfoIb3y1h5VuV-1DG22ugWthUPaxWTZWmdXgjEl8sf2NrPPhxfgU2i-ww4pwPtcrZtO2784JdErSCgUyE0dv82AMm3TIy4nlgSLpDEVJTCNEHNt9_OnibjOEB_ZeiVJVoj9F9wdY9yXe15Mm_lMN4waICRH5f4K-bLiUbeii4LcrkjV9iFSupfUY4kknmbQLUIBmYjG_RrQPwd7MZmOzmXhT6YzOaAztS0DweK5MlkoSRL1srm9TjmPwkoxRtjUX8yxjJ4IE4jNyAXZBHOetfobWNhfcelKK3wpnferBRs_uDpeRATfq2bwjSaXVeajDpXxQIA_5Nqf1Zbv6hfH-V-UNnS-JrVFwJ_JKNRZ5yQXqeZQRk2G-7oLt5ezkL6BeVY_OrVH4RC9CWJ1wY6SPK5ZSW_QR5vP1fTtx2lljhuUQoF50x2zeiLOejm241H-fQnH-QXFgiYcidKldGYxnLnVK1QA3QSa94S8Y8=w800-h455-no)

這樣就可以成功連接MongoDB啦！

---
####參考資料
- [Easily Develop Node.js and MongoDB Apps with Mongoose](https://scotch.io/tutorials/using-mongoosejs-in-node-js-and-mongodb-applications)