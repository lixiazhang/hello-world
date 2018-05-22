# hello-world
Get FK
SELECT
        *
        , 'ALTER TABLE ['+ FKTableSchema +'].[' + FKTableName + '] DROP CONSTRAINT ' 
            + ForeignKeyName AS FKDrop
        , 'ALTER TABLE ['+ FKTableSchema +'].[' +FKTableName + '] ADD CONSTRAINT ' 
            + ForeignKeyName + ' FOREIGN KEY(' +
          FKColumnList +') REFERENCES ['+ PKTableSchema+'].['
            + PKTableName + '] ('+PKColumnList +')' AS FKCreate
        , 'ALTER TABLE ['+ PKTableSchema +'].[' +PKTableName + '] DROP CONSTRAINT ' 
            + PrimaryKeyName AS PKDrop
        , 'ALTER TABLE ['+PKTableSchema +'].[' +PKTableName + '] ADD CONSTRAINT ' 
            + PrimaryKeyName + ' PRIMARY KEY('+PKColumnList+')' AS PKCreate
    FROM
        (
        SELECT DISTINCT
            FKTableSchema = 
                (
                SELECT DISTINCT 
                    table_schema 
                FROM 
                    INFORMATION_SCHEMA.KEY_COLUMN_USAGE 
                WHERE 
                    constraint_name=rc.constraint_Name
                ),
            FKTableName = 
                (
                SELECT DISTINCT 
                    table_Name 
                FROM 
                    INFORMATION_SCHEMA.KEY_COLUMN_USAGE 
                WHERE 
                    constraint_name=rc.constraint_Name
                    ),
            rc.constraint_name AS ForeignKeyName,
            FKColumnList = 
                (
                SELECT 
                    left(t.column_name,len(t.column_name)-1) AS 'ColumnList' 
                FROM
                    (
                    SELECT Column_Name + ',' FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
                    WHERE constraint_name=rc.constraint_name
                    FOR XML PATH('')
                    ) AS t(column_Name)
                ),
            cu.table_schema AS PKTableSchema,
            cu.table_name AS PKTableName,
            cu.constraint_Name AS PrimaryKeyName,
            PKColumnList = 
                (
                SELECT 
                    left(t.column_name,len(t.column_name)-1) AS 'ColumnList' 
                FROM
                    (
                    SELECT 
                        Column_Name + ',' 
                    FROM 
                        INFORMATION_SCHEMA.KEY_COLUMN_USAGE
                    WHERE 
                        constraint_name=cu.constraint_name
                    FOR XML PATH('')
                    ) AS t(column_Name)
                )
        FROM
            INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS cu
        INNER JOIN
            INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS AS rc
        ON  rc.Unique_Constraint_name= cu.Constraint_name
    ) AS tab
