## 题目要求：

1、把去重复后的lagou表内的公司、城市和地区、及其他字段分离成三个表(lagou_company,lagou_city,lagou_position)
思路整理
取得公司相关的字段company_id、company_short_name、company_full_name、company_size、financestage去除重复， 把这些数据插入到一个新的表格lagou_company中

2、删除lagou表原来关于公司内容的字段company_short_name、company_full_name、company_size、financestage，只保留company_id

3、根据p_province表获得区id、省、市、区的数据表

4、根据p_province表获得市id、省、市、null的数据表

5、创建lagou_city表，把步骤3和4的数据插入进去，获得一个包含省市区和省市的表，虽然数据有些冗余但是查询时可大大提升效率，按需求来设计表。

6、由于lagou表中的字段city（城市）和district(地区)的字段存在冗余，并且不能包含所有的城市和地区，所以这两个字段不能分割出来作为一个表来使用。

7、但是lagou表中的city和district这两个字段是不能保留的，假设我们要查询某个省的招聘信息，很明显这两个字段无法满足我们的要求，所以我们要把他们 更改成lagou_city表中其对应的id

8、把city和district更换成lagou_city的id时，我们要考虑一个问题，就是当district的字段为空时，我们获得id就是city（城市）的id，如果district(地区) 字段不为空时，我们获得的id就是district(地区)的id,这是两种截然不同的条件，所以我们可分开查询。

9、当lagou表中district为空时,我们可以把lagou表和lagou_city表连接查询，需要获得的是city（城市）的id,但是条件限制一定要确定清楚。首先肯定是lagou和lagou_city中 的district（地区）的值一定是为空的。其次我们lagou_city表中的所有的城市都带有“市”，而lagou表中的城市则不带“市”，这也是一个要解决的问题（使用like关键词）。所以当前两个表连接查询的条件就是要解决这些问题。

10、当lagou表中的district不为空时，我们需要获得是district的id。这时lagou表和lagou_city表的连接条件就是二者的city（城市）相同，以及district(区县)相同。 还有值得注意的一点便是二者表中的字段district(地区)都不能为空，不然的会产生数据的混淆无法准确得出district(区)的id,可能会拿到city（城市）的id。

11、在步骤9和步骤10时便可以把最终表的字段给确定下来，二者查询完毕后可直接使用union all关联两个查询结果，创建新表lagou_position,插入其中。但是这样 做的效率太过于低下，所以在这里选择可以分开执行。

### 具体实现代码

    --步骤1
    --创建lagou_company公司表。根据查询到去除重复的公司数据建立lagou_company
    create table lagou_company as 
      select distinct company_id cid,company_short_name short_name,company_full_name full_name,
          company_size size,financestage from lagou;
 
 
--步骤2

    --删除原表字段
    alter table lagou drop company_short_name;
    alter table lagou drop company_full_name;
    alter table lagou drop company_size;
    alter table lagou drop financestage;


--步骤3、4、5
--创建lagou_city省市县表。

    create table lagou_city as 
      select d.id as id,p.cityName as province,c.cityName as city,d.cityName as district from s_provinces d 
        inner join s_provinces c on c.id=d.parentid 
        inner join s_provinces p on p.id=c.parentid
      where c.depth=2 --内连接西过滤city列的数据
      union
      select c.id as id,p.cityName as province,c.cityName as city,null from s_provinces p
        inner join s_provinces c on p.id=c.parentid 
      where c.depth=2; 


--步骤9、10、11
--创建lagou_position表，将查询的数据插入其中
create table lagou_position as
  --lagou表中district为空的数据，为地区为空的城市取得编号
  
      select l.pid,c.id cid,l.position,l.field,l.salary_min,l.salary_max,l.workyear,
      l.education,l.ptype,l.pnature,l.advantage,l.published_at,l.updated_at,l.company_id
      from lagou_city c,lagou l where c.city like concat(l.city,'%') and c.district is null and l.district is null;
      
--查询district不为空时，取得区的编号，并插入数据

    insert into lagou_position 
      select l.pid,c.id cid,l.position,l.field,l.salary_min,l.salary_max,l.workyear,
      l.education,l.ptype,l.pnature,l.advantage,l.published_at,l.updated_at,l.company_id
      from lagou_city c,lagou l where c.city like concat(l.city,'%') and c.district like (l.district,'%') 
      and c.district is not null and l.district is not null;