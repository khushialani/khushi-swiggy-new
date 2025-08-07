use role sysadmin;

use database sandbox;
use schema stage_sch;
use warehouse adhoc_wh;

list @stage_sch.csv_stg/initial/delivery/;

-- this table may have additional information like picked time, accept time etc.
create or replace table stage_sch.delivery (
    deliveryid text comment 'Primary Key (Source System)',                           -- foreign key reference as text (no constraint in snowflake)
    orderid text comment 'Order FK (Source System)',                           -- foreign key reference as text (no constraint in snowflake)
    deliveryagentid text comment 'Delivery Agent FK(Source System)',                   -- foreign key reference as text (no constraint in snowflake)
    deliverystatus text,                    -- delivery status as text
    estimatedtime text,                     -- estimated time as text
    addressid text comment 'Customer Address FK(Source System)',                         -- foreign key reference as text (no constraint in snowflake)
    deliverydate text,                      -- delivery date as text
    createddate text,                       -- created date as text
    modifieddate text,                      -- modified date as text

    -- audit columns with appropriate data types
    _stg_file_name text,
    _stg_file_load_ts timestamp,
    _stg_file_md5 text,
    _copy_data_ts timestamp default current_timestamp
)
comment = 'This is the delivery stage/raw table where data will be copied from internal stage using copy command. This is as-is data represetation from the source location. All the columns are text data type except the audit columns that are added for traceability.';

create or replace stream stage_sch.delivery_stm 
on table stage_sch.delivery
append_only = true
comment = 'this is the append-only stream object on delivery table that only gets delta data';


copy into stage_sch.delivery (deliveryid,orderid, deliveryagentid, deliverystatus, 
                    estimatedtime, addressid, deliverydate, createddate, 
                    modifieddate, _stg_file_name, _stg_file_load_ts, 
                    _stg_file_md5, _copy_data_ts)
from (
    select 
        t.$1::text as deliveryid,
        t.$2::text as orderid,
        t.$3::text as deliveryagentid,
        t.$4::text as deliverystatus,
        t.$5::text as estimatedtime,
        t.$6::text as addressid,
        t.$7::text as deliverydate,
        t.$8::text as createddate,
        t.$9::text as modifieddate,
        metadata$filename as _stg_file_name,
        metadata$file_last_modified as _stg_file_load_ts,
        metadata$file_content_key as _stg_file_md5,
        current_timestamp as _copy_data_ts
    from @stage_sch.csv_stg/initial/delivery/delivery-initial-load.csv t
)
file_format = (format_name = 'stage_sch.csv_file_format')
on_error = abort_statement;


CREATE OR REPLACE TABLE clean_sch.delivery (
    delivery_sk INT AUTOINCREMENT PRIMARY KEY comment 'Surrogate Key (EDW)', -- Primary key with auto-increment
    delivery_id INT NOT NULL comment 'Primary Key (Source System)',
    order_id_fk NUMBER NOT NULL comment 'Order FK (Source System)',                        -- Foreign key reference, converted to numeric type
    delivery_agent_id_fk NUMBER NOT NULL comment 'Delivery Agent FK (Source System)',               -- Foreign key reference, converted to numeric type
    delivery_status STRING,                 -- Delivery status, stored as a string
    estimated_time STRING,                  -- Estimated time, stored as a string
    customer_address_id_fk NUMBER NOT NULL  comment 'Customer Address FK (Source System)',                      -- Foreign key reference, converted to numeric type
    delivery_date TIMESTAMP,                -- Delivery date, converted to timestamp
    created_date TIMESTAMP,                 -- Created date, converted to timestamp
    modified_date TIMESTAMP,                -- Modified date, converted to timestamp

    -- Audit columns with appropriate data types
    _stg_file_name STRING,                  -- Source file name
    _stg_file_load_ts TIMESTAMP,            -- Source file load timestamp
    _stg_file_md5 STRING,                   -- MD5 checksum of the source file
    _copy_data_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- Metadata timestamp
)
comment = 'Delivery entity under clean schema with appropriate data type under clean schema layer, data is populated using merge statement from the stage layer location table. This table does not support SCD2';

create or replace stream CLEAN_SCH.delivery_stm 
on table CLEAN_SCH.delivery
comment = 'This is the stream object on delivery agent table table to track insert, update, and delete changes';

MERGE INTO 
    clean_sch.delivery AS target
USING 
    stage_sch.delivery_stm AS source
ON 
    target.delivery_id = TO_NUMBER(source.deliveryid) and
    target.order_id_fk = TO_NUMBER(source.orderid) and
    target.delivery_agent_id_fk = TO_NUMBER(source.deliveryagentid)
WHEN MATCHED THEN
    -- Update the existing record with the latest data
    UPDATE SET
        delivery_status = source.deliverystatus,
        estimated_time = source.estimatedtime,
        customer_address_id_fk = TO_NUMBER(source.addressid),
        delivery_date = TO_TIMESTAMP(source.deliverydate),
        created_date = TO_TIMESTAMP(source.createddate),
        modified_date = TO_TIMESTAMP(source.modifieddate),
        _stg_file_name = source._stg_file_name,
        _stg_file_load_ts = source._stg_file_load_ts,
        _stg_file_md5 = source._stg_file_md5,
        _copy_data_ts = source._copy_data_ts
WHEN NOT MATCHED THEN
    -- Insert new record if no match is found
    INSERT (
        delivery_id,
        order_id_fk,
        delivery_agent_id_fk,
        delivery_status,
        estimated_time,
        customer_address_id_fk,
        delivery_date,
        created_date,
        modified_date,
        _stg_file_name,
        _stg_file_load_ts,
        _stg_file_md5,
        _copy_data_ts
    )
    VALUES (
        TO_NUMBER(source.deliveryid),
        TO_NUMBER(source.orderid),
        TO_NUMBER(source.deliveryagentid),
        source.deliverystatus,
        source.estimatedtime,
        TO_NUMBER(source.addressid),
        TO_TIMESTAMP(source.deliverydate),
        TO_TIMESTAMP(source.createddate),
        TO_TIMESTAMP(source.modifieddate),
        source._stg_file_name,
        source._stg_file_load_ts,
        source._stg_file_md5,
        source._copy_data_ts
    );
