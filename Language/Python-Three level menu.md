Title: python 三级菜单
Date: 13:40 2016/9/21
Modified: 13:40 2016/9/21
Category: language
Tags: Python
Slug: 
Author: allposs
##适用环境

操作系统：python3*

##版本号
V1.0

##说明（README.md）

###1.基本概述

&#160; &#160; &#160; &#160;三级菜单程序是由两个文件组成，由主程序和数据存储文件组成，都是基于python语言写的。
	
###2.技巧运用

&#160; &#160; &#160; &#160;其中运用了字典、元组的读取与格式转换，for循环，if循环还有while循环，其中有利用continue跳出循环的小技巧制作了返回功能，还有自定义模块的导入与导出功能。
	
###3.程序运行概述

&#160; &#160; &#160; &#160;当用户运行主程序文件时，主程序文件会读取zone.py模块，然后主程序进行格式转换打印出一级菜单，然后用户输入选项，然后主程序进行配对，如果输入正确则进行二级菜单操作，
此时二级菜单会拉取一级菜单的数据进行格式转换并打印二级菜单，然后提示用户输入选项，如果输入正确则进入三级菜单操作，此时三级菜单会拉取二级菜单数据并格式转换并打印三级菜单，然后提示用户输入选项，
如果输入正确则进入三级菜单内容操作，输入正确打印菜单内容，并提示用户是否选择返回上一层或者退出程序。
##流程图

![](http://images.allposs.com/20160921134011.png)

##代码
	python代码：
	
	#!/usr/bin/env python
	#_*_coding:utf-8_*_
	#Version:
	#data:
	import zone
	Menu1=[]
	Menu2=[]
	Menu3=[]
	Return_flage = 0
	Exit_flage = False
	while Exit_flage is not True:
	#无限循环
	#第一级菜单
		if Return_flage == 0 :
			print ("-------------------------------------------------")
			print ("+            +")
			print ("+            +")
			print ("+   欢迎来到大中华地区景点查询系统   +")
			print ("+            +")
			print ("+            +")
			print ("-------------------------------------------------")
			Return_flage += 1
			Zone1 = zone.Menu
			for index,i in enumerate(Zone1):
				Menu1.append(str(i))
				print(index, i)
	#转换成元组并打印索引编号与元组
	#第二级菜单
		elif Return_flage == 1 :
			Option = input("(q=退出,r=返回上一层)省级菜单请输入选择:")
			if Option.isdigit() :
				Return_flage += 1
				Option = int(Option)
				if len(Menu1) > Option :
				#判断输入数值是否超出选项范围
					Input_Option = Menu1[Option]
					Zone2 = Zone1.get(Input_Option)
					for index,i in enumerate(Zone1.get(Input_Option)):
						Menu2.append(str(i))
						print(index, i)
				else:
					print("请输入正确选项！")
					Return_flage -= 1
					continue
					#超出范围的输入则跳出本次循环并再进行一次
			elif Option == "quit" or Option == "q" :
				Exit_flage = True
			elif Option == "Return" or Option == "r":
				Return_flage -= 1
				continue
				#返回功能
			else :
				print("请输入正确选项！")
				continue
	# 第三级菜单
		elif Return_flage == 2:
			Option = input("(q=退出,r=返回上一层)市级菜单请输入选择:")
			if Option.isdigit():
				Return_flage += 1
				Option = int(Option)
				if len(Menu2) > Option :
					Input_Option = Menu2[Option]
					Zone3 = Zone2.get(Input_Option)
					for index, i in enumerate(Zone2.get(Input_Option)):
						Menu3.append(str(i))
						print(index, i)
				else:
					print("请输入正确选项！")
					Return_flage -= 1
					continue
			elif Option == "quit" or Option == "q":
				Exit_flage = True
			elif Option == "Return" or Option == "r":
				Return_flage -= 1
				continue
			else:
				print("请输入正确选项！")
				continue
	# 第三级菜单内容
		elif Return_flage == 3:
			Option = input("(q=退出,r=返回上一层)区级菜单请输入选择 :")
			if Option.isdigit():
				Return_flage += 1
				Option = int(Option)
				if len(Menu3) > Option:
					Input_Option = Menu3[Option]
					Zone4 = Zone3.get(Input_Option)
					print(Zone4)
					Option = input("(q=退出,r=返回上一层)请输入返回上一层或者退出:")
					if Option.isdigit():
						print("请输入正确选项！")
						continue
					elif Option == "quit" or Option == "q":
						Exit_flage = True
						continue
					elif Option == "Return" or Option == "r":
						Return_flage -= 1
						continue
					else:
						print("请输入正确选项！")
						continue
				else:
					print("请输入正确选项！")
					Return_flage -= 1
					continue
			elif Option == "quit" or Option == "q":
				Exit_flage = True
			elif Option == "Return" or Option == "r":
				Return_flage -= 1
				continue
			else:
				print("请输入正确选项！")
				continue




	
	zone.py文件：
		#!/usr/bin/env python
		#_*_coding:utf-8_*_
		#Version:
		#data:
		Menu = {
			'山东' : {
				'青岛' : {
					'四方':
						['山','河','湖','城'],
					'黄岛':
						['山','河','湖','城'],
					'崂山':
						['山','河','湖','城'],
					'李沧':
						['山','河','湖','城'],
					'城阳':
						['山','河','湖','城']
				},
				'济南' : {
					'历城':['山','河','湖','城'],
					'槐荫':['山','河','湖','城'],
					'高新':['山','河','湖','城'],
					'长青':['山','河','湖','城'],
					'章丘':['山','河','湖','城']
				},
				'烟台' : {
					'龙口':['山','河','湖','城'],
					'莱山':['山','河','湖','城'],
					'牟平':['山','河','湖','城'],
					'蓬莱':['山','河','湖','城'],
					'招远':['山','河','湖','城']
				}
			},
			'江苏' : {
				'苏州' : {
					'沧浪':['山','河','湖','城'],
					'相城':['山','河','湖','城'],
					'平江':['山','河','湖','城'],
					'吴中':['山','河','湖','城'],
					'昆山':['山','河','湖','城']
					},
				'南京' : {
					'白下':['山','河','湖','城'],
					'秦淮':['山','河','湖','城'],
					'浦口':['山','河','湖','城'],
					'栖霞':['山','河','湖','城'],
					'江宁':['山','河','湖','城'],
				},
				'无锡' : {
					'崇安':['山','河','湖','城'],
					'南长':['山','河','湖','城'],
					'北塘':['山','河','湖','城'],
					'锡山':['山','河','湖','城'],
					'江阴':['山','河','湖','城']
				}
			},
			'浙江' : {
				'杭州' : {
					'西湖':['山','河','湖','城'],
					'江干':['山','河','湖','城'],
					'下城':['山','河','湖','城'],
					'上城':['山','河','湖','城'],
					'滨江':['山','河','湖','城']
				},
				'宁波' : {
					'海曙':['山','河','湖','城'],
					'江东':['山','河','湖','城'],
					'江北':['山','河','湖','城'],
					'镇海':['山','河','湖','城'],
					'余姚':['山','河','湖','城']
				},
				'温州' : {
					'鹿城':['山','河','湖','城'],
					'龙湾':['山','河','湖','城'],
					'乐清':['山','河','湖','城'],
					'瑞安':['山','河','湖','城'],
					'永嘉':['山','河','湖','城']
				}
			},
			'安徽' : {
				'合肥' : {
					'蜀山':['山','河','湖','城'],
					'庐阳':['山','河','湖','城'],
					'包河':['山','河','湖','城'],
					'经开':['山','河','湖','城'],
					'新站':['山','河','湖','城']
				},
				'芜湖' : {
					'镜湖':['山','河','湖','城'],
					'鸠江':['山','河','湖','城'],
					'无为':['山','河','湖','城'],
					'三山':['山','河','湖','城'],
					'南陵':['山','河','湖','城']
				},
				'蚌埠' : {
					'蚌山':['山','河','湖','城'],
					'龙子湖':['山','河','湖','城'],
					'淮上':['山','河','湖','城'],
					'怀远':['山','河','湖','城'],
					'固镇':['山','河','湖','城']
				}
			},
			'广东' : {
				'深圳' : {
					'罗湖':['山','河','湖','城'],
					'福田':['山','河','湖','城'],
					'南山':['山','河','湖','城'],
					'宝安':['山','河','湖','城'],
					'布吉':['山','河','湖','城']
				},
				'广州' : {
					'天河':['山','河','湖','城'],
					'珠海':['山','河','湖','城'],
					'越秀':['山','河','湖','城'],
					'白云':['山','河','湖','城'],
					'黄埔':['山','河','湖','城']
				},
				'东莞' : {
					'莞城':['山','河','湖','城'],
					'长安':['山','河','湖','城'],
					'虎门':['山','河','湖','城'],
					'万江':['山','河','湖','城'],
					'大朗':['山','河','湖','城']
				}
			}
		}