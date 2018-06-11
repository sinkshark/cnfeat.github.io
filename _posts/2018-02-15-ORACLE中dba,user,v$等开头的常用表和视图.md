---
layout: post
title: ORACLE中dba,user,v$等开头的常用表和视图
date: 2018-02-15
tag: 转载
---

### 一.Oracle表明细及说明
```
1.dba_开头表 
   dba_users           数据库用户信息 
   dba_segments    表段信息 
   dba_extents        数据区信息 
   dba_objects        数据库对象信息 
   dba_tablespaces 数据库表空间信息 
   dba_data_files     数据文件设置信息 
   dba_temp_files    临时数据文件信息 
   dba_rollback_segs         回滚段信息 
   dba_ts_quotas               用户表空间配额信息 
   dba_free_space              数据库空闲空间信息 
   dba_profiles                    数据库用户资源限制信息 
   dba_sys_privs                用户的系统权限信息 
   dba_tab_privs                 用户具有的对象权限信息 
   dba_col_privs                用户具有的列对象权限信息 
   dba_role_privs                用户具有的角色信息 
   dba_audit_trail                审计跟踪记录信息 
   dba_stmt_audit_opts       审计设置信息 
   dba_audit_object             对象审计结果信息 
   dba_audit_session          会话审计结果信息 
   dba_indexes                   用户模式的索引信息

2.user_开头表
   user_objects            用户对象信息 
   user_source             数据库用户的所有资源对象信息 
   user_segments        用户的表段信息 
   user_tables              用户的表对象信息 
   user_tab_columns    用户的表列信息 
   关于这个还涉及到两个常用的例子如下：
   2.1.oracle中查询某个字段属于哪个表 
       Sql代码 
       select table_name,owner from dba_tab_columns t where t.COLUMN_NAME like upper(‘%username%’);
   2.2.oracle中查询某个表的列数 
       Sql代码   
       select count(*) from user_tab_columns where table_name= upper(‘sys_operate’)
   注:这两个例子都用到了upper这个函数，是因为在这里表名得大写，否则查出的结果不是正确的
   user_constraints 用户的对象约束信息 
   user_sys_privs   当前用户的系统权限信息 
   user_tab_privs    当前用户的对象权限信息 
   user_col_privs    当前用户的表列权限信息 
   user_role_privs   当前用户的角色权限信息 
   user_indexes              用户的索引信息 
   user_ind_columns       用户的索引对应的表列信息 
   user_cons_columns    用户的约束对应的表列信息 
   user_clusters              用户的所有簇信息 
   user_clu_columns      用户的簇所包含的内容信息 
   user_cluster_hash_expressions   散列簇的信息

3.v$开头表
   vdatabase数据库信息vdatafile        数据文件信息 
   vcontrolfile控制文件信息vlogfile          重做日志信息 
   vinstance数据库实例信息vlog               日志组信息 
   vloghist日志历史信息vsga             数据库SGA信息 
   vparameter初始化参数信息vprocess       数据库服务器进程信息 
   vbgprocess数据库后台进程信息vcontrolfile_record_section   控制文件记载的各部分信息 
   vthread线程信息vdatafile_header     数据文件头所记载的信息 
   varchivedlog归档日志信息varchive_dest         归档日志的设置信息 
   vlogmnrcontents归档日志分析的DMLDDL结果信息vlogmnr_dictionary 日志分析的字典文件信息 
   vlogmnrlogs日志分析的日志列表信息vtablespace        表空间信息 
   vtempfile临时文件信息vfilestat              数据文件的I/O统计信息 
   vundostatUndo数据信息vrollname           在线回滚段信息 
   vsession会话信息vtransaction       事务信息 
   vrollstat回滚段统计信息vpwfile_users     特权用户信息 
   vsqlarea当前查询过的sql语句访问过的资源及相关的信息vsql&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 与vsqlarea基本相同的相关信息vsysstat            数据库系统状态信息 

4.all_开头表 
   all_users                数据库所有用户的信息 
   all_objects              数据库所有的对象的信息 
   all_def_audit_opts   所有默认的审计设置信息 
   all_tables                所有的表对象信息 
   all_indexes             所有的数据库对象索引的信息 

5.session_开头表 
   session_roles    会话的角色信息 
   session_privs    会话的权限信息 

6.index_开头表 
    index_stats      索引的设置和存储信息
```
### 二.oracle最重要的9个动态性能视图
```
    vsession+vsession_wait (在10g里功能被整合,凑合算1个吧.) 
    vprocessvsql 
    vsqltextvbh&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (更宁愿是xbh)vlock 
    vlatchchildrenvsysstat 
    v$system_event
```
### 三.按组分的几组重要的性能视图 
```
1.System的over view

     vsysstat,vsystem_event,v$parameter 

2.某个session的当前情况

     vprocess,vsession,vsessionwait,vsession_event,v$sesstat 

3.SQL的情况

     vsql,vsqlarea,vSQLPLAN,VSQL_PLAN_STATISTICS,v$sqltext_with_newlines 

4.Latch/lock/ENQUEUE

     vlatch,vlatch_children,vlatchholder,vlock,VENQUEUESTAT,VENQUEUE_LOCK 

5.IO方面的

     vsegstat,vfilestat,vtempstat,vdatafile,v$tempfile 

6.share pool/Libary cache

     vLibrarycache,vrowcache,x$ksmsp 

7.几个advice也不错

     vdbcacheadvice,vPGA_TARGET_ADVICE,v$SHARED_POOL_ADVICE

2.分类
视图有三种实例：user_*(当前用户所拥有对象的有关信息)，all_*(当前用户可访问对象的信息)，dba_*(数据库中所有对象的信息)。
查询数据字典数据时使用大写字母。可用upper\lower函数转换。
一下以user_*实例举例（如没有user_*，则依次以all_*、dba_*举例）。
3.类别
3.1 关系群集、表、视图
群集      user_clusters                   群集           
           user_cluster_hash_expressions   群集使用的群集散列函数
           user_clu_columns                表列到群集列的映射(无all_*)
表和视图  user_tables                     关系表
           user_all_tables                 表(user_all_tables是user_tables与user_object_tables的集合)
           user_tab_columns                表列
           user_tab_comments               表的注释
        user_col_comments               表和视图的列的注释
           user_refs                       对象类型列的属性和REF列
           user_partial_drop_tabs          被部分放弃的表              
              user_unused_col_tabs            带有未使用列的表
              user_updatable_columns          联合视图中更新的列
              user_views                      视图 
3.2 集合、LOB、对象类型和对象表
    集合      user_coll_types                 集合类型
              user_varrays                    varray数组类型
              user_nested_tables              嵌套表
    大型对象  user_lobs                       LOB
    对象表类型user_types                      对象类型
        user_type_attrs                 对象类型的属性
              user_type_methods               对象类型的方法
              user_object_tables              对象表             
3.3 oracle视图
设备环境  all_conext(all_context)         设备环境
维数      user_dimensions                 维数
           user_dim_hierarchies            维数分层 
           user_dim_levels                 维数的级别
           user_dim_level_key              维数级别的列 
           user_dim_attributes             维数的相关列和维数级之间的关系
           user_dim_child_of               维数级别之间的关系
           user_dim_join_key               维数间的联合
操作符    user_operators                  操作符的基本信息
           user_opancillary                操作符的辅助信息
           user_oparguments                操作符的参数
           user_opbindings                 操作符的绑定功能           
摘要      user_outlines                   摘要
           user_outline_hints              摘要的提示
策略      user_policies                   表和视图的策略 
3.4 其他数据库对象
数据库任务 user_jobs                      数据库任务
数据库连接 user_db_links                  数据库连接
目录       all_directories                目录
库         user_libraries                 库(字典)
序列       user_sequences                 序列
替代名     user_synonyms                  替代名(同义词)
3.5 分区和子分区
user_part_tables                          已分区表
    user_tab_partitions                       表的分区
    user_tab_subpartitions                    表的子分区        
user_part_indexes                         已分区索引
user_ind_partitions                       索引分区
user_ind_subpartitions                    索引子分区
user_part_lobs                            表中的LOB数据分区
user_lob_partitions                       LOB分区
    user_lob_subpartitions                    LOB子分区
user_part_key_columns                     已分区对象的分区关键字列
user_subpart_key_columns                  使用组合范围/散列法分区的表的子分区关键字列 
user_part_col_statistics                  表分区统计和其他信息
user_subpart_col_statistics               表子分区的列统计
user_part_histograms                      表的分区的直方图
user_subpart_histograms                   表的子分区的直方图
3.6 索引
user_indexes                               索引
    user_ind_columns                           索引列
    user_ind_expressions                       索引的函数索引表达式
    user_indextypes                            索引类型
    user_indextype_operators                   索引类型支持的操作符
3.7 实现视图、摘要、快照
实现视图  user_mviews                      物化视图
           user_mview_logs                  物化视图日志
           user_mview_comments              物化视图注释
      user_mview_refresh_times         物化视图刷新时间
      user_mview_analysis              物化视图的附加信息                       
      user_mview_detail_relations      物化视图FROM列表的详细关系           
           user_mview_keys                  物化视图列（或者 GROUP BY子句中的列）
           user_mview_joins                 物化视图WHERE子句中各列间的联合
     user_mview_aggregates            物化视图选择列表中的成组功能      
    快照      user_refresh                     快照刷新组
              user_refresh_children            快照刷新组的对象
              user_snapshots                   快照
              user_snapshot_logs               快照日志              
              user_snapshot_refresh_time       快照的刷新次数
              user_registered_snapshots        已注册快照
              all_refresh_dependencies         快照的从属或容器表 （只要all一种）             
    摘要      user_summaries                   摘要
              user_summary_detail_tables       摘要FROM列表的详细关系
              user_summary_keys                摘要列（或者 GROUP BY子句中的列）
              user_summary_joins               摘要WHERE子句中各列间的联合
              user_summary_aggregates          摘要选择列表中的成组功能             
3.8 子程序、方法、触发器
    子程序    user_procedures                  子程序名（包括过程、函数、包）
              user_arguments                   子程序参数（包括过程、函数、包）
    方法      user_method_params               对象类型方法的参数
           user_method_results              对象类型方法的返回值
    触发器    user_triggers                    触发器
        user_trigger_cols                触发器的列           
3.9 源代码和编译错误
    源代码    user_source        包，包体，函数，过程，对象类型，对象类型体的源代码
    编译错误  user_errors        视图和包，包体，函数，过程的编译错误
3.10 相关和限制
相关     user_dependencies                对象之间的相关（引用）
限制     user_constraints                 表的限制(约束)
          user_cons_columns                约束的列
          user_cons_obj_columns          
3.11 统计和审计
    统计     user_ustats                     对象的统计
             user_tab_col_statistics         表列的统计
             user_tab_histograms             表和视图的直方图
             user_associations               数据库对象的用户自定义统计
    审计     all_def_audit_opts              对象的默认审计选项
             audit_actions                   审计跟踪类型码的说明    
3.12 权限和授权
    系统权限   user_sys_privs                用户系统权限
    表权限     user_tab_privs                授予作为拥有者、授权者、权限受让者对象的权限
               user_tab_privs_made           授予当前用户对象的权限
               all_tab_privs_recd            授予作为权限受让者的用户对象的权限
    列权限     user_col_privs                授予作为拥有者、授权者、或受让者的用户的可授访问表或视图列的权限
               user_col_privs_made           授予当前用户表或视图列的权限
               user_col_privs_recd           授予作为权限受让者用户表或视图列权限
    
4.属性
    表(视图)的属性(列)信息查看sql如下(以视图user_coll_types举例):
select lower(column_name) column_name,nullable,decode(data_type,’VARCHAR2’,data_type||’(‘||char_length||’)’,data_type) data_type
from dba_tab_columns 
where lower(table_name)=’user_coll_types’ order by column_id;
–》调整类型
select lower(column_name) column_name,nullable,data_type||’(‘||data_length||’)’ data_type
from dba_tab_columns 
where lower(table_name)=’user_coll_types’ order by column_id;  
4.1 集合、LOB、对象类型和对象表
1.集合
  user_coll_types                 集合类型
     user_varrays                    varray数组类型
     user_nested_tables              嵌套表
   user_coll_types
     列       是否为空      类型                    说明
  type_name N VARCHAR2(30)                   集合名称
  coll_type N VARCHAR2(30)                   集合类型，可以是表或可变数组
  upper_bound Y NUMBER                         数组类型元素的最大上限
  elem_type_mod Y VARCHAR2(7)                元素类型修改符（如REF）
  elem_type_owner Y VARCHAR2(30)               元素类型的拥有者（只在与集合自身的拥有者不同时有效）
  elem_type_name Y VARCHAR2(30)               元素类型的名称
  length Y NUMBER                             如果元素类型是CHAR或VARCHAR2，则为其长度
  precision Y NUMBER                         如果元素是NUMBER类型，则为精度
  scale Y NUMBER                             如果元素是NUMBER类型，则为比例
  character_set_name Y VARCHAR2(44)           在说明了CHARCS或NCHARCS时为字符集名称。
  elem_storage Y VARCHAR2(7)                Oracle8i中使用的若干varray元素的存储字符。
  nulls_stored Y VARCHAR2(3)                Oracle8i中在存储了varray元素空信息时使用。
     user_varrays  
     列       是否为空      类型                    说明     
  parent_table_name Y VARCHAR2(30)           包括表的名称
  parent_table_column Y VARCHAR2(4000)         带有集合的包括表的拥有者
  type_owner Y VARCHAR2(30)                   集合类型的拥有者
  type_name Y VARCHAR2(30)                   集合类型的名称
  lob_name Y VARCHAR2(30)                   如果在LOB中有集合的话，则为该LOB的名称
  storage_spec Y VARCHAR2(30)               为DEFAULT或USER SPECIFIED。
  return_type Y VARCHAR2(20)                   列的返回类型
  element_substitutable Y VARCHAR2(25)
   user_nested_tables
     列       是否为空      类型                    说明   
  table_name Y VARCHAR2(30)                   如果在LOB中有集合的话，则为该LOB的名称
  table_type_owner Y VARCHAR2(30)           集合类型的拥有者
  table_type_name Y VARCHAR2(30)               集合类型的名称
  parent_table_name Y VARCHAR2(30)           包括表的名称
  parent_table_column Y VARCHAR2(4000)         带有集合的包括表的拥有者
  storage_spec Y VARCHAR2(30)               为DEFAULT或USER SPECIFIED。
  return_type Y VARCHAR2(20)                   列的返回类型
  element_substitutable Y VARCHAR2(25)
    2.大型对象  
      user_lobs                       LOB
     列       是否为空      类型                    说明      
  table_name Y VARCHAR2(30)                   包括LOB的表的名称
  column_name Y VARCHAR2(4000)                 LOB列或属性的名称
  segment_name Y VARCHAR2(30)               LOB段的名称
  tablespace_name Y VARCHAR2(30)               LOB所在表空间
  index_name Y VARCHAR2(30)                   LOB索引的名称
  chunk Y NUMBER                             以字节为分配或操作单位的LOB块长度。
  pctversion Y NUMBER                         用于存储版本信息的LOB的最大百分比。
  retention Y NUMBER
  freepools Y NUMBER
  cache Y VARCHAR2(10)                       如果LOB可使用缓冲区为YES，否则为NO。
  logging Y VARCHAR2(7)                        如果记录了LOB的变更则为YES，否则为NO。
  in_row Y VARCHAR2(3)                        如果LOB使用基行存储的话为YES，否则为NO。   
  format Y VARCHAR2(15)
  partitioned Y VARCHAR2(3)
    3.对象表类型
     user_types                      对象类型
  user_type_attrs                 对象类型的属性
  user_type_methods               对象类型的方法
  user_object_tables              对象表
      user_types                      
  type_name N VARCHAR2(30)                   对象类型的名称
  type_oid N RAW(16)                        类型的对象标识符（OID）
  typecode Y VARCHAR2(30)                   类型OBJECT，TABLE，VARCHAR2，NUMBER等的类型码
  attributes Y NUMBER(22)                     类型属性个数
  methods Y NUMBER(22)                         类型方法的个数
  predefined Y VARCHAR2(3)                    如果类型是预定义的则为YES，如果是用户定义的则为NO。
  incomplete Y VARCHAR2(3)                    如果类型不完整则为YES，否则为NO。 
  final Y VARCHAR2(3)
  instantiable Y VARCHAR2(3)
  supertype_owner Y VARCHAR2(30)
  supertype_name Y VARCHAR2(30)
  local_attributes Y NUMBER(22)
  local_methods Y NUMBER(22)
  typeid Y RAW(16)    
   user_type_attrs    
  type_name N VARCHAR2(30)            对象类型的名称
  attr_name N VARCHAR2(30)            属性的名称
  attr_type_mod Y VARCHAR2(7)         属性的类型修改符（如REF）
  attr_type_owner Y VARCHAR2(30)        如果是用户定义的类型，则为属性类型的拥有者
  attr_type_name Y VARCHAR2(30)        如果是用户定义的类型，则为属性类型的名称
  length Y NUMBER                      CHAR或VARCHAR2属性的长度。
  precision Y NUMBER                  NUMBER属性的精度。
  scale Y NUMBER                      NUMBER属性的比例
  character_set_name Y VARCHAR2(44)    说明的属性字符集
  attr_no N NUMBER                      在起始语句CREATE TYPE中说明的属性位置。 
  inherited Y VARCHAR2(3)                
   user_type_methods              
  type_name N VARCHAR2(30)            对象类型的名称
  method_name N VARCHAR2(30)            方法的名称
  method_no N NUMBER(22)              用于区别重载方法的方法号
  method_type Y VARCHAR2(6)             方法类型，可以是M A P、O R D E R、P U B L I C之一。
  parameters N NUMBER(22)              方法的参数个数
  results N NUMBER(22)                  方法返回结果个数
  final Y VARCHAR2(3)
  instantiable Y VARCHAR2(3)
  overriding Y VARCHAR2(3)
  inherited Y VARCHAR2(3)   
   user_object_tables             
  table_name N VARCHAR2(30)
  tablespace_name Y VARCHAR2(30)
  cluster_name Y VARCHAR2(30)
  iot_name Y VARCHAR2(30)
  status Y VARCHAR2(8)
  pct_free Y NUMBER(22)
  pct_used Y NUMBER(22)
  ini_trans Y NUMBER(22)
  max_trans Y NUMBER(22)
  initial_extent Y NUMBER(22)
  next_extent Y NUMBER(22)
  min_extents Y NUMBER(22)
  max_extents Y NUMBER(22)
  pct_increase Y NUMBER(22)
  freelists Y NUMBER(22)
  freelist_groups Y NUMBER(22)
  logging Y VARCHAR2(3)
  backed_up Y VARCHAR2(1)
  num_rows Y NUMBER(22)
  blocks Y NUMBER(22)
  empty_blocks Y NUMBER(22)
  avg_space Y NUMBER(22)
  chain_cnt Y NUMBER(22)
  avg_row_len Y NUMBER(22)
  avg_space_freelist_blocks Y NUMBER(22)
  num_freelist_blocks Y NUMBER(22)
  degree Y VARCHAR2(10)
  instances Y VARCHAR2(10)
  cache Y VARCHAR2(5)
  table_lock Y VARCHAR2(8)
  sample_size Y NUMBER(22)
  last_analyzed Y DATE(7)
  partitioned Y VARCHAR2(3)
  iot_type Y VARCHAR2(12)
  object_id_type Y VARCHAR2(16)
  table_type_owner Y VARCHAR2(30)
  table_type Y VARCHAR2(30)
  temporary Y VARCHAR2(1)
  secondary Y VARCHAR2(1)
  nested Y VARCHAR2(3)
  buffer_pool Y VARCHAR2(7)
  row_movement Y VARCHAR2(8)
  global_stats Y VARCHAR2(3)
  user_stats Y VARCHAR2(3)
  duration Y VARCHAR2(15)
  skip_corrupt Y VARCHAR2(8)
  monitoring Y VARCHAR2(3)
  cluster_owner Y VARCHAR2(30)
  dependencies Y VARCHAR2(8)
  compression Y VARCHAR2(8)
  dropped Y VARCHAR2(3)   
4.2 其他数据库对象
1.数据库任务 
   user_jobs    
  job N NUMBER                         任务ID号。只要该任务存在，该ID就保持不变
  log_user N VARCHAR2(30)           提交任务的用户
  priv_user N VARCHAR2(30)           默认权限适用于该任务的用户
  schema_user N VARCHAR2(30)           任务的默认模式
  last_date Y DATE                   任务上一次执行成功的日期
  last_sec Y VARCHAR2(8)            意义与last_date相同，为HH24：MI：SS格式（只有时间格式）
  this_date Y DATE                   任务开始执行的日期。如果任务没有开始则为空。
  this_sec Y VARCHAR2(8)            任务开始执行的时间，只有时间格式HH24：MI：SS
  next_date N DATE                   当任务将在下一次执行时的日期
  next_sec Y VARCHAR2(8)            当任务将在下一次执行时的时间，只有时间格式HH24：MI：SS
  total_time Y NUMBER                 系统在任务上的总时间开销（以秒为单位）
  broken Y VARCHAR2(1)                如果任务中断则为Y，否则为N。
  interval N VARCHAR2(200)          时间间隔：用来计算next_date值的日期函数。
  failures Y NUMBER                 自从成功执行上一个任务后的失败次数
  what Y VARCHAR2(4000)             构成匿名PL/SQL块的包体。限长4000字节
  nls_env Y VARCHAR2(4000)             任务的NLS环境（由ALTER SESSION说明）     
  misc_env Y RAW                    任务其他会话的参数
  instance Y NUMBER             在Oracle8i下运行该任务的OPS环境的实例(数据库实例)
   dba_jobs_running
  sid Y NUMBER                          正在运行任务的进程的进程标识符
  job Y NUMBER                          任务号
  failures Y NUMBER                  自从上次成功运行后任务执行失败的次数
  last_date Y DATE                    该任务成功运行的最后日期
  last_sec Y VARCHAR2(8)             与last_date相同，但以字符格式返回，只有时间格式HH24：MI：SS
  this_date Y DATE                    按计划下次运行任务的日期
  this_sec Y VARCHAR2(8)             与this_date相同，但以字符格式返回，只有时间格式HH24：MI：SS
  instance Y NUMBER                 在Oracle8i下运行该任务的OPS环境的实例(数据库实例)
2.数据库连接 user_db_links   
  db_link N VARCHAR2(128)               数据库连接名
  username Y VARCHAR2(30)            将使用连接的用户名
  password Y VARCHAR2(30)            将使用连接的用户密码
  host Y VARCHAR2(2000)              用于连接的Net8字符串：主机地址
  created N DATE                     数据库连接创建的日期
3.目录       all_directories                
  owner N VARCHAR2(30)                 所有者
  directory_name N VARCHAR2(30)         目录名
  directory_path Y VARCHAR2(4000)      目录所在的文件系统的操作系统路径
4.库(字典)         user_libraries                
  library_name N VARCHAR2(30)         库名
  file_spec Y VARCHAR2(2000)           库所在文件目录的操作系统路径及库文件(如:dll)
  dynamic Y VARCHAR2(1)                  如果该库是动态的(.dll)，则为Y，否则为N
  status Y VARCHAR2(7)                  库状态—VALID或INVALID
5.序列       user_sequences    
  sequence_name N VARCHAR2(30)         序列名
  min_value Y NUMBER                   序列的起始值
  max_value Y NUMBER                   序列的终止值
  increment_by N NUMBER               步长：为每个NEXTVAL增加的序列数的取值
  cycle_flag Y VARCHAR2(1)              如果在极限达到时回转则为Y，否则为N
  order_flag Y VARCHAR2(1)              如果按顺序生成序列数则为Y，否则为N
  cache_size N NUMBER                   缓冲序列数的个数
  last_number N NUMBER                  写入磁盘的最后序列数。该数可能与CURRVAL不同
6.替代名(同义词)     user_synonyms     
  synonym_name N VARCHAR2(30)         替代名的名称
  table_owner Y VARCHAR2(30)             由synonym引用对象的拥有者
  table_name N VARCHAR2(30)             由synonym引用对象的名称
  db_link Y VARCHAR2(128)              由远程synonym引用的数据库连接 
4.3 子程序、方法、触发器
    1.子程序    user_procedures                  子程序名（包括过程、函数、包）
                user_arguments                   子程序参数（包括过程、函数、包）
   user_procedures  
  object_name N VARCHAR2(30)             子程序的名称
  procedure_name Y VARCHAR2(30)         包下子程序的名称
  aggregate Y VARCHAR2(3)                聚集；集合
  pipelined Y VARCHAR2(3)                管道；传递途径
  impltypeowner Y VARCHAR2(30)
  impltypename Y VARCHAR2(30)
  parallel Y VARCHAR2(3)                并行
  interface Y VARCHAR2(3)                接口
  deterministic Y VARCHAR2(3)
  authid Y VARCHAR2(12)               
      user_arguments
  object_name Y VARCHAR2(30)             子程序的名称
  package_name Y VARCHAR2(30)         包名：如果子程序在包中的话，则为该包的名称
  object_id N NUMBER                   对象号：对子程序进行的编号
  overload Y VARCHAR2(40)             重载子程序的唯一标识符
  argument_name Y VARCHAR2(30)         参数名
  position N NUMBER                   参数在表中的位置，或在函数返回值为空
  sequence N NUMBER                   包括在嵌套层中的参数序列
  data_level N NUMBER                   复合类型（表或记录）参数的层次
  data_type Y VARCHAR2(30)             参数的数据类型
  default_value Y LONG                 说明的默认值
  default_length Y NUMBER               默认参数的长度
  in_out Y VARCHAR2(9)                  参数模式为IN，OUT，IN OUT之一
  data_length Y NUMBER                   按字节计算的参数长度
  data_precision Y NUMBER               参数精度
  data_scale Y NUMBER                   参数比例
  radix Y NUMBER                       参数的表示基数
  character_set_name Y VARCHAR2(44)     说明的参数字符集
  type_owner Y VARCHAR2(30)             用户定义参数类型的拥有者
  type_name Y VARCHAR2(30)             用户定义参数类型的名称
  type_subname Y VARCHAR2(30)         用户定义附属类型的名
  type_link Y VARCHAR2(128)            远程用户定义类型时的数据库连接名称
  pls_type Y VARCHAR2(30)
  char_length Y NUMBER
  char_used Y VARCHAR2(1)                   
    2.方法      user_method_params               对象类型方法的参数
             user_method_results              对象类型方法的返回值
   user_method_params
  type_name N VARCHAR2(30)             对象类型的名称
  method_name N VARCHAR2(30)             方法名
  method_no N NUMBER                   方法号（用于区别重载的方法）
  param_name N VARCHAR2(30)             参数名
  param_no N NUMBER                   参数个数或位置
  param_mode Y VARCHAR2(6)              参数模式（IN、OUT、IN OUT）
  param_type_mod Y VARCHAR2(7)          参数类型修改符（如REF）
  param_type_owner Y VARCHAR2(30)     参数类型拥有者
  param_type_name Y VARCHAR2(30)         参数类型名
  character_set_name Y VARCHAR2(44)     定义的参数字符集
    user_method_results  
  type_name N VARCHAR2(30)             对象类型名称
  method_name N VARCHAR2(30)             方法名称
  method_no N NUMBER                   方法号（用于区别重载的方法）
  result_type_mod Y VARCHAR2(7)          返回值的类型修改符（如REF）
  result_type_owner Y VARCHAR2(30)     如果是用户定义的类型，则为返回值类型的拥有者
  result_type_name Y VARCHAR2(30)     如果是用户定义的类型，则为返回值类型的名称
  character_set_name Y VARCHAR2(44)     定义的返回值字符集         
    3.触发器    user_triggers                    触发器
          user_trigger_cols                触发器的列
   user_triggers    
  trigger_name Y VARCHAR2(30)         触发器名称
  trigger_type Y VARCHAR2(16)         触发器类型，包括：BEFORE EACH ROW，AFTER EACH ROW，BEFORE STATEMENT，AFTER STATEMENT，INSTEAD OF
  triggering_event Y VARCHAR2(227)    触发事件，包括：INSERT，UPDATE，DELETE及其任意组合(如:INSERT OR UPDATE OR DELETE)
  table_owner Y VARCHAR2(30)             表的所有者
  base_object_type Y VARCHAR2(16)     基本对象类型(表的类型)，包括：TABLE，VIEW
  table_name Y VARCHAR2(30)             表名(含视图名)
  column_name Y VARCHAR2(4000)           列名:用于触发器的列名称
  referencing_names Y VARCHAR2(128)    参照名:REFERENCING NEW AS NEW OLD AS OLD
  when_clause Y VARCHAR2(4000)           条件：触发动作需要满足的条件
  status Y VARCHAR2(8)                  状态：enable,disable
  description Y VARCHAR2(4000)           描述：触发器头
  action_type Y VARCHAR2(11)             动作类型：PL/SQL  
  trigger_body Y LONG              触发器体： 触发器体完全放置在Long字段中，导致触发器不能很长，可将独立功能用PROCEDURE实现，在触发器中调用即可。
   user_trigger_cols
  trigger_owner Y VARCHAR2(30)         触发器的所有者
  trigger_name Y VARCHAR2(30)         触发器名称
  table_owner Y VARCHAR2(30)             表的所有者
  table_name Y VARCHAR2(30)             表名(含视图名)
  column_name Y VARCHAR2(4000)           用于触发器的列名称
  column_list Y VARCHAR2(3)              如果在update子句中说明该列的话，则为YES，否则为NO。
  column_usage Y VARCHAR2(17)         说明列在触发器中的引用方式。它可以带有操作符NEW，OLD，IN，OUT，IN OUT 的组合。        
4.4 源代码和编译错误
    1.源代码    user_source        包，包体，函数，过程，对象类型，对象类型体的源代码
  name Y VARCHAR2(30)                  内置对象的名称
  type Y VARCHAR2(12)                  对象类型
  line Y NUMBER                        当前源代码行的行号
  text Y VARCHAR2(4000)                当前行的源文本  
    2.编译错误  user_errors        视图和包，包体，函数，过程的编译错误
  name N VARCHAR2(30)                  对象名
  type Y VARCHAR2(12)                  对象类型
  sequence N NUMBER                    错误序号（针对同一个对象的多个错误）
  line N NUMBER                        错误行号：错误所在的行号
  position N NUMBER                    错误位置号：错误所在的行中以零为基数的偏移量
  text N VARCHAR2(4000)                包括错误代码和错误信息在内的错误文本
  attribute Y VARCHAR2(9)               错误类型
  message_number Y NUMBER                错误编号：oracle对错误的解析编号
4.5 相关和限制
1.相关     user_dependencies                对象之间的相关（引用）
   user_dependencies 
  name N VARCHAR2(30)                      对象名称
  type Y VARCHAR2(17)                      对象类型，可以是PROCEDURE、FUNCTION、PACKAGE、PACKAGE BODY、TYPE、TYPE BODY、TRIGGER或JAVA CLASS(Oracle8i使用)。
  referenced_owner Y VARCHAR2(30)          相关对象的所有者
  referenced_name Y VARCHAR2(64)              相关对象名称
  referenced_type Y VARCHAR2(17)              相关对象类型
  referenced_link_name Y VARCHAR2(128)     与引用对象连接的数据库连接名称（在引用对象为远程数据库时）
  schemaid Y NUMBER                        模式序号(等价于userid的值)
  dependency_type Y VARCHAR2(4)               相关类型：HARD（确实、接近），REF（参考）物化视图与表的相关性  
2.限制     user_constraints                 表的限制(约束)
   user_constraints                               约束
  owner N VARCHAR2(30)                     所有者
  constraint_name N VARCHAR2(30)             约束名
  constraint_type Y VARCHAR2(1)              约束类型 包括：P，U，R，C
  table_name N VARCHAR2(30)                 表名
  search_condition Y LONG(0)              约束类型为C时的约束(条件)
  r_owner Y VARCHAR2(30)                     外键关联的主键的所有者
  r_constraint_name Y VARCHAR2(30)         外键关联的主键
  delete_rule Y VARCHAR2(9)                  级联删除规则：NO ACTION 不做处理，SET NULL 设置为空，CASCADE 级联删除
  status Y VARCHAR2(8)                      状态：enable 有效,disable 无效
  deferrable Y VARCHAR2(14)                   是否延期  NOT DEFERRABLE
  deferred Y VARCHAR2(9)                    延期处理类型  IMMEDIATE
  validated Y VARCHAR2(13)                   经过验证的  VALIDATED
  generated Y VARCHAR2(14)                   生成的；发生的
  bad Y VARCHAR2(3)
  rely Y VARCHAR2(4)                      依赖
  last_change Y DATE(7)                      最末一次修改时间
  index_owner Y VARCHAR2(30)                 相关索引的所有者
  index_name Y VARCHAR2(30)                 相关索引(名)
  invalid Y VARCHAR2(7)
  view_related Y VARCHAR2(14)
```
[转载链接](http://blog.csdn.net/donggua3694857/article/details/74347855)


转载请注明 : [sinkshark的博客](http://sinkshark.com/)