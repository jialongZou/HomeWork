##实现的方法
####1.

    select a.cityName 省级, b.cityName 市级, c.cityName 区级
    from s_provinces a 
           join s_provinces b on b.parentId = a.id
           join s_provinces c on c.parentId = b.id
    where a.cityName = '广东省';
    
    
####2.
    select a.cityName 省级, b.cityName 市级, c.cityName 区级
    from s_provinces a,
         s_provinces b,
         s_provinces c
    where a.id = b.parentId
      and b.id = c.parentId
      and a.cityName = '广东省';
      
这两个都是自联查询，自己关联自己查询，用这个的原因是因为要显示的数据<br>
都是在一张表里面的,并且有可以体现**数据间不同深度且相互关联的字段**，所以<br>
可以这样用。

第一种实现的写法是先以省为主和己表的市查，然后再以市为主和己表的区查，最后取<br>省市区
三个列显示

第二种实现的写法则是自联三次直接判断。