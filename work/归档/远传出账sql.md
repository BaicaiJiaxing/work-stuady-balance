### 查表数量统计

```
-- 新系统全部查表计划数量（支）
select count(*) from water_meter_read;
-- 新系统户表查表计划数量(支)
select count(*) from water_meter_read where client_type = '1';
-- 新系统户表远传纳入数量(支)
select count(*) from water_meter_read where client_type = '1' and account_opening_plan = '1';
-- 新系统大路表查表计划数量(支)
select count(*) from water_meter_read where client_type = '2';
-- 新系统大路表远传纳入数量(支)
select count(*) from water_meter_read where client_type = '2' and account_opening_plan = '1';

-- 现状系统全部查表计划数量（支）
select count(*) from wmis_meter_read;
-- 现状系统户表查表计划数量(支)
   select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT ciid FROM WMIS_CUSTINFO wc WHERE wc.CICUSTTYPE ='1')
-- 现状系统户表远传纳入数量(支)
   select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT ciid FROM WMIS_CUSTINFO wc WHERE wc.CICUSTTYPE ='1')
   AND MR_CID IN (SELECT cid FROM WMIS_YCB_CZ_CID wycc);
-- 现状系统大路表查表计划数量(支)
   select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT ciid FROM WMIS_CUSTINFO wc WHERE wc.CICUSTTYPE ='2')
-- 现状系统大路表远传纳入数量(支)
   select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT ciid FROM WMIS_CUSTINFO wc WHERE wc.CICUSTTYPE ='2')
   AND MR_CID IN (SELECT cid FROM WMIS_YCB_CZ_CID wycc);
   
-- 无老系统生产库的备用方法
   select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT client_code  FROM water_client wc WHERE wc.client_type ='2')
   union all
    select count(*) from wmis_meter_read wmr 
   where MR_CID IN (SELECT client_code  FROM water_client wc WHERE wc.client_type ='2')
   
   
-- 新系统远传入账数量（大路表）
select count(*) from water_meter_read where client_type ='2' and  mr_enter_staff = '远传' ;
-- 新系统远传入账数量（户表）
select count(*) from water_meter_read where client_type ='1' and mr_is_submit = 'Y' and mr_enter_staff = '远传' ;
-- 现状系统远传入账数量（大路表）
select count(*) from wmis_meter_read wms  left join  water_client wc
on wms.mr_cid = wc.client_code where wc.client_type = '1' and wms.mr_inputer = '远传' ; 
-- 现状系统远传入账数量（户表）
select count(*) from wmis_meter_read wms  left join  water_client wc
on wms.mr_cid = wc.client_code where wc.client_type = '2' and wms.mr_inputer = '远传' ; 
```
### 查表计划差异
```
SELECT
    CASE
        wmr.client_type
        WHEN '1' THEN '户表'
        WHEN '2' THEN '大路表'
        WHEN '3' THEN '楼门表'
    END AS "用户类型",
    wmr.abode_code AS "营业所编号",
    wmr.meter_code AS "新系统水表编号",
    WMS."MR_MID" AS "旧系统水表编号",
    wmr.mr_old_last AS "新系统起码",
    WMS."MR_SCODE" AS "旧系统起码",
    wmr.mr_last_read_time AS "新系统上次查表时间",
    WMS."MR_PRDATE" AS "旧系统上次查表时间",
    wmr.create_time as "创建时间",
    wmr.mb_code AS "新系统查表路",
    WMS."MR_BFID" AS "旧系统查表路",
    wmr.account_opening_plan as "纳入远传",
    CASE wmr.mr_manual_source
        WHEN '01' THEN '正常'
        WHEN '02' THEN '自报数'
        WHEN '03' THEN '自来水app自报数'
        WHEN '08' THEN '预约查表'
        WHEN '09' THEN '半年未见数'
        WHEN '05' THEN '营销计划管理'
        WHEN '04' THEN '追补'
    END AS "新系统计划来源",
    wmb.mb_task_month as "查表任务月份",
    wc.mb_period as "用户查表周期",
    wc.client_status as "用户状态",
    wm.meter_state as "水表状态",
    wc.create_time as "立户时间"
FROM
    water_meter_read_back_20251001 wmr
    FULL JOIN water_revenue.wmis_meter_read_20251001 WMS
        ON WMS."MR_MID" = wmr.meter_code
    left join water_meter wm on wmr.meter_code = wm.meter_code
    left join water_meter_book wmb on wmr.mb_code = wmb.mb_code
    left join water_client wc on wmr.client_code = wc.client_code
WHERE
    (wmr.meter_code IS NULL OR WMS."MR_MID" IS NULL)
    OR wmr.mr_old_last <> WMS."MR_SCODE"
    OR DATE(mr_last_read_time) <> DATE(WMS."MR_PRDATE");


-- 新系统全部查表计划数量（支）
select count(*) from water_meter_read_back_20251001;
-- 新系统户表查表计划数量(支)
select count(*) from water_meter_read_back_20251001 where client_type = '1';
-- 新系统户表远传纳入数量(支)
select count(*) from water_meter_read_back_20251001 where client_type = '1' and account_opening_plan = '1';
-- 新系统大路表查表计划数量(支)
select count(*) from water_meter_read_back_20251001 where client_type = '2';
-- 新系统大路表远传纳入数量(支)
select count(*) from water_meter_read_back_20251001 where client_type = '2' and account_opening_plan = '1';
select * from water_meter_book wmb where wmb.mb_type ='1'
-- 现状系统全部查表计划数量（支）
select count(*) from wmis_meter_read_20251001;
-- 现状系统户表查表计划数量(支)
   select count(*) from wmis_meter_read_20251001 wmr 
   where "MR_CID" IN (SELECT ciid FROM WMIS_CUSTINFO wc WHERE wc.cibfid in (select mb_code from water_meter_book wmb where wmb.mb_type ='2'
)) AND "MR_CID" IN (SELECT cid FROM WMIS_YCB_CZ_CID wycc);

```

### 查表数据

```
SELECT
    CASE wmr.meter_is_main
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "是否父表",
    CASE wmr.client_type
        WHEN '1' THEN '户表'
        WHEN '2' THEN '大路表'
    END AS "用户类型",
    CASE wmr.meter_is_billing 
        WHEN 'Y' THEN '计费'
        WHEN 'N' THEN '不计费'
    END AS "计费标识",
    wmr.client_code AS "新系统用户编号",
    wms.mr_cid AS "旧系统用户编号",
    wmr.meter_code AS "新系统水表编号",
    wms.mr_mid AS "旧系统水表编号",
    wmr.work_day as "新系统查表工作日",
     wms.mr_workdate as "旧系统查表工作日",
    wmr.mr_old_last AS "新系统起码",
    wms.mr_scode AS "旧系统起码",
    wmr.mr_old_now AS "新系统止码",
    wms.mr_ecode AS "旧系统止码",
    wmr.mr_use AS "新系统水量",
    wms.mr_usenum AS "旧系统水量",
    wms.mr_readdate AS "旧系统查表时间",
    wmr.mr_read_time AS "新系统查表时间",
    wmr.mr_last_read_time AS "上次查表时间",
    CASE wmr.mr_state
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "新系统出账标志",
    CASE wms.mr_flagra
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "旧系统出账标志",
    CASE wmr.account_status 
        WHEN '2' THEN '是'
        WHEN '1' THEN '否'
    END AS "新系统二次入账标志",
    wmr.check_status  AS "新系统查表状态",
    CASE wms.mr_flagecrz
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "旧系统二次入账标志",
    wms.mr_last_buss AS "旧系统二次入账原因",
    wms.mr_inputer AS "旧系统录入员",
    wmr.mr_enter_staff as "新系统录入员",
		wmr.mr_input_time as "新系统录入时间",
		wms.mr_inputdate as "旧系统录入时间",
		 CASE wmr.mr_manual_source
        WHEN '01' THEN '正常'
        WHEN '02' THEN '自报数'
        WHEN '03' THEN '自来水app自报数'
        WHEN '08' THEN '预约查表'
        WHEN '09' THEN '半年未见数'
        WHEN '05' THEN '营销计划管理'
        WHEN '04' THEN '追补'
    END AS "新系统计划来源",
    CASE wmr.account_opening_plan
        WHEN '1' THEN '是'
        WHEN '0' THEN '否'
    end as "是否纳入远传"
FROM
    water_meter_read wmr
    FULL JOIN wmis_meter_read wms ON wms.mr_mid = wmr.meter_code 
WHERE 
    (wmr.mr_enter_staff = '远传') or 
--     wmr.mr_enter_staff = '远传' OR wms.mr_inputer = '远传'
    (wms.mr_inputer = '远传')
--     and work_day = '1'
and client_type = '1'
    order by wmr.meter_code;
```


### 父子表
```
SELECT 
  wmm.parent_meter,
  STRING_AGG(wmm.child_meter, ',') AS child_meters,
  STRING_AGG(wmr_child.mr_use::TEXT, ',') AS child_mr_uses,
  STRING_AGG(wmr_child.check_status::TEXT, ',') AS child_check_status,
  wmr_parent.mr_use AS parent_mr_use,
  wmr_parent.check_status AS parent_check_status,
  SUM(CASE WHEN wmr_child.mr_use < 0 THEN 0 ELSE wmr_child.mr_use END) AS total_child_mr_use
FROM water_meter_mix wmm
LEFT JOIN water_meter_read wmr_child 
  ON wmm.child_meter = wmr_child.meter_code
LEFT JOIN (
  SELECT DISTINCT ON (meter_code) meter_code, mr_use, check_status
  FROM water_meter_read
  ORDER BY meter_code -- 按日期取最新记录
) wmr_parent 
  ON wmm.parent_meter = wmr_parent.meter_code
where wmm.parent_meter in 
()
GROUP BY wmm.parent_meter, wmr_parent.mr_use, wmr_parent.check_status
order by  wmm.parent_meter
;
```
### 周期换表
```
SELECT 
  wmr.meter_code,
  wmr.mr_old_last as "上次查表示数",
  wmr.mr_old_now as "本次查表示数",
  wcm.data_status as "换表审核状态",           -- 水表读数状态
  wcm.cm_change_date as "换表日期",        -- 更换日期
  wcm.cm_old_stop  as "旧表拆除示数",           -- 旧表止码
  wcm.cm_nrw_start as "新表起数",          -- 新表起码
  wcm.cm_change_audit_time,  -- 更换审核时间
  wcm.cm_old_stop - mr_old_last + mr_old_now - cm_nrw_start as "加表底水量",
  wcm.meter_is_billing,
  wcm.cm_change_type         -- 更换类型
FROM water_meter_read wmr
LEFT JOIN (
  SELECT DISTINCT ON (cm_meter_code)
    cm_meter_code,   
    meter_is_billing,       -- 被更换的旧表编号
    cm_change_date,
    cm_old_stop,
    cm_nrw_start,
    cm_change_audit_time,
    cm_change_type,
    data_status
  FROM water_change_meter
  where cm_change_date is not null
  ORDER BY cm_meter_code, cm_change_date DESC  -- 按更换日期取最新记录
) wcm ON wmr.meter_code = wcm.cm_meter_code
WHERE wmr.meter_code in ()
order by wmr.meter_code;
```
### 应收差异

城区数据量大需要针对当月应收建表
```
CREATE TEMP TABLE tmp_clients AS
SELECT DISTINCT wmr.client_code
FROM water_meter_read wmr
JOIN wmis_meter_read wms ON wms.mr_mid = wmr.meter_code
WHERE wmr.mr_enter_staff = '远传'
  AND wms.mr_inputer = '远传'
  AND wmr.mr_use = wms.mr_usenum
  AND wmr.mr_read_time = wms.mr_readdate
  AND wmr.mr_old_now = wms.mr_ecode
  AND wmr.mr_old_now IS NOT NULL
  AND wmr.account_status = '1'
  AND wms.mr_flagecrz = 'N'
  AND wmr.client_type = '1'
  AND wmr.mr_month = '2025-10';

CREATE INDEX idx_tmp_clients ON tmp_clients(client_code);

create table account_charge_meter_1010 as 
SELECT a.*
FROM account_charge_meter a
JOIN tmp_clients t ON a.client_code = t.client_code
WHERE a.mr_month = '2025-10';

create table account_charge_client_1010 AS
select * from account_charge_client where acm_id in (select cm_id from account_charge_meter_1010);   


with base_data as (
    SELECT 
         acm.cm_id AS cmid, 
         wra.raid AS raid,
         acm.client_code,
         wra.rausenum AS rause,
         acm.charge_count AS acmcount
    FROM account_charge_meter_1010   acm
    INNER JOIN wmis_rec_acc wra  -- 改为 INNER JOIN，避免不必要的无效数据
        ON (acm.charge_month, acm.client_code) = (wra.ramonth, wra.racid)
    WHERE  acm.client_code is not null
)
, detail AS (
    SELECT 
        b.client_code as "用户编号",
        wrac.racpiid as "老系统费用项",
        MAX(b.rause) AS "老系统出账水量",  -- 保持最大值，检查是否是你业务所需
        MAX(b.acmcount) AS "新系统出账水量",  -- 保持最大值，检查是否是你业务所需
        SUM(DISTINCT wrac.racmoney) AS "老系统出账金额",  -- 去重，避免重复计算
        acc.pi_code as "新系统费用项",
        SUM(DISTINCT acc.cc_price) AS "新系统出账金额"  -- 去重，避免重复计算
    FROM wmis_rec_acc_content wrac
    LEFT JOIN account_charge_client_1010  acc
           ON (wrac.racmid, wrac.racpiid) = (acc.meter_code, acc.pi_code)
    JOIN base_data b
           ON wrac.racraid = b.raid
          AND acc.acm_id = b.cmid
    GROUP BY b.client_code, wrac.racpiid, acc.pi_code
)
SELECT * from detail;
-- 


SELECT 
    COUNT(DISTINCT "用户编号") AS "用户数",
      -- 使用 DISTINCT 避免重复计算
      sum("老系统出账水量"),sum("新系统出账水量"),
    detail."老系统费用项",
    SUM("新系统出账金额") AS "新系统出账金额",
    SUM("老系统出账金额") AS "旧系统出账金额"
FROM detail
GROUP BY "老系统费用项";  


```

### 异常审核明细
```
SELECT
    CASE wmr.meter_is_main
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "是否父表",
    CASE wmr.client_type
        WHEN '1' THEN '户表'
        WHEN '2' THEN '大路表'
    END AS "用户类型",
    CASE wmr.meter_is_billing 
        WHEN 'Y' THEN '计费'
        WHEN 'N' THEN '不计费'
    END AS "计费标识",
    wmr.client_code AS "新系统用户编号",
    wms.mr_cid AS "旧系统用户编号",
    wmr.meter_code AS "新系统水表编号",
    wms.mr_mid AS "旧系统水表编号",
    wmr.work_day as "新系统查表工作日",
     wms.mr_workdate as "旧系统查表工作日",
    wmr.mr_old_last AS "新系统起码",
    wms.mr_scode AS "旧系统起码",
    wmr.mr_old_now AS "新系统止码",
    wms.mr_ecode AS "旧系统止码",
    wmr.mr_use AS "新系统水量",
    wms.mr_usenum AS "旧系统水量",
    wms.mr_readdate AS "旧系统查表时间",
    wmr.mr_read_time AS "新系统查表时间",
    wmr.mr_last_read_time AS "上次查表时间",
    CASE wmr.mr_state
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "新系统出账标志",
    CASE wms.mr_flagra
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "旧系统出账标志",
    CASE wmr.account_status 
        WHEN '2' THEN '是'
        WHEN '1' THEN '否'
    END AS "新系统二次入账标志",
    wmr.check_status  AS "新系统查表状态",
    CASE wms.mr_flagecrz
        WHEN 'Y' THEN '是'
        WHEN 'N' THEN '否'
    END AS "旧系统二次入账标志",
    wms.mr_last_buss AS "旧系统二次入账原因",
    wms.mr_inputer AS "旧系统录入员",
    wmr.mr_enter_staff as "新系统录入员",
		wmr.mr_input_time as "新系统录入时间",
		wms.mr_inputdate as "旧系统录入时间",
		 CASE wmr.mr_manual_source
        WHEN '01' THEN '正常'
        WHEN '02' THEN '自报数'
        WHEN '03' THEN '自来水app自报数'
        WHEN '08' THEN '预约查表'
        WHEN '09' THEN '半年未见数'
        WHEN '05' THEN '营销计划管理'
        WHEN '04' THEN '追补'
    END AS "新系统计划来源",
    CASE wmr.account_opening_plan
        WHEN '1' THEN '是'
        WHEN '0' THEN '否'
    end as "是否纳入远传"
FROM
    water_meter_read wmr
    FULL JOIN wmis_meter_read wms ON wms.mr_mid = wmr.meter_code 
WHERE 
    (wmr.mr_enter_staff = '远传') or 
--     wmr.mr_enter_staff = '远传' OR wms.mr_inputer = '远传'
    (wms.mr_inputer = '远传')
--     and work_day = '1'
and client_type = '1'
    order by wmr.meter_code;
    
    
    SELECT
    wmr.client_code as "新系统用户编号",
    wc.client_type as "用户类型",
    wc.is_ladderhouse AS "阶梯标识",
    wc.meter_is_billing AS "计费标识",
    STRING_AGG(DISTINCT wmr.mr_use::text, ',') AS "查表水量（多表，分隔）",
    STRING_AGG(DISTINCT wmr.check_status, ',') AS "当前查表状态",
    SUM(CASE WHEN wmr.mr_use < 0 THEN 0 ELSE wmr.mr_use END) AS "累计水量（负水量按0相加）",
    wc.year_total_sl AS "出账前年累计",
    wc.year_total_sl + SUM(CASE WHEN wmr.mr_use < 0 THEN 0 ELSE wmr.mr_use END) AS "出账后年累计",
    wc.meter_people_amt AS "人口数",
    MAX(wmr.mr_last_read_time) AS "上次查表时间",
    MAX(wc.wp_code) AS "水价",
    MAX(wmr.mr_read_time) AS "本次查表时间",
    CASE
    WHEN wmra.check_type = 'cqwjs' THEN '长期未见数'
    WHEN wmra.check_type = 'zeroWater' THEN '水量为0'
    WHEN wmra.check_type = 'waterIsNegative' THEN '水量为负'
    WHEN wmra.check_type = 'waterFluctuationAnomaly' THEN '水量波动'
    WHEN wmra.check_type = 'superLadder' THEN '超阶梯'
    WHEN wmr.check_status = '15' THEN '提级审核'
    WHEN wmr.check_status = '5' THEN '一户多表待出账'
    WHEN wmr.check_status = '51' THEN '待子表出账'
END AS "新系统审核类型",
    wms.mr_last_buss AS "旧系统二次入账原因"
FROM
    water_meter_read wmr
left join wmis_meter_read wms on wmr.meter_code = wms.mr_mid
LEFT JOIN
    water_client wc ON wmr.client_code = wc.client_code
LEFT JOIN
    water_meter_read_abnormal wmra ON wmr.client_code = wmra.client_code AND wmr.mr_month = wmra.mr_month
WHERE
    wmr.client_code in (
    SELECT
    wmr.client_code
FROM
    water_meter_read wmr
    FULL JOIN wmis_meter_read wms ON wms.mr_mid = wmr.meter_code 
WHERE 
    wmr.mr_enter_staff = '远传' and wms.mr_inputer = '远传'
    and wmr.mr_read_time = wms.mr_readdate and wmr.mr_use = wms.mr_usenum and wmr.mr_old_now = wms.mr_ecode 
    and wmr.account_status = '2' and wms.mr_flagecrz = 'Y'
--     and work_day = '1'
and client_type = '1'
    )
GROUP BY
    wmr.client_code,
    wc.client_type,
    wc.is_ladderhouse,
    wc.meter_is_billing,
    wc.year_total_sl,
    wc.meter_people_amt,
    wmra.check_type,
    wms.mr_last_buss,
    wmr.check_status;
    
    
    
    
    
    
    
    
    
    
    
    
    
  -- 新系统远传所有审核明细  
    SELECT
    wmr.client_code as "新系统用户编号",
    wc.client_type as "用户类型",
    wc.is_ladderhouse AS "阶梯标识",
    wc.meter_is_billing AS "计费标识",
    STRING_AGG(DISTINCT wmr.mr_use::text, ',') AS "查表水量（多表，分隔）",
    STRING_AGG(DISTINCT wmr.check_status, ',') AS "当前查表状态",
    SUM(CASE WHEN wmr.mr_use < 0 THEN 0 ELSE wmr.mr_use END) AS "累计水量（负水量按0相加）",
    wc.year_total_sl AS "出账前年累计",
    wc.year_total_sl + SUM(CASE WHEN wmr.mr_use < 0 THEN 0 ELSE wmr.mr_use END) AS "出账后年累计",
    wc.meter_people_amt AS "人口数",
    MAX(wmr.mr_last_read_time) AS "上次查表时间",
    MAX(wc.wp_code) AS "水价",
    MAX(wmr.mr_read_time) AS "本次查表时间",
    CASE
    WHEN wmra.check_type = 'cqwjs' THEN '长期未见数'
    WHEN wmra.check_type = 'zeroWater' THEN '水量为0'
    WHEN wmra.check_type = 'waterIsNegative' THEN '水量为负'
    WHEN wmra.check_type = 'waterFluctuationAnomaly' THEN '水量波动'
    WHEN wmra.check_type = 'superLadder' THEN '超阶梯'
    WHEN wmr.check_status = '15' THEN '提级审核'
    WHEN wmr.check_status = '5' THEN '一户多表待出账'
    WHEN wmr.check_status = '51' THEN '待子表出账'
END AS "新系统审核类型",
    wms.mr_last_buss AS "旧系统二次入账原因"
FROM
    water_meter_read wmr
left join wmis_meter_read wms on wmr.meter_code = wms.mr_mid
LEFT JOIN
    water_client wc ON wmr.client_code = wc.client_code
LEFT JOIN
    water_meter_read_abnormal wmra ON wmr.client_code = wmra.client_code AND wmr.mr_month = wmra.mr_month
WHERE
    wmr.mr_enter_staff = '远传' and account_status ='2' and wmr.client_type ='1'
GROUP BY
    wmr.client_code,
    wc.client_type,
    wc.is_ladderhouse,
    wc.meter_is_billing,
    wc.year_total_sl,
    wc.meter_people_amt,
    wmra.check_type,
    wms.mr_last_buss,
    wmr.check_status;
    


```