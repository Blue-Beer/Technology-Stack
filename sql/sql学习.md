# sql知识汇总

## 一、后端使用sql 的场景

1. 使用flyway进行建表的版本管理

   环境变量中进行配置：

   ```yml
   spring:
     flyway:
       enable: true
       clean-disabled: true
       locations: classpath:db/migration
       baseline-on-migrate: true
   ```

   在classpath指定的目录下按照v__xxx.sql的命名方式来管理数据库的表结构。
   建表脚本：

   ```sql
   CREATE TABLE QUANTITY_SUBMITTING_POSITIONS(
     ID                        NVARCHAR2(32) NOT NULL,
     QUANTITY_SUBMITTING_ID    NVARCHAR2(32) NOT NULL,
     POSITION_ID               NVARCHAR2(32) NOT NULL,
     CONSTRAINT QUANTITY_SUBMITTING_POSITIONS_PK PRIMARY KEY(ID)
   );
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.ID IS '主键ID';
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.QUANTITY_SUBMITTING_ID IS '提量ID';
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.POSITION_ID IS '点部位ID';
   COMMENT ON TABLE QUANTITY_SUBMITTING_POSITIONS IS '提量点部位';
   CREATE UNIQUE INDEX QUANTITY_SUBMITTING_POSITIONS_UNIQUE ON QUANTITY_SUBMITTING_POSITIONS(QUANTITY_SUBMITTING_ID,POSITION_ID);
   ```

2. 使用mapper + sql.xml 在代码中对数据进行持久化操作和复杂的查询
  
   mapper:

   ```java
   import com.baomidou.mybatisplus.core.mapper.BaseMapper;
   import org.apache.ibatis.annotations.Mapper;
   import org.apache.ibatis.annotations.Param;
   
   import java.util.List;
   
   @Mapper
   public interface AAAAMapper extends BaseMapper<AAAAPo> {
       ProjectPricingPo findById(@Param("id") String id);
   
       List<LaborTeamPo> findByAAAAId(@Param("id") String id);
   
   }
   ```
  
   增删改的持久化操作的实现是通过 extends 的BaseMapper 来实现的，而复杂的查询操作通过在配置文件中关联的xml文件使用sql语句实现，这里只需要规定变量名称。
  
   依赖配置：
  
   ```yml
   # mybatis-plus 配置
   mybatis-plus:
     tenant-enable: ture       #多租户模式
     mapper-locations: classpath:/mapper/**/*.xml   #同名的xml文件的存储位置
     global-config:
       banner: false
       db-config:
         db-type: oracle
         id-type: auto
         select-strategy: not_empty
         insert-strategy: not_empty
         update-strategy: not_empty
     type-handlers-package:  com.cr121.communal.common.data.handler 
     configuration:
       jdbc-type-for-null: 'null'
   ```
  
   xml具体的sql语句： 这里的id 和 mapper中的代码名称相同, 代码中的变量已 #{XX} 的形式引入
  
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.cr121.yangong.biz.adapter.driven.persistence.oracle.pricing.AAAAMapper">
       <select id="findById" resultType="com.cr121.yangong.biz.adapter.driven.persistence.oracle.pricing.AAAAPo">
           select t.ID,
                  t.PROJECT_ID,
                  t.QUANTITY_SUBMIT_CUT_OFF_DATE as bizCutOffDate,
                  t.DATA_CUT_OFF_DATE            as dataCutOffDate,
                  t.PRICING_CUT_OFF_DATE         as pricingCutOffDate,
                  t.PERIOD,
                  t.PRICING_TYPE,
                  t.EXCLUDING_TAX_AMOUNT,
                  t.INCLUDING_TAX_AMOUNT,
                  t.TAXES_AMOUNT,
                  t.STATUS
           from SUBMITTING_PROJECT_PRICING t
           where t.id = #{id}
       </select>
       
       <select id="findByAAAAId" resultType="com.cr121.yangong.biz.adapter.driven.persistence.oracle.pricing.BBBBPo">
           select t.TEAM_ID,
                  t.TEAM_NAME
           from QUANTITY_SUBMITTING t
           where t.PRICING_ID = #{id}
       </select>
   </mapper>
   ```
  
3. LambdaQueryWrapper 在代码中执行简单的查询
   LambdaQueryWrapper 的基础使用：

   ```java
   LambdaQueryWrapper<AdditionalPricingPo> wrapper = new LambdaQueryWrapper<>();
           wrapper.eq(AdditionalPricingPo::getQuantitySubmittingId, quantitySubmittingId);
           List<AdditionalPricingPo> pricingPos = mapper.selectList(wrapper);
   ```

## 二、 SQL基础语法

1. **数据定义语言 DDL**
  
   ```sql
   CREATE DATABASE ZYC;
   DROP DATABASE ZYC;

   CREATE TABLE QUANTITY_SUBMITTING_POSITIONS(
     ID                        NVARCHAR2(32) NOT NULL,
     QUANTITY_SUBMITTING_ID    NVARCHAR2(32) NOT NULL,
     POSITION_ID               NVARCHAR2(32) NOT NULL,
     CONSTRAINT QUANTITY_SUBMITTING_POSITIONS_PK PRIMARY KEY(ID)
   );
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.ID IS '主键ID';
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.QUANTITY_SUBMITTING_ID IS '提量ID';
   COMMENT ON COLUMN QUANTITY_SUBMITTING_POSITIONS.POSITION_ID IS '点部位ID';
   COMMENT ON TABLE QUANTITY_SUBMITTING_POSITIONS IS '提量点部位';
   CREATE UNIQUE INDEX QUANTITY_SUBMITTING_POSITIONS_UNIQUE ON QUANTITY_SUBMITTING_POSITION (QUANTITY_SUBMITTING_ID,POSITION_ID);
   
   DROP TABLE QUANTITY_SUBMITTING_POSITIONS;

   ALTER TABLE QUANTITY_SUBMITTING_POSITIONS ADD (QUANTITY_STATUS VARCHAR(2));

   ALTER TABLE QUANTITY_SUBMITTING_POSITIONS RENAME COLUMN QUANTITY_STATUS TO QUANTITY_STATE;

   ALTER TABLE QUANTITY_SUBMITTING_POSITIONS MODIFY COLUMN QUANTITY_STATUS NUMBER(10,2);

   ALTER TABLE QUANTITY_SUBMITTING_POSITIONS DROP COLUMN QUANTITY_STATUS;
   
   ```
  
   SQL基本数据类型(oracle)：

   目前涉及到的数据类型有：varchar2(), nvarchar2(), number(,), date ...
   **varchar2(x)**: 长度最大为x的变长字符串
   **nvarchar2(x)**: 不区分中英文的长度最大为x的变长字符串
   **number(精度，尺度)**：精度表示一个number可以显示的总位数。尺度表示精确度，对小数点前后多少位维持精确度
   **date**: 用7个字节存储 年 月 日 时 分 秒。 默认顺序是：DD-MON-YY HH.MIN.SS。 在建表时，如果让后续插入的每一行数据不需要输入date而是自动生成一个默认的时间，可以使用`CREATE_TIME DATE DEFAULT SYSDATE`

2. **数据操作语言 DML**
   目前后端使用MAPPER进行基础的插入，修改和删除操作。只进行基础的使用演示：
  
   ```sql
   INSERT INTO QUANTITY_SUBMITTING_POSITIONS (QUANTITY_SUBMINTTING_ID, POSITION_ID) VALUES ('001','001');
   UPDATE QUANTITY_SUBMITTING_POSITIONS SET POSITION_ID = '002' WHERE ID '000001';
   DELETE FROM QUANTITY_SUBMITTING_POSITIONS WHERE ID = '000001';
   ```

3. **数据查询语言 DQL**
   **条件查询**：
   ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE ***** `
   - 等值查询
     ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE POSITION_ID = '0001' `
     如果查询时只需要满足多个值中的一个，可以引入 IN 操作符： ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE POSITION_ID IN ('0001','0002','0003') `
   - 范围查询
     ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE PRICE <= 10.0 `
   - 模糊查询
     ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME LIKE 'XXX' OR KEYWORD LIKE 'XXX' `
     ` SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME LIKE 'XXX' AND KEYWORD LIKE 'XXX' `
   - 子查询
     ` SELECT * FROM QUANTITY_SUBMITTING_POSITION WHERE POSITION_ID = (SELECT POSITION_ID FROM QUANTITY_SUBMITTING WHERE TEAM_ID = '1111' ) `
   - 正则查询
    `SELECT column1, column2 FROM your_table WHERE REGEXP_LIKE(column3, 'pattern');`

   **（反向）模糊查询**：
   `SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE ******`
   - 前后缀匹配 以PREFIX开头 或者 SUFFIX结尾
   `SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE 'PREFIX%'`
   `SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE '%SUFFIX'`
   - 包含匹配
   `SELECT * FROM QUANTITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE '%SUBSTRING%'`
   - 单字符匹配 正数或倒数第2个字符是Z
   `SELECT * FROM QUNATITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE ''-Z%`
   `SELECT * FROM QUNATITY_SUBMITTING_POSITIONS WHERE NAME (NOT) LIKE ''%Z-`

   **查询结果排序**：ASC 该列升序，DESC该列降序
   `**** ORDER BY QUANTITY_ID ASC, POSITION_ID DESC`

   **分组查询**：
   - 以 position_id 为分组条件来查询
   `SELECT *** FROM **** WHERE ***** GROUP BY POSITION_ID`
   - HAVING 子句限制：
    在进行查询时，优先将表内的数据先以**WHERE**进行筛选，再以**GROUP BY**分组，最后以**HAVING**子句进行约束。

   **多表连接查询**：
   ![Alt text](http://192.168.100.92:9000/picbed/2024/01/25/202401250958305.png)  
   在进行多表查询时，条件发生的优先级: on 规定的条件在连接的时候生效，不满足的均被置为NULL，where 在连表生成后生效，最后生效的是having。
   - INNER JOIN

   ```sql
   SELECT COLUMN_NAME
   FROM TABLE1
   INNER JOIN TABLE2
   ON TABLE1.COLUMN_NAME=TABLE2.COLUMN_NAME
   WHERE ******;
   ```

   - LEFT/RIGHT JOIN 返回的结果有左表的所有行，右表不具备的会显示为null。RIGHT JOIN 相反

   ```sql
   SELECT COLUMN_NAME
   FROM TABLE1
   LEFT JOIN TABLE2
   ON TABLE1.COLUMN_NAME=TABLE2.COLUMN_NAME
   WHERE ******;
   ```

   - FULL JOIN 等价于左右连接同时生效

4. **动态sql**

   对需要进行操作的语句中可能使用的条件个数不确定的时候，引入动态生成的sql语句。

   ```xml
   <delete id="deleteByIds">
     delete
     from   CONSTRUCTION_PART_AGGREGATION
     where (ID in
     <!-- 处理in的集合超过1000条时  Oracle不支持的情况 -->
     <trim suffixOverrides=" OR   (ID) IN()">
       <!-- 表示删除最后一个条件 -->
       <foreach collection="ids"   item="item" index="index"   open="(" close=")">
         <if test="index != 0">
           <choose>
             <when test="index %   1000 == 999">
               ) OR ID IN (
             </when>
             <otherwise>
             ,
             </otherwise>
           </choose>
         </if>
         (#{item})
       </foreach>
     </trim>
     )
   </delete>
   ```

  这里使用trim的字段内的sql语句是根据mapper中`void deleteByIds(@Param("ids") List<String> ids);` 传入的List中的id个数决定的。

- `suffixOverrides`：表明在trim处理结束后尾部要删除的字段。
- `collection item index open close`：
  - collection:指定输入对象中的集合属性
  - item:每次遍历生成的对象
  - open:开始遍历时的拼接字符串
  - close:结束时拼接的字符串
  - separator:遍历对象之间需要拼接的字符串

## 三、ORACLE TIPS

1. **group by VS partition by**

   group by 使用后，只能额外select用来分组的这个字段，对其他的字段来说，因为分组，信息已经被压缩消失了。
   partition by 使用后，可以查看其他所有列的信息，因为信息没有被压缩，而是在每一行都存储了分类后的信息。同时要注意，在select其他信列信息时，要加上表的名字前缀。

   使用 parttion by 的方法：`SELECT b.*, COUNT(*) OVER(PARTITION BY COLOUR) FROM TABLEB b;`

2. **注意处理NULL 的数据**

   尤其是在做反向操作NOT IN ，NOT EXIST 的时候。
