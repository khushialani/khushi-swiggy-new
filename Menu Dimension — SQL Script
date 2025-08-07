use role sysadmin;
use database sandbox;
use schema stage_sch;
use warehouse adhoc_wh;


create or replace table stage_sch.menu (
    menuid text comment 'Primary Key (Source System)',                   -- primary key as text
    restaurantid text comment 'Restaurant FK(Source System)',             -- foreign key reference as text (no constraint in snowflake)
    itemname text,                 -- item name as text
    description text,              -- description as text
    price text,                    -- price as text (no decimal constraint)
    category text,                 -- category as text
    availability text,             -- availability as text
    itemtype text,                 -- item type as text
    createddate text,              -- created date as text
    modifieddate text,             -- modified date as text

    -- audit columns with appropriate data types
    _stg_file_name text,
    _stg_file_load_ts timestamp,
    _stg_file_md5 text,
    _copy_data_ts timestamp default current_timestamp
)
comment = 'This is the menu stage/raw table where data will be copied from internal stage using copy command. This is as-is data represetation from the source location. All the columns are text data type except the audit columns that are added for traceability.';

-- Stream object to capture the changes. 
create or replace stream stage_sch.menu_stm 
on table stage_sch.menu
append_only = true
comment = 'This is the append-only stream object on menu entity that only gets delta data';


list @stage_sch.csv_stg/initial/menu;

copy into stage_sch.menu (menuid, restaurantid, itemname, description, price, category, 
                availability, itemtype, createddate, modifieddate,
                _stg_file_name, _stg_file_load_ts, _stg_file_md5, _copy_data_ts)
from (
    select 
        t.$1::text as menuid,
        t.$2::text as restaurantid,
        t.$3::text as itemname,
        t.$4::text as description,
        t.$5::text as price,
        t.$6::text as category,
        t.$7::text as availability,
        t.$8::text as itemtype,
        t.$9::text as createddate,
        t.$10::text as modifieddate,
        metadata$filename as _stg_file_name,
        metadata$file_last_modified as _stg_file_load_ts,
        metadata$file_content_key as _stg_file_md5,
        current_timestamp as _copy_data_ts
    from @stage_sch.csv_stg/initial/menu t
)
file_format = (format_name = 'stage_sch.csv_file_format')
on_error = abort_statement;

select * from menu limit 10;

select *
from table(information_schema.copy_history(table_name=>'MENU', start_time=> dateadd(hours, -1, current_timestamp())));


CREATE OR REPLACE TABLE clean_sch.menu (
    Menu_SK INT AUTOINCREMENT PRIMARY KEY comment 'Surrogate Key (EDW)',  -- Auto-incrementing primary key for internal tracking
    Menu_ID INT NOT NULL UNIQUE comment 'Primary Key (Source System)' ,             -- Unique and non-null Menu_ID
    Restaurant_ID_FK INT comment 'Restaurant FK(Source System)' ,                      -- Identifier for the restaurant
    Item_Name STRING not null,                        -- Name of the menu item
    Description STRING not null,                     -- Description of the menu item
    Price DECIMAL(10, 2) not null,                   -- Price as a numeric value with 2 decimal places
    Category STRING,                        -- Food category (e.g., North Indian)
    Availability BOOLEAN,                   -- Availability status (True/False)
    Item_Type STRING,                        -- Dietary classification (e.g., Vegan)
    Created_dt TIMESTAMP_NTZ,               -- Date when the record was created
    Modified_dt TIMESTAMP_NTZ,              -- Date when the record was last modified

    -- Audit columns for traceability
    _STG_FILE_NAME STRING,                  -- Source file name
    _STG_FILE_LOAD_TS TIMESTAMP_NTZ,        -- Timestamp when data was loaded from the staging layer
    _STG_FILE_MD5 STRING,                   -- MD5 hash of the source file
    _COPY_DATA_TS TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP -- Timestamp when data was copied to the clean layer
)
comment = 'Menu entity under clean schema with appropriate data type under clean schema layer, data is populated using merge statement from the stage layer location table. This table does not support SCD2';

create or replace stream CLEAN_SCH.menu_stm 
on table CLEAN_SCH.menu
comment = 'This is the stream object on menu table table to track insert, update, and delete changes';


MERGE INTO clean_sch.menu AS target
USING (
    SELECT 
        TRY_CAST(menuid AS INT) AS Menu_ID,
        TRY_CAST(restaurantid AS INT) AS Restaurant_ID_FK,
        TRIM(itemname) AS Item_Name,
        TRIM(description) AS Description,
        TRY_CAST(price AS DECIMAL(10, 2)) AS Price,
        TRIM(category) AS Category,
        CASE 
            WHEN LOWER(availability) = 'true' THEN TRUE
            WHEN LOWER(availability) = 'false' THEN FALSE
            ELSE NULL
        END AS Availability,
        TRIM(itemtype) AS Item_Type,
        TRY_CAST(createddate AS TIMESTAMP_NTZ) AS Created_dt,  -- Renamed column
        TRY_CAST(modifieddate AS TIMESTAMP_NTZ) AS Modified_dt, -- Renamed column
        _stg_file_name,
        _stg_file_load_ts,
        _stg_file_md5,
        _copy_data_ts
    FROM stage_sch.menu
) AS source
ON target.Menu_ID = source.Menu_ID
WHEN MATCHED THEN
    UPDATE SET
        Restaurant_ID_FK = source.Restaurant_ID_FK,
        Item_Name = source.Item_Name,
        Description = source.Description,
        Price = source.Price,
        Category = source.Category,
        Availability = source.Availability,
        Item_Type = source.Item_Type,
        Created_dt = source.Created_dt,  
        Modified_dt = source.Modified_dt,  
        _STG_FILE_NAME = source._stg_file_name,
        _STG_FILE_LOAD_TS = source._stg_file_load_ts,
        _STG_FILE_MD5 = source._stg_file_md5,
        _COPY_DATA_TS = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (
        Menu_ID,
        Restaurant_ID_FK,
        Item_Name,
        Description,
        Price,
        Category,
        Availability,
        Item_Type,
        Created_dt, 
        Modified_dt,  
        _STG_FILE_NAME,
        _STG_FILE_LOAD_TS,
        _STG_FILE_MD5,
        _COPY_DATA_TS
    )
    VALUES (
        source.Menu_ID,
        source.Restaurant_ID_FK,
        source.Item_Name,
        source.Description,
        source.Price,
        source.Category,
        source.Availability,
        source.Item_Type,
        source.Created_dt,  
        source.Modified_dt,  
        source._stg_file_name,
        source._stg_file_load_ts,
        source._stg_file_md5,
        CURRENT_TIMESTAMP
    );


CREATE OR REPLACE TABLE consumption_sch.menu_dim (
    Menu_Dim_HK NUMBER primary key comment 'Menu Dim HK (EDW)',                         -- Hash key generated for Menu Dim table
    Menu_ID INT NOT NULL comment 'Primary Key (Source System)',                       -- Unique and non-null Menu_ID
    Restaurant_ID_FK INT NOT NULL comment 'Restaurant FK (Source System)',                          -- Identifier for the restaurant
    Item_Name STRING,                            -- Name of the menu item
    Description STRING,                         -- Description of the menu item
    Price DECIMAL(10, 2),                       -- Price as a numeric value with 2 decimal places
    Category STRING,                            -- Food category (e.g., North Indian)
    Availability BOOLEAN,                       -- Availability status (True/False)
    Item_Type STRING,                           -- Dietary classification (e.g., Vegan)
    EFF_START_DATE TIMESTAMP_NTZ,               -- Effective start date of the record
    EFF_END_DATE TIMESTAMP_NTZ,                 -- Effective end date of the record
    IS_CURRENT BOOLEAN                         -- Flag to indicate if the record is current (True/False)
)
COMMENT = 'This table stores the dimension data for the menu items, tracking historical changes using SCD Type 2. Each menu item has an effective start and end date, with a flag indicating if it is the current record or historical. The hash key (Menu_Dim_HK) is generated based on Menu_ID and Restaurant_ID.';


MERGE INTO 
    consumption_sch.MENU_DIM AS target
USING 
    CLEAN_SCH.MENU_STM AS source
ON 
    target.Menu_ID = source.Menu_ID AND
    target.Restaurant_ID_FK = source.Restaurant_ID_FK AND
    target.Item_Name = source.Item_Name AND
    target.Description = source.Description AND
    target.Price = source.Price AND
    target.Category = source.Category AND
    target.Availability = source.Availability AND
    target.Item_Type = source.Item_Type
WHEN MATCHED 
    AND source.METADATA$ACTION = 'DELETE' 
    AND source.METADATA$ISUPDATE = 'TRUE' THEN
    -- Update the existing record to close its validity period
    UPDATE SET 
        target.EFF_END_DATE = CURRENT_TIMESTAMP(),
        target.IS_CURRENT = FALSE
WHEN NOT MATCHED 
    AND source.METADATA$ACTION = 'INSERT' 
    AND source.METADATA$ISUPDATE = 'TRUE' THEN
    -- Insert new record with current data and new effective start date
    INSERT (
        Menu_Dim_HK,               -- Hash key
        Menu_ID,
        Restaurant_ID_FK,
        Item_Name,
        Description,
        Price,
        Category,
        Availability,
        Item_Type,
        EFF_START_DATE,
        EFF_END_DATE,
        IS_CURRENT
    )
    VALUES (
        hash(SHA1_hex(CONCAT(source.Menu_ID, source.Restaurant_ID_FK, 
            source.Item_Name, source.Description, source.Price, 
            source.Category, source.Availability, source.Item_Type))),  -- Hash key
        source.Menu_ID,
        source.Restaurant_ID_FK,
        source.Item_Name,
        source.Description,
        source.Price,
        source.Category,
        source.Availability,
        source.Item_Type,
        CURRENT_TIMESTAMP(),       -- Effective start date
        NULL,                      -- Effective end date (NULL for current record)
        TRUE                       -- IS_CURRENT = TRUE for new record
    )
WHEN NOT MATCHED 
    AND source.METADATA$ACTION = 'INSERT' 
    AND source.METADATA$ISUPDATE = 'FALSE' THEN
    -- Insert new record with current data and new effective start date
    INSERT (
        Menu_Dim_HK,               -- Hash key
        Menu_ID,
        Restaurant_ID_FK,
        Item_Name,
        Description,
        Price,
        Category,
        Availability,
        Item_Type,
        EFF_START_DATE,
        EFF_END_DATE,
        IS_CURRENT
    )
    VALUES (
        hash(SHA1_hex(CONCAT(source.Menu_ID, source.Restaurant_ID_FK, 
            source.Item_Name, source.Description, source.Price, 
            source.Category, source.Availability, source.Item_Type))),  -- Hash key
        source.Menu_ID,
        source.Restaurant_ID_FK,
        source.Item_Name,
        source.Description,
        source.Price,
        source.Category,
        source.Availability,
        source.Item_Type,
        CURRENT_TIMESTAMP(),       -- Effective start date
        NULL,                      -- Effective end date (NULL for current record)
        TRUE                       -- IS_CURRENT = TRUE for new record
    );


-- Part -2
list @stage_sch.csv_stg/delta/menu;
copy into stage_sch.menu (menuid, restaurantid, itemname, description, price, category, 
                availability, itemtype, createddate, modifieddate,
                _stg_file_name, _stg_file_load_ts, _stg_file_md5, _copy_data_ts)
from (
    select 
        t.$1::text as menuid,
        t.$2::text as restaurantid,
        t.$3::text as itemname,
        t.$4::text as description,
        t.$5::text as price,
        t.$6::text as category,
        t.$7::text as availability,
        t.$8::text as itemtype,
        t.$9::text as createddate,
        t.$10::text as modifieddate,
        metadata$filename as _stg_file_name,
        metadata$file_last_modified as _stg_file_load_ts,
        metadata$file_content_key as _stg_file_md5,
        current_timestamp as _copy_data_ts
    from @stage_sch.csv_stg/delta/menu/ t
)
file_format = (format_name = 'stage_sch.csv_file_format')
on_error = abort_statement;
