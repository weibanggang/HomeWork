#进行分割lagou_position表
        分离出 company 和 city 两张表
       -- 要求，创建三张表
       -- lagou_city (cityid, province, city, district)  # 全国的省市县信息，需要从 china_city 表中提取出来
       -- lagou_company (company_id,company_short_name,company_full_name,company_size,financestage) # 所有的公司表，从 lagou_position_bk 中分离出来
       -- lagou_position (pid, cityid, cid, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at)
   
#首先进行筛选出为空的数据
        第一步:除了distrct字段可以为空，其他字段为空的进行筛选掉
        第二步:字段不能重复，用关键字distinct进行筛选掉重复的字段值
       create table lagou as
       select distinct * from lagou_position where pid is not null and city is not null and field is not null and salary_min is not null
       and salary_max is not null and workyear is not null and education is not null and ptype is not null and pnature is not null
       and advantage is not null and company_id is not null and company_short_name is not null
       and company_full_name is not null and company_size is not null and financestage is not null and position is not null;
#在进行分割出company表
    第一步:在lagou表上进行查询出所有的公司，用group by company_id 进行筛选重复的公司
    第二步:筛选出数据后，进行把公司的相关字段进行查询company_id,company_short_name,company_full_name,company_size,financestage
    第三步:进行根据数据创建出公司表即可
    create table company as
     select company_id,company_short_name,company_full_name,company_size,financestage from  lagou group by company_id;
#进行分割s_provinces表(地区表)
    select count(*) from s_provinces;
    所有数据为:3750条
    select count(*) from s_provinces where depth=1;
    所有为省的数据为:35条
    select count(*) from s_provinces where depth=2;
    所有为市的数据为:373条
    select count(*) from s_provinces where depth=3;
    所有为区、县的数据为3341条
        在表中可以看出每条数据中都有depth字段，0代表国家，1代表省，2代表市，3代表区、县
    第一步:将不需要的数据进行筛选，只需要cityid, province, city, district四个字段
            1、查询出所有的区、县 
                区、县的depth=3
            select * from s_provinces where depth=3
            2、查出所有的区、县后，然后进行多表查询出该区、县是属于哪个市
              市的depth=2
              方式一:
            select * from s_provinces r
            join s_provinces f on r.parentId=f.id
            where r.depth=3 and f.depth=2
            方式二:
                 select * from s_provinces r
                 join s_provinces f on r.parentId=f.id and f.depth=2
                 where r.depth=3 
            方式三:
            select * from (
                (select * from s_provinces where depth=3)r
                join s_provinces f on r.parentId=f.id and f.depth=2
                )
             3、查出所有的区、县、市后，再进行多表查询出该市是属于哪个省的
             三种
             方式一:
             select r.id,z.cityName,f.cityName,r.cityName from s_provinces r
             join s_provinces f on r.parentId=f.id
             join s_provinces z on f.parentId=z.id
             where r.depth=3 and f.depth=2 and z.depth=1;
             
             4、查询出所有省、市、null(null代表区、县为空) 有些字段区、县是空的
             select distinct r.id cid, f.cityName province,r.cityName distrid,null from s_provinces r
             join s_provinces f on r.parentId=f.id
             where f.depth=1
               
             5、进行用union连接两个查询，得出3714条数据
             select count(*) from(
             select r.id,z.cityName province,f.cityName city,r.cityName district  from s_provinces r
             join s_provinces f on r.parentId=f.id
             join s_provinces z on f.parentId=z.id
             where z.depth=1
             union
             select r.id,f.cityName province,r.cityName city,null district  from s_provinces r
             join s_provinces f on r.parentid=f.id
             where f.depth=1)a;
             6、进行根据数据创建出地区表即可
             create table lagou_city as
             select distinct r.id cid, z.cityName province,f.cityName city,r.cityName distrid from s_provinces r
             join s_provinces f on r.parentId=f.id
             join s_provinces z on f.parentId=z.id
             where z.depth=1
             union all
             select distinct r.id cid, f.cityName province,r.cityName distrid,null from s_provinces r
             join s_provinces f on r.parentId=f.id
             where f.depth=1;
#创建新的表，将主表的地区和公司进行分割为三张表
        该表字段为pid,cid,position,field,salary_min,salary_max,workyear,education,ptype,pnature,advantage,published_at,updated_at,company_id
       第一步:查询出lagou表中district为空的字段
       select * from lagou where district is null
       第二步:进行多表连接  连接lagou_citu表
        查出lagou_citu表中的市区和lagou表中的市区比配相同，并且lagou表中的区、县为空
         select p.*, c.cid from (select * from lagou where district is null) p
        join lagou_city c on c.city like concat(p.city, '%') and c.distrid is null
       第三步:进行多表连接  连接lagou_citu表
        查出lagou_citu表中的市区和lagou表中的市区比配相同，并且lagou表中的区、县为不空
         select p.*, c.cid from (select * from lagou where district is not  null) p
         join lagou_city c on c.city like concat(p.city, '%') and c.distrid like concat(p.district,'%')
     第四步: 进行根据表的数据创建
##方式一：
          将数据用union进行融合
        select p.*, c.cid from (select * from lagou where district is null) p
        join lagou_city c on c.city like concat(p.city, '%') and c.distrid is null
        union all
        select p.*, c.cid from (select * from lagou where district is not  null) p
        join lagou_city c on c.city like concat(p.city, '%') and c.distrid like concat(p.district,'%')
         进行根据表的数据创建
        create table lagou_position
          as
            select pid, cid as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at from
         (
         select p.*, c.cid from (select * from lagou where district is null) p
         join lagou_city c on c.city like concat(p.city, '%') and c.distrid is null
         union all
         select p.*, c.cid from (select * from lagou where district is not  null) p
         join lagou_city c on c.city like concat(p.city, '%') and c.distrid like concat(p.district,'%')
         )  aa;
##方式二
        根据第二步或者第三步进行创建数据表
        1、根据第二步进行创建一个数据表
         create table lagou_position
          as select pid, cid as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at from
         (select p.*, c.cid from (select * from lagou where district is null) p
         join lagou_city c on c.city like concat(p.city, '%') and c.distrid is null)s;
        2、进行添加数据
            insert into lagou_position
            select pid, cid as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at from (select * from lagou where district is not  null) p
            join lagou_city c on c.city like concat(p.city, '%') and c.distrid like concat(p.district,'%')
            
            