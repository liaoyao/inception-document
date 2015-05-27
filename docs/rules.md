#Inception 审核规范及原则
##写在前面的话

MySQL语句的审核，在业界都已经基本被认同了，实际上也是对MySQL语句写法的统一化，标准化，而之前的人工审核，针对标准这个问题其实是很吃力的，标准越多，DBA越累，开发也越累。那么Inception出现之后，它是不怕标准多的，只要DBA定义了为了更好的管理数据库的规则时，Inception就可以很好的执行，并且返回审核结果，那么这一章主要介绍当前Inception在审核时，是使用什么样的规则来做的，并且哪些规则是可以配置的，哪些规则是不可以配置的，这样针对不同的部门或者公司，可以定制不同的规则，而配置参数将在<<**Inception所支持的参数变量**>>中详细介绍。

下面就是有关审核规范的具体信息，但一些人为的或者逻辑的东西机器没办法搞定的，需要我们DBA自己定义，所有制定的规则，都是以语法正确为前提的，当然首先审核的就是语法问题，如果语法不通过，则整个语句块中，语法出问题的位置前面的语句审核完成，后面则不能将各个语句分开，报错时后面就被认为是一个语句。
##支持的语句类型
* use db：此时会检查这个库是不是存在，需要连接到线上服务器来判断。  
* set option：现在只需要支持set names charset，设置其它变量时都报错不支持。  
* 创建数据库语句  
* 插入语句（包括多值插入）  
* 查询插入语句  
* 删除语句（包括多表删除）  
* 更新语句（包括多表更新）  
* 创建表语句  
* 删除表语句  
* 修改表语句  
* Truncate表语句  
* inception命令集语句（包括管理命令）  
**注**：暂时不支持表链接更新及删除操作

##插入语句检查项
* 表是否存在  
* 必须指定插入列表，也就是要对哪几个列指定插入值，如insert into t (id,id2) values(...)，（可配置）  
* 必须指定值列表，与上面对应的列，插入的值是什么，必须要指定。  
* 插入列列表与值列表个数相同，上面二者的个数需要相同，如果没有指定列列表（因为可配置），则值列表长度要与表列数相同。  
* 不为null的列，如果插入的值是null，报错（可配置）  
* 插入指定的列名对应的列必须是存在的。  
* 插入指定的列列表中，同一个列不能出现多次。
* 插入值列表中的简单表达式会做检查，但具体包括什么不一一指定

##更新、删除语句检查项
* 表是否存在  
* 必须有where条件（可配置）  
* delete语句不能有limit条件（可配置）  
* 不能有order by语句（可配置）  
* 影响行数大于10000条，则报警（数目可配置）  
* 对WHERE条件这个表达式做简单检查，具体包括什么不一一指定
* 对更新列的值列表表达式做简单检查，具体不一一指定
* 对更新列对象做简单检查，主要检查列是不是存在等
* 多表更新、删除时，每个表必须要存在

##建表语句检查项
###表属性的检查项
* 这个表不存在  
* 对于create table like，会检查like的老表是不是存在。  
* 对于create table db.table，会检查db这个数据库是不是存在
* 表名、列名、索引名的长度不大于64个字节  
* 如果建立的是临时表，则必须要以tmp为前缀  
* 必须要指定建立innodb的存储引擎（可配置）  
* 必须要指定utf8的字符集（字符串可配置，指定支持哪些字符集）  
* 表必须要有注释（可配置）  
* 表不能建立为分区表（可配置）  
* 只能有一个自增列  
* 索引名字不能是Primay  
* 不支持Foreign key（可配置）  
* 建表时，如果指定auto_increment的值不为1，报错（可配置）  
* 如果自增列的名字不为id，说明有可能是有意义的，MySQL这样使用比较危险，所以报警（可配置）  
###列属性的检查项
* 不能设置列的字符集（可配置）  
* 列的类型不能使用集合、枚举、位图类型。（可配置）  
* 列必须要有注释（可配置）  
* char长度大于20的时候需要改为varchar（长度可配置）  
* 列的类型不能是BLOB/TEXT。（可配置）
* 每个列都使用not null（可配置）  
* 如果列为BLOB/TEXT类型的，则这个列不能设置为NOT NULL。
* 如何是自增列，则使用无符号类型（可配置）  
* 如果自增列，则长度必须要大于等于4个字节（可配置）  
* 如果是timestamp类型的，则要必须指定默认值。  
* 对于MySQL5.5版本（包含）以下的数据库，不能同时有两个TIMESTAMP类型的列，如果是DATETIME类型，则不能定义成DATETIME DEFAULT CURRENT_TIMESTAMP及ON UPDATE CURRENT_TIMESTAMP等语句。
* 每个列都需要定义默认值，除了自增列、主键列及大字段列之外（可配置）
* 不能有重复的列名  
###索引属性检查项
* 索引必须要有名字
* 不能有外键（可配置）
* Unique索引必须要以uniq_为前缀（可配置）  
* 普通索引必须要以idx_为前缀（可配置）  
* 索引的列数不能超过5个（数目可以配置）  
* 表必须要有一个主键（可配置）  
* 最多有5个索引（数目可配置）  
* 建索引时，指定的列必须存在。  
* 索引中的列，不能重复  
* BLOB列不能建做KEY  
* 索引长度不能超过766  
* 不能有重复的索引，名字及内容  
###默认值检查项
* BLOB/TEXT类型的列，不能有非NULL的默认值
* MySQL5.5以下（含）的版本，对于DATETIME类型的列，不能有函数NOW()的默认值。
* 如果设置默认值为函数，则只能是NOW()。
* 如果默认值为NULL，但列类型为NOT NULL，或者是主键列，或者定义为自增列，则报错。
* 自增列不能设置默认值。

##修改表语句检查项
* 表是不是存在  
###创建索引检查项
* 同上面创建表中的索引检查项
###加列检查项
* 同上面创建表中的列检查项
###修改表检查项
* 表是不是存在 
* 如果语句块中存在多条对同一个表的修改语句，则建议合并成一个ALTER语句
* 列是否存在  
* 剩下的同上面创建表，创建索引，创建列，默认值等检查项一样
###删除索引检查项
* 表是不是存在  
* 检查索引是不是存在  
###修改列的默认值检查项
* 同默认值检查项
###修改表属性
* 表属性只支持对存储引擎、表注释、自增值及默认字符集的修改操作。
* 修改存储引擎时检查是不是Innodb（可配置）。
* 字符集修改检查是不是属于设置参数的值（支持字符集可配置）。
###转换表字符集
* 字符集修改检查是不是属于设置参数的值（支持字符集可配置）。

#声明
* 针对线上MySQL服务器是不是5.6以上（包含）版本，有不同的处理策略，比如在预估影响行数时，5.6可以直接对任何DML语句做EXPLAIN操作，而在5.5版本中，只支持对SELECT语句执行EXPLAIN操作，而在5.5版本中，有些DML语句是不容易直接转换为SELECT语句去做EXPLAIN，这样导致预估行数为0。
* 还是针对5.6以上版本与5.5版本的不同，DATETIME、TIMESTAMP系列类型在执行时，5.5的限制比较多，而5.6基本通用，所以这上面的处理可能在5.6及5.5版本上，相同语句返回的结果集是不同的（规则是在5.5中以在执行时报的错误信息为准），这与线上版本有关系。