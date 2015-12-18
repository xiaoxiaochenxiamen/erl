# 项目需求

第三轮开发：
1、配置文件：{http_send_cycle, *}, http请求一次数据周期(秒)
2、配置文件：{mysql_table_key_range,"{[{"cid1:10,cid3:25,pager:200"},{"cid1:20,cid3:35,pager:100"},...]}"},
该机器分配的数据范围。
3、脚本启动时，按照2中配置读取数据库中的指定表数据（id和value）进入内存。查询SQL格式为：where (cid1=10 and cid3=25) or (cid1=20 and cid3=35))
4、使用2的数据拼接成url地址并发访问，url地址为http://115.28.144.211:8080/jtest.php?functionId=getCat&body=%7B%22catelogyIdLevel1%22%3A%22{cid1}%22%2C%22stock%22%3A%221%22%2C%22pagesize%22%3A%2210%22%2C%22catelogyId%22%3A%22{cid3}%22%2C%22page%22%3A%22{pagenum}%22%2C%22cid%22%3A%22{cid3}%22%7D
这里的pagenum是从1开始到Pager的序列数字。如2中cid1的pager为200，则pagenum从1到200顺序访问。其中的内容格式为json数据urlencode的结果。
5、解析返回的json数据，每个json数据包含最多10条：
{"page":"1","wareInfo":[

{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345670","jdPrice":"3450.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345671","jdPrice":"3459.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345672","jdPrice":"3451.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345673","jdPrice":"3452.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345674","jdPrice":"3453.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345675","jdPrice":"3454.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345676","jdPrice":"3455.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345677","jdPrice":"3456.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345678","jdPrice":"3457.00"},
{"cid1":"1318","cid2":"1466","catid":"1698","wareId":"12345679","jdPrice":"3458.00"}

]"isUsed":false}

如果超出读取范围，返回json数据为空。具体如下：
{"page":"***","wareInfo":[]"isUsed":false}

6、判断逻辑
* wareID对应配置表中的id，jdPrice对应value。如果对应的数字和本地内存中一致，略过
* 如果对应的数字和本地内存中不一致，修改本地内存数据，update数据库记录，然后插入明细表

7、所有完成后，规定间隔还没到，则sleep。等待下一轮循环
