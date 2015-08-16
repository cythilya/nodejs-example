#Node - 隱藏Response Headers資訊
Response Headers給駭客一個可以入侵網站的地方，具有資安意識的網站會隱藏或給予錯誤的資訊。假設要隱藏X-Powered-By資訊(見紫框)，使用這一行指令：

	app.disable('x-powered-by');

![隱藏Response Headers資訊](https://lh3.googleusercontent.com/JwvXnbIOYXC8paKXibSam22bItOOa515W5dnDqIyM5M=w559-h402-no)  

資訊隱藏。  

![隱藏Response Headers資訊](https://lh3.googleusercontent.com/dku9j2GcJMxlRAWxMWcSiICeuZhIruz-XeGqMi6w5xM=w557-h361-no)
