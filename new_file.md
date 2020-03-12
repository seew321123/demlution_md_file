# 2020.3.12小程序商城笔记
>### 1.详情页点击立即购买后的流程
>>首先应该跳转到订单确认页面，页面内容包括：  
>>1).收货地址  
>>2).产品信息以及产品数量  
>>3).优惠方案（此次学习过程暂时不考虑）  
>>4).备注信息  
>>5).提交订单按钮

>### 2.跳转到订单确认页面后，需要查询的数据表分别是： 
>>1).根据siteuser查询其对应的地址表信息  
>>2).根据跳转到此页面时传入的产品id，查询产品的信息  

>### 3.知识总结： 
>>1).在后端py文件中获取当前用户id的方式为:  
>>>siteuser_id = await context.async_get_siteuser_id()  

>>2).js以及py文件有一种比较简洁的赋值方式用于测试方法。比如你的方法需要传入一个参数，但你所处于的环境让你无法传入这个参数。
  在js中你可以通过：  
>>>const a = b || c  

>>在py中你可以通过：  
>>> a = b or c  

>>你在文件伊始事先声明这个c变量，当b变量不存在时就可以将这个c变量赋值给a用于测试。

>>3).小程序wxml文件中的图片内容src属性可以通过过滤器将数据库中的图片数据转化为url,方式如下  
>>> <wx-image class="b-product_img":src="dwtools.qiniu(slide.dwdata.product_data.image)"></wx-image>   

>>4).在商城的订单确认界面，通常要显示当前用户在地址库中的地址，
如果你需要显示这个地址，可以在py文件中如下操作：  
>>>from shared.order.repos import AddressBookRepo  
导入地址库的数据表以后，根据当前的id,上文已经知道siteuser_id的获取方式。  
理论上通过AddressBookRepo().filter(siteuser_id=XXX)就能够获得这个用户所有的地址信息所组成的列表。你可以默认选择第一个地址。  
当用户想要切换地址时，再次加载所有的地址供其选择。  
在api文档中也有对应的接口可以使用，
await context.async_call('get/paddressbook/get_list_by_siteuser')
在开发阶段因为当前登录的测试账号没有相关的地址，因此我没有采用这种方式。

>>5).提交订单的方式  
>>>提交订单的云函数接口为  
context.call('post/order_ccode/submit_papp_order', papp_order_params)  
其中参数papp_prder_params是一个字典。字典中包含的参数如下  
>>>>papp_order_params = {  
        "papp_slug": data['slug'],  
        "return_url": return_url,  
        "dwapp_return_url": dwapp_return_url,  
        "addressbook_id": papporder_data['addressbook_id'],  
        "need_express": True,  
        "orderitems_data": orderitems_data,  
        "orderitem_data": orderitems_data[0]  
    }  
	
>>>其中每个属性的意义可以参考  
https://cloud.demlution.com/store/admin/page.html#/dapi/schema)

>>>其中部分属性的获取方式:  
>>>>应用slug可以在前端通过data:['slug'] = dw.app.getPappSlug()

>>>调用提交订单api时，数据检查比较严格。有几个需要注意的是：  
1.siteuser_id必须要和address_book中的用户id匹配  
2.paymethod_id（支付方式id）必须要存在，不能为null.  
3.系统默认的siteuser在数据库中没有任何地址簿记录，因此我尝试通过调用
create_by_siteuser这个api为他创建一条地址簿数据。  
虽然显示成功，但是依然不能以正常方式调取这个数据。  
因此我个人猜测是因为调试环境和正常环境所用的不是同一个数据库，所以我失败了。