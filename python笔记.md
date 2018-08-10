## Python基础语法

* ord()函数获取字符的整数表示，chr()函数把编码转换为对应的字符

* 指定编码转换 `bytes`
```
	'ABC'.encode('ascii')
	'中文'.encode('utf-8')
	b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
	b'ABC'.decode('ascii')
	b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
```

* 修改全局变量使用`global `关键字进行声明，修改外部作用域非全局变量使用`nonlocal`关键字进行声明，例如:
	
	```
	num = 1		//全局变量
	def test():
		global num 	//使用global声明
		num = 10
		print(num)
				
		count = 0
		def inner():	//内部方法
			nonlocal count	//使用nonlocal声明
			count = 2
			print(count)	

	```

* round函数使用的坑： [链接](http://www.runoob.com/w3cnote/python-round-func-note.html)


* 列表推导式 




			