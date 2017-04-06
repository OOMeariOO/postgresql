1.提取表中的字段和字段类型
select  string_agg(context,'\n') from (
select 'create table dss_phist.${tablename}'||'(' as context
--create table
union all
--字段
select string_agg(filed,',\n') from (    
select column_name||' '||data_type||
coalesce(case
when  data_type='numeric'  then
('('||numeric_precision||','||numeric_scale||')')
else
'('||character_maximum_length||')'
end , '') 
||case when is_nullable = 'YES' then '' else ' NOT NULL' end  
||case when column_default is not null then ' default '||column_default else '' end 
as filed
from information_schema.columns where table_schema = 'dss_phist' and table_name = '${tablename}'   
order by ordinal_position
) t 
)t

2.提取表的授权和字段描述
select 'GRANT '|| privilege_type ||' ON TABLE dss_phist.${tablename} TO ' ||grantee||';' from INFORMATION_SCHEMA.role_table_grants where  
table_schema='dss_phist' and  table_name='${tablename}'
union all
select 'COMMENT ON COLUMN dss_phist..' || b.attname || ' IS ''' || COALESCE(a.description,'') ||''';'
        from pg_catalog.pg_description a,pg_catalog.pg_attribute b   
        where objoid='dss_phist.${tablename}'::regclass   
        and a.objoid=b.attrelid
        and a.objsubid=b.attnum

3.搜索所有的表名
SELECT   tablename   FROM   pg_tables
WHERE   tablename    LIKE   '%_hd'
ORDER   BY   tablename;
