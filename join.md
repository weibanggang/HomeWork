
##第一步：查出广东省
	首先进行查询这个表的数据，条件为 cityName="广东省"
	select * from s_provinces sn where sn.cityName="广东省"
	可以查看到只有一条数据
##第二步:查出广东省的所有市
	进行多表连接，条件parentId=id  并且  sn.cityName="广东省"
	select * from s_provinces sn
	join s_provinces sh on sh.parentId=sn.id
	where sn.cityName="广东省"
	这样可以查出广东省的所有市
##第三步:查出广东省所有的市和县 此时需要三个进行查询
	规律就是自己查看已经，然后进行parentId=sn.id
	进行多表连接，条件parentId=id  并且  sn.cityName="广东省"
	
	select sc.id,sn.cityName,sh.cityName,sc.cityName from s_provinces sn
	join s_provinces sh on sh.parentId=sn.id
	join s_provinces sc on sc.parentId=sh.id
	where sn.cityName="广东省"
	这样查出广东省所有的市和县

##关于union和union all的理解
	使用union进行查询时候会把相同的数据融合在一起
	使用union all进行查询，数据不会融合
	
##总结
		需要全部查出数据就得使用union all