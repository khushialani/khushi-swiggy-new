use role sysadmin;
use database sandbox;
use schema stage_sch;
use warehouse adhoc_wh;

create or replace table stage_sch.deliveryagent (
    deliveryagentid text comment 'Primary Key (Source System)',         -- primary key as text
    name text,           -- name as text, required field
    phone text,            -- phone as text, unique constraint indicated
    vehicletype text,             -- vehicle type as text
    locationid text,              -- foreign key reference as text (no constraint in snowflake)
    status text,                  -- status as text
    gender text,                  -- status as text
    rating text,                  -- rating as text
    createddate text,             -- created date as text
    modifieddate text,            -- modified date as text

    -- audit columns with appropriate data types
    _stg_file_name text,
    _stg_file_load_ts timestamp,
    _stg_file_md5 text,
    _copy_data_ts timestamp default current_timestamp
)
comment = 'This is the delivery stage/raw table where data will be copied from internal stage using copy command. This is as-is data represetation from the source location. All the columns are text data type except the audit columns that are added for traceability.';

create or replace stream stage_sch.deliveryagent_stm 
on table stage_sch.deliveryagent
append_only = true
comment = 'This is the append-only stream object on delivery agent table that only gets delta data';


copy into stage_sch.deliveryagent (deliveryagentid, name, phone, vehicletype, locationid, 
                         status, gender, rating, createddate, modifieddate,
                         _stg_file_name, _stg_file_load_ts, _stg_file_md5, _copy_data_ts)
from (
    select 
        t.$1::text as deliveryagentid,
        t.$2::text as name,
        t.$3::text as phone,
        t.$4::text as vehicletype,
        t.$5::text as locationid,
        t.$6::text as status,
        t.$7::text as gender,
        t.$8::text as rating,
        t.$9::text as createddate,
        t.$10::text as modifieddate,
        metadata$filename as _stg_file_name,
        metadata$file_last_modified as _stg_file_load_ts,
        metadata$file_content_key as _stg_file_md5,
        current_timestamp as _copy_data_ts
    from @stage_sch.csv_stg/initial/delivery-agent t
)
file_format = (format_name = 'stage_sch.csv_file_format')
on_error = abort_statement;

select count(*) from stage_sch.deliveryagent;

select * from stage_sch.deliveryagent_stm;


CREATE OR REPLACE TABLE clean_sch.delivery_agent (
    delivery_agent_sk INT AUTOINCREMENT PRIMARY KEY comment 'Surrogate Key (EDW)', -- Primary key with auto-increment
    delivery_agent_id INT NOT NULL UNIQUE comment 'Primary Key (Source System)',               -- Delivery agent ID as integer
    name STRING NOT NULL,                -- Name as string, required field
    phone STRING NOT NULL,                 -- Phone as string, unique constraint
    vehicle_type STRING NOT NULL,                 -- Vehicle type as string
    location_id_fk INT comment 'Location FK(Source System)',                     -- Location ID as integer
    status STRING,                       -- Status as string
    gender STRING,                       -- Gender as string
    rating number(4,2),                        -- Rating as float
    created_dt TIMESTAMP_NTZ,          -- Created date as timestamp without timezone
    modified_dt TIMESTAMP_NTZ,         -- Modified date as timestamp without timezone

    -- Audit columns with appropriate data types
    _stg_file_name STRING,               -- Staging file name as string
    _stg_file_load_ts TIMESTAMP,         -- Staging file load timestamp
    _stg_file_md5 STRING,                -- Staging file MD5 hash as string
    _copy_data_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- Data copy timestamp with default value
)
comment = 'Delivery entity under clean schema with appropriate data type under clean schema layer, data is populated using merge statement from the stage layer location table. This table does not support SCD2';


create or replace stream CLEAN_SCH.delivery_agent_stm 
on table CLEAN_SCH.delivery_agent
comment = 'This is the stream object on delivery agent table table to track insert, update, and delete changes';


MERGE INTO clean_sch.delivery_agent AS target
USING stage_sch.deliveryagent_stm AS source
ON target.delivery_agent_id = source.deliveryagentid
WHEN MATCHED THEN
    UPDATE SET
        target.phone = source.phone,
        target.vehicle_type = source.vehicletype,
        target.location_id_fk = TRY_TO_NUMBER(source.locationid),
        target.status = source.status,
        target.gender = source.gender,
        target.rating = TRY_TO_DECIMAL(source.rating,4,2),
        target.created_dt = TRY_TO_TIMESTAMP(source.createddate),
        target.modified_dt = TRY_TO_TIMESTAMP(source.modifieddate),
        target._stg_file_name = source._stg_file_name,
        target._stg_file_load_ts = source._stg_file_load_ts,
        target._stg_file_md5 = source._stg_file_md5,
        target._copy_data_ts = source._copy_data_ts
WHEN NOT MATCHED THEN
    INSERT (
        delivery_agent_id,
        name,
        phone,
        vehicle_type,
        location_id_fk,
        status,
        gender,
        rating,
        created_dt,
        modified_dt,
        _stg_file_name,
        _stg_file_load_ts,
        _stg_file_md5,
        _copy_data_ts
    )
    VALUES (
        TRY_TO_NUMBER(source.deliveryagentid),
        source.name,
        source.phone,
        source.vehicletype,
        TRY_TO_NUMBER(source.locationid),
        source.status,
        source.gender,
        TRY_TO_NUMBER(source.rating),
        TRY_TO_TIMESTAMP(source.createddate),
        TRY_TO_TIMESTAMP(source.modifieddate),
        source._stg_file_name,
        source._stg_file_load_ts,
        source._stg_file_md5,
        CURRENT_TIMESTAMP()
    );

select * from CLEAN_SCH.delivery_agent_stm ;


CREATE OR REPLACE TABLE consumption_sch.delivery_agent_dim (
    delivery_agent_hk number primary key comment 'Delivery Agend Dim HK (EDW)',               -- Hash key for unique identification
    delivery_agent_id NUMBER not null comment 'Primary Key (Source System)',               -- Business key
    name STRING NOT NULL,                   -- Delivery agent name
    phone STRING UNIQUE,                    -- Phone number, unique
    vehicle_type STRING,                    -- Type of vehicle
    location_id_fk NUMBER NOT NULL comment 'Location FK (Source System)',                     -- Location ID
    status STRING,                          -- Current status of the delivery agent
    gender STRING,                          -- Gender
    rating NUMBER(4,2),                     -- Rating with one decimal precision
    eff_start_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- Effective start date
    eff_end_date TIMESTAMP,                 -- Effective end date (NULL for active record)
    is_current BOOLEAN DEFAULT TRUE
)
comment =  'Dim table for delivery agent entity with SCD2 support.';


MERGE INTO consumption_sch.delivery_agent_dim AS target
USING CLEAN_SCH.delivery_agent_stm AS source
ON 
    target.delivery_agent_id = source.delivery_agent_id AND
    target.name = source.name AND
    target.phone = source.phone AND
    target.vehicle_type = source.vehicle_type AND
    target.location_id_fk = source.location_id_fk AND
    target.status = source.status AND
    target.gender = source.gender AND
    target.rating = source.rating
WHEN MATCHED 
    AND source.METADATA$ACTION = 'DELETE' 
    AND source.METADATA$ISUPDATE = 'TRUE' THEN
    -- Update the existing record to close its validity period
    UPDATE SET 
        target.eff_end_date = CURRENT_TIMESTAMP,
        target.is_current = FALSE
WHEN NOT MATCHED 
    AND source.METADATA$ACTION = 'INSERT' 
    AND source.METADATA$ISUPDATE = 'TRUE' THEN
    -- Insert new record with current data and new effective start date
    INSERT (
        delivery_agent_hk,        -- Hash key
        delivery_agent_id,
        name,
        phone,
        vehicle_type,
        location_id_fk,
        status,
        gender,
        rating,
        eff_start_date,
        eff_end_date,
        is_current
    )
    VALUES (
        hash(SHA1_HEX(CONCAT(source.delivery_agent_id, source.name, source.phone, 
            source.vehicle_type, source.location_id_fk, source.status, 
            source.gender, source.rating))), -- Hash key
        delivery_agent_id,
        source.name,
        source.phone,
        source.vehicle_type,
        location_id_fk,
        source.status,
        source.gender,
        source.rating,
        CURRENT_TIMESTAMP,       -- Effective start date
        NULL,                    -- Effective end date (NULL for current record)
        TRUE                    -- IS_CURRENT = TRUE for new record
    )
WHEN NOT MATCHED 
    AND source.METADATA$ACTION = 'INSERT' 
    AND source.METADATA$ISUPDATE = 'FALSE' THEN
    -- Insert new record with current data and new effective start date
    INSERT (
        delivery_agent_hk,        -- Hash key
        delivery_agent_id,
        name,
        phone,
        vehicle_type,
        location_id_fk,
        status,
        gender,
        rating,
        eff_start_date,
        eff_end_date,
        is_current
    )
    VALUES (
        hash(SHA1_HEX(CONCAT(source.delivery_agent_id, source.name, source.phone, 
            source.vehicle_type, source.location_id_fk, source.status,
            source.gender, source.rating))), -- Hash key
        source.delivery_agent_id,
        source.name,
        source.phone,
        source.vehicle_type,
        source.location_id_fk,
        source.status,
        source.gender,
        source.rating,
        CURRENT_TIMESTAMP,       -- Effective start date
        NULL,                    -- Effective end date (NULL for current record)
        TRUE                   -- IS_CURRENT = TRUE for new record
    );




-- part-2

copy into deliveryagent (deliveryagentid, name, phone, vehicletype, locationid, 
                         status, gender, rating, createddate, modifieddate,
                         _stg_file_name, _stg_file_load_ts, _stg_file_md5, _copy_data_ts)
from (
    select 
        t.$1::text as deliveryagentid,
        t.$2::text as name,
        t.$3::text as phone,
        t.$4::text as vehicletype,
        t.$5::text as locationid,
        t.$6::text as status,
        t.$7::text as gender,
        t.$8::text as rating,
        t.$9::text as createddate,
        t.$10::text as modifieddate,
        metadata$filename as _stg_file_name,
        metadata$file_last_modified as _stg_file_load_ts,
        metadata$file_content_key as _stg_file_md5,
        current_timestamp as _copy_data_ts
    from @stage_sch.csv_stg/delta/delivery-agent/day-02-delivery-agent.csv t
)
file_format = (format_name = 'stage_sch.csv_file_format')
on_error = abort_statement;
